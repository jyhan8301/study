# 맥 미니 OpenClaw에 다른 기기에서 접속하기

윈도우·아이폰·아이패드는 클라이언트로만 사용하고, 기존 Gateway와 세션, 작업 실행은 맥 미니에 둔다. 먼저 **Tailscale Serve + 브라우저**를 연결하고, 윈도우 앱은 필요할 때 추가한다.

## 1. 맥 미니와 윈도우를 같은 tailnet에 연결

1. 두 기기에 [Tailscale](https://tailscale.com/download)을 설치하고 같은 사설 네트워크(tailnet)에 로그인한다.
2. 맥 미니에서 `tailscale status`로 연결을 확인한다. 명령을 찾지 못하면 설치한 macOS 버전의 [CLI 안내](https://tailscale.com/kb/1080/cli)를 확인한다.
3. Serve 설정 중 HTTPS 인증서 활성화 안내가 나오면 Tailscale 관리 콘솔에서 해당 안내를 따른다.

## 2. 맥 미니의 기존 Gateway에 Serve 설정

기존 설정을 확인한다. 이미 Serve가 있거나 443 포트를 다른 서비스가 쓰고 있다면 덮어쓰지 말고 먼저 해당 경로의 용도를 확인한다.

```sh
openclaw gateway status
openclaw config get gateway.bind
openclaw config get gateway.tailscale.mode
tailscale serve status
```

기존 Gateway가 기본 로컬 바인딩(loopback)이라면 그대로 유지하고 맥 미니 터미널에서 다음을 실행한다. 재시작 중 기존 연결이 잠깐 끊길 수 있다.

```sh
openclaw config set gateway.tailscale.mode serve
openclaw gateway restart
openclaw gateway status
tailscale serve status
```

마지막 출력에 나온 HTTPS 주소를 사용한다. Serve는 tailnet 안에서만 접근할 수 있다. 공개용 Funnel이나 공유기 포트 포워딩은 필요 없다. 기존 Gateway 인증은 그대로 둔다.

## 3. 윈도우에서 기존 세션 확인

1. Tailscale이 연결된 상태에서 Edge나 Chrome으로 Serve의 HTTPS 주소를 연다.
2. 인증을 요청하면 기존 Gateway 자격 증명을 로그인 화면에 입력한다. 기기 승인이 필요하면 맥 미니에서 `openclaw devices list`로 본인의 요청인지 확인한 뒤 `openclaw devices approve <request-id>`로 승인한다. 토큰은 문서나 채팅에 올리지 않는다.
3. 새 채팅 대신 기존 세션을 열어 이전 메시지를 확인하고 테스트 메시지를 보낸다. 다른 기기에서도 동일한 세션을 선택해야 같은 대화가 이어진다.

## 4. 아이폰·아이패드 및 외부 접속

기기에도 Tailscale을 설치해 같은 tailnet에 가입한 다음 Safari에서 같은 HTTPS 주소와 같은 세션을 연다. 아이폰은 Wi-Fi를 끄고 셀룰러로 시험하면 외부 접속을 확인할 수 있다. 맥 미니는 켜져 있고 Gateway와 Tailscale이 실행 중이어야 한다.

## 선택: Windows Hub 앱

[공식 Windows Hub](https://docs.openclaw.ai/platforms/windows#recommended-windows-hub)를 설치한 뒤 **Advanced setup → Connections → Remote Gateway**를 선택한다. **Set up locally**는 선택하지 않는다. Serve 호스트의 Gateway WebSocket 주소(`wss://…`)와 앱에서 요구하는 인증 정보를 사용한다. Windows Node 모드는 필요 없다. 앱 연결과 원하는 세션 표시 여부는 설치 후 확인하고, 우선 브라우저를 기준으로 삼는다.

## 막힐 때

- **주소가 안 열림:** 양쪽 Tailscale 연결, 맥 미니의 절전 상태, `tailscale serve status`에 나온 주소와 경로부터 확인한다.
- **화면은 열리지만 연결 안 됨:** Gateway 상태, 로그인·기기 승인 메시지를 확인한다.
- **이전 대화가 안 보임:** 같은 맥 미니 Gateway에 연결했는지, 같은 세션을 열었는지, 해당 세션을 볼 권한이 있는지 확인한다.
- **브라우저 채팅은 되는데 일부 위젯만 비어 있음:** 앱·HTML 위젯은 별도 샌드박스 주소가 필요할 수 있다. 기본 채팅과는 별개다.

공식 문서: [Tailscale Serve 설정](https://docs.openclaw.ai/gateway/tailscale) · [Control UI](https://docs.openclaw.ai/web/control-ui) · [Windows Hub](https://docs.openclaw.ai/platforms/windows)
