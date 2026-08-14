# 구내식당 메뉴 알리미 - Claude Code 루틴 설정 가이드

## 1. 이 폴더를 git 저장소로 만들기

로컬(또는 GitHub)에 새 저장소를 만들고 아래 파일들을 커밋하세요:
- `send_kakao.sh`
- `ROUTINE_PROMPT.md` (참고용, 실제 루틴 프롬프트는 여기서 복사해서 씀)

```bash
git init
git add send_kakao.sh
chmod +x send_kakao.sh
git commit -m "카카오 메뉴 알리미 초기 설정"
git remote add origin <본인의 GitHub 저장소 URL>
git push -u origin main
```

`menu.json`은 루틴이 매주 자동으로 만들고 덮어쓰므로 미리 만들 필요 없습니다.

## 2. Google Drive에 폴더 만들기

Google Drive에 **"구내식당메뉴"** 라는 폴더를 하나 만들어두세요.
매주 금요일, kt그룹희망나눔재단 채널에 새 주간메뉴표가 올라오면:
- 카카오톡에서 그 이미지를 길게 눌러 **"공유" → "드라이브에 저장"** → 방금 만든 "구내식당메뉴" 폴더 선택

(폴더 이름을 다르게 하고 싶으면 `ROUTINE_PROMPT.md`의 폴더명도 같이 바꿔주세요.)

## 3. Claude Code에서 루틴 만들기

Claude Code (웹/데스크톱/모바일)에서:

1. **저장소 연결**: 위에서 만든 GitHub 저장소를 선택
2. **커넥터**: Google Drive 활성화 (이 루틴 세션에서 사용할 수 있도록)
3. **시크릿(환경변수) 등록**:
   - `KAKAO_REST_API_KEY` = `5aaca31d7477bdedfead35aefbc7b166`
   - `KAKAO_CLIENT_SECRET` = (Kakao Developers에서 확인한 클라이언트 시크릿)
   - `KAKAO_REFRESH_TOKEN` = (이번에 발급받은 refresh_token)
4. **스케줄**: 평일 매일 아침 원하는 시간 (예: 매일 08:00, 요일 월~금)
5. **지시사항(프롬프트)**: `ROUTINE_PROMPT.md`의 내용을 그대로 복사해서 붙여넣기

## 4. 유지보수 참고사항

- 카카오 refresh_token 유효기간은 발급 후 약 60일입니다. 60일 이내에 만료 임박 시 카카오가 새 refresh_token을 자동 재발급하기도 하는데, 이 경우 루틴 실행 로그에 경고 메시지가 남습니다 — 그 값으로 `KAKAO_REFRESH_TOKEN` 시크릿을 갱신해주세요.
- 만약 60일 이상 루틴을 사용 안 하다가 refresh_token이 완전히 만료되면, 처음에 했던 로그인 인증(6~7단계)을 다시 한번 해서 새 refresh_token을 받아야 합니다.
