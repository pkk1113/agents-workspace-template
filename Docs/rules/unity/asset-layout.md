# 에셋 구성

새 폴더·Feature를 만들거나 에셋·스크립트의 위치를 정할 때 읽는다. 경로는 Unity 프로젝트 루트를 기준으로 하며, 프로젝트가 직접 관리하는 애플리케이션 에셋은 `Assets/__App__` 아래에 둔다.

## 에셋 구성

`__App__` 하위 에셋은 소유 범위와 재사용 수준에 따라 배치한다.

- `Common`에는 특정 기능에 속하지 않고 여러 기능·씬에서 재사용되는 구성 요소를 둔다. 세부 구조는 아래 `Common 폴더 구성`을 따른다.
- `Infrastructure`에는 서버, 메시지 브로커, 기기 SDK, OS 등 앱 외부 시스템과 통신하거나 이를 조회하는 코드를 둔다. 세부 구조는 아래 `Infrastructure 폴더 구성`을 따른다.
- `Features`는 제품 기능 단위로 구성한다. 한 씬에서만 사용하는 UI 프리팹·모델·재질·오디오도 독립된 제품 기능이 소유하면 해당 Feature에 둔다.
- `Scenes`는 씬 이름별 하위 디렉터리로 구성한다. 해당 씬 파일, Composition(Root Composition과 보조 클래스), 씬 조립 도구, 씬 배치에 직접 종속되는 데이터와 베이크 결과를 함께 둔다. Composition의 구성은 `composition.md`를 따른다.
  - 씬 파일은 씬 폴더 바로 아래에 두고, Composition 스크립트는 `Scripts`, 씬 조립 도구는 `Scripts/Editor`, 씬 배치에 종속되는 데이터 에셋은 `Data`에 둔다. 베이크 결과 등은 역할에 맞는 하위 폴더에 둔다.
- `Data`, `Font`, `RenderPipeline`처럼 기능이나 특정 씬에 종속되지 않는 프로젝트 전역 데이터·리소스·설정은 `__App__` 바로 아래의 역할별 폴더에 둔다. 여러 Feature 또는 여러 시각 세트가 공유하는 Custom Shader는 `RenderPipeline/Shaders`에 두며, 특정 Feature 또는 하나의 시각 세트에만 쓰이는 Shader는 그 소유 Feature 또는 시각 세트에 둔다.

배치 위치가 모호할 때의 판단 기준과 폴더 간 참조 규칙은 `core.md`에 있다.

## Common 폴더 구성

`Common`은 특정 Feature가 소유하지 않고 여러 Feature 또는 씬에서 재사용되는 구성 요소를 관리한다.

- Inspector에 부착해 사용하는 Behaviour는 `Components`에 둔다.
- 스크립트에서 호출하는 공용 보조 기능은 `Utils`에 둔다.
- 재사용 가능한 Prefab 기반 UI 부품과 여러 UI 부품이 공유하는 시각 에셋은 `UI`에 둔다.
- UI에 한정되지 않는 공용 Material, Texture, Sprite 같은 시각 에셋은 `Visuals`에 둔다.

## Infrastructure 폴더 구성

`Infrastructure`는 앱 외부 시스템과의 연동을 연동 대상별 모듈로 구성한다. 예: `Infrastructure/Network`(REST 클라이언트, 메시지 큐, DTO), `Infrastructure/Device`(기기 SDK, OS 상태 조회).

- 각 모듈의 스크립트는 `Scripts`에 두고, 필요하면 `Scripts/Api`, `Scripts/Mq`, `Scripts/Dto`처럼 역할별 하위 폴더로 나눈다.
- 외부 시스템의 데이터 계약(DTO)과 통신 수단만 제공한다.
- Composition이 Infrastructure 객체를 소유하고, 외부 데이터를 Feature 데이터로 변환해 Feature의 `Command`로 전달한다. 변환 책임이 커지면 `Adapter`로 분리한다(`composition.md`의 `Composition 구성`).
- 폴더와 모듈 이름은 연동 대상을 나타내는 단수형 명사를 사용한다. 예: `Network`, `Device`.

