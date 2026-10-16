# Unity Editor·MCP 서버 기동

에이전트가 터미널에서 Unity Editor를 띄우고 mcp-for-unity 연결까지 올리는 절차와, 에디터 UI 없이 EditorPrefs를 바꾸는 방법이다. Unity CLI와 CoplayDev mcp-for-unity(HTTP Local 전송)를 전제로 한다. 2026-10-16에 Unity 2022.3 빈 프로젝트로 전 과정을 실험한 결과를 반영했다.

## 구성

| 구성 요소 | 내용 |
|---|---|
| Unity CLI | `unity` 명령. 에디터 설치·실행·종료·빌드·테스트를 다룬다. 설치는 [Unity CLI 안내](https://unity.com/blog/meet-the-unity-cli)를 따른다. |
| mcp-for-unity 서버 | 에디터가 `uvx mcp-for-unity --transport http`로 띄우는 Python 프로세스. 기본 주소 `http://127.0.0.1:8080/mcp`. `uv`가 설치돼 있어야 한다. 한 PC에서 8080 서버는 하나만 뜨고, 여러 에디터가 같은 서버에 인스턴스로 붙는다. |
| 에디터 브리지 | MCP for Unity 창의 Transport=HTTP Local. 서버가 뜨면 에디터가 연결한다. |

## 새 프로젝트에 무인 설치

`project-setup.md`의 프로젝트 생성 절차로 쓴다. 사용자가 "프로젝트 만들자"라고만 하면 아래 세 가지를 질문 하나로 묻고, 답을 받은 뒤 1\~4단계를 에이전트가 끝까지 한다.

| 항목 | 정해지지 않았을 때 |
|---|---|
| 이름(폴더 이름) | 묻는다. 기본값 없음 |
| 에디터 버전 | `unity editors --format json`으로 설치된 버전 목록을 보여 주고 고르게 한다. 설치된 것이 없으면 `unity releases`에서 최신 LTS를 제안하고 `unity install`로 설치한다 |
| 템플릿 | `com.unity.template.3d`를 기본으로 제안하고 확인만 받는다. 다른 템플릿은 `unity templates list -e <버전> --installed`로 보여 준다 |

1. 프로젝트 생성: `unity projects new <이름> --path <상위 폴더> --editor-version <버전> --template com.unity.template.3d --format json --non-interactive`. 배치모드 에디터가 한 번 돌아 `Library`까지 만들고 끝난다(약 1분). 템플릿 ID는 `unity templates list -e <버전> --installed`로 본다.
2. 패키지 추가: `Packages/manifest.json`의 `dependencies`에 `"com.coplaydev.unity-mcp": "https://github.com/CoplayDev/unity-mcp.git?path=/MCPForUnity#main"`을 넣는다.
3. 사용자 단위 EditorPrefs 확인: 아래 네 키가 1이어야 첫 기동부터 대화상자 없이 서버가 올라온다. 이 PC에서 한 번이라도 MCP for Unity 창을 설정했다면 이미 1이다. 새 PC라면 [EditorPrefs를 UI 없이 바꾸기](#editorprefs를-ui-없이-바꾸기windows) 방법으로 에이전트가 쓴다. 네 키 모두 사용자 단위라 한 번 쓰면 같은 PC의 다른 프로젝트에도 적용된다.

   | 키 | 역할 |
   |---|---|
   | `MCPForUnity.AutoStartOnLoad` | 에디터 로드 뒤 서버·브리지 자동 기동(기본 꺼짐) |
   | `MCPForUnity.UseHttpTransport` | HTTP 전송 사용 |
   | `MCPForUnity.SetupCompleted` | 패키지 첫 임포트 때 뜨는 Setup 창 억제 |
   | `MCPForUnity.HttpServerLaunchConfirmed` | 서버 첫 기동 확인 대화상자 억제(자동 기동 경로는 어차피 묻지 않는다) |

4. 기동: 아래 기동 절차를 따른다. 패키지 임포트·컴파일을 포함해 `unity open` 뒤 약 1분 안에 서버가 LISTEN하고 브리지가 연결된다.

### 단계별 자동화 가능 여부

| 단계 | 사용자 개입 | 근거 |
|---|---|---|
| 프로젝트 생성(`unity projects new`) | 없음 | 배치모드로 생성 후 종료. 다른 에디터가 떠 있어도 충돌 없음 |
| manifest.json 편집 | 없음 | 파일 편집 |
| EditorPrefs 쓰기 | 없음 | 아래 레지스트리 절차. 단, MSIX 가상화 주의 |
| 에디터 기동(`unity open`) | 없음 | 로그인 안 된 상태 경고만 나오고 정상 기동 |
| 서버 자동 기동·브리지 연결 | 없음 | `AutoStartOnLoad`=1이면 Editor.log에 "Starting local HTTP server" → "Server ready" → "Session connected". 0이면 110초 뒤에도 이 줄이 없고 instances가 0개 |
| `tools/call` 조작 | 없음 | curl로 `manage_scene`·`manage_gameobject`·`execute_code` 성공 |
| 떠 있는 에디터 종료 | 필요할 수 있음 | 아래 "에디터 종료" 참고. 에이전트 권한 정책이 다른 작업의 프로세스 종료를 막을 수 있다 |
| 종료 뒤 남은 서버 프로세스 정리 | 필요 | 에디터 종료 정리 코드가 서버를 못 내린다(아래) |

## 기동 절차

기존 프로젝트를 열 때도 아래 순서를 따른다. 자동 기동 설정 확인은 사용자에게 맡기지 않고 에이전트가 한다.

1. 자동 기동 설정 확인: `unity open` 전에 `MCPForUnity.AutoStartOnLoad`를 [아래 WMI 경유 `reg query`](#msix-패키지-앱-안에서-쓸-때필수)로 읽는다. 셸에서 직접 읽은 값은 가상 하이브가 섞여 믿을 수 없다. 값이 0이거나 없으면 같은 방법으로 1을 쓴다. 에디터가 이미 떠 있으면 이 값은 다음 기동부터 반영되므로, 그 에디터는 수동 Connect나 재기동이 필요하다.
2. 에디터 실행: `unity open "<프로젝트 경로>" --format json --non-interactive`. `ProjectSettings/ProjectVersion.txt`에 맞는 에디터가 뜬다. Hub 프로젝트 목록에 없는 프로젝트는 "파일 경로로 열립니다" 경고가 나오지만 정상이다.
3. 자동 기동 대기: 에디터 로드 뒤 서버와 브리지가 자동으로 올라온다. 로드 후 1분 안팎 걸린다. 8080 서버가 이미 떠 있으면 새로 띄우지 않고 브리지만 연결한다.
4. 확인: 8080 포트를 python 프로세스가 LISTEN하고, MCP `initialize` 뒤 `mcpforunity://instances` 리소스에 `<프로젝트명>@…` 인스턴스가 나오면 끝까지 연결된 것이다. Editor.log(`%LOCALAPPDATA%\Unity\Editor\Editor.log`)의 `MCP-FOR-UNITY` 줄로도 단계를 확인할 수 있다.

배치모드(`-batchmode`)에서는 환경변수 `UNITY_MCP_ALLOW_BATCH`가 없으면 자동 기동과 종료 정리를 건너뛴다.

에이전트 세션은 MCP 도구를 시작 시에만 로드한다. 세션 시작 때 연결이 실패했으면 에디터를 띄운 뒤 새 세션을 열거나, 아래 curl 절차로 직접 호출한다.

### curl로 직접 호출

1. `initialize` 요청 → 응답 헤더 `mcp-session-id`를 저장한다.
2. 같은 세션 ID로 `notifications/initialized`를 보낸다.
3. `tools/call`·`resources/read`를 보낸다. 헤더 `Accept: application/json, text/event-stream`, 응답은 `data:` 줄에 JSON이 온다.
4. refresh 직후 `read_console`은 도메인 리로드로 "disconnected"가 올 수 있으므로 10초 간격으로 재시도한다.

## 에디터 종료

- `unity projects close <경로>`는 `com.unity.pipeline`(Unity 6 이상)이 있어야 정상 종료한다. 그 아래 버전은 `--force`(강제 종료)만 되고, 강제 종료는 `EditorApplication.quitting`이 돌지 않는다.
- 브리지가 연결돼 있으면 `execute_code`로 `EditorApplication.delayCall += () => EditorApplication.Exit(0);`를 보내 정상 종료할 수 있다. `EditorApplication.Exit`는 차단 패턴이라 `safety_checks=false`가 필요하고, 저장하지 않으므로 먼저 `manage_scene save`를 한다.
- 에디터가 정상 종료돼도 서버는 남는다. 종료 정리 코드(`McpEditorShutdownCleanup`)가 돌고 pid 파일도 지우지만, `cmd → uvx → uv → mcp-for-unity.exe → python → python3.12` 프로세스 트리는 두 번의 실험 모두 살아남아 8080을 계속 LISTEN했다. 다음 에디터는 이 서버를 재사용하므로 기동에는 지장이 없다. 완전히 내리려면 python 프로세스를 직접 끝낸다.
- 프로세스 수를 셀 때 `Unity.exe`에는 에디터 외에 `AssetImportWorker`가, `unity.exe`(소문자)에는 Unity CLI 인증 브로커가 섞인다. 명령줄의 `-projectpath` 유무로 구분한다.

## EditorPrefs를 UI 없이 바꾸기(Windows)

EditorPrefs는 파일이 아니라 레지스트리 `HKCU\Software\Unity Technologies\Unity Editor 5.x`에 사용자 단위로 저장된다(macOS는 plist, Linux는 prefs 파일).

- 값 이름은 `<키>_h<hash>`다. hash는 키 이름의 UTF-8 바이트를 djb2 XOR 변형으로 돌린 uint32 값이다: `h = 5381; 각 바이트 b마다 h = ((h << 5) + h) ^ b`, 매 단계 `& 0xFFFFFFFF`. 예: `MCPForUnity.AutoStartOnLoad_h2539145689`. 계산은 Python으로 한다. PowerShell 5.1의 `-shl`은 uint32 범위를 넘겨 다른 값이 나온다. 위 네 키를 포함해 여섯 키로 실제 레지스트리 이름과 일치함을 확인했다.
- bool·int는 DWORD, 문자열은 REG_BINARY(UTF-8)다.
- 사용자 단위라 같은 PC의 모든 프로젝트에 적용된다.
- 레지스트리 쓰기만으로 토글이 반영된다. 에디터는 기동 시 레지스트리 값을 읽어 그대로 따른다(1이면 자동 기동, 0이면 하지 않음을 모두 확인).
- 에디터가 떠 있을 때의 동작: 에디터의 `EditorPrefs.Set`은 즉시 레지스트리에 써진다. 에디터가 아직 읽지 않은 키는 레지스트리에서 바로 읽지만, 이미 읽은 키(예: 로드 시 읽는 `AutoStartOnLoad`)는 캐시돼 레지스트리를 바꿔도 다시 읽지 않는다. 정상 종료 때 캐시를 레지스트리에 덮어쓰지는 않는다. 따라서 에디터가 떠 있는 동안 써도 값은 남지만, 반영은 다음 기동부터다.

### MSIX 패키지 앱 안에서 쓸 때(필수)

에이전트가 MSIX로 설치된 앱(예: Store 배포 Claude 데스크톱)의 자식 셸에서 실행되면 PowerShell·Bash·cmd의 `HKCU\Software` 쓰기는 그 앱의 가상 하이브로 가고, Unity가 읽는 실제 레지스트리에는 반영되지 않는다. 읽기는 실제 값과 가상 값을 합쳐 보여 주므로 쓴 값이 보여도 성공이 아니다. 실제 레지스트리 1을 셸에서 0으로 바꾼 뒤 에디터가 계속 1로 동작한 것으로 확인했다.

- 판별: 셸 프로세스의 조상에 `C:\Program Files\WindowsApps\...` 경로의 실행 파일이 있으면 해당된다. `Get-CimInstance Win32_Process`로 `ParentProcessId`를 따라 올라가 본다.
- 쓰기·확인: 패키지 컨텍스트 밖에서 프로세스를 만든다. WMI가 만든 프로세스는 상속받지 않는다.

  ```powershell
  Invoke-CimMethod -ClassName Win32_Process -MethodName Create -Arguments @{ CommandLine = 'cmd.exe /c reg add "HKCU\Software\Unity Technologies\Unity Editor 5.x" /v MCPForUnity.AutoStartOnLoad_h2539145689 /t REG_DWORD /d 1 /f > "C:\Temp\reg.txt" 2>&1' }
  ```

  출력은 파일로 받아 읽는다. 확인도 같은 방법으로 `reg query`를 돌린다. 셸에서 직접 `Get-ItemProperty`로 읽은 값은 가상 하이브가 섞여 있어 믿을 수 없다.
- 셸에서 `Remove-ItemProperty`를 하면 가상 하이브에 삭제 표시가 남아 실제 값이 셸에 보이지 않게 된다. 가상 하이브는 `%LOCALAPPDATA%\Packages\<패키지>\SystemAppData\Helium\`에 있다.

## Unity CLI의 한계

- `unity mcp`·`unity command`·`unity status`·`unity recompile`·`unity projects close`(정상 종료)는 `com.unity.pipeline` 패키지(Unity 6.0 LTS 이상)가 필요하다. 그 아래 버전 프로젝트에서는 mcp-for-unity를 쓴다.
- `unity build`·`unity test`·`unity run`·`unity projects new`는 배치모드로 에디터를 띄우므로 버전과 무관하게 쓸 수 있다.
- 에디터 UI(창의 토글·버튼)는 에이전트가 직접 누르지 못한다. 위 EditorPrefs 방법으로 대신하거나 사용자에게 요청한다.
