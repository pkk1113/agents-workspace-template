# Unity 공통 규칙

Unity 프로젝트에서 작업할 때 항상 따른다. 이 문서와 `Docs/rules/unity/`의 다른 문서에 나오는 경로는 해당 Unity 프로젝트 루트(Workspace의 프로젝트 폴더)를 기준으로 한다. 프로젝트가 직접 관리하는 애플리케이션 에셋은 `Assets/__App__` 아래에 둔다.

## 작업별 규칙

아래 작업을 할 때는 해당 문서를 먼저 읽는다. 이 문서에는 매번 필요한 규칙만 둔다.

| 작업 | 문서 |
|---|---|
| Feature의 외부 접점이나 Composition(Root, Sub-Composition, Adapter)을 만들거나 수정 | `composition.md` |
| 새 폴더·Feature를 만들거나 에셋·스크립트의 위치를 결정 | `asset-layout.md` |
| UI 객체·스크립트를 만들거나 이름을 짓거나 화면의 기본 선택을 정하거나 UI 이벤트를 추가 | `ui.md` |
| `ConstData`, `RuntimeData`, `LocalData`, `PlayerPrefs`를 다룸 | `global-data.md` |
| 씬에 객체를 추가하거나 Hierarchy 구조를 변경 | `scene-hierarchy.md` |
| 에셋·폴더 이동·이름 변경, `[SerializeField]` 필드 이름 변경 | `asset-move.md` |
| 새 Unity 프로젝트를 만들거나 Editor를 터미널에서 띄우거나 mcp-for-unity 연결을 복구하거나 EditorPrefs를 UI 없이 변경 | `editor-launch.md` |

## 💻 코드

### C# 클래스 구성

C# 클래스를 만들거나 수정할 때는, 해당하는 멤버가 있는 경우 아래 순서로 `#region` 블록을 구성한다.

1. `Constants`
2. `Fields`
3. `Properties`
4. `Events`
5. `MonoBehaviour`
6. `Methods`
7. 인터페이스·상위 클래스 이름 영역 (아래 `인터페이스 구현` 참고)
8. `Helpers`

클래스에 필요한 영역만 사용하고, 각 멤버는 역할에 맞는 영역에 둔다. 목적: 어떤 클래스든 같은 위치에서 외부 접점과 구현을 찾을 수 있게 한다.

- `Methods`: public 메서드와 생성자를 둔다.
- `Helpers`: private, protected, internal 등 외부 공개용이 아닌 구현 메서드를 둔다.
- `MonoBehaviour`: 접근 제한자와 관계없이 Unity 생명주기 메서드를 둔다.
- 중첩 타입(`enum`, `class`, `struct`)은 영역 밖, 클래스 본문의 최상단에 둔다.

### 제어문 중괄호

`if`, `else if`, `else`, `for`, `foreach`, `while`, `do` 등 제어문의 본문에는 실행문이 한 줄이어도 반드시 중괄호 `{ }`를 사용한다. 조기 반환, `continue`, `break`만 있는 본문도 예외 없이 적용한다. 중괄호는 각각 별도 줄에 작성한다. 목적: 본문에 줄을 추가할 때 생기는 제어 흐름 오류를 막고 diff를 명확하게 한다.

### 인터페이스 구현

클래스가 인터페이스를 구현하고 인터페이스 전용 멤버를 가진다면, 해당 멤버는 접근 제한자와 관계없이 인터페이스 이름의 별도 영역으로 묶는다. 예: `#region IPoseAdjustmentView`

상위 클래스의 멤버를 override하는 경우에도 같은 방식으로 상위 클래스 이름의 영역으로 묶는다. 단, Unity 생명주기 메서드는 `MonoBehaviour` 영역에 둔다. 예: `#region AuthClient`

### 명명 규칙

#### 클래스

- `Controller`: 수명 주기 세션을 갖지 않고, 상태나 세션에 대한 명령을 전달·조정하는 클래스에 사용한다. 이력 스택이나 선택 상태처럼 해제가 필요 없는 단순 상태는 소유해도 된다.
- `Manager`: 수명 주기(LifeCycle) 세션을 소유하거나 관리하는 클래스에 사용한다.
- 수명 주기 세션은 시작과 종료가 있고 종료 시 해제가 필요한 상태나 자원을 말한다. 예: 로그인 세션, 트래킹 세션, 네트워크·메시지 큐 연결.

#### 필드와 식별자

- 일반 클래스의 필드는 `m_` 접두사와 camelCase를 사용한다. `[SerializeField]` 필드에도 동일하게 적용한다. 예: `m_currentIndex`.
- 가변 `static` 필드는 가급적 사용하지 않는다. 필요하면 `s_` 접두사와 camelCase를 사용한다. 예: `s_instanceCount`.
- 상수가 아닌 식별자에서는 약어도 일반 단어 하나로 취급해 Camel/Pascal 표기를 따른다. 예: `HardwareId`, `QrTrackingHud`, `m_uiRoot`.
- 상수와 고정값 `static readonly` 멤버는 대문자 스네이크 케이스를 사용한다. 예: `SERVER_MQ_IP`.
- DTO의 멤버는 전달 대상의 데이터 계약과 명명 규칙을 따른다. DTO에는 `m_` 필드 규칙을 적용하지 않는다.

### Editor 전용 코드

Editor에서만 쓰는 테스트·디버그 코드(키보드 입력 시뮬레이션 등)는 `#if UNITY_EDITOR`로 감싸거나 Editor 전용 스크립트로 분리한다. 런타임 동작이 Editor 전용 입력이나 객체의 존재 여부에 의존해 조기 반환하거나 생략되지 않도록 한다.

### Inspector 값

