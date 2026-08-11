# Sendgo OpenAPI Specification

Sendgo API의 OpenAPI 3.0.3 스펙입니다. 코드 생성기, API 클라이언트, AI 코딩 도구에
그대로 넣어 쓸 수 있는 기계 판독용 계약입니다.

- 최신 스펙: <https://sendgo.io/openapi.yaml>
- 서버: `https://api.sendgo.io/api`

## 엔드포인트

| 채널 | 메서드 | 경로 |
| --- | --- | --- |
| 토큰 발급 | `POST` | `/{version}/token` |
| 알림톡 (Alimtalk) | `POST` | `/{version}/notices/send` |
| 친구톡 (Friendtalk) | `POST` | `/{version}/friends/send` |
| 브랜드메시지 발송 | `POST` | `/{version}/brand-messages/send` |
| 브랜드메시지 캠페인 목록 | `GET` | `/{version}/brand-messages` |
| 브랜드메시지 캠페인 상세 | `GET` | `/{version}/brand-messages/{campaign_id}` |
| SMS/LMS/MMS | `POST` | `/{version}/messages/send` |

`{version}` 은 `v1` 또는 `v2` 입니다. 브랜드메시지는 **v2 전용**입니다.

친구톡은 카카오의 정책에 따라 종료될 예정이므로 신규 연동은 브랜드메시지를 사용하세요.

## 인증

2단계입니다.

1. **토큰 발급** — `accessKey:secretKey` 를 Base64로 인코딩해 Basic 인증으로 `POST /{version}/token` 호출
2. **API 호출** — 발급받은 토큰으로 Bearer 인증
   - v1: `Authorization: Bearer base64(token)`
   - v2: `Authorization: Bearer token`

토큰은 발급 후 **24시간** 유효합니다. 응답의 `expiresAt`(v2) / `expires_at`(v1) 값을
보고 만료 전에 재발급하세요.

## 사용법

### 브라우저에서 살펴보기

[Swagger Editor](https://editor.swagger.io)에 `openapi.yaml` 을 붙여넣으면
엔드포인트와 스키마를 시각적으로 탐색할 수 있습니다.

### 클라이언트 코드 생성

```bash
# TypeScript
npx @openapitools/openapi-generator-cli generate \
  -i https://sendgo.io/openapi.yaml -g typescript-fetch -o ./sendgo-client

# Python
openapi-generator-cli generate \
  -i https://sendgo.io/openapi.yaml -g python -o ./sendgo-client
```

공식 SDK가 이미 있는 언어라면 생성된 클라이언트보다 SDK를 쓰는 게 낫습니다.
SDK는 토큰 캐싱과 만료 시 자동 재발급을 처리해주지만, 생성된 클라이언트는 직접
구현해야 합니다. 지원 언어 목록은 <https://sendgo.io/ko/sdk> 를 참고하세요.

### 스펙 검증

```bash
npx @redocly/cli lint openapi.yaml
```

## 변경 사항

### 1.1.0 (2026-08-11)

- 짧은 URL 5개 경로 추가
- 토큰 유효시간을 '약 50분' → '24시간' 으로 수정 (실제 구현은 `Carbon::now()->addDay()`)
- paths 에서 쓰이는데 선언되지 않았던 `브랜드메시지` 태그 선언 추가

## 라이선스

MIT © Sendgo — https://sendgo.io
