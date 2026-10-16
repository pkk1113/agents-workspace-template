# Agents Workspace 템플릿

> 이 Workspace의 프로젝트 소개와 목록은 [`WORKSPACE.md`](WORKSPACE.md)를 본다. 이 문서는 Workspace 구조와 템플릿 사용법을 설명한다.

Claude Code·Codex 같은 코딩 에이전트와 함께 일하기 위한 Workspace 구조다. 범용 작업 지침과 기술 스택별 규칙을 두고, 그 아래에 프로젝트를 추가해 쓴다.

템플릿 저장소: `https://github.com/pkk1113/agents-workspace-template.git`

## 구조

```text
<Workspace>/
  AGENTS.md            범용 작업 방식 (항상 로드)
  CLAUDE.md            Claude Code용 진입점 (AGENTS.md, WORKSPACE.md import)
  WORKSPACE.md         이 Workspace의 프로젝트 목록과 고유 정보
  README.md            이 문서
  Docs/
    index.md           문서 진입점
    rules/             작성 규칙과 양식 (템플릿과 동기화)
      index.md
      project-setup.md 프로젝트 추가 절차
      templates/       진입 지침, WORKSPACE.md, knowledge 양식
      <stack>/         스택별 규칙 (core.md + 작업별 문서 + templates/)
    knowledge/         이 Workspace의 사실
      index.md
    todo.md
  <Project>/           프로젝트 폴더 (AGENTS.md, CLAUDE.md 진입점 포함)
```

| 구분 | 파일 | 새 Workspace에서 |
|---|---|---|
| 범용 | `AGENTS.md`, `CLAUDE.md`, `README.md`, `Docs/index.md`, `Docs/rules/` | 그대로 사용 |
| 고유 | `WORKSPACE.md`, `Docs/knowledge/`, `Docs/todo.md`, 프로젝트 폴더 | 양식에서 새로 작성 |

## 새 Workspace 시작

```bash
git clone <템플릿 저장소> <Workspace 이름>
```

```bash
git -C <Workspace 이름> remote rename origin template
```

```bash
git -C <Workspace 이름> remote add origin <새 Workspace 저장소>
```

이력을 유지한 채 원격만 바꾸므로 나중에 템플릿 개선을 `git merge template/master`로 받을 수 있다. 이후 에이전트에게 "프로젝트를 추가해줘"라고 요청하면 `Docs/rules/project-setup.md`에 따라 진행한다.

### 에이전트에게 시키기

빈 폴더에서 코딩 에이전트를 열고 "이 템플릿으로 Workspace를 만들어줘 `<템플릿 저장소>`"라고 하면 에이전트가 아래를 한다. 사용자가 치는 명령은 없다.

1. 현재 폴더에 clone한다(`git clone <템플릿 저장소> .`). 폴더가 비어 있지 않으면 멈추고 사용자에게 확인한다.
2. `origin`을 `template`으로 바꾼다.
3. 새 Workspace 저장소 주소를 묻는다. 있으면 `origin`으로 추가하고, 아직 없으면 건너뛴다.
4. `WORKSPACE.md`의 제목과 설명을 묻고 채운다.
5. 커밋하고, 사용자가 "프로젝트 만들자"라고 하면 `Docs/rules/project-setup.md`로 넘어간다. 필요한 정보는 그때 되묻는다.

## 템플릿과 동기화

범용 파일 변경은 Workspace에서 별도 커밋으로 남긴다(`AGENTS.md`의 `🔀 Git`).

- **템플릿 개선 받기**: `git fetch template` 후 `git merge template/master`.
- **Workspace 개선 올리기**: 템플릿 저장소에서 해당 커밋을 cherry-pick하거나 변경 파일을 복사해 커밋한다. 범용 파일에 고유 정보가 섞이지 않았는지 확인한다.

## 템플릿 저장소 갱신

템플릿 저장소에는 범용 파일과, 고유 파일 자리에 들어갈 빈 양식만 둔다. Workspace에서 템플릿을 다시 만들 때는 범용 파일을 복사하고, 고유 파일은 `Docs/rules/templates/`의 양식으로 채운다.

| 템플릿 파일 | 출처 |
|---|---|
| `WORKSPACE.md` | `Docs/rules/templates/WORKSPACE.md` |
| `Docs/knowledge/index.md` | `Docs/rules/templates/knowledge-index.md` |
| `Docs/knowledge/known-exceptions.md` | `Docs/rules/templates/known-exceptions.md` |
| `Docs/todo.md` | `Docs/rules/templates/todo.md` |
