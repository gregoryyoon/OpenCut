# 개발 완료 보고서: Next.js 웹 앱의 Tauri 네이티브 데스크탑 포팅

## 1. 개요
* **목표:** 기존 `apps/web`에 구축된 Next.js 기반 웹 애플리케이션을 Windows 데스크탑 환경에서 실행 가능한 독립적인 네이티브 앱으로 동작하도록 Tauri 프레임워크로 이식.
* **접근 방식:** 원본 오픈소스 프로젝트(upstream)의 코드를 보존하기 위해 기존의 `apps/desktop` 폴더(GPUI 기반)를 건드리지 않고, 완전히 독립된 새로운 `apps/tauri` 구성 요소를 추가하여 웹 앱을 래핑(Wrapping)하는 아키텍처를 채택함.

## 2. 주요 작업 내용 및 아키텍처 변경사항
1. **신규 패키지 추가 (`apps/tauri`)**
   * Tauri v2 CLI를 활용하여 `apps/tauri` 디렉토리에 새로운 Rust 기반 프로젝트 뼈대를 구축.
   * 필요 없는 기본 웹 템플릿(vanilla HTML/JS)을 제거하고 순수 Tauri 래퍼(Wrapper)로 구성.
2. **Next.js - Tauri 런타임 연결 브릿지 구축**
   * `apps/tauri/src-tauri/tauri.conf.json` 수정.
   * `frontDist` 경로를 `../../web/out` 으로 연결하여 Next.js의 정적 빌드 결과물(Static Export)을 읽어오도록 구성.
   * 개발 모드 시 원클릭 실행을 위해 `beforeDevCommand`에 `bun run --filter=@opencut/web dev` 바인딩(`localhost:3000` 라우팅).
3. **의존성 밎 패키지 매니저 연동**
   * 프로젝트 루트의 모노레포(Workspaces) `Cargo.toml` 에 `apps/tauri/src-tauri` 멤버 추가 완료.
   * 모노레포 관리 도구(Bun)에 맞춰 루트 `package.json`과 `apps/web/package.json`의 Script (dev/build) 연동.
4. **업스트림 동기화 통과 검증**
   * Upstream 깃허브 원본과의 Pull/Merge 동기화 과정에서 발생하는 `Cargo.lock`, `bun.lock` 파일의 충돌을 자동으로 수용하고 재빌드할 수 있는 스크립트 기반을 다짐.

## 3. 성과 및 결과
* 기존 웹 코드를 전혀 수정하지 않고도 네이티브 데스크탑의 이점을 취할 수 있는 견고한 아키텍처 수립 완료.
* 개발 핫 리로딩(HMR) 환경 지원으로 인해 향후 유지보수 시 생산성 극대화.
* Microsoft Visual C++ Build Tools 및 Rust 환경에서 정상 빌드됨을 검증 후 GitHub 메인 브랜치 반영 완료.
