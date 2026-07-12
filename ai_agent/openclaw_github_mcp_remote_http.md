# OpenClaw GitHub MCP 원격 서버 설정 정리

이 문서는 Mac mini에서 Docker 없이 GitHub MCP를 OpenClaw에 연결하는 방법을 정리한 것이다. 로컬에서 GitHub MCP Docker 컨테이너를 실행하지 않고, GitHub Copilot MCP의 원격 HTTP endpoint에 직접 연결한다.

실제 GitHub PAT 값은 문서에 기록하지 않는다.

## 전체 구조

원격 MCP 서버 방식은 다음 흐름으로 동작한다.

```text
OpenClaw
  -> GitHub Copilot MCP remote HTTP endpoint
  -> GitHub API
  -> GitHub repository
```

Docker 방식과 달리 로컬 컨테이너를 띄우지 않는다. OpenClaw가 `streamable-http` transport로 GitHub Copilot MCP endpoint에 연결하고, 요청 헤더의 bearer token으로 인증한다.

## 토큰을 env 파일에 저장

GitHub MCP 접속에 사용할 토큰은 OpenClaw 설정에 직접 쓰지 않고 `~/.openclaw/.env`에 저장한다.

간단한 예시는 다음과 같다.

```bash
echo 'GITHUB_MCP_TOKEN=github_pat_xxxxx' >> ~/.openclaw/.env
```

다만 실제 토큰을 입력할 때는 shell history에 남지 않도록 다음처럼 저장하는 편이 더 안전하다.

```bash
install -d -m 700 ~/.openclaw

touch ~/.openclaw/.env
chmod 600 ~/.openclaw/.env

read -rsp "GitHub MCP token: " GITHUB_MCP_TOKEN; echo
printf '\nGITHUB_MCP_TOKEN=%s\n' "$GITHUB_MCP_TOKEN" >> ~/.openclaw/.env
unset GITHUB_MCP_TOKEN
```

`~/.openclaw/.env`에는 다음 형식의 값이 들어간다.

```env
GITHUB_MCP_TOKEN=github_pat_xxxxx
```

이미 `GITHUB_MCP_TOKEN` 항목이 있으면 중복으로 추가하지 말고 기존 값을 교체한다.

## OpenClaw에 GitHub MCP 설정

GitHub MCP 서버를 원격 HTTP endpoint로 등록한다.

```bash
openclaw mcp set github '{
  "url": "https://api.githubcopilot.com/mcp/",
  "transport": "streamable-http",
  "headers": {
    "Authorization": "Bearer ${GITHUB_MCP_TOKEN}"
  }
}'
```

여기서 중요한 점은 `Authorization` 값에 실제 토큰을 직접 쓰지 않고 `${GITHUB_MCP_TOKEN}` 형태로 남겨두는 것이다. OpenClaw가 실행 시점에 `~/.openclaw/.env`의 값을 읽어 치환한다.

## 설정 확인

저장된 MCP 정의를 확인한다.

```bash
openclaw mcp show github
```

확인할 점은 `Authorization` 헤더가 실제 토큰 값이 아니라 다음처럼 환경변수 참조로 남아 있는지다.

```text
Bearer ${GITHUB_MCP_TOKEN}
```

실제 연결과 사용 가능한 tool 개수를 확인한다.

```bash
openclaw mcp probe github
```

`probe`가 성공하면 OpenClaw가 원격 GitHub MCP 서버에 연결했고, GitHub 관련 MCP tool 목록을 사용할 수 있는 상태다.

## Docker 방식과의 차이

Docker 방식은 로컬에서 GitHub MCP 서버 컨테이너를 실행한다.

```text
OpenClaw -> docker run -> local GitHub MCP server -> GitHub API
```

원격 MCP 서버 방식은 OpenClaw가 GitHub Copilot MCP endpoint에 직접 연결한다.

```text
OpenClaw -> GitHub Copilot MCP remote endpoint -> GitHub API
```

따라서 Mac mini에서는 Docker Desktop이나 Colima 없이 더 간단하게 설정할 수 있다.

## 핵심 요약

Mac mini에서는 Docker 없이 GitHub Copilot MCP의 원격 endpoint를 사용할 수 있다. 토큰은 `~/.openclaw/.env`에 `GITHUB_MCP_TOKEN`으로 저장하고, OpenClaw MCP 설정에는 `${GITHUB_MCP_TOKEN}` 환경변수 참조만 남긴다. 설정 후 `openclaw mcp show github`로 정의를 확인하고, `openclaw mcp probe github`로 실제 연결을 검증한다.
