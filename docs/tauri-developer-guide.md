# 데스크탑(Tauri) 앱 개발자 가이드

이 문서에서는 `apps/web` (Next.js)과 `apps/tauri` (Rust)로 구성된 모노레포 데스크탑 환경에서 개발을 시작하고 유지보수하는 방법을 안내합니다.

## 1. 개발 환경 필수 요구사항 (Prerequisites)
데스크탑 앱을 로컬에서 빌드하고 실행하기 위해서는 다음 도구들이 시스템 환경변수(PATH)에 올바르게 설치되어 있어야 합니다.

1. **Bun (v1.2 이상)**: 패키지 매니저 및 스크립트 실행
2. **Rust & Cargo**: Rust 컴파일러
    * 설치 스크립트: `powershell -ExecutionPolicy Bypass -File .\script\setup-rust.ps1`
3. **Microsoft C++ Build Tools (Windows 필수)**:
    * Rust가 Windows 시스템 라이브러리를 링킹(Link)하기 위해 필요합니다(`link.exe`).
    * Visual Studio Build Tools 설치 시 **"C++를 사용한 데스크톱 개발"** 워크로드를 반드시 포함하여 설치해야 합니다.

---

## 2. 앱 실행 및 빌드 방법

터미널을 프로젝트 루트(`c:\projects\OpenCut\`)에 위치시킨 후 아래 명령어를 사용하세요.

### 개발 모드 (Development)
```bash
bun run dev:tauri
```
* **작동 방식:** `apps/web` 폴더에서 Next.js 개발 서버(`http://localhost:3000`)를 백그라운드로 띄운 뒤, Tauri 윈도우가 열리면서 이 주소를 띄워줍니다.
* **특징:** React/Next.js 코드를 수정하고 저장하면 **창을 껐다 켤 필요 없이 실시간으로 변경사항이 반영(HMR)** 됩니다.

### 실제 배포/릴리즈 빌드 (Production Build)
```bash
bun run build:tauri
```
* **작동 방식:** Next.js 코드를 `output: "export"` 형태로 최적화하여 `apps/web/out`에 정적 파일로 내보낸(export) 뒤, Tauri가 이 파일을 품고 단일 실행 파일(`.exe`) 또는 설치 패키지로 압축합니다.

---

## 3. 업스트림(원본 레포지토리) 동기화 시 주의사항

오픈소스 프로젝트인 만큼 업스트림(Upstream) 저장소의 변경사항을 Sync(Merge/Pull) 받는 경우가 잦습니다. 이때 의존성 파일들에 충돌이 일어날 수 있습니다.

### 패키지 모듈 동기화
업스트림 갱신 후 반드시 Root에서 패키지를 동기화해주세요:
```bash
bun install
```

### Cargo.lock 충돌 해결법
다른 사람이 `Cargo.toml` 기반의 새로운 패키지를 추가했거나 버전을 바꾼 경우 `Cargo.lock`에 충돌 마커(`<<<<<<<`)가 생길 수 있습니다.
이 경우 다음 과정을 통해 해결하십시오:
1. `git checkout --theirs Cargo.lock` (또는 수동으로 충돌 마커 삭제)
2. `cargo metadata --format-version 1 --no-deps` 실행 (워크스페이스 멤버 기반으로 `Cargo.lock` 자동 복구 및 갱신)
3. `git add Cargo.lock` 후 `git commit`

---

## 4. 아키텍처 이해하기
* **`apps/web`**: **UI와 순수 프론트엔드 비즈니스 로직**을 담당합니다. 웹 브라우저에서도 돌아가고 Tauri 데스크탑 창 내부에서도 돌아갑니다.
* **`apps/tauri`**: 오직 데스크탑 네이티브 윈도우를 띄워주는 **"크롬 없는 껍데기(Wrapper)"** 역할만 합니다. 
* 파일 시스템 제어, 하드웨어 접근 등의 복잡한 로직이 추후 필요하다면, `apps/tauri/src-tauri` 또는 모노레포의 `rust/crates` 내에 네이티브 Rust 코드로 작성한 뒤, `tauri::command`를 통해 프론트엔드로 브릿지(Bridge)하여 사용하십시오.
