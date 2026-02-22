# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 저장소 개요

Claude Code 플러그인 저장소로, **프로젝트 경계 보호** 기능을 제공합니다 — 현재 프로젝트 디렉토리 외부의 파일 수정을 방지합니다. PreToolUse 훅을 통한 하드 가드레일과 AI 스킬 문서를 통한 소프트 가드레일, 두 가지 계층으로 보호합니다.

사용자 문서의 기본 언어는 한국어입니다. 셸 스크립트 주석은 영어를 사용합니다.

## 아키텍처

```
marketplace.json                    # 플러그인 배포용 레지스트리
project-boundary-guard/
├── .claude-plugin/plugin.json      # 플러그인 메타데이터 (이름, 버전)
├── hooks/hooks.json                # PreToolUse 훅 설정
├── scripts/
│   ├── guard-file-boundary.sh      # Write/Edit 도구 가드 — 외부 파일 경로 차단
│   └── guard-bash-boundary.sh      # Bash 도구 가드 — 수정 명령어, 리다이렉트, 글로벌 설치 차단
└── skills/
    ├── brainstorming/SKILL.md      # 설계 우선 개발 프로세스
    ├── frontend-design/SKILL.md    # UI/UX 디자인 가이드라인
    ├── handoff/SKILL.md            # 에이전트 간 인수인계 문서화
    └── project-boundary/SKILL.md   # 훅이 감지 못하는 패턴에 대한 소프트 가드레일
```

**훅 시스템:** `hooks.json`에 두 개의 PreToolUse 훅이 등록되어 있습니다. Claude가 Edit/Write를 호출하면 `guard-file-boundary.sh`가 대상 파일이 `$CLAUDE_PROJECT_DIR` 내부에 있는지 확인합니다. Bash를 호출하면 `guard-bash-boundary.sh`가 수정 명령어(`rm`, `mv`, `cp` 등), 출력 리다이렉트(`>`, `>>`, `tee`), 글로벌 설치(`npm -g`, venv 없는 `pip`)를 파싱합니다. 두 스크립트 모두 stdin으로 JSON을 읽고, `realpath`로 경로를 정규화한 후, JSON 권한 결정을 출력하거나 허용 시 조용히 종료합니다.

**핵심 설계:** `--dangerously-skip-permissions` 모드에서도 PreToolUse 훅은 항상 실행되므로, 권한 모드와 무관하게 경계 보호가 동작합니다.

**소프트 가드레일 계층:** `project-boundary/SKILL.md`는 훅이 감지할 수 없는 간접 실행 패턴을 문서화합니다 (예: `bash -c "rm /path"`, `xargs rm`, `find -exec`, `eval`, 변수 간접 참조, Python/Node 파일 조작). 이러한 패턴은 스크립트가 아닌 AI의 인지에 의존합니다.

## 빌드 / 테스트 / 린트

빌드 시스템, 테스트 스위트, 린터가 없습니다. 정적 셸 스크립트, JSON 설정, 마크다운 문서로만 구성되어 있으며 컴파일이나 의존성 설치가 필요하지 않습니다.

## 개발 참고사항

- 훅 스크립트는 외부 의존성(`jq`) 없이 인라인 Node.js(`node -e`)로 JSON을 파싱합니다.
- `guard-bash-boundary.sh`는 화이트리스트 방식: 읽기 전용 명령어(`cat`, `ls`, `grep`, `git log` 등)만 외부 경로에서 명시적으로 허용하고, 나머지는 모두 검사합니다.
- 두 훅 스크립트 모두 `hooks.json`에서 10초 타임아웃이 설정되어 있습니다.
- 경로 검사는 `realpath` 정규화 후 `$CLAUDE_PROJECT_DIR`(기본값 `$PWD`) 대비 접두사 매칭을 사용합니다.
