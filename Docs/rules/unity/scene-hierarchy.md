# 씬 Hierarchy 구성

씬에 객체를 추가하거나 씬 Hierarchy 구조를 바꿀 때 읽는다. 씬 변경은 `core.md`의 `Unity 에셋 변경 원칙`에 따라 mcp-for-unity로 한다.

- 씬 Composition 객체는 씬 루트의 최상단에 둔다. Composition이 조립하고 씬 전체에서 함께 사용하는 제어 객체는 Composition의 자식으로 둔다. 예: `SceneComposition/UiNavigation`.
- Composition 그룹 아래에는 XR·카메라 등 플랫폼 구성, 화면·콘텐츠 순으로 배치한다. 개발 플랫폼과 SDK가 요구하는 부모·자식 관계 및 Prefab 구조는 따른다. 이러한 필수 구조는 유지하되, 씬 루트에서 Composition 그룹을 최상단에 두는 정리 기준은 유지한다.
- UI Feature는 Canvas 등 자신이 동작하는 데 필요한 계층에 둔다. 모든 Feature를 Composition의 자식으로 옮기는 것은 아니다. Hierarchy의 부모·자식 관계와 Feature 간 코드 책임 경계는 구분한다.
- 역할 그룹 사이에는 `=================` 이름의 빈 GameObject를 구분선으로 배치한다. 구분선에는 Transform 외 기능 컴포넌트나 자식 객체를 두지 않는다.
- Hierarchy 순서는 가독성과 객체 그룹화를 위한 기준이다. 초기화나 실행 순서는 Hierarchy 순서에 의존하지 않는다.

예시:

```text
SceneComposition
    ├─ UiNavigation
    └─ SessionComposition
=================
XR Plugin
=================
Canvas
    ├─ DefaultLayout
    ├─ MenuLayout
    └─ OperationLayout
```

위 예시의 XR·카메라와 Canvas의 실제 계층은 개발 플랫폼 및 SDK의 요구사항에 맞춘다.
