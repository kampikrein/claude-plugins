# project-boundary-guard

Claude Code 플러그인 — 프로젝트 경계 보호 (외부 파일 수정 방지)

## 기능

- **Write/Edit 가드**: 프로젝트 외부 파일 수정 시 사용자 확인 요청
- **Bash 가드**: 외부 경로 대상 수정 명령어, 리다이렉트, 글로벌 설치 감지
- **읽기 전용 바이패스**: `cat`, `ls`, `grep`, `git log` 등은 외부 경로도 자유롭게 허용
- **소프트 가드레일**: Hook이 감지 못하는 간접 실행 패턴에 대한 AI 지침

## 설치

```bash
# 로컬 테스트
claude --plugin-dir /path/to/project-boundary-guard

# 마켓플레이스에서 설치
claude marketplace add https://github.com/kampikrein/claude-plugins/marketplace.json
claude plugin install project-boundary-guard --scope user
```

## 동작 원리

`--dangerously-skip-permissions` 모드에서도 **PreToolUse 훅은 항상 실행**됩니다.
이 플러그인은 이 특성을 활용하여 권한 우회 모드에서도 프로젝트 경계를 보호합니다.
