# 프로젝트 추가

Workspace에 새 프로젝트를 만들거나 기존 프로젝트를 들여올 때 읽는다. 양식은 `Docs/rules/templates/`와 스택별 `Docs/rules/<stack>/templates/`에 있다.

## 절차

1. **정보 확인**: 폴더 이름과 스택이 정해지지 않았으면 한 번의 질문으로 함께 묻는다. 스택 규칙이 생성에 필요한 정보(버전, 템플릿 등)를 더 정해 두었으면 같은 질문에 넣는다. 기본값이 있는 항목은 기본값을 제안하고 확인만 받는다.
2. **폴더**: 프로젝트는 Workspace 루트 바로 아래 폴더 하나로 둔다. 스택 규칙에 프로젝트 생성 절차가 있으면 그 절차로 에이전트가 직접 만든다. 없으면 빈 폴더를 만들고 시작한다.
3. **스택 규칙 확인**: `Docs/rules/<stack>/core.md`가 있는지 확인한다.
   - 있으면 그대로 연결한다.
   - 없으면 규칙 없이 시작한다. 새 스택 규칙이 필요하면 에이전트는 초안까지만 작성하고, 사용자와 합의한 뒤 `Docs/rules/<stack>/`에 확정한다. rules는 합의로만 바뀐다.
4. **진입 지침**: 양식으로 프로젝트 폴더에 진입 지침을 만든다.
   - `templates/project-AGENTS.md` → `<프로젝트>/AGENTS.md`
   - `templates/project-CLAUDE.md` → `<프로젝트>/CLAUDE.md`
   - 스택 규칙이 없으면 `core.md`를 가리키는 줄을 뺀다.
5. **git 설정**: 스택 양식이 있으면 프로젝트 폴더 안에 둔다. 예: Unity는 `unity/templates/unity.gitignore` → `<프로젝트>/.gitignore`, `unity.gitattributes` → `<프로젝트>/.gitattributes`. 프로젝트 전용 무시 항목을 루트 `.gitignore`에 넣지 않는다.
6. **등록**: `WORKSPACE.md`의 프로젝트 표에 폴더, 내용, 스택, 작업 전에 읽을 문서를 추가한다.
7. **커밋**: `[init] <프로젝트> 프로젝트 추가`로 커밋한다.

## 기존 프로젝트를 들여올 때

위 절차에 아래를 더한다. 각 단계는 사용자에게 확인받고 진행한다.

- **기존 git 이력**: 프로젝트에 `.git`이 있으면 처리 방법을 사용자에게 묻는다. 이력을 유지하려면 프로젝트 이력을 하위 폴더로 옮겨 Workspace 저장소에 합치고(예: `git subtree add` 또는 `git filter-repo --to-subdirectory-filter` 후 merge), 이력이 필요 없으면 `.git`을 제외하고 파일만 들여온다.
- **기존 지침**: 프로젝트에 지침 문서가 있으면 범용 내용은 루트 `AGENTS.md`와, 스택 규칙은 `Docs/rules/<stack>/`와 비교해 차이를 보고한다. 규칙 통합은 사용자와 합의한 뒤 한다. 프로젝트 고유 사실은 `Docs/knowledge/`로 옮긴다.
- **knowledge 초안**: 코드와 기존 문서를 분석해 코드만으로 알기 어려운 사실(외부 계약, 단위, 기기 동작 등)의 초안을 `Docs/knowledge/`에 작성하고 사용자에게 확인받는다.
- **규칙 예외**: 스택 규칙을 따르지 않는 기존 코드는 고치거나 `Docs/knowledge/known-exceptions.md`에 기록한다.

## 새 Workspace를 시작할 때

템플릿 저장소에서 새 Workspace를 만드는 방법은 `README.md`를 따른다. 시작한 뒤 `WORKSPACE.md`의 제목과 설명을 채우고, 프로젝트마다 위 절차를 따른다.
