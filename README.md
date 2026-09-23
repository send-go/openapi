# Sendgo OpenAPI Specification

Sendgo API의 OpenAPI 3.0.3 스펙입니다. 코드 생성기, API 클라이언트, AI 코딩 도구에
그대로 넣어 쓸 수 있는 기계 판독용 계약입니다.

- 최신 스펙: <https://sendgo.io/openapi.yaml>
- 서버: `https://sendgo.io/api`

## 엔드포인트

### 발송

| 채널 | 메서드 | 경로 |
| --- | --- | --- |
| 토큰 발급 | `POST` | `/{version}/token` |
| 알림톡 (Alimtalk) | `POST` | `/{version}/notices/send` |
| 친구톡 (Friendtalk) — **Deprecated** | `POST` | `/{version}/friends/send` |
| 브랜드메시지 발송 | `POST` | `/{version}/brand-messages/send` |
| 브랜드메시지 캠페인 목록 | `GET` | `/{version}/brand-messages` |
| 브랜드메시지 캠페인 상세 | `GET` | `/{version}/brand-messages/{campaign_id}` |
| SMS/LMS/MMS | `POST` | `/{version}/messages/send` |
| 짧은 URL | `GET`·`POST`·`DELETE` | `/{version}/short-urls[/{code}[/stats]]` |

### 관리 — 등록 · 심사 (v2 전용)

콘솔 화면에서만 되던 일을 코드로 처리합니다.

| 대상 | 메서드 | 경로 |
| --- | --- | --- |
| 채널 인증번호 발송 | `POST` | `/{version}/kakao-senders/token` |
| 발신프로필 등록 | `POST` | `/{version}/kakao-senders` |
| 발신프로필 목록·카테고리 | `GET` | `/{version}/kakao-senders`, `/kakao-senders/categories` |
| 발신프로필 동기화 | `POST` | `/{version}/kakao-senders[/{key}]/sync` |
| 브랜드메시지 M/N 증적·신청 | `POST` | `/{version}/kakao-senders/{key}/brand-message/{evidence\|apply}` |
| 알림톡 템플릿 CRUD | `GET`·`POST`·`PUT`·`DELETE` | `/{version}/notice-templates[/{templateCode}]` |
| 알림톡 검수 요청·취소 | `POST`·`DELETE` | `/{version}/notice-templates/{templateCode}/inspection` |
| 알림톡 승인 취소 | `DELETE` | `/{version}/notice-templates/{templateCode}/approval` |
| 알림톡 휴면 해제 | `POST` | `/{version}/notice-templates/{templateCode}/release` |
| 브랜드메시지 템플릿 CRUD | `GET`·`POST`·`PUT`·`DELETE` | `/{version}/brand-templates[/{templateCode}]` |
| 발신번호 등록·수정·삭제 | `POST`·`PATCH`·`DELETE` | `/{version}/senders[/{senderKey}]` |
| 발신번호 유형·중복 확인 | `GET`·`POST` | `/{version}/senders/number-types`, `/senders/validate` |
| 문자 템플릿 CRUD | `GET`·`POST`·`PUT`·`DELETE` | `/{version}/message-templates[/{templateKey}]` |
| 카카오 이미지 업로드 | `GET`·`POST` | `/{version}/kakao-images/types`, `/kakao-images/{type}` |
| 수신거부(080) 조회 | `GET` | `/{version}/rejected-numbers` |
| 이벤트 웹훅 구독 | `GET`·`PUT`·`DELETE`·`POST` | `/{version}/webhook[/test]` |

`{version}` 은 `v1` 또는 `v2` 입니다. 브랜드메시지와 관리 API 는 **v2 전용**입니다.

