
## Installation

Envio CLI는 Windows 환경에서 Scoop을 통해 설치할 수 있습니다.

### 1. Scoop 설치

PowerShell을 열고 아래 명령어를 실행합니다.

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression
````

이미 Scoop이 설치되어 있다면 이 단계는 건너뛰어도 됩니다.

### 2. Envio bucket 추가

```powershell
scoop bucket add envio https://github.com/Team-NEEEE/scoop-bucket.git
```

### 3. Envio CLI 설치

```powershell
scoop install envio
```

### 4. 설치 확인

```powershell
envio --version
```

정상적으로 설치되었다면 Envio CLI 버전 정보가 출력됩니다.

````

추가로 README에 바로 붙일 수 있는 짧은 버전은 아래처럼 써도 됩니다.

```md
## Installation

Windows에서는 Scoop을 사용해 Envio CLI를 설치할 수 있습니다.

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression

scoop bucket add envio https://github.com/Team-NEEEE/scoop-bucket.git
scoop install envio
````

설치 후 아래 명령어로 정상 설치 여부를 확인합니다.

```powershell
envio --version
```
