# Agorago Releases

Agorago 는 tmux 위의 GUI 터미널 멀티플렉서입니다. 앱을 닫아도(또는 SSH 로 붙던 윈도우가
재부팅돼도) 셸이 계속 돌고, 다시 열면 같은 셸에 붙습니다.

이 저장소는 Agorago 의 공개 배포 저장소입니다. 소스는 들어 있지 않고, 설치 파일과 업데이트
서명, 앱이 업데이트를 확인하는 `latest.json` 만 올라옵니다.

## 설치

[Releases](https://github.com/southglory/agorago-releases/releases/latest) 에서 받습니다.

| 운영체제 | 파일 |
|---|---|
| Windows | `Agorago_<버전>_x64-setup.exe` (또는 `.msi`) |
| Linux | `Agorago_<버전>_amd64.AppImage`, `.deb`, `.rpm` |

Windows 설치 파일은 아직 코드 서명이 없어 SmartScreen 이 처음 한 번 경고합니다.
**추가 정보 → 실행**을 누르면 됩니다.

## 업데이트

설치한 뒤에는 앱이 이 저장소의 `latest.json` 을 확인해 스스로 업데이트합니다.
각 설치 파일 옆의 `.sig` 는 업데이트 서명이고, 앱에 들어 있는 공개키로만 검증됩니다.

v0.12.0 이하에서 올라오는 경우에는 서명 키가 바뀌어 자동 업데이트가 되지 않습니다.
이 저장소의 설치 파일로 한 번 직접 설치하면 그다음부터는 자동입니다. 설정과 작업대는
그대로 남습니다.

## 에이전트용 안내 (스킬)

Claude Code, Codex, Gemini CLI 같은 에이전트가 Agorago 의 기능, 설정 위치, 문제 해결을 알도록
안내 문서를 스킬로 둡니다. 릴리스마다 `agorago-skill.md` 로 함께 올라옵니다.

- Agorago 안에서: **설정 → 에이전트 → agorago 스킬 설치** 의 **지금 설치**. 앱에 든 판이 그대로 들어갑니다.
- 앱 밖에서 직접 (이미 있는 파일은 건드리지 않습니다):

```sh
for d in ~/.claude/skills/agorago ~/.agents/skills/agorago; do
  if [ -e "$d/SKILL.md" ]; then echo "$d/SKILL.md 이 이미 있어 그대로 둡니다"; else
    mkdir -p "$d" && curl -fsSL https://github.com/southglory/agorago-releases/releases/latest/download/agorago-skill.md -o "$d/SKILL.md"; fi
done
```

```powershell
foreach ($d in "$HOME\.claude\skills\agorago", "$HOME\.agents\skills\agorago") {
  if (Test-Path "$d\SKILL.md") { "$d\SKILL.md 이 이미 있어 그대로 둡니다" }
  else { New-Item -ItemType Directory -Force $d | Out-Null
         Invoke-WebRequest https://github.com/southglory/agorago-releases/releases/latest/download/agorago-skill.md -OutFile "$d\SKILL.md" } }
```

설치한 앱과 같은 판이 필요하면 `latest/download` 를 `download/v<버전>` 으로 바꿉니다. WSL 이나 SSH 서버에서 도는
에이전트는 그 기계 안에서 위 명령을 실행합니다. 새 판으로 바꾸려면 파일을 지우고 다시 실행합니다.

## 릴레이 직접 운영

같은 네트워크 안에서 공유할 때는 필요 없습니다. 휴대폰 데이터나 다른 네트워크에서 공유 터미널을 열려면 양쪽이
닿는 릴레이가 필요합니다. Agorago 에는 내장 릴레이도 기본 주소도 없으므로, 자기 서버에 직접 띄워 씁니다.

릴리스마다 `agorago-relay-<버전>.tar.gz` 가 함께 올라옵니다. 릴레이, 관리 페이지, 그 판의 뷰어 페이지, Dockerfile,
compose 파일이 들어 있어 소스 없이 띄울 수 있습니다. 압축을 풀고 안의 `README.md` 를 따르십시오.

```sh
tar -xzf agorago-relay-<버전>.tar.gz && cd agorago-relay-<버전>
cp env.example .env   # RELAY_ADMIN_SECRET 을 채웁니다
docker compose up -d --build
```

앱과 같은 판의 묶음을 쓰십시오. 릴레이 모드에서는 터미널 내용이 그 서버를 거쳐 가므로, 남이 운영하는 릴레이를 쓸 때는
P2P 모드를 고르는 것이 안전합니다.

## 라이선스

Apache License 2.0. [LICENSE](LICENSE), [NOTICE](NOTICE) 를 보십시오.
