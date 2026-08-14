#!/bin/bash
# 사용법: ./send_kakao.sh "보낼 메시지 텍스트"
#
# 필요한 환경변수 (루틴 시크릿으로 등록):
#   KAKAO_REST_API_KEY
#   KAKAO_CLIENT_SECRET
#   KAKAO_REFRESH_TOKEN
set -e

MESSAGE="$1"

if [ -z "$MESSAGE" ]; then
  echo "에러: 보낼 메시지가 없습니다. 사용법: ./send_kakao.sh \"메시지\""
  exit 1
fi

if [ -z "$KAKAO_REST_API_KEY" ] || [ -z "$KAKAO_CLIENT_SECRET" ] || [ -z "$KAKAO_REFRESH_TOKEN" ]; then
  echo "에러: KAKAO_REST_API_KEY / KAKAO_CLIENT_SECRET / KAKAO_REFRESH_TOKEN 환경변수가 설정되어 있지 않습니다."
  exit 1
fi

echo "[1/2] access token 갱신 중..."
TOKEN_RESPONSE=$(curl -s -X POST "https://kauth.kakao.com/oauth/token" \
  -d "grant_type=refresh_token" \
  -d "client_id=${KAKAO_REST_API_KEY}" \
  -d "client_secret=${KAKAO_CLIENT_SECRET}" \
  -d "refresh_token=${KAKAO_REFRESH_TOKEN}")

ACCESS_TOKEN=$(echo "$TOKEN_RESPONSE" | grep -o '"access_token":"[^"]*' | cut -d'"' -f4)
NEW_REFRESH_TOKEN=$(echo "$TOKEN_RESPONSE" | grep -o '"refresh_token":"[^"]*' | cut -d'"' -f4)

if [ -z "$ACCESS_TOKEN" ]; then
  echo "토큰 갱신 실패: $TOKEN_RESPONSE"
  exit 1
fi

if [ -n "$NEW_REFRESH_TOKEN" ]; then
  echo "⚠️  카카오가 새 refresh_token을 발급했습니다. 만료 방지를 위해 루틴 시크릿(KAKAO_REFRESH_TOKEN)을 아래 값으로 업데이트해주세요:"
  echo "$NEW_REFRESH_TOKEN"
fi

echo "[2/2] 메시지 전송 중..."
SEND_RESPONSE=$(curl -s -X POST "https://kapi.kakao.com/v2/api/talk/memo/default/send" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  --data-urlencode "template_object={\"object_type\":\"text\",\"text\":\"${MESSAGE}\",\"link\":{\"web_url\":\"https://developers.kakao.com\",\"mobile_web_url\":\"https://developers.kakao.com\"}}")

echo "$SEND_RESPONSE"

if echo "$SEND_RESPONSE" | grep -q '"result_code":0'; then
  echo "✅ 전송 성공"
else
  echo "❌ 전송 실패"
  exit 1
fi
