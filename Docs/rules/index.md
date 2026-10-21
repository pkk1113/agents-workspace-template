# rules 목록

다른 Workspace에도 그대로 가져가는 작성 규칙과 양식이다. 특정 프로젝트·제품의 이름이나 파일을 적지 않는다. 변경은 사용자와 합의한 경우에만 한다.

## 공통

- [agent-collaboration.md](agent-collaboration.md): 작업 규모에 따라 설계·개발·통합 역할을 나누고 계약 변경과 검증을 조율하는 선택적 협업 절차.
- [project-setup.md](project-setup.md): 프로젝트를 추가하거나 기존 프로젝트를 들여오는 절차.
- [templates/](templates/): 진입 지침, `WORKSPACE.md`, knowledge index, known-exceptions, todo 양식.

## Unity (`unity/`, 경로는 Unity 프로젝트 루트 기준)

- [core.md](unity/core.md): Unity 작업 시 항상 따르는 규칙. 클래스 구성, 명명, 참조 규칙, 에셋 변경 원칙, 스크립트 검증, 작업별 규칙 표.
- [collaboration.md](unity/collaboration.md): 협업 절차의 Unity 적용. Feature·Common·Composition 담당과 Editor·컴파일 검증 조율.
- [composition.md](unity/composition.md): Feature 접점(Property·Command·Event)과 Root·Sub-Composition·Adapter 구성.
- [asset-layout.md](unity/asset-layout.md): `__App__` 하위 Common·Infrastructure·Features·Scenes 폴더 구성.
- [ui.md](unity/ui.md): UI 이름(Layout·View·Menu·Form·Slate·Hud·Bar·Panel)과 UI 이벤트 명명.
- [global-data.md](unity/global-data.md): ConstData·RuntimeData·LocalData 경계와 예시.
- [scene-hierarchy.md](unity/scene-hierarchy.md): 씬 루트 배치 순서와 구분선.
- [asset-move.md](unity/asset-move.md): 에셋 이동·이름 변경 검증과 mcp-for-unity 주의사항.
- [editor-launch.md](unity/editor-launch.md): Unity CLI로 새 프로젝트를 만들고 mcp-for-unity를 무인 설치·기동하는 절차, 단계별 자동화 가능 여부, curl 직접 호출, 에디터 종료, EditorPrefs를 UI 없이 바꾸는 방법(레지스트리 키 해시, MSIX 가상화 우회).
- [templates/](unity/templates/): Unity 프로젝트용 `.gitignore`, `.gitattributes` 양식.
