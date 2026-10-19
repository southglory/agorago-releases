# Agorago Releases

Agorago 의 공개 배포 저장소입니다. 소스는 들어 있지 않고, 설치 파일과 업데이트
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

## 라이선스

Apache License 2.0. [LICENSE](LICENSE), [NOTICE](NOTICE) 를 보십시오.
