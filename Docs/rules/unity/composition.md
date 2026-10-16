# Feature Composition

Feature의 외부 접점이나 Composition(Root, Sub-Composition, Adapter)을 만들거나 수정할 때 읽는다. 경로는 Unity 프로젝트 루트를 기준으로 한다.

## Feature Composition

각 Feature는 Composition이 연결에 사용할 다음 세 종류의 접점을 제공한다. 참조할 수 있는 범위는 `core.md`의 `참조 규칙`을 따른다.

- `Property`: Feature가 현재 보유한 상태나 값을 Composition에 제공한다.
- `Command`: Feature가 외부에 공개하는 public 메서드로, Composition이 동작이나 상태 변경을 요청할 때 사용한다. `Methods` 영역에 배치한다.
- `Event`: Feature에서 발생한 상태 변화나 동작을 Composition에 알린다.

Composition은 한 Feature의 `Event`를 구독하고, 필요한 값을 다른 Feature의 `Property`에서 읽어 해당 Feature의 `Command`를 호출하는 방식으로 Feature들을 연결한다. UI 이벤트도 같은 방식으로 Composition이 기능에 연결하며, UI 클래스는 기능 동작을 직접 호출하지 않는다. 목적: Feature를 서로 모르는 채로 변경·재사용·테스트할 수 있게 하고, 기능 사이의 연결을 Composition 한 곳에서 파악할 수 있게 한다.

이 규칙은 Feature의 **외부 연결 방식과 책임 경계**를 정의한다. `Property`, `Command`, `Event`는 클래스 분류가 아닌 개념적 접점이다.

Feature는 기본적으로 하나의 클래스에서 상태 프로퍼티, 명령 메서드, 변경 이벤트를 함께 제공한다. 서로 다른 수명 주기, 독립적인 재사용, 또는 분리된 소유 책임이 명확할 때에만 별도 클래스로 분리한다. 세 접점이라는 이유만으로 `Property`, `Command`, `Event` 클래스를 각각 만들지 않는다.

## Composition 구성

씬은 하나의 Root Composition이 조립하며, 연결 책임이 커지면 Composition 보조 클래스로 나눈다. 보조 클래스는 Composition의 일부로 취급한다. Feature와 Infrastructure를 직접 참조할 수 있고, interface는 구현 교체가 필요할 때만 도입한다. 단, Feature의 외부 접점(`Property`, `Command`, `Event`)만 사용하고 Feature 내부 구현에 의존하지 않는다.

보조 클래스는 역할에 따라 다음 두 형태 중 하나로 만든다.

- `Adapter`: 외부 데이터와 Feature 데이터를 변환한다. 변환에 필요한 참조 외의 자체 상태를 갖지 않는 일반 클래스이며, Composition이 호출한다. 예: 서버 DTO를 Feature 상태로 반영하는 `OperationDtoAdapter`.
- `Sub-Composition`: Root Composition의 연결 일부를 독립적으로 맡는다. 맡은 Feature의 `Event`를 직접 구독하고, `Property`를 읽어 `Command`를 호출한다. Root는 연결 동작을 위해 Sub-Composition의 메서드를 호출하지 않는다. 이름은 `<책임>Composition`으로 짓는다.

씬 전용 클래스(Root Composition, Sub-Composition, Adapter)는 씬 이름을 접두사로 붙인다. 예: `OperationSessionComposition`, `OperationDtoAdapter`.

Sub-Composition은 다음을 따른다.

- MonoBehaviour로 만들고 Root Composition의 자식으로 둔다.
- 씬 객체(Feature, UI)는 자신이 `[SerializeField]`로 받는다. 여러 Sub-Composition이 같은 씬 객체를 참조해도 된다.
- 일반 객체는 생성한 쪽이 소유하고 해제한다. 여러 Composition이 공유하면 Root가 생성해 초기화 메서드로 전달하고, 한 Sub-Composition만 쓰면 그 Sub-Composition이 생성한다.
- Root가 `Awake`에서 각 Sub-Composition의 초기화 메서드를 정해진 순서로 호출한다. Sub-Composition은 자신의 `Awake`·`Start`에서 연결하지 않으며, 구독 해제는 `OnDestroy`에서 한다.
- Sub-Composition끼리 참조하지 않으며, Sub-Composition은 `Event`를 두지 않는다. 연결 묶음 사이는 각 묶음이 참조하는 Feature의 외부 접점으로 이어진다.
- 그래도 이어지지 않으면 나눈 경계를 다시 정한다. 정해진 순서로 함께 움직여야 하는 연결은 하나로 합치고, 여러 묶음이 쓰는 일반 객체는 Root가 소유한다. 연결을 나누기 위해 Feature 접점을 추가하거나 바꾸지 않는다.
- Composition에 남아 있는 상태가 어느 Feature의 책임인지는 분리와 별개로 판단한다.
- public 멤버는 초기화 메서드로 한정한다.

나누는 기준은 다음과 같다.

- 다루는 Feature와 수명이 분명한 연결 묶음을 Sub-Composition으로 나눈다. 예: 로그인부터 로그아웃까지의 세션 연결, 작전 상태를 HUD에 반영하는 연결.
- 여러 Sub-Composition의 순서와 수명을 조율하는 코드는 Root에 둔다.
- 줄 수만으로 나누지 않는다. Editor 전용 디버그 코드처럼 Composition 내부를 두루 건드리는 코드는 `<Composition>.EditorDebug.cs` 같은 partial 파일로 분리하고 파일 전체를 `#if UNITY_EDITOR`로 감싼다.

목적: 연결을 여러 파일로 나누더라도 각 연결 묶음이 무엇에 의존하고 누가 수명을 책임지는지 한 곳에서 보이게 하고, Root가 모든 참조를 전달하는 통로가 되지 않게 한다.

## Feature 외부 접점의 명명과 주석

`Property`, `Event`, `Command`는 이름과 타입만으로 역할과 사용 의도를 이해할 수 있도록 작성한다. 주석으로 모호한 이름을 보완하기 전에 명명과 접점 설계를 개선한다.

이름과 타입만으로 드러나지 않는 사용 계약이 있으면 XML 문서 주석으로 명시한다. 예: 단위·좌표계, 유효 시점, null 허용, 이벤트 발생 순서, 반복 호출 동작, 비동기 완료의 의미.

이름을 반복하는 주석이나 내부 구현을 나열하는 주석은 작성하지 않는다. 접점이나 관련 구현을 수정할 때는 기존 주석이 수정된 코드와 일치하는지 함께 확인하고, 설명이나 계약이 달라지면 같은 작업에서 주석도 갱신한다.
