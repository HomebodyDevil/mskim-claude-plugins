# mskim-claude-plugins

mskim 개인 Claude Code 플러그인 marketplace입니다. 플러그인 `mskim` 하나에 여러 스킬을 담고, 각 스킬은 `/mskim:<스킬 이름>`으로 호출합니다.

## 스킬 목록

| 스킬 | 사용법 | 설명 |
|---|---|---|
| `wanna` | `/mskim:wanna [작업 지침서를 만들기 위한 내용]` | 애매한 점을 먼저 질문한 뒤 작업 지침서(Markdown)를 저장하고, 함께 넘길 작업 지시문을 응답 끝에 준다 |
| `why-codebase` | `/mskim:why-codebase [현상/문제] [doc]` | 현상이 왜 일어나는지 표면×경로로 추적해 근본 원인을 분석한다 (코드·환경 포함, 수정은 제안만) |
| `advice` | `/mskim:advice [현상/문제/분석 문서 경로] [doc]` | 원인 분석(why)에 이어, 코드·환경(OS, 런타임, 의존성, 설정) 양쪽 개선안을 비교하고 권장안·검증·재발 방지를 제시한다 (적용은 하지 않음) |
| `resolve` | `/mskim:resolve [extend \| refactor] [지침서 경로/작업 내용]` | 지침서·why·advice·review 보고서나 작업 내용을 바탕으로 실제 작업을 수행한다. `extend`는 기존 구조를 유지하고 새 단위를 추가하며, `refactor`는 필요하면 구조까지 재설계한다 |
| `ai-browser` | `/mskim:ai-browser [수행할 브라우저 작업]` | 사용자 입력과 분리된 ego lite TaskSpace(에이전트 전용 탭·커서)에서 브라우저 작업을 수행한다. 로그인·2FA·CAPTCHA·비밀번호/결제 입력처럼 에이전트가 할 수 없는 단계에서는 `handOff()`로 제어를 넘기고 같은 턴에서 사용자에게 묻고 기다린 뒤, 제어를 되찾아 차단이 풀렸는지 확인하고 이어서 진행한다. 같은 사이트 요청 사이에는 서버 부하를 줄이려고 고정 간격을 두고, `403`·`429`·비정상 트래픽 화면이 나오면 재시도 없이 바로 멈춰 사용자에게 다음 방법을 묻는다. 명시적으로 호출할 때만 실행된다 |
| `work-git` | `/mskim:work-git [commit \| push \| pr \| merge] [변경 내용·PR 번호·추가 지시]` | 회사 Git 규칙(`references/git-rules.md`)에 맞춰 커밋·푸시·PR을 진행한다: `dev`에서 딴 `feature/<이름>` 브랜치, `<type>(<scope>): <subject>` 커밋(제목 50자 이하·마침표 없음·명령형, 본문 줄당 75자 이하), 변경 하나당 커밋 하나, `feature/* → dev → main`, PR 본문 3개 섹션과 리뷰어 1명 이상. `main`·`dev` 직접 push와 자기 PR merge는 하지 않는다. 명시적으로 호출할 때만 실행된다 |
| `total-review` | `/mskim:total-review [diff 파일 \| 브랜치 \| PR \| A..B] [PR 설명·이슈·문서]` | 명시적으로 호출할 때만 실행되는 통합 리뷰다. 트리아지로 필요한 분야만 고르고, 공통 컨텍스트 패키지를 한 번 만들어 5개 분야 리뷰 스킬을 subagent로 병렬 실행한 뒤 중복 병합·severity 재조정·critical/high 검증을 거쳐 머지 가능 / 수정 후 머지 / 머지 차단을 판정한다. 소규모 변경(50줄 미만·파일 2개 이하, SKILL.md 상단에서 조정)은 subagent 없이 직접 검토한다 |
| `bug-review` | `/mskim:bug-review [diff 파일 \| 브랜치 \| PR \| A..B]` | 변경에서 기능적 결함(분기·경계값, null, 예외 처리, 데이터 손실, 동시성)만 찾아 severity·confidence·file:line·재현 시나리오로 보고한다. 명시적으로 호출할 때만 실행된다. 보안·테스트·컨벤션·배포 영향은 다루지 않는다 (total-review가 내부에서 쓰는 분야 스킬 5개 중 하나) |
| `test-review` | `/mskim:test-review [diff 파일 \| 브랜치 \| PR \| A..B]` | 변경이 테스트로 보호되는지 본다: 핵심 경로·실패 경로 테스트, 버그 수정의 회귀 테스트, 실제로 검증하는 테스트인지(assertion 없음, 과도한 mock, 구현 결합). 지적마다 "어떤 동작이 깨져도 현재 테스트가 통과하는가"를 적는다. 명시적으로 호출할 때만 실행되고, 제품 코드 버그는 다루지 않는다 (total-review가 내부에서 쓰는 분야 스킬 5개 중 하나) |
| `security-review` | `/mskim:security-review [diff 파일 \| 브랜치 \| PR \| A..B]` | 변경에서 보안 취약점(인증·인가, 인젝션·XSS·경로 조작·역직렬화, 비밀정보·개인정보 노출, SSRF·안전하지 않은 기본값·새 의존성·새 엔드포인트)만 찾아 severity·confidence·file:line·공격 시나리오로 보고한다. 명시적으로 호출할 때만 실행된다. 정상 사용 중 오작동과 테스트 부족은 다루지 않는다 (total-review가 내부에서 쓰는 분야 스킬 5개 중 하나) |
| `context-review` | `/mskim:context-review [diff 파일 \| 브랜치 \| PR \| A..B] [PR 설명·이슈·문서]` | 변경이 diff 바깥과 맞는지 본다: PR 설명·이슈와의 일치(빠진 요구사항, 요청 밖 동작 변경), 변경 범위(무관한 변경, 복제 로직 중 일부만 수정), 기존 유틸·관례와의 일관성, 레이어 경계, 도메인 규칙·ADR, 주석·이름. diff 밖 코드와 문서를 직접 탐색하고, 의도를 모르면 그 사실을 먼저 적는다. 명시적으로 호출할 때만 실행되고, 기능 결함과 외부 계약·운영 영향은 다루지 않는다 (total-review가 내부에서 쓰는 분야 스킬 5개 중 하나) |
| `change-impact-review` | `/mskim:change-impact-review [diff 파일 \| 브랜치 \| PR \| A..B]` | 변경이 코드 바깥에서 깨뜨리는 것만 본다: API 하위 호환(필드 삭제·이름·타입, 응답 형식, 에러 코드), DB 마이그레이션 안전성(대형 테이블 락, 기본값 없는 NOT NULL, 롤백, 배포 순서), 새 환경변수·설정 키와 누락 시 동작, 배포 순서·피처 플래그·롤백. 지적마다 "어떤 소비자·환경에서 무엇이 깨지는가"를 적는다. 명시적으로 호출할 때만 실행되고, 로직 결함과 내부 정합성은 다루지 않는다 (total-review가 내부에서 쓰는 분야 스킬 5개 중 하나) |
| `research` | `/mskim:research [질문/요청/의도] [doc]` | 공식 문서·논문·전문 사이트·커뮤니티를 웹에서 조사해 출처와 함께 답한다. 지식·알고리즘은 근거 있는 설명으로, 게임 시스템·콘텐츠는 실제 게임 사례와 적용·개선안으로, 방향·컨셉은 게임·소설·영화·논문(분야 무관)의 요소와 적용 방안으로 답한다. 명시적으로 호출할 때만 실행된다 |

