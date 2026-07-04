# 배포 체크리스트 (GitHub Pages) — git 초심자용

작성: 백엔드·데이터 엔지니어 · 2026-07-04 · 대상: 오너(권유철)

이 문서 하나만 위에서 아래로 따라 하면 사이트가 공개됩니다. 명령어는 **Windows PowerShell**에 그대로 복사해 붙여넣으면 됩니다. 프로젝트 폴더 경로는 `C:\Users\82108\Claude\Projects\넷플릭스 데이터 기반 플젝` 입니다.

## 무엇이 공개되나 (중요)
GitHub Pages에는 **`data/` 폴더 + 프론트엔드 파일(index.html 등)만** 올라갑니다.
`backend/`, `analysis/`, `agents/`, `.env`(TMDB 키)는 `.gitignore`로 **자동 제외**됩니다. → 키가 공개될 경로가 없습니다.

---

## 사전 준비 (한 번만)
1. **GitHub 계정** — 없으면 https://github.com 에서 가입.
2. **Git 설치** — https://git-scm.com/download/win 에서 받아 설치(전부 기본값 Next). 설치 후 새 PowerShell 창에서 확인:
   ```powershell
   git --version
   ```
   버전이 나오면 OK. (안 나오면 PowerShell을 껐다 다시 여세요.)
3. **TMDB 키가 `backend\.env`에 있는지 확인** — 없으면 데이터 생성이 안 됩니다.

---

## 단계 1 — 데이터 최신화 + 검증 (게이트 ①②)
> ⚠️ **pending 상태로 배포 금지.** 아래 두 명령을 반드시 먼저 통과시키세요.

프로젝트 폴더로 이동:
```powershell
cd "C:\Users\82108\Claude\Projects\넷플릭스 데이터 기반 플젝"
```
① 데이터 전량 생성(리뷰 insight 포함):
```powershell
python backend\build_data.py
```
`=== 완료 ===` 가 뜨고 `data\` 에 5개 파일(genres/trending_by_genre/opportunity/charts/meta)이 생기면 성공.

② 배포 전 검증:
```powershell
python backend\verify_deploy.py
```
마지막 줄에 **`=== 검증 통과 → 배포 진행 가능 ===`** 이 나와야 다음 단계로 갑니다.
`검증 실패`가 나오면 그 위 `FAIL` 항목을 해결하고 ①부터 다시 하세요. (예: pending 있으면 ① 재실행)

---

## 단계 2 — GitHub에서 저장소(repo) 만들기
1. https://github.com/new 접속.
2. **Repository name**: 예) `netflix-trend-dashboard` (영문·하이픈).
3. **Public** 선택. (Pages 무료 공개에 필요)
4. "Add a README" 등 **체크박스는 모두 비워두기**.
5. **Create repository** 클릭.
6. 다음 화면에 나오는 주소를 복사해 둡니다: `https://github.com/<내아이디>/netflix-trend-dashboard.git`

---

## 단계 3 — 로컬 파일을 GitHub에 올리기(push)
> 아래 명령을 **한 줄씩 순서대로** 실행하세요. `<내아이디>` 부분만 본인 것으로 바꾸면 됩니다.

```powershell
cd "C:\Users\82108\Claude\Projects\넷플릭스 데이터 기반 플젝"
git init
git add .
```
**여기서 확인(중요):** 무엇이 올라갈지 점검합니다.
```powershell
git status
```
목록에 **`data/`와 프론트엔드 파일(index.html 등)만** 보여야 합니다.
혹시 `backend/`, `analysis/`, `.env`, `*.csv`가 보이면 **멈추고** 알려주세요(=.gitignore가 안 먹은 것). 정상이면 계속:
```powershell
git commit -m "MVP 대시보드 배포"
git branch -M main
git remote add origin https://github.com/<내아이디>/netflix-trend-dashboard.git
git push -u origin main
```
> 처음 push하면 GitHub 로그인 창이 뜹니다. 브라우저로 로그인(Authorize)하면 됩니다.

---

## 단계 4 — GitHub Pages 켜기
1. 방금 만든 저장소 페이지 → 상단 **Settings**.
2. 왼쪽 메뉴 **Pages**.
3. **Build and deployment → Source**: `Deploy from a branch`.
4. **Branch**: `main` / 폴더 `/ (root)` 선택 → **Save**.
5. 1~2분 뒤 페이지 상단에 사이트 주소가 뜹니다: `https://<내아이디>.github.io/netflix-trend-dashboard/`

---

## 단계 5 — 라이브 확인
- 위 주소를 열어 대시보드가 뜨는지 확인.
- 화면이 비었으면: 브라우저에서 `주소/data/meta.json` 을 직접 열어 JSON이 보이는지 확인(데이터 경로 점검용). 안 보이면 알려주세요.

---

## 데이터 갱신할 때(다음부터 반복)
카탈로그·인기·리뷰를 새로고침해 재배포하는 절차:
```powershell
cd "C:\Users\82108\Claude\Projects\넷플릭스 데이터 기반 플젝"
python backend\build_data.py
python backend\verify_deploy.py      # '검증 통과' 확인
git add data
git commit -m "데이터 갱신"
git push
```
1~2분 뒤 사이트에 반영됩니다.

---

## 🔐 안전 수칙 (꼭)
- **`.env`(TMDB 키)는 절대 커밋/푸시 금지.** `.gitignore`가 막지만, `git status`에 `.env`가 보이면 멈추세요.
- **공개 배포 직전 TMDB 키를 재발급**하세요(이번 세션에서 키가 한 번 외부 호출에 쓰였습니다). themoviedb.org → Settings → API → 키 재생성 후 `backend\.env`만 새 키로 교체(재배포 불필요, 데이터엔 키가 없음).
- 저장소를 **Public**으로 두되, 올라간 파일에 개인정보·키가 없는지 최초 1회 눈으로 확인.
- `backend/`·`analysis/` 코드도 공개하고 싶다면 **별도 저장소**를 쓰세요(단 `.env`는 어디에도 올리지 말 것).
