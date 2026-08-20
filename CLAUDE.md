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

## 배포 전 검사 — 반드시 실행

이 저장소를 고쳤으면 **push 하기 전에** 아래를 돌린다. 셋 다 통과해야 한다.

```bash
node .github/check-syntax.js && node .github/check-guards.js && node .github/check-bulk.js
```

| 검사 | 무엇을 잡나 |
|---|---|
| `check-syntax.js` | 화면이 통째로 안 뜨는 문법 오류 (기준선보다 늘면 실패) |
| `check-guards.js` | 한 번 고쳐둔 로직이 사라졌는지 (`.github/guards.json` 목록) |
| `check-bulk.js` | 한 파일에서 1,500줄 이상 또는 35% 넘게 삭제 — 옛 파일 위에 덮어쓴 사고 |

**버그를 고쳤으면 `.github/guards.json` 에 한 줄 추가한다.** 이게 이 저장소의 핵심 규칙이다.
같은 버그가 다른 작업에 밀려 조용히 되돌아가는 일이 실제로 여러 번 있었고(해외 조회기간은
고친 뒤 3개 커밋 만에 사라졌다), 화면은 멀쩡히 뜨고 숫자만 틀리기 때문에 눈으로는 못 잡는다.

```json
{ "id": "짧은이름", "why": "빠지면 무슨 일이 생기는지", "must": "파일에 남아 있어야 할 코드 한 조각" }
```

`check-bulk.js` 가 막아설 때는 대개 **최신 상태에서 작업하지 않은 것**이다. `git pull --rebase`
후 다시 확인하고, 정말 의도한 대규모 정리라면 커밋 **제목**에 `[대량변경]` 을 넣는다.
