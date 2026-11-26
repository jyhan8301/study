# 윈도우에서 맥 미니 OpenClaw의 기존 대화 이어 쓰기

**윈도우 브라우저에서 기존 세션을 열어 대화를 이어갈 수 있음을 확인했다.** 윈도우는 화면과 입력만 담당하고, 대화 기록과 작업 실행은 맥 미니의 기존 OpenClaw Gateway에 남는다. SSH, 별도 Windows 앱, 공유기 포트 포워딩은 필요 없다.

## 1. 맥 미니와 윈도우를 Tailscale에 연결

1. 두 기기에 [Tailscale](https://tailscale.com/download)을 설치한다. 처음이라 계정이 없으면 한 번만 **Sign up** 하고, 두 기기에서 같은 계정으로 **Sign in** 한 뒤 연결한다.
2. [Tailscale 관리자 콘솔](https://console.tailscale.com/admin)의 **Machines** 탭에 맥 미니와 윈도우 PC가 모두 보이는지 확인한다. 처음 보이는 **Add first device / Add device**는 기기 추가 안내다. 두 기기가 이미 목록에 있다면 다시 추가하지 않아도 된다.
3. 같은 콘솔의 **DNS** 탭에서 **MagicDNS**와 **HTTPS Certificates**가 켜져 있는지 확인한다. 이미 켜져 있으면 그대로 둔다.

## 2. 맥 미니에서 OpenClaw의 Tailscale Serve 켜기

기존 Gateway가 실행 중인 맥 미니의 터미널에서 아래 명령을 실행한다. Gateway의 기존 로컬 바인딩과 인증은 그대로 둔다. 공개용 **Funnel**이 아니라 같은 Tailscale 네트워크에서만 접근하는 **Serve**를 사용한다.

```sh
openclaw config set gateway.tailscale.mode serve
```

이번에는 설정 적용 시 Gateway가 자동으로 재시작됐다. 수동 재시작을 먼저 반복하지 말고 아래 명령으로 실행 상태를 확인한 뒤, 다음 단계의 HTTPS 주소를 열어 본다.

```sh
openclaw gateway status --deep
```

수동 재시작에서 `GATEWAY_RESTART_PREPARATION_REFUSED`나 `Gateway was not signaled`가 나와도, 이미 Gateway가 정상이고 HTTPS 주소가 열리면 재시작할 필요가 없다. Gateway가 실제로 멈춰 있고 상태 출력에 시작 오류가 있다면 그 원인을 먼저 해결한 뒤 `openclaw gateway start`를 사용한다.

## 3. 윈도우에서 접속 주소 찾고 기존 세션 열기

1. 윈도우에서 Tailscale을 연결한 상태로 [Tailscale 관리자 콘솔](https://console.tailscale.com/admin)을 연다.
2. **Machines → 맥 미니 선택 → Full domain**에 표시된 전체 도메인을 복사한다. 앞에 `https://`를 붙여 Edge나 Chrome에서 연다. 예: `https://mac-mini.example.ts.net/`. **Full domain**은 호스트 이름이고, OpenClaw 화면은 맥 미니에서 Serve가 켜진 뒤에 열린다.
3. `pairing required`가 뜨는 경우에만 맥 미니에서 `openclaw devices list`를 실행한다. 방금 접속한 브라우저의 요청 ID를 확인해서 `openclaw devices approve <요청-ID>`로 승인한 뒤 윈도우 브라우저를 새로고침한다.
4. OpenClaw 사이드바에서 **새 채팅 대신 이어가려는 기존 대화**를 선택한다. 이전 메시지가 보이는지 확인하고 테스트 메시지를 보내면 된다.

**주소가 안 열리면:** 두 기기의 Tailscale 연결, 맥 미니의 절전 여부, Gateway 상태를 확인한다. 맥 미니의 `tailscale serve status --json`에서 `Web` 아래의 호스트 이름으로 주소를 다시 확인할 수 있다. `Proxy`의 `127.0.0.1` 주소는 맥 미니 내부용이다. macOS에서 `tailscale` 명령을 찾지 못하고 Tailscale 앱이 기본 위치에 설치되어 있다면 `/Applications/Tailscale.app/Contents/MacOS/Tailscale serve status --json`을 실행한다.

## 아이폰·아이패드나 외부에서 쓰려면

이번 확인 범위는 윈도우 브라우저다. 아이폰·아이패드에서도 Tailscale을 설치해 같은 네트워크에 연결한 뒤 Safari에서 같은 HTTPS 주소와 기존 대화를 열면 된다. 외부 접속은 Wi-Fi를 끄고 셀룰러에서 시험해 볼 수 있다. 맥 미니는 절전 상태가 아니어야 하고, Tailscale과 Gateway가 실행 중이어야 한다.

공식 문서: [Tailscale Serve 설정](https://docs.openclaw.ai/gateway/tailscale) · [OpenClaw Control UI](https://docs.openclaw.ai/web/control-ui)