목적: 외부 시스템의 변경(서버 계약, SDK 교체, OS 차이)이 Infrastructure와 Composition의 변환 코드에서 멈추고 Feature까지 번지지 않게 한다.

## Features 폴더 구성

`Features`는 제품 기능을 기준으로 구성한다. 폴더 구조는 기능의 범위와 책임을 드러내야 한다.

### 기능 계층

Feature 폴더는 제품 기능과 그 하위 기능의 소유 범위를 나타낸다. Feature가 직접 책임지는 동작, 상태 또는 UI가 있다면 그 Feature 폴더에 `Scripts`, `Textures`, `Models` 등의 구현 에셋을 바로 둔다. 최상위 Feature와 중간 Feature도 단순 분류 공간이 아니라 직접 책임을 가진다면 구현 에셋을 소유할 수 있다.

하위 Feature는 독립적으로 소유·변경되는 책임이 있을 때만 만든다. 하위 Feature들을 함께 감싸는 폴더는 `Menus`, `Panels`, `Tracking`처럼 그 항목들이 공유하는 책임을 설명할 수 있을 때만 사용한다. 공통 책임이 없다면 억지로 감싸지 말고 상위 Feature 바로 아래에 둔다.

`Scripts`, `Textures`, `Materials`, `Prefabs`, `Models` 같은 전통적인 에셋 종류 이름은 Feature 내부에서 구현물을 정리하는 데만 사용한다. 하위 Feature를 표현하거나 분류하는 이름으로 사용하지 않는다. 순수하게 하위 Feature를 묶는 분류 폴더에는 구현 에셋을 직접 두지 않는다.

### Feature 구현 에셋

각 Feature는 자신이 소유한 구현 스크립트와 에셋을 함께 관리한다. 에셋 수와 상관없이 역할별 하위 폴더를 사용한다.

- 스크립트는 `Scripts`에 둔다.
- 텍스처는 `Textures`에 둔다.
- 모델과 모델에 종속되는 Materials, Textures, Animation Controller, Clips, Prefabs는 모두 `Models` 하위에 둔다.

### 적용 원칙

- 기능의 이름과 폴더 경로만으로도 어떤 책임을 가지는지 알 수 있어야 한다.
- 하나의 기능이 커지면 역할이 다른 하위 기능으로 나눌 것을 제안하고, 각 하위 기능 안에서 에셋 종류를 분류한다.
- 여러 Feature가 같은 로직을 쓰지만 프로젝트 고유 값이 섞여 있으면 둘로 나눈다. 프로젝트 개념을 모르는 범용 로직·타입은 `Common/Utils`에 두어 Feature가 참조하게 하고, 프로젝트 고유 데이터(ScriptableObject 등)는 `__App__/Data` 또는 씬의 `Data`에 두어 Composition만 참조한다. Composition이 데이터로 범용 타입을 만들어 각 Feature의 `Command`로 전달한다. 공유를 이유로 Feature끼리 참조하거나 공유용 Feature를 만들지 않는다. 목적: Feature 경계를 지키면서 공유 로직을 한 곳에 두고, 범용 부분만 나중에 패키지로 옮길 수 있게 한다.
- 한 Feature의 구성 요소를 다른 Feature에서도 써야 하면, 공유 범위에 맞는 공용 소유 위치를 제안하고 이동할지 먼저 사용자에게 묻는다. 예를 들어 여러 버튼이 공유하는 아이콘은 `Common/UI/Icons`, UI에 한정되지 않는 공용 Material, Texture, Sprite는 `Common/Visuals`를 제안한다. 사용자의 선택 없이 Feature 경계를 넘는 참조를 추가하거나 `Common`으로 이동하지 않는다.