## 배포하지 않는 스킬 (`unpublished-skills/`)

아래 스킬은 저장소에만 두고 플러그인에는 넣지 않는다. 플러그인은 `mskim/` 폴더만 배포하므로 설치한 PC에서는 `/mskim:<이름>`으로 호출할 수 없다. 다시 배포하려면 폴더를 `mskim/skills/`로 옮기고 `version`을 올린다.

| 스킬 | 사용법 | 설명 |
|---|---|---|
| `pr-review` | `/mskim:pr-review <PR 번호·URL 또는 base...head> [previous=review.json] [format=json] [limit=n]` | merge-base 기준 PR 정적 리뷰. 이전 지적을 다시 확인하고 전체 지적의 심각도로 판정한다. 기본 출력은 한국어 Markdown이다. |
| `review` | `/mskim:review [대상] [reviewers=...] [vendors=codex,gemini\|none] [doc]` | 변경을 runtime·correctness·security(+필요 시 concurrency·compatibility·design·flow) 관점으로 Claude subagent와 타 벤더 모델(Codex, Gemini 등)이 병렬 리뷰하고, validator가 모든 지적을 누락 없이 검증·통합해 보고한다 (수정은 하지 않음) |
| `review-loop` | `/mskim:review-loop [대상] [extend \| refactor] [n=3] [major=0] [minor=2] [count-p3] [pre-existing | change-only] [fast] [vendors=...] [reviewers=...]` | review → resolve → re-review를 반복해, 남은 major(P0+P1)·minor(P2) 지적이 기준 이하가 되면 멈춘다. 최대 n라운드이고, 개선이 멈추거나 악화되거나 사용자 판단이 필요하면 일찍 멈춘다. 라운드마다 snapshot과 보고서를 남긴다 |
| `original-review-loop` | `/mskim:original-review-loop [대상] [extend \| refactor] [n=3] [important=0] [nits=<k>] [pre-existing] [level=medium]` | 내장 `code-review` → resolve → `code-review`를 반복해, 남은 `important` 지적(옵션으로 nit 포함)이 기준 이하가 되면 멈춘다. `review-loop`보다 라운드당 비용이 훨씬 적은 대신 리뷰어가 하나뿐이다. 멈춤 규칙·snapshot·보고서는 `review-loop`와 같다 |

