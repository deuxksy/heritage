# The Heritage Collection

> A sophisticated system for a more civilized archive.

## 프로젝트 개요

**The Heritage Collection**은 단순한 관리를 넘어선 **'프라이빗 라이브러리 큐레이션'**의 완성을 목표로 하는 고급 미디어 아카이브 시스템입니다. 15년 차 DevOps 엔지니어이자 고급 미디어 큐레이터인 사용자를 위해 설계되었습니다.

## 핵심 인프라 (Core Infrastructure)

*   **OS:** Fedora 43
*   **Runtime:** Podman 5.7.1 (Rootless, Daemonless)
*   **Hardware:** Intel N100 (Chatreey R1)
*   **Storage Strategy:**
    *   **SSD (nvme0n1):** OS, 설정, DB 저장소 (`/home/crong/heritage`)
    *   **HDD 1 (sda1):** Acquisition Area (수집 구역, 신규 다운로드)
    *   **HDD 2 (sdb1):** The Vault (저장고, 최종 아카이브)

## 기술 스택 (The Stack)

*   **Homepage:** 시스템 관제 및 리소스 모니터링
*   **Prowlarr:** 인덱서 통합 관리
*   **ruTorrent:** 메인 수집 엔진 (Autotools 활용)
*   **JDownloader 2:** 직접 다운로드 도구 (JVM 최적화)
*   **Whisparr:** 콘텐츠 자동 정리 (Content Librarian)
*   **Stash:** 프라이빗 DB 관리 (Archive Vault)
*   **Jellyfin:** 4K 스트리밍 (The Grand Cinema, Intel QSV 가속)

## 운영 원칙

*   **IaC (Infrastructure as Code):** 모든 인프라는 `podman-compose`와 Git으로 관리합니다.
*   **Gentlemanly Terminology:** 품격 있는 용어 사용 (예: '프라이빗 큐레이션', '시네마틱 아카이브').
*   **Performance:** I/O Wait 최소화 및 하드링크 우선 적용.

자세한 기술 명세와 설정 파일은 [ContextFile.md](./ContextFile.md)를, AI 어시스턴트 지침은 [GEMINI.md](./GEMINI.md)를 참조하십시오.
