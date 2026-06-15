# building-premium-homepages — Claude Code Skill

재사용 가능한 프리미엄 멀티링구얼 홈페이지 빌드 방법론 스킬.

레퍼런스 기반 디자인, codex `$imagegen` 에셋 파이프라인(단일 머티리얼 스펙), 시그니처 인터랙션(X-ray 스캐너·단어↔이미지 동기·비디오월), 다국어/콘텐츠 레이어, Supabase 리드 수집, Gemini 챗봇, Vercel 배포까지의 워크플로우와 안티패턴을 담았습니다.

## 설치 (Claude Code)
```bash
git clone <this-repo> 
cp -r building-premium-homepages ~/.claude/skills/
```
또는 `~/.claude/skills/` 아래에 `building-premium-homepages/SKILL.md`로 배치.

스킬은 "프리미엄/멀티링구얼 홈페이지를 새로 만들 때" 자동 트리거됩니다.