## review: 벤더·모델 설정

`review`는 기본으로 Claude와 `codex`, `gemini` CLI를 함께 사용한다. 각 CLI는 미리 설치하고 로그인해 둔다. 설치되지 않았거나 실패한 벤더는 건너뛰지 않고 보고서 맨 위에 실패로 표시한다.

모델이나 벤더를 바꾸려면 덮어쓸 키만 담은 JSON을 만든다. 프로젝트별로는 `.claude/mskim-review.json`, 전역으로는 `~/.claude/mskim-review.json`에 둔다. 기본값은 `unpublished-skills/review/references/default-config.json`에 있다.

```json
{
  "vendors": {
    "codex":  { "model": "gpt-6.2-sol", "effort": "high", "split": true },
    "gemini": { "perspectives": ["security", "correctness"] },
    "qwen":   { "enabled": true, "command": "qwen -p \"Follow the review instructions given on stdin.\" < {prompt} > {out}", "timeout_sec": 1200 }
  }
}
```

`split: true`를 주면 그 벤더를 관점마다 따로 실행한다. 리뷰는 깊어지지만 실행 횟수와 비용이 관점 수만큼 늘어난다. 저장소에 `.codex/`나 `.gemini/`가 있으면 실행 전에 물어본다. 이 폴더들은 MCP 서버나 hook을 실행할 수 있기 때문이다. Gemini는 기본적으로 빈 임시 폴더에서 실행하고, 저장소는 `--include-directories`로 읽게 해서 저장소 설정을 불러오지 않는다.

한 번만 바꿀 때는 `vendors=gemini`, `vendors=none`처럼 인자로 지정한다. 새 벤더는 `command`와 `timeout_sec`만 있으면 추가된다. 명령은 headless로 실행되어야 하고, `{prompt}` 파일을 읽어 `{out}`에 결과를 써야 하며, 읽기 전용이어야 한다.

## 설치 (새 PC / 새 환경)

Claude Code가 설치되어 있으면 터미널에서:

```shell
claude plugin marketplace add HomebodyDevil/mskim-claude-plugins
claude plugin install mskim@mskim-claude-plugins --scope user
```

`--scope user`는 그 PC의 모든 프로젝트에서 쓰도록 전역 설치한다. Claude Code 안에서는 `/plugin marketplace add HomebodyDevil/mskim-claude-plugins` 후 `/plugin install mskim@mskim-claude-plugins`로도 된다.

## 업데이트

```shell
claude plugin marketplace update mskim-claude-plugins
claude plugin update mskim@mskim-claude-plugins
```

업데이트 후 새 세션을 열거나 `/reload-skills`를 실행한다.

## 스킬 추가하기

1. `mskim/skills/<스킬 이름>/SKILL.md`를 만든다 (frontmatter에 `name`, `description`, 필요하면 `argument-hint`).
2. `mskim/.claude-plugin/plugin.json`의 `version`을 올린다. 버전이 같으면 설치된 PC가 변경을 받지 않는다.
3. commit 후 push하고, 각 PC에서 위 업데이트 명령을 실행한다.

폴더 이름이 곧 호출 이름이 된다: `mskim/skills/resolve/` → `/mskim:resolve`.

## 구조

```
.claude-plugin/marketplace.json   marketplace 정의
mskim/.claude-plugin/plugin.json  플러그인 정의 (version)
mskim/skills/<name>/SKILL.md      스킬 본문
mskim/skills/<name>/evals/        스킬 테스트 케이스 (claude plugin eval 형식)
mskim/agents/<name>.md            스킬 전용 subagent (`mskim:<name>`)
```