사용자의 명시적 조정 요구가 없는 구현 값은 기본적으로 `Constants`에 코드 상수로 관리한다. 사용자가 씬에서 값을 바꿔가며 확인하고 싶다고 요청하는 등 Inspector 조정이 필요하다고 밝힌 경우에만 Inspector에 노출한다. 목적: 씬에서 바꿀 수 있는 값을 의도한 것으로 한정해, 씬 데이터와 코드 중 어디가 기준인지 헷갈리지 않게 한다.

Inspector에서 참조하는 필드는 필수 연결을 전제로 한다. `SerializeField` Reference에 대한 null 검사나 대체 동작을 작성하지 않으며, 연결 누락은 Unity Editor에서 즉시 드러나게 둔다. 목적: 대체 동작이 연결 누락을 가려 늦게 발견되는 것을 막는다.

선택적 Reference는 `SerializeField`로 두지 않는다. 있을 수도 없을 수도 있는 의존성은 Composition이 초기화 `Command`로 전달하며, null 허용 여부는 해당 `Command`의 XML 문서 주석에 명시한다.

### 종류별 표현 값

종류(enum)에 따라 달라지는 표현(모양·색·크기·아이콘 등)은 코드 분기 대신 종류별 데이터(프리팹, ScriptableObject, Inspector 값)에 둔다. 코드는 "종류 → 데이터"를 고르는 한 곳에서만 종류로 분기한다. 이 규칙은 종류별로 달라지는 표현 값에만 적용하며, 그 밖의 구현 값은 위 `Inspector 값`을 따른다. 목적: 종류별 차이를 데이터 한 곳에서 보고 고칠 수 있게 하고, 같은 종류 분기가 여러 메서드에 흩어지지 않게 한다.

## 📦 구조 경계

### Feature 연결

Feature는 `Property`(상태), `Command`(public 메서드), `Event`(변화 알림)만 외부에 제공하고, Feature 사이의 연결은 Composition이 한다. UI 클래스는 기능 동작을 직접 호출하지 않고 UI 이벤트(`SaveButtonPressed`처럼 UI 동작 기준 이름)만 낸다. 상세는 `composition.md`, `ui.md`.

### 배치 판단

배치 위치가 모호하면 소유 범위를 기준으로 결정한다: 앱 외부 시스템과의 연동이면 `Infrastructure`, 여러 곳에서 재사용하면 `Common`, 하나의 제품 기능에만 속하면 `Features`, 한 씬을 구성하기 위해서만 필요하면 `Scenes/<SceneName>`, 그 밖의 전역 리소스와 설정이면 `__App__` 바로 아래다. 폴더별 세부 구조는 `asset-layout.md`.

### 참조 규칙

폴더 간 코드 참조는 아래 표를 따른다. 의존 방향을 한쪽으로 고정해 Feature를 독립적으로 변경·재사용하고, 연결 변경은 Composition 한 곳에서만 일어나게 하기 위함이다. 이 표는 asmdef로 경계를 강제할 때의 기준이기도 하다.

| 참조하는 쪽 | 참조할 수 있는 대상 | 참조하지 않는 대상 |
|---|---|---|
| `Scenes` (Composition, 보조 클래스) | `Features`, `Infrastructure`, `Common`, `Data` | — |
| `Features` | `Common`, Unity·범용 라이브러리 | 다른 Feature, `Infrastructure`, `Data`, `Scenes`, 기기 SDK·OS API |
| `Infrastructure` | `Common`, 외부 SDK·패키지·OS API | `Features`, `Data`, `Scenes` |
| `Data` | `Common`, `Infrastructure`의 데이터 계약(DTO) | `Features`, `Scenes` |
| `Common` | Unity·범용 라이브러리 | `Features`, `Infrastructure`, `Data`, `Scenes` |

- 같은 Feature 안의 하위 폴더끼리는 자유롭게 참조한다. 하위 Feature 사이는 서로 다른 Feature로 취급한다.
- 표에 없는 참조가 필요하면 구현 전에 근거와 대안을 제시하고 논의한다.

## Unity 에셋 변경 원칙

목적: Unity 직렬화 파일과 `.meta`의 GUID 참조는 Editor를 거치지 않고 수정하면 깨지기 쉽고, 깨져도 늦게 드러난다.

- `.unity`, `.prefab`, `.meta` 등 Unity 직렬화 파일을 직접 수정하지 않는다.
- Scene 및 Prefab 변경은 반드시 mcp-for-unity를 통해 수행한다.
- Scene·Prefab 변경은 서브에이전트에 맡기지 않고 메인이 직접 한다. 여러 에이전트가 동시에 Editor를 조작하지 않게 한다.
- 에셋과 폴더의 이동·이름 변경은 `.meta`의 GUID 참조가 유지되도록 mcp-for-unity로 수행한다. 파일 탐색기, 셸 명령, git으로 옮기지 않는다.
- mcp-for-unity를 사용할 수 없으면 파일 직접 수정으로 대체하지 말고 사용자에게 알린다.
- mcp-for-unity 패키지가 설치되어 있지 않으면 직접 설치하지 말고, [CoplayDev/unity-mcp](https://github.com/CoplayDev/unity-mcp) 설치를 진행할지 사용자에게 묻는다.
- 패키지는 설치되어 있지만 MCP 연결을 사용할 수 없으면 재설치하지 말고, Unity Editor의 MCP 세션 시작 또는 에이전트의 MCP 재연결이 필요한지 사용자에게 알린다.

## ✅ 스크립트 변경 검증

- Unity 컴파일이 끝날 때까지 기다린 뒤 Console의 오류를 확인한다. mcp-for-unity의 refresh와 Console 조회를 사용한다. 오류가 남은 상태로 완료를 보고하지 않는다.
- 가능한 범위에서 Play 모드로 동작을 확인한다.