> **관리 API 는 즉시 완료되지 않습니다.** 알림톡 템플릿은 카카오가, 발신번호는
> sendgo 운영자가 심사합니다. 등록 호출이 성공했다는 것은 "접수됐다"는 뜻이지
> "쓸 수 있다"는 뜻이 아닙니다 — `PUT /{version}/webhook` 으로 구독해 결과를
> 받으세요.
>
> 리셀러는 **sendgo.io 콘솔에 들어올 일이 없습니다.** 휴대폰 발신번호는 PASS
> 대신 신분증 사본을 받아 sendgo 운영자가 대신 심사합니다. 사람이 개입하는
> 지점은 카카오 채널 인증번호 하나뿐이고, 그것도 리셀러 화면에서 받습니다.
>
> 카카오 관련 관리 API 는 **기업(Team) 소유 애플리케이션**만 사용할 수 있습니다.

> ⚠️ **친구톡은 카카오 정책에 따라 2025-12-31 종료되었습니다.**
> 2026-01-01 부터 `/{version}/friends/send` 로 들어온 요청은 카카오 측에서
> **브랜드메시지(자유형)** 로 자동 대체 발송됩니다. 호출은 계속 성공하지만 실제로
> 나가는 것은 브랜드메시지입니다.
>
> 엔드포인트는 제거되지 않습니다 — 자유 본문 타입(`FT`/`FI`/`FW`)을 개별 수신자에게
> 보내는 경로는 현재 이것뿐이며, `/{version}/brand-messages/send` 는 같은 조합에 대해
> `NOT_A_BRAND_MESSAGE` 를 반환합니다.
>
> 다음의 경우에는 브랜드메시지를 사용하세요.
> - 템플릿 기반 리치 타입 (`FL`/`FC`/`FM`/`FP`/`FA`)
> - 채널 친구가 **아닌** 수신자 (`targeting` = `N` / `I`)
> - 수신 동의한 전체 채널 친구 동보 (`targeting` = `F`)
>
> 메시지 타입은 1:1 대응됩니다 — `FT`→`BT`, `FI`→`BI`, `FW`→`BW`, `FL`→`BL`,
> `FC`→`BC`, `FM`→`BM`, `FP`→`BP`, `FA`→`BA`. 변환은 서버가 처리하므로 요청에는
> 친구톡 코드를 그대로 넘깁니다.

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

## 계정 API (본체 3.3.0)

`/v2/account`의 12개 연산과 `AgentToken` 인증 스키마를 동기화했습니다.
계정 조회, 조직 목록·선택, API 키 CRUD·발송 토큰 발급, 허용 IP 목록·추가·삭제를 제공합니다.
애플리케이션 토큰 대신 콘솔에서 발급받은 에이전트 토큰을 Bearer로 전달합니다.

## 템플릿 폴더 (1.5.0)

기업 계정의 발송용 API 키와 `apiVersion=v2` 설정으로 사용하는 서버 전용 API입니다.
폴더는 알림톡·브랜드메시지가 공유하며, 목록의 `templateType`은 `notice` 또는 `brand`입니다.
목록은 `data.folders` 트리와 `total`, `uncategorised` 개수를 반환합니다.
`templateCount`는 하위 폴더를 제외한 해당 폴더의 템플릿 수입니다.

- 생성: `name`, 선택 `parentUuid`. 최대 5단계이며 같은 부모 아래 이름 중복은 409입니다.
- 이동: 동일 발신프로필의 `templateCodes` 1~100개. `folderUuid`는 필수이며 `null`이면 미분류로 이동합니다.
- 템플릿 목록: `folderUuid=none`은 미분류, UUID는 해당 폴더, 생략은 전체입니다.
- 템플릿 등록: 선택 필드 `folderUuid`로 폴더를 지정합니다. 기존 템플릿 수정 API 대신 폴더 이동 API를 사용하세요.

승인되지 않은 키의 `403 ACCESS_KEY_NOT_APPROVED`는 토큰 재발급·재시도 없이 반환합니다.
계정 API의 `autoApprove`는 서버 설정의 실제 승인 정책을 나타냅니다.

동기화 기준: 본체 `eb9e43d9` (2026-09-24).

| 메서드 | 경로 | 기능 |
| --- | --- | --- |
| GET | `/v2/template-folders` | 폴더 트리와 개수 |
| POST | `/v2/template-folders` | 폴더 생성 |
| PATCH | `/v2/template-folders/templates` | 일괄 이동·미분류로 이동 |
