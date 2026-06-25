# OpenClaw GitHub MCP 설정 정리

이 문서는 OpenClaw에서 GitHub MCP 서버를 Docker로 연결하는 방법과 `openclaw.json`에 들어간 MCP 설정의 의미를 정리한 것이다. 실제 GitHub PAT 값은 문서에 기록하지 않는다.

## 전체 구조

OpenClaw는 MCP 설정에 등록된 명령을 실행해서 MCP 서버 프로세스를 띄운다. GitHub MCP 연결은 다음 흐름으로 동작한다.

```text
OpenClaw
  -> docker run ...
  -> 로컬 Docker 컨테이너 안의 GitHub MCP server
  -> GitHub API
  -> GitHub repository
```

GitHub MCP server 프로세스는 로컬 Docker 컨테이너 안에서 실행된다. 서버가 GitHub 저장소를 읽거나 수정할 때는 GitHub API에 접속하며, 이때 env 파일에 저장한 GitHub PAT를 사용한다.

## Docker 설치

Ubuntu 22.04 기준 Docker 설치 예시는 다음과 같다.

```bash
apt-get update
apt-get install -y docker.io
systemctl enable --now docker
docker --version
docker run --rm hello-world
```

root가 아닌 일반 사용자로 Docker를 실행하려면 사용자를 `docker` 그룹에 추가한다.

```bash
usermod -aG docker "$USER"
```

그룹 변경은 로그아웃 후 다시 로그인해야 적용된다.

## GitHub PAT env 파일 저장

PAT는 `openclaw.json`에 직접 쓰지 않고 별도 env 파일에 보관한다.

```bash
install -d -m 700 ~/.openclaw/secrets

read -rsp "GitHub PAT: " GITHUB_PAT; echo
printf 'GITHUB_PERSONAL_ACCESS_TOKEN=%s\n' "$GITHUB_PAT" > ~/.openclaw/secrets/github.env
chmod 600 ~/.openclaw/secrets/github.env
unset GITHUB_PAT
```

env 파일에는 다음 형식의 값이 들어간다.

```env
GITHUB_PERSONAL_ACCESS_TOKEN=YOUR_GITHUB_PAT
```

권한은 필요한 범위만 부여한다. 저장소 파일 읽기/쓰기가 목적이면 fine-grained PAT에서 대상 저장소를 제한하고 `Contents: Read and write`를 우선 부여한다. PR이나 issue 작업이 필요하면 `Pull requests: Read and write`, `Issues: Read and write`를 추가한다.

## GitHub MCP Docker 이미지 받기

GitHub 공식 MCP 서버 이미지는 GitHub Container Registry에 올라와 있다.

```bash
docker pull ghcr.io/github/github-mcp-server
```

`ghcr.io/github/github-mcp-server`는 Docker 이미지 이름이다.

- `ghcr.io`: GitHub Container Registry
- `github/github-mcp-server`: registry 안의 이미지 경로
- Docker는 이 이름으로 이미지를 찾고, 로컬에 없으면 registry에서 내려받아 실행한다.

이미지 안에는 GitHub MCP 서버 실행 파일과 기본 실행 설정이 들어 있다. 현재 이미지의 실행 설정은 다음 형태다.

```text
Entrypoint: ["/server/github-mcp-server"]
Cmd: ["stdio"]
```

즉 컨테이너를 실행하면 GitHub MCP 서버가 `stdio` 모드로 시작된다.

## OpenClaw MCP 등록

OpenClaw에는 MCP 서버를 `openclaw mcp` 명령으로 등록할 수 있다.

처음 등록할 때 `openclaw mcp add`를 사용할 수 있다.

```bash
openclaw mcp add github \
  --command docker \
  --arg run \
  --arg -i \
  --arg --rm \
  --arg --env-file \
  --arg /root/.openclaw/secrets/github.env \
  --arg ghcr.io/github/github-mcp-server
```

