# 원본 프로젝트 실행·배포 안내

이 문서는 문서형 포트폴리오에서 소개하는 `마음의 날씨` 애플리케이션을 원본 팀 저장소의 소스로 실행하는 방법입니다. 아래 파일 경로와 명령은 **원본 저장소**를 기준으로 합니다.

- 원본: [guddlrdl123/WeatherOfTheHeart-](https://github.com/guddlrdl123/WeatherOfTheHeart-)
- 확인 기준: 2026년 10월 1일, 원본 `main` 커밋 [`40a127e`](https://github.com/guddlrdl123/WeatherOfTheHeart-/commit/40a127ecd8baf8b110a438e9ef1a858a0bf6e77b)
- 구현·배포 점검 근거: [README 점검 기록](README_AUDIT.md)
- [포트폴리오 README로 돌아가기](../README.md)

## 1. 실행에 필요한 환경

| 항목 | 기준 | 확인한 파일 |
| --- | --- | --- |
| Java | JDK 21 | `backend/build.gradle` |
| Spring Boot | 4.0.6 | 같은 파일의 플러그인 선언 |
| Gradle | 프로젝트의 Wrapper 사용, 9.4.1 | `backend/gradle/wrapper/gradle-wrapper.properties` |
| Node.js | 22.12 이상 사용. 잠금 파일의 Vite 요구 범위는 `^20.19.0` 또는 `>=22.12.0` | `frontend/package-lock.json` |
| 패키지 설치 | npm과 `npm ci` | `frontend/package-lock.json` |
| 로컬 DB | Docker Compose, MySQL 8.4 | `compose.yaml` |

현재 소스에서 확인된 Spring Boot 버전은 4.0.6입니다. 다른 버전으로 바꿔 실행한 결과를 이 안내의 검증 결과로 취급하지 않습니다.

## 2. 원본 저장소 받기

```bash
git clone https://github.com/guddlrdl123/WeatherOfTheHeart-.git
cd WeatherOfTheHeart-
```

이하 명령은 별도 설명이 없으면 이 저장소의 루트에서 시작합니다. 백엔드와 프론트엔드 실행은 각각 별도 터미널을 사용합니다.

## 3. 루트 `.env` 설정

백엔드는 `application.yaml`의 `spring.config.import`와 `spring-dotenv`를 사용합니다. 프론트엔드는 `vite.config.ts`의 `envDir: '..'`로 루트 환경 파일을 읽습니다.

아래 예시의 자리표시자를 실제 값으로 교체합니다.

```env
SERVER_PORT=5000

DB_URL=jdbc:mysql://localhost:3306/woth
DB_USERNAME=root
DB_PASSWORD=replace_with_local_mysql_password

MEMORY_ENCRYPTION_KEY=replace_with_generated_base64_key
MEMORY_ENCRYPTION_MIGRATE=false

JWT_SECRET=replace_with_a_long_random_signing_secret
JWT_EXPIRES_IN_SECONDS=86400

OPENAI_API_KEY=replace_with_openai_api_key

MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=replace_with_email_address
MAIL_PASSWORD=replace_with_smtp_app_password

GOOGLE_OAUTH_CLIENT_ID=replace_with_google_client_id
GOOGLE_OAUTH_CLIENT_SECRET=replace_with_google_client_secret
KAKAO_REST_API_KEY=replace_with_kakao_rest_api_key
KAKAO_CLIENT_SECRET=replace_with_kakao_client_secret
NAVER_CLIENT_ID=replace_with_naver_client_id
NAVER_CLIENT_SECRET=replace_with_naver_client_secret
OAUTH_ALLOWED_REDIRECT_URIS=http://localhost:5173/oauth/callback/google,http://localhost:5173/oauth/callback/kakao,http://localhost:5173/oauth/callback/naver

AWS_REGION=ap-northeast-2
AWS_S3_BUCKET_NAME=replace_with_image_bucket_name
AWS_S3_PUBLIC_BASE_URL=https://replace_with_image_bucket_name.s3.ap-northeast-2.amazonaws.com

VITE_API_BASE_URL=http://localhost:5000
VITE_S3_ASSET_BASE_URL=https://replace_with_image_bucket_name.s3.ap-northeast-2.amazonaws.com
VITE_OBJECT_CATALOG_MODE=api
```

| 설정 | 필요한 기능·주의점 |
| --- | --- |
| `DB_URL`, `DB_USERNAME`, `DB_PASSWORD` | 백엔드 DB 연결. 로컬 Compose의 DB 이름은 `woth`, 계정은 `root` |
| `MEMORY_ENCRYPTION_KEY` | 애플리케이션 시작에 필요. Base64를 해독했을 때 정확히 32바이트여야 함 |
| `JWT_SECRET` | 로그인 토큰의 HMAC 서명 키. 로컬 기본값을 운영에 재사용하지 않도록 별도 지정 |
| `OPENAI_API_KEY` | AI 분석과 광장 이미지 생성. 개인 기억 생성도 백엔드에서 AI 분석을 호출하므로 필요 |
| `MAIL_*` | 이메일 인증, 비밀번호 재설정, 이메일 변경과 관련 기능 |
| OAuth 설정 | 각 제공자의 인가 코드 교환·프로필 조회. 허용 콜백은 제공자 콘솔과 백엔드에서 동일하게 등록 |
| AWS·S3 설정 | 오브젝트 이미지 읽기와 광장 결과 이미지 저장. 버킷 이름뿐 아니라 사용 주체의 권한도 필요 |
| `VITE_*` | 프론트엔드에 공개되는 주소·기능 설정. 비밀값을 넣지 않음 |

프론트엔드 설정은 빌드할 때 결과 파일에 반영됩니다. 주소를 바꾼 뒤에는 개발 서버를 다시 시작하거나 다시 빌드합니다. `apiClient.ts`에는 기존 `NEXT_PUBLIC_API_BASE_URL`이 `VITE_API_BASE_URL`보다 우선하는 호환 설정이 있으므로 중복된 값도 확인합니다. `VITE_`와 `NEXT_PUBLIC_` 접두어의 변수는 브라우저에 공개될 수 있습니다.

문서 보관용 S3 설정도 원본에 있습니다. 해당 기능을 사용할 때만 추가합니다.

```env
AWS_S3_RAG_BUCKET_NAME=replace_with_document_bucket_name
AWS_S3_RAG_RAW_PREFIX=raw/
AWS_S3_RAG_PROCESSED_PREFIX=processed/
AWS_S3_RAG_FAILED_PREFIX=failed/
```

현재 `RagDocumentStorageService`에서 확인되는 기능은 원본 문서의 S3 업로드입니다. 이를 근거로 문서 검색·벡터 저장소·검색 결과를 반영한 AI 답변까지 완료됐다고 설명하지 않습니다.

### AES 키 생성

macOS/Linux:

```bash
openssl rand -base64 32
```

Windows PowerShell:

```powershell
$memoryKeyBytes = New-Object byte[] 32
$memoryKeyGenerator = [System.Security.Cryptography.RandomNumberGenerator]::Create()
$memoryKeyGenerator.GetBytes($memoryKeyBytes)
[Convert]::ToBase64String($memoryKeyBytes)
$memoryKeyGenerator.Dispose()
```

출력값을 `MEMORY_ENCRYPTION_KEY`에 넣습니다. 기존 암호화된 데이터를 계속 읽으려면 같은 키를 보관해야 합니다. `MEMORY_ENCRYPTION_MIGRATE`는 기존 평문 기억을 옮기는 일회성 작업에 사용하는 설정이며 기본 실행에서는 `false`로 둡니다.

## 4. 로컬 MySQL 시작

```bash
docker compose up -d mysql
docker compose ps
```

MySQL은 호스트의 `3306` 포트를 사용하며 데이터는 Compose의 `woth_mysql_data` 볼륨 정의에 유지됩니다. 다른 MySQL이 이미 같은 포트를 사용하면 실행 중인 환경에 맞춰 포트와 JDBC URL을 함께 조정합니다.

현재 JPA 설정은 `ddl-auto: update`, SQL 초기화 설정은 `spring.sql.init.mode: never`입니다. `schema.sql`은 실행 때 자동으로 적용되지 않습니다. 빈 DB에서 화면에 필요한 오브젝트 카탈로그와 S3 이미지 자료가 저절로 채워진다고 가정하지 말고 별도로 준비합니다.

## 5. 백엔드 실행과 상태 확인

macOS/Linux:

```bash
cd backend
./gradlew bootRun
```

Windows PowerShell:

```powershell
cd backend
.\gradlew.bat bootRun
```

기본 주소는 `http://localhost:5000`입니다. 기본 포트는 `application.yaml`의 `${SERVER_PORT:5000}`으로 정해집니다.

macOS/Linux:

```bash
curl -i http://localhost:5000/actuator/health
```

Windows PowerShell:

```powershell
Invoke-RestMethod http://localhost:5000/actuator/health
```

정상 상태의 대표 응답은 다음과 같습니다.

```json
{"status":"UP"}
```

| 경로 | 구현과 검사 범위 |
| --- | --- |
| `/actuator/health` | Actuator 의존성과 기본 자동 설정. 자동 등록된 DB·메일 등의 상태가 결과에 영향을 줄 수 있음 |
| `/health` | `HealthCheckController`가 항상 HTTP 200과 `OK`를 반환 |
| `/` | 같은 컨트롤러가 HTTP 200과 `OK`를 반환 |

이번 점검에서 실행 중인 서버에 이 요청을 보내지는 않았습니다. 정상 응답 예시는 실제 실행 때 확인할 기준입니다. `/actuator/health`도 S3·OpenAI·소셜 로그인·전체 화면 기능을 모두 검사하지 않으므로, 연결된 기능은 별도로 확인합니다.

로드밸런서 검사 경로를 `/actuator/health`로 정할 때는 DB·메일 상태가 응답에 주는 영향과 실제 HTTP 상태 코드를 확인합니다. 당시 Elastic Beanstalk에 설정된 검사 경로는 소스만으로 확정할 수 없습니다.

## 6. 프론트엔드 실행

별도 터미널에서 원본 저장소 루트로 이동한 뒤 실행합니다.

```bash
cd frontend
npm ci
npm run dev
```

터미널에 표시된 개발 주소로 접속합니다. 기본 포트는 `5173`이며, 다른 포트를 사용하면 OAuth 허용 콜백 주소도 맞춥니다.

| 화면 | 주소 |
| --- | --- |
| 랜딩·로그인·회원가입 | `/`, `/login`, `/signup` |
| 비밀번호 재설정 | `/reset-password` |
| 소셜 로그인 콜백 | `/oauth/callback/:provider` |
| 개인 방 | `/room` |
| 광장 목록·상세 | `/plaza`, `/plaza/:plazaId` |
| 편지함·마이페이지 | `/mailbox`, `/mypage` |
| 문의·공지 | `/qna`, `/notices` |
| 관리자 신고 | `/admin/reports` |

## 7. 주요 API

인증이 필요한 요청은 `Authorization: Bearer <accessToken>`을 사용합니다. `@CurrentUser` 인자 처리기가 토큰을 해석하고, 각 기능에서 사용자·소유자·관리자 조건을 확인합니다. JSON 응답은 공통 `ApiResponse` 형식을 사용하며 상태 확인 문자열과 이미지 파일 응답은 별도입니다.

| 구분 | 방식 | 경로 | 역할 |
| --- | --- | --- | --- |
| 상태 | GET | `/actuator/health` | Actuator 상태 확인 |
| 상태 | GET | `/health`, `/` | 고정 `OK` 응답 |
| 인증 | POST | `/api/auth/signup` | 회원가입 |
| 인증 | POST | `/api/auth/login` | 로그인 |
| 인증 | GET | `/api/auth/oauth/{provider}/authorize` | 소셜 로그인 인가 URL |
| 인증 | POST | `/api/auth/oauth/{provider}/login` | 인가 코드 교환과 로그인 |
| 인증 | POST | `/api/auth/email/send` | 이메일 인증번호 발송 |
| 인증 | POST | `/api/auth/email/verify` | 이메일 인증번호 확인 |
| 인증 | POST | `/api/auth/password/reset/request` | 비밀번호 재설정 요청 |
| 인증 | POST | `/api/auth/password/reset/verify` | 재설정 코드 확인 |
| 인증 | POST | `/api/auth/password/reset/confirm` | 새 비밀번호 확정 |
| AI | POST | `/api/ai/analyze` | 글의 정서에 따른 날씨 분석 |
| 사용자 | GET | `/api/users/me` | 내 프로필 |
| 사용자 | PATCH | `/api/users/me` | 내 프로필 변경 |
| 사용자 | POST | `/api/users/me/email/change/send` | 이메일 변경 인증번호 발송 |
| 사용자 | PATCH | `/api/users/me/email` | 이메일 변경 |
| 사용자 | GET | `/api/users/me/plazas` | 내가 만든 광장 |
| 사용자 | GET | `/api/users/me/plaza-entries` | 내가 작성한 광장 글 |
| 사용자 | DELETE | `/api/users/me` | 회원 탈퇴 |
| 사용자 | POST | `/api/users/me/social-withdraw/email/send` | 소셜 회원 탈퇴 인증번호 발송 |
| 사용자 | DELETE | `/api/users/me/social-withdraw` | 소셜 회원 탈퇴 |
| 기억 | GET | `/api/memories` | 개인 기억 목록 |
| 기억 | POST | `/api/memories` | 개인 기억 생성과 AI 날씨 분석 |
| 기억 | PATCH | `/api/memories/{memoryId}` | 제목·본문·감정 태그 변경 |
| 기억 | PUT/PATCH | `/api/memories/{memoryId}/position` | 좌표·반전·기울기·레이어 변경 |
| 기억 | DELETE | `/api/memories/{memoryId}` | 개인 기억 삭제 |
| 오브젝트 | GET | `/api/objects` | 활성 카탈로그 목록 |
| 오브젝트 | GET | `/api/objects/{objectKey}/image` | 이미지 파일 |
| 광장 | GET | `/api/plazas` | 광장 목록 |
| 광장 | GET | `/api/plazas/{plazaId}` | 광장 상세 |
| 광장 | POST | `/api/plazas` | 광장 생성 |
| 광장 | POST | `/api/plazas/with-first-entry` | 첫 글과 함께 생성 |
| 광장 | DELETE | `/api/plazas/{plazaId}` | 광장 삭제 |
| 광장 | GET | `/api/plazas/entries` | 광장 글 통합 조회 |
| 광장 | GET | `/api/plazas/{plazaId}/entries` | 광장별 글 조회 |
| 광장 | POST | `/api/plazas/{plazaId}/entries` | 광장 참여 글 작성 |
| 광장 | PATCH | `/api/plazas/entries/{entryId}` | 광장 글 변경 |
| 광장 | PATCH | `/api/plazas/entries/{entryId}/position` | 오브젝트 위치 변경 |
| 광장 | POST | `/api/plazas/entries/{entryId}/likes` | 좋아요 전환 |
| 광장 | POST | `/api/plazas/entries/{entryId}/reports` | 글 신고 |
| 광장 | DELETE | `/api/plazas/entries/{entryId}` | 광장 글 삭제 |
| 광장 | PATCH | `/api/plazas/{plazaId}/complete` | 방장의 완료 요청 |
| 편지함 | GET | `/api/mailbox` | 편지 목록 |
| 편지함 | GET | `/api/mailbox/unread-count` | 읽지 않은 편지 수 |
| 편지함 | PATCH | `/api/mailbox/{letterId}/read` | 읽음 처리 |
| 편지함 | PATCH | `/api/mailbox/read-all` | 전체 읽음 |
| 편지함 | DELETE | `/api/mailbox/{letterId}` | 편지 삭제 |
| 편지함 | GET | `/api/mailbox/{letterId}/image` | 결과 이미지 다운로드 |
| 공지 | GET | `/api/notices` | 공지 목록 |
| 공지 | POST | `/api/notices` | 관리자 공지 작성 |
| 공지 | PATCH | `/api/notices/{noticeId}` | 관리자 공지 변경 |
| 공지 | DELETE | `/api/notices/{noticeId}` | 관리자 공지 삭제 |
| 문의 | POST | `/api/inquiries` | 문의 작성 |
| 문의 | GET | `/api/inquiries` | 문의 목록 |
| 문의 | PATCH | `/api/inquiries/{inquiryId}/answer` | 관리자 답변 |
| 문의 | DELETE | `/api/inquiries/{inquiryId}` | 문의 삭제 |
| 관리자 | GET | `/api/admin/reports` | 신고 목록 |
| 관리자 | DELETE | `/api/admin/reports/entries/{entryId}` | 신고된 글 삭제 |
| 관리자 | PATCH | `/api/admin/users/{userId}/suspension` | 사용자 정지 상태 변경 |

AI 분석 응답의 데이터는 `weatherKey`, `weatherLabel`, `confidence`, `reason` 네 필드입니다. 개인 기억 생성 시 감정 태그와 오브젝트는 사용자 선택값이며, 날씨는 서버가 AI 결과로 결정합니다. 같은 날짜의 기억 중복을 제한하고, 해당 연·월의 개인 방이 없으면 자동 생성합니다.

## 8. 테스트와 빌드

백엔드, macOS/Linux:

```bash
cd backend
./gradlew test
./gradlew build
```

백엔드, Windows PowerShell:

```powershell
cd backend
.\gradlew.bat test
.\gradlew.bat build
```

프론트엔드:

```bash
cd frontend
npm ci
npm run lint
npm run build
```

현재 백엔드에는 컨텍스트 로드 테스트 1건과 OAuth 탈퇴 후 재가입 회귀 테스트 1건이 있습니다. 컨텍스트 로드에는 DB 연결·암호화 키 등 애플리케이션 설정이 필요합니다. 위 명령은 실행 안내이며 이번 문서 점검에서 통과했다고 보고한 결과는 아닙니다.

## 9. 실제 백엔드 배포 구성

원본 `.github/workflows/deploy-backend.yml`과 `backend/Procfile`에서 확인한 흐름입니다.

| 단계 | 현재 설정 |
| --- | --- |
| 실행 조건 | `main`에 `backend/**` 또는 `.github/workflows/**` 변경을 push하거나 수동 실행 |
| Java | Amazon Corretto 21 |
| 빌드 | `./gradlew clean build -x test` |
| 실행 파일 | `*-plain.jar`을 제외한 JAR을 `application.jar`로 복사 |
| 배포 묶음 | `application.jar`와 `Procfile`을 ZIP 루트에 배치 |
| 시작 명령 | `web: java -jar application.jar` |
| 포트 | `SERVER_PORT=5000` 사용 |
| AWS 인증·리전 | GitHub Secrets의 `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION` |
| 배포 대상 | AWS Elastic Beanstalk. 애플리케이션·환경 이름은 원본 워크플로에서 관리 |
| DB | 운영 RDS for MySQL을 `DB_URL`, `DB_USERNAME`, `DB_PASSWORD`로 연결 |

[2026년 7월 1일 실행](https://github.com/guddlrdl123/WeatherOfTheHeart-/actions/runs/28488782913)에서 빌드·패키징·배포 단계가 모두 성공했습니다. 대상 커밋 [`7ec6142`](https://github.com/guddlrdl123/WeatherOfTheHeart-/commit/7ec61429f6b6e4e92933e39f67df6ec8a58ecba4)의 `build.gradle`도 4.0.6을 선언하며, 현재 `main`의 빌드 파일과 같은 blob입니다.

당시 배포 빌드는 테스트를 제외했습니다. 이 성공 기록을 자동 테스트 통과 기록이나 현재 데모 운영 상태로 해석하지 않습니다.

백엔드 실행 환경에는 DB·암호화 키·JWT 서명 키·메일·OAuth·OpenAI·S3 설정을 별도로 등록합니다. GitHub Actions의 배포용 자격 증명은 애플리케이션 실행 환경의 S3 권한과 구분합니다. 로컬 S3 SDK는 AWS 프로필·환경변수 등의 자격 증명 설정이 필요하고, Elastic Beanstalk에서는 인스턴스 역할의 버킷 접근 권한을 확인합니다.

## 10. React·Vite 프론트엔드 배포 재현 예시

이 프로젝트의 프론트엔드는 AWS Amplify에 배포했습니다. 다음은 현재 소스와 공식 Amplify 작성 형식에 맞춘 **재현용 예시**입니다. 당시 AWS 콘솔의 설정을 그대로 추출한 파일은 아닙니다.

원본 저장소를 연결하고 앱 경로와 `AMPLIFY_MONOREPO_APP_ROOT`를 `frontend`로 맞춥니다.

```yaml
version: 1
applications:
  - appRoot: frontend
    frontend:
      phases:
        preBuild:
          commands:
            - npm ci
        build:
          commands:
            - npm run build
      artifacts:
        baseDirectory: dist
        files:
          - '**/*'
      cache:
        paths:
          - node_modules/**/*
```

빌드 환경의 Node 버전은 위 실행 조건에 맞춥니다. `appRoot: frontend` 기준이므로 산출물은 `dist`입니다. Amplify 환경변수의 `VITE_API_BASE_URL`에는 백엔드 HTTPS 주소를 넣고 S3 이미지 기준 주소도 맞춥니다.

React Router를 사용하는 경로는 직접 접속하거나 새로고침해도 `index.html`로 연결되도록 SPA 주소 재작성 규칙을 설정합니다. 실제 이미지·자바스크립트 등 정적 파일 요청은 유지하는 AWS 공식 SPA 예시를 기준으로 적용합니다.

배포 도메인을 바꾸면 백엔드 `CorsConfig`의 허용 출처, `OAUTH_ALLOWED_REDIRECT_URIS`, 각 OAuth 제공자 콘솔의 콜백 주소도 함께 맞춥니다. 현재 원본 CORS 설정은 코드에 허용 출처를 직접 지정합니다.

공식 형식 참고:

- [Amplify 여러 프로젝트 저장소 빌드 설정](https://docs.aws.amazon.com/amplify/latest/userguide/monorepo-configuration.html)
- [Amplify SPA 주소 재작성 예시](https://docs.aws.amazon.com/amplify/latest/userguide/redirect-rewrite-examples.html)
- [Spring Boot 4.0 Actuator 엔드포인트](https://docs.spring.io/spring-boot/4.0/reference/actuator/endpoints.html)

## 11. 데이터와 결과 처리의 확인 범위

- 개인 기억의 제목·본문은 저장 때 AES-256-GCM 암호화, 읽을 때 복호화합니다. 암호화 범위를 광장 글·이미지·모든 DB 컬럼까지 확대하지 않습니다.
- 광장 완료는 저장 트랜잭션 커밋 뒤 비동기 작업으로 처리합니다. 이미지·편지 조회 가능 시점은 외부 API 처리에 따라 지연될 수 있습니다.
- OpenAI 요청에는 연결 오류·시간 초과·429·5xx에 대한 조건부 재시도가 있습니다. 완료 작업 자체를 보존하는 영속 큐·서버 재시작 후 자동 복구와는 구분합니다.
- AI 이미지 생성 실패 시 이미지가 없는 편지를, S3 업로드 실패 시 원본 data URL을 이용하는 흐름이 있습니다. 생성 중 상태는 `finally`에서 해제합니다.
- `imageGenerating` 확인은 DB의 원자적 잠금이 아니므로 동시 작업에 대한 완전한 중복 방지 보장으로 설명하지 않습니다.
- 원본의 [ERD 개요](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/main/docs/db-erd.md)와 현재 JPA 엔티티를 함께 확인합니다. 일부 값 참조는 실제 DB 외래 키와 다릅니다.
- 이 포트폴리오의 데모 상태는 운영 종료입니다. 이번 작업에서는 AWS 자원을 생성하거나 서비스를 재배포하지 않았습니다.
