# Envio CLI 업데이트 안내

## v0.3.0

### Added

- `envio history` 명령을 추가했습니다.
  - 서버에 저장된 환경변수 버전 목록을 조회합니다.
  - 목록 단계에서는 복호화하지 않고 메타데이터만 표시합니다.
  - 사용자가 선택한 버전을 로컬 마스터 키로 즉시 복호화해 `.env` 내용을 출력합니다.
- history API 클라이언트와 응답 파싱을 추가했습니다.
  - `data: [...]` 배열 응답과 `{ "histories": [...] }` 응답을 모두 지원합니다.
  - `histories_id`와 `snake_case` 필드명을 수용합니다.
- 대화형 history 화면에서 복호화 후 버전 목록으로 돌아가거나 종료할 수 있는 작업 선택 흐름을 추가했습니다.
- 릴리스 전 검증과 태그 push를 돕는 `scripts/release.ps1` 배포 helper를 추가했습니다.

### Changed

- history 목록 출력에 번호, 버전, 생성 시각, 작성자, 상태 컬럼과 선택 안내 문구를 표시합니다.
- 서버가 latest 값을 주지 않으면 가장 큰 `version_id`를 latest로 보정합니다.
- Linux race 테스트 환경에서 legacy `.envio` 파일 fallback이 동작하도록 CLI 테스트 헬퍼를 보강했습니다.

### Notes

- `envio history v3`처럼 버전을 직접 지정하면 대화형 목록 없이 해당 버전을 바로 복호화합니다.
- `envio diff` 명령은 이번 릴리스 범위에 포함되지 않았습니다.
- 서버에는 암호문만 저장되며, 복호화는 CLI 로컬 마스터 키로 수행됩니다.


## v0.2.0

### Added

- `envio push` 명령을 추가했습니다.
  - 로컬 환경변수 파일을 읽고 암호화한 뒤 Envio 프로젝트에 업로드합니다.
  - 업로드 성공 후 서버가 반환한 version ID를 로컬 동기화 상태로 저장합니다.
- `envio pull` 명령을 추가했습니다.
  - 서버의 최신 암호화 환경변수 데이터를 내려받아 로컬에서 복호화합니다.
  - 복호화한 환경변수 파일을 로컬 경로에 저장하고 version ID를 갱신합니다.
- 프로젝트 환경변수 동기화 API 클라이언트와 요청/응답 타입을 추가했습니다.
- dotenv 파싱과 환경변수 암복호화 흐름을 추가했습니다.
- 동기화 충돌, 미초기화 버전, 접근 거부, 프로젝트 미연결 상황에 대한 오류 처리를 추가했습니다.

### Changed

- 프로젝트 로컬 세션과 link config에 동기화 기준 version ID를 저장하도록 확장했습니다.
- Go 품질 게이트 지적 사항을 정리했습니다.

### Notes

- 기본 환경변수 파일 경로는 저장소 루트의 `.env`입니다.
- `VERSION_CONFLICT`가 발생하면 `envio pull` 후 다시 `envio push`를 실행해야 합니다.
- `ENVIRONMENT_VERSION_NOT_INITIALIZED`가 발생하면 `envio push`로 첫 환경변수 버전을 생성해야 합니다.



---
## v0.1.0

`v0.1.0`은 Envio CLI의 MVP 릴리스입니다. GitHub OAuth 로그인, 프로젝트 생성/연동, 다국어 CLI 출력, GoReleaser 기반 배포 파이프라인까지 CLI를 배포 가능한 형태로 구성한 버전입니다.

### 주요 기능

#### GitHub OAuth 로그인

- `envio login` 명령을 통해 GitHub OAuth 기반 로그인 흐름을 제공합니다.
- CLI 기기를 등록하고, 서버 응답의 사용자 ID, GitHub ID, 기기 ID를 로컬 세션에 저장합니다.
- 이미 로그인된 상태에서는 중복 로그인 대신 경고 결과를 반환합니다.
- `--device-name` 옵션으로 등록할 CLI 기기 이름을 지정할 수 있습니다.

```bash
envio login
envio login --device-name my-laptop
```

#### 로컬 키 및 세션 관리

- RSA 키를 생성하고 서버 계약에 맞는 `ssh-rsa` 공개키 형식으로 등록합니다.
- 프로젝트 마스터 키와 기기별 wrapped key 처리를 위한 암호화 유틸리티를 포함합니다.
- 로그인 세션은 전역 세션으로 관리하고, 프로젝트별 세션은 저장소 단위로 분리합니다.
- 민감한 키 자료는 평문 출력에 노출하지 않는 방향으로 렌더링 경계를 구성했습니다.

#### 프로젝트 생성

- `envio create <repository-url>` 명령으로 현재 Git 저장소를 Envio 프로젝트로 등록합니다.
- Git origin과 입력한 GitHub 저장소 URL을 검증합니다.
- 프로젝트 생성 API를 호출하고, 프로젝트 마스터 키를 멤버 공개키별로 암호화해 저장합니다.
- 생성 후 로컬 저장소에 프로젝트 세션 정보를 저장합니다.

```bash
envio create https://github.com/owner/repo
envio create --repo https://github.com/owner/repo
```

#### 기존 프로젝트 연결

- `envio link [repository-url]` 명령으로 현재 Git 저장소를 기존 Envio 프로젝트에 연결합니다.
- repository URL을 생략하면 현재 Git origin을 기준으로 프로젝트 연결을 시도합니다.
- 서버에서 받은 wrapped master key를 로컬에서 복호화하고 OS Keyring에 저장합니다.
- `.envio/config` 기반 로컬 연결 정보를 저장해 이후 프로젝트 명령에서 재사용합니다.

```bash
envio link
envio link https://github.com/owner/repo
```

#### 출력 모드와 다국어 지원

- 기본 출력 외에 plain text, JSON, debug JSON 출력을 지원합니다.
- `--lang en`, `--lang ko` 옵션 및 `ENVIO_LANG` 환경변수를 통해 출력 언어를 선택할 수 있습니다.
- 사용자 입력 오류는 사용법과 허용값을 함께 보여주는 CLI 입력 오류 경로로 렌더링됩니다.

```bash
envio login --plain
envio login --json
envio login --debug
envio login --lang ko
```

#### 버전 명령과 배포 파이프라인

- `envio version` 명령이 CLI 버전, 커밋, 빌드 시각을 출력합니다.
- GoReleaser가 빌드 시 `main.version`, `main.commit`, `main.date` 값을 주입합니다.
- `v*` 태그 push 시 GitHub Actions release workflow가 실행됩니다.
- 릴리스 전 `go test ./...`와 `go vet ./...`를 실행하도록 구성되어 있습니다.
- GitHub Release, Homebrew Cask, Scoop 배포 설정을 포함합니다.

```bash
envio version
```

### v0.1.0에서 사용 가능한 주요 명령

```bash
envio login                 # GitHub OAuth 로그인 및 기기 등록
envio create <repo-url>     # 현재 Git 저장소를 새 Envio 프로젝트로 등록
envio link [repo-url]       # 현재 Git 저장소를 기존 Envio 프로젝트에 연결
envio version               # CLI 버전, 커밋, 빌드 시각 출력
```
