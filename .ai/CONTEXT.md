# ContextFile: Technical Specifications

이 문서는 The Heritage Collection의 구체적인 기술 설정 및 구성 파일을 포함합니다.

## 1. Directory Structure & Persistence Strategy

본 프로젝트는 **"Config as Code"** 원칙을 따르며, 핵심 설정 파일만을 Git으로 관리합니다.

### 1.1. File System Hierarchy

```text
/home/crong/git/heritage/
├── .gitignore          # 바이너리, 로그, 캐시, 미디어 파일 제외 설정
├── docker-compose.yml  # 인프라 정의 (Podman)
├── homepage/
│   └── config/         # [Git] 대시보드 설정 (services.yaml, widgets.yaml 등)
├── jdownloader-2/      # [Local] 전체 제외 (바이너리 및 수시로 변하는 설정)
├── jellyfin/           # [Local] 전체 제외 (메타데이터 및 DB 용량 과다)
├── prowlarr/           # [Local] 전체 제외 (인덱서 캐시 및 로그)
├── stash/              # [Local] 전체 제외 (대용량 DB 및 메타데이터)
├── torrent/            # [Local] 전체 제외 (P2P 세션 및 설정)
├── whisparr/           # [Local] 전체 제외 (라이브러리 메타데이터)
```

### 1.2. Persistence Policy

| Data Type | Storage Location | Backup Strategy | Git Tracking |
| :--- | :--- | :--- | :--- |
| **Config** | `/home/crong/heritage/*` | Git Commit & Push | ✅ Yes |
| **Media** | `/mnt/data1`, `/mnt/data2` | NAS / Cold Storage | ❌ No |
| **Runtime** | `logs/`, `cache/`, `tmp/` | Ephemeral (삭제 가능) | ❌ No |
| **State** | `*.db`, `*.zip`, `*.pid` | Local Persistence | ❌ No |

---

## 2. docker-compose.yml (Podman Optimized)

`podman-compose` 명령어로 실행하며, 모든 설정과 DB는 SSD(`/home/crong/heritage`)에 저장되어 최상의 응답 속도를 보장합니다.

```yaml
version: '3.9'

networks:
  media-net:
    name: media-net

services:
  # --- 🛡️ Infrastructure (관제 및 인프라) ---
  homepage:
    image: ghcr.io/gethomepage/homepage:latest
    container_name: homepage
    ports:
      - "3000:3000"
    volumes:
      - /home/crong/heritage/homepage/config:/app/config # Git Managed
      - /run/user/1000/podman/podman.sock:/var/run/docker.sock:ro
      - /mnt:/mnt:ro
    networks:
      - media-net
    restart: unless-stopped

  prowlarr:
    image: ghcr.io/hotio/prowlarr:latest
    container_name: prowlarr
    ports:
      - "9696:9696"
    environment:
      - PUID=1000
      - PGID=1000
    volumes:
      - /home/crong/heritage/prowlarr:/config # Local State (Excluded)
    networks:
      - media-net
    restart: unless-stopped

  # --- 📥 Acquisition Engines (수집 엔진) ---
  rtorrent-rutorrent:
    image: crazymax/rtorrent-rutorrent:latest
    container_name: rtorrent-rutorrent
    environment:
      - PUID=1000
      - PGID=1000
    ports:
      - "8080:8080"
    volumes:
      - /home/crong/heritage/rtorrent:/data # Local State (Excluded)
      - /mnt/data1/torrent:/downloads      # sda1: Inbound Staging
      - /mnt/data2/torrent:/completed_hdd2 # sdb1: The Vault
    networks:
      - media-net
    restart: always

  jdownloader:
    image: jlesage/jdownloader-2
    container_name: jdownloader
    ports:
      - "5800:5800"
    environment:
      - PUID=1000
      - PGID=1000
      - JAVA_OPTIONS=-Xmx512m -Xms256m -XX:+UseSerialGC
    volumes:
      - /home/crong/heritage/jdownloader-2:/config # Local State (Excluded from Git)
      - /mnt/data1/torrent/jd2_downloads:/output
    mem_limit: 1g
    networks:
      - media-net
    restart: unless-stopped

  # --- 🏛️ Curatorial Tools (기록 및 분류) ---
  whisparr:
    image: ghcr.io/hotio/whisparr:latest
    container_name: whisparr
    ports:
      - "6969:6969"
    environment:
      - PUID=1000
      - PGID=1000
    volumes:
      - /home/crong/heritage/whisparr:/config # Local State (Excluded)
      - /mnt/data1/torrent:/data1
      - /mnt/data2/torrent:/data2
    networks:
      - media-net
    restart: unless-stopped

  stash:
    image: stashapp/stash:latest
    container_name: stash
    ports:
      - "9999:9999"
    environment:
      - STASH_CONFIG_FILE=/config/config.yml
    volumes:
      - /home/crong/heritage/stash:/config # Local State (Excluded)
      - /mnt/data1/torrent:/data1
      - /mnt/data2/torrent:/data2
    networks:
      - media-net
    restart: unless-stopped

  # --- 🎭 Digital Theater (전시 및 스트리밍) ---
  jellyfin:
    image: jellyfin/jellyfin:latest
    container_name: jellyfin
    ports:
      - "8096:8096"
    devices:
      - /dev/dri/renderD128:/dev/dri/renderD128
    volumes:
      - /home/crong/heritage/jellyfin:/config # Local State (Excluded)
      - /mnt/data1/torrent:/data1
      - /mnt/data2/torrent:/data2
    networks:
      - media-net
    restart: unless-stopped
```

## 3. services.yaml (Homepage - Gentleman Edition)

`/home/crong/heritage/homepage/config/services.yaml`

이 설정은 대시보드에 신사다운 품격을 더하며, 각 앱의 API 키를 입력하여 실시간 상태를 모니터링합니다.

```yaml
- Infrastructure:
    - Prowlarr:
        label: "Search Gateway"
        icon: prowlarr.png
        href: http://192.168.1.108:9696
        container: prowlarr
        widget:
          type: prowlarr
          url: http://prowlarr:9696
          key: YOUR_PROWLARR_API_KEY

- Acquisition Engines:
    - ruTorrent:
        label: "Direct Inbound"
        icon: rutorrent.png
        href: http://192.168.1.108:8080
        container: rtorrent-rutorrent
        widget:
          type: rutorrent
          url: http://rtorrent-rutorrent:8080
    - JDownloader:
        label: "Web Collector"
        icon: jdownloader.png
        href: http://192.168.1.108:5800
        container: jdownloader
        widget:
          type: jdownloader
          url: http://jdownloader:5800

- Curatorial Tools:
    - Whisparr:
        label: "Content Librarian"
        icon: whisparr.png
        href: http://192.168.1.108:6969
        container: whisparr
        widget:
          type: whisparr
          url: http://whisparr:6969
          key: YOUR_WHISPARR_API_KEY
    - Stash:
        label: "Archive Vault"
        icon: stash.png
        href: http://192.168.1.108:9999
        container: stash
        widget:
          type: stash
          url: http://stash:9999
          key: YOUR_STASH_API_KEY

- Digital Theater:
    - Jellyfin:
        label: "The Grand Cinema"
        icon: jellyfin.png
        href: http://192.168.1.108:8096
        container: jellyfin
        widget:
          type: jellyfin
          url: http://jellyfin:8096
          key: YOUR_JELLYFIN_API_KEY
```