줄바꿈이나 shell quoting 때문에 인자 전달이 헷갈릴 때는 JSON 형태로 등록하는 방법이 안정적이다.

```bash
openclaw mcp set github '{"command":"docker","args":["run","-i","--rm","--env-file","/root/.openclaw/secrets/github.env","ghcr.io/github/github-mcp-server"]}'
```

기존 등록을 지우고 다시 설정하려면 다음 순서로 실행한다.

```bash
openclaw mcp unset github
openclaw mcp set github '{"command":"docker","args":["run","-i","--rm","--env-file","/root/.openclaw/secrets/github.env","ghcr.io/github/github-mcp-server"]}'
```

## openclaw.json MCP 설정 풀이

현재 GitHub MCP 설정은 다음 구조다.

```json
{
  "mcp": {
    "servers": {
      "github": {
        "command": "docker",
        "args": [
          "run",
          "-i",
          "--rm",
          "--env-file",
          "/root/.openclaw/secrets/github.env",
          "ghcr.io/github/github-mcp-server"
        ]
      }
    }
  }
}
```

각 항목의 의미는 다음과 같다.

- `mcp.servers.github`: `github`라는 이름의 MCP 서버 등록
- `command: "docker"`: OpenClaw가 실행할 명령
- `args`: `docker` 명령에 넘길 인자 목록
- `run`: Docker 컨테이너 실행
- `-i`: stdin을 열어 MCP client와 서버가 통신할 수 있게 함
- `--rm`: 컨테이너 종료 후 자동 삭제
- `--env-file /root/.openclaw/secrets/github.env`: GitHub PAT가 들어 있는 env 파일 주입
- `ghcr.io/github/github-mcp-server`: 실행할 GitHub MCP 서버 Docker 이미지

OpenClaw가 GitHub MCP 서버를 시작할 때 실제로 실행하는 명령은 다음과 같은 형태다.

```bash
docker run -i --rm --env-file /root/.openclaw/secrets/github.env ghcr.io/github/github-mcp-server
```

## 설정 확인 명령어

등록된 MCP 서버 목록을 본다.

```bash
openclaw mcp list
```

GitHub MCP 서버 설정을 확인한다.

```bash
openclaw mcp show github --json
```

GitHub MCP 서버에 실제로 연결해 사용 가능한 도구를 확인한다.

```bash
openclaw mcp probe github
```

설정을 다시 읽게 한다.

```bash
openclaw mcp reload
```

OpenClaw 설정 유효성을 확인한다.

```bash
openclaw config validate
```

## reload 시 동작

`openclaw mcp reload`를 실행하면 OpenClaw는 MCP 런타임 캐시를 비우고 다음 사용 시 새 설정으로 MCP 서버를 시작한다. GitHub MCP 서버가 필요해지면 `openclaw.json`의 `mcp.servers.github` 설정을 읽고 등록된 Docker 명령을 실행한다.

## GitHub 작업 지시 예시

GitHub MCP 연결 후에는 OpenClaw에게 다음처럼 지시할 수 있다.

```text
jyhan8301/study 저장소의 파일 목록을 확인해줘.
jyhan8301/study에 README.md를 만들어줘.
jyhan8301/study에 브랜치를 만들고 PR을 열어줘.
jyhan8301/study의 issue 목록을 확인해줘.
```

현재 확인된 GitHub MCP 접근 정보는 다음과 같다.

- GitHub 계정: `jyhan8301`
- 접근 가능한 저장소: `jyhan8301/study`
- 저장소 공개 상태: private
- 기본 브랜치 이름: main

## 핵심 요약

OpenClaw는 `openclaw.json`의 MCP 설정을 읽고 Docker 명령을 실행한다. Docker는 `ghcr.io/github/github-mcp-server` 이미지를 로컬 컨테이너로 실행한다. 컨테이너 안의 GitHub MCP 서버는 `stdio`로 OpenClaw와 통신하고, GitHub PAT를 사용해 GitHub API에 접근한다.
