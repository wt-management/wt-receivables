# 국내 미수금 관리 사이트 (wt-receivables)

담당자가 Claude Code로 직접 수정·배포하는 repo입니다. main에 push하면 1~2분 내 라이브에 반영됩니다.
라이브: https://wt-management.github.io/wt-receivables/

## 구조
- **index.html 단일 파일** 사이트입니다. HTML/CSS/JS가 전부 이 파일 안에 있습니다.
- 화면: 대시보드 / 채권 목록 / 관리대상 / 손실처리 / 주간 추이 / 임원 보고 / 데이터 업로드 (해시 라우팅 `#home` 등)

## 데이터 (절대 HTML에 넣지 말 것)
- 채권 데이터는 Supabase `cons_cache` 테이블: key=`ar_kr`(원본), key=`ar_kr_edits`(웹 수정 오버레이)
- **민감 재무데이터(거래처·금액)를 index.html에 직접 임베드 금지** — 이 repo는 public이라 그대로 노출됩니다.
- **Supabase service key(sb_secret_...)를 커밋 금지.** 코드에는 publishable key만 사용.
- 주간 데이터 갱신은 코드 수정이 아니라 사이트의 "데이터 업로드" 탭에서 엑셀 업로드로 합니다.

## 수정 규칙
1. 수정 전 `git pull` 먼저 (다른 사람 작업분 반영)
2. 수정 후 **인라인 JS 문법검증 필수** — 문법오류 1건이면 사이트 전체가 죽습니다:
   ```bash
   node -e "const fs=require('fs');const h=fs.readFileSync('index.html','utf8');const re=/<script(?![^>]*src)[^>]*>([\s\S]*?)<\/script>/g;let m,all='';while((m=re.exec(h))){all+=m[1]+'\n;\n';}fs.writeFileSync('_check.js',all);" && node --check _check.js && del _check.js
   ```
3. `git add index.html && git commit -m "..." && git pull --rebase && git push`
4. 배포 후 사이트 열어서 강력 새로고침(Ctrl+Shift+R)으로 확인
5. 잘못 올렸으면 `git revert HEAD && git push` 로 즉시 되돌리기

## 건드리면 안 되는 것
- `arAllowed()` 접근 게이트 — 미수금은 민감 재무데이터라 master + access 'ar' 보유자만 허용. 임의 완화 금지(권한 추가는 관리자 페이지에서).
- `WT_ERRLOG_V1` 스니펫(오류수집)과 `WT_SWITCHER` 블록(사이트 이동) — 전 사이트 공용 모듈이므로 개별 수정 금지.
- `rid()` 해시 함수 — 파이썬 시드 스크립트와 동일해야 웹 수정분이 유지됩니다. 변경 금지.
