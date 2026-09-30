# README 구현·배포 점검 기록

- 확인일: **2026년 10월 1일**
- 원본 팀 저장소: [guddlrdl123/WeatherOfTheHeart-](https://github.com/guddlrdl123/WeatherOfTheHeart-)
- 원본 최신 `main`: [`40a127ecd8baf8b110a438e9ef1a858a0bf6e77b`](https://github.com/guddlrdl123/WeatherOfTheHeart-/commit/40a127ecd8baf8b110a438e9ef1a858a0bf6e77b), 2026년 7월 8일 README 수정 커밋
- 반영 대상: [junyeong2107/weather-of-the-heart-portfolio](https://github.com/junyeong2107/weather-of-the-heart-portfolio)
- 범위: 원본 README·구현·설정·GitHub Actions 기록을 대조해 포트폴리오 README와 관련 문서를 갱신
- [README로 돌아가기](../README.md) · [전체 실행·배포 안내](RUN_GUIDE.md)

## 1. 주요 판정

| 항목 | 확인한 근거 | 문서 반영 |
| --- | --- | --- |
| Spring Boot | 최신 `main`의 `build.gradle`에 **4.0.6** 선언. 최근 배포 성공 대상 커밋의 빌드 파일도 동일 | `Spring Boot 4`를 정확한 **4.0.6**으로 명시. 3.5.x로 변경하지 않음 |
| Java·Gradle | Java toolchain 21, Gradle Wrapper 9.4.1, Actions의 Corretto 21 | 기존 개발 도구 설명을 현재 파일 기준으로 명확히 표기 |
| 상태 확인 | Actuator 의존성, 기본 경로를 바꾸는 설정 없음. 직접 만든 컨트롤러의 `/health`·`/`는 고정 `OK` | `/actuator/health` 중심 실행 안내와 세 경로의 검사 범위를 구분 |
| 백엔드 배포 | Actions에서 실행 JAR+Procfile ZIP을 Elastic Beanstalk로 배포. 2026년 7월 1일 실행 성공 | 실제 파일·트리거·시작 명령·성공 기록을 연결 |
| 테스트 | 배포 명령은 `clean build -x test` | 배포 성공을 테스트 통과로 표현하지 않음 |
| 프론트엔드 | React 19.2.6·Vite 8.0.14, `tsc -b && vite build`. 당시 Amplify 콘솔 원본은 저장소에 없음 | `frontend` 앱 경로와 `dist` 산출물로 재현 예시 작성. 배포 경험과 예시를 구분 |
| AI 분석 범위 | 응답은 `weatherKey`, `weatherLabel`, `confidence`, `reason` | 개인 기억의 날씨는 AI 결과, 감정 태그·오브젝트는 사용자 선택으로 설명 |
| AES 암호화 | 32바이트 키, AES/GCM, 임의 12바이트 IV. `PrivateMemory.title/content`에 변환기 적용 | 개인 기억 제목·본문의 저장 암호화라는 강점과 적용 범위 유지 |
| 광장·편지함 | 커밋 후 비동기 완료 검사 → 이미지 생성 → S3 업로드 → 완성 편지 저장 | 핵심 흐름 유지. 이미지 제공 지연과 실패 시 처리도 정확히 설명 |
| 재시도·중복 제어 | OpenAI HTTP 요청에 조건부 재시도. 완료 작업의 영속 큐·원자적 DB 잠금은 없음 | HTTP 재시도와 작업 자동 복구를 구분. 생성 상태 확인을 완전한 중복 방지로 과장하지 않음 |
| DB 버전 | 로컬 Compose는 MySQL 8.4. 현재 RDS 엔진 버전은 저장소에서 확인 불가 | 로컬 DB 버전과 RDS 운영 구성 분리 |
| 개인 기여 | 작성자 이메일 기준으로 `bamkkayo` 명의 원본 커밋 63개 재확인 | 대표 기여·팀원 후속 개선·수상 증빙을 보존 |

## 2. Spring Boot 버전 정정의 근거

최신 원본 `README.md`의 `Spring Boot 4`는 현재 빌드 선언과 계열이 일치합니다. 따라서 소스 기준의 오류로 판정해 3.5.x로 바꾸기보다, 모호한 표기를 **4.0.6**으로 정확히 적었습니다.

확인한 두 빌드 파일은 GitHub blob SHA `430429ada39cfb3264d61d6fbd712d20bfdfda95`로 동일합니다.

- [최신 `main`의 build.gradle](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/40a127ecd8baf8b110a438e9ef1a858a0bf6e77b/backend/build.gradle)
- [최근 배포 성공 대상 커밋의 build.gradle](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/7ec61429f6b6e4e92933e39f67df6ec8a58ecba4/backend/build.gradle)
- [Gradle Wrapper 설정](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/40a127ecd8baf8b110a438e9ef1a858a0bf6e77b/backend/gradle/wrapper/gradle-wrapper.properties)

별도로 배포된 3.5.x 산출물이나 시작 로그는 이번 점검에서 확인되지 않았습니다. 과거의 버전 권고를 현재 저장소의 실제 버전으로 대신 사용하지 않았습니다.

## 3. 상태 확인 경로의 근거와 한계

- [HealthCheckController](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/40a127ecd8baf8b110a438e9ef1a858a0bf6e77b/backend/src/main/java/com/woth/backend/global/health/HealthCheckController.java): `/`·`/health`에 HTTP 200, 문자열 `OK` 반환
- [application.yaml](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/40a127ecd8baf8b110a438e9ef1a858a0bf6e77b/backend/src/main/resources/application.yaml): 기본 포트 5000. Actuator 경로 변경 설정 없음
- [Spring Boot 4.0 공식 문서](https://docs.spring.io/spring-boot/4.0/reference/actuator/endpoints.html): 기본 상태 확인 경로 `/actuator/health`

Actuator의 기본 경로 안내는 의존성과 기본 설정을 대조한 결과입니다. 이번 점검에서 애플리케이션을 직접 실행해 200 응답을 측정하지는 않았습니다. 실제 DB·메일 연결 상태나 AWS 환경 속성에 따라 결과가 달라질 수 있습니다.

S3·OpenAI와 전체 서비스 기능이 자동으로 검사된다고 설명하지 않았습니다. 현재 로드밸런서 설정 원본도 확보하지 않아 검사 경로를 당시 실제 적용값으로 단정하지 않았습니다. 원본에 없는 `/api/health/full` 같은 별도 프로젝트의 경로를 추가하지 않았습니다.

## 4. 배포 성공 기록

| 항목 | 확인값 |
| --- | --- |
| 실행 | [GitHub Actions 28488782913](https://github.com/guddlrdl123/WeatherOfTheHeart-/actions/runs/28488782913) |
| 실행 생성 시각 | 2026년 7월 1일 오전 11시 12분 24초, 한국 시간 |
| 대상 커밋 | `7ec61429f6b6e4e92933e39f67df6ec8a58ecba4` |
| 빌드 단계 | `Build with Gradle` 성공 |
| 패키징 단계 | `Prepare Deploy Package` 성공 |
| 배포 단계 | `Deploy to Elastic Beanstalk` 성공 |
| 테스트 실행 | 빌드 명령의 `-x test`로 제외 |

- [배포 워크플로](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/40a127ecd8baf8b110a438e9ef1a858a0bf6e77b/.github/workflows/deploy-backend.yml)
- [Procfile](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/40a127ecd8baf8b110a438e9ef1a858a0bf6e77b/backend/Procfile)

이 기록은 해당 실행의 단계 성공을 뒷받침합니다. 현재 AWS 리소스가 가동 중이거나 모든 사용자 기능이 통과했다는 증거로 확대하지 않습니다. 기존 포트폴리오에 기록된 데모 운영 종료 상태를 유지했습니다.

프론트엔드의 실제 Amplify 배포 경험과 담당 범위도 기존 문서에서 유지했습니다. 이번에 제시한 빌드 YAML은 현재 Vite 소스·공식 Amplify 형식에 맞춘 재현 예시입니다.

## 5. 기능 설명을 확인한 주요 코드

| 강점 | 근거 |
| --- | --- |
| Google·Kakao·Naver 소셜 로그인 | [SocialAuthService](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/40a127ecd8baf8b110a438e9ef1a858a0bf6e77b/backend/src/main/java/com/woth/backend/auth/SocialAuthService.java), [AuthController](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/40a127ecd8baf8b110a438e9ef1a858a0bf6e77b/backend/src/main/java/com/woth/backend/auth/AuthController.java) |
| 개인 기억 저장 암호화 | [AesGcmTextEncryptor](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/40a127ecd8baf8b110a438e9ef1a858a0bf6e77b/backend/src/main/java/com/woth/backend/global/crypto/AesGcmTextEncryptor.java), [PrivateMemory](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/40a127ecd8baf8b110a438e9ef1a858a0bf6e77b/backend/src/main/java/com/woth/backend/memory/PrivateMemory.java) |
| AI 날씨 분석 | [EmotionAnalysisResponse](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/40a127ecd8baf8b110a438e9ef1a858a0bf6e77b/backend/src/main/java/com/woth/backend/ai/dto/EmotionAnalysisResponse.java), [AiResponseParser](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/40a127ecd8baf8b110a438e9ef1a858a0bf6e77b/backend/src/main/java/com/woth/backend/ai/service/AiResponseParser.java), [MemoryService](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/40a127ecd8baf8b110a438e9ef1a858a0bf6e77b/backend/src/main/java/com/woth/backend/memory/MemoryService.java) |
| 광장 완료·비동기 처리 | [PlazaCompletionService](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/40a127ecd8baf8b110a438e9ef1a858a0bf6e77b/backend/src/main/java/com/woth/backend/plaza/PlazaCompletionService.java) |
| AI 요청·조건부 재시도 | [OpenAiClient](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/40a127ecd8baf8b110a438e9ef1a858a0bf6e77b/backend/src/main/java/com/woth/backend/ai/service/OpenAiClient.java) |
| 이미지 보관·편지함 | [S3ImageStorageService](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/40a127ecd8baf8b110a438e9ef1a858a0bf6e77b/backend/src/main/java/com/woth/backend/storage/S3ImageStorageService.java), [MailboxService](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/40a127ecd8baf8b110a438e9ef1a858a0bf6e77b/backend/src/main/java/com/woth/backend/mailbox/MailboxService.java) |
| React·Vite 실행·배포 | [package.json](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/40a127ecd8baf8b110a438e9ef1a858a0bf6e77b/frontend/package.json), [package-lock.json](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/40a127ecd8baf8b110a438e9ef1a858a0bf6e77b/frontend/package-lock.json), [vite.config.ts](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/40a127ecd8baf8b110a438e9ef1a858a0bf6e77b/frontend/vite.config.ts) |

## 6. 검증 내용

- 최신 원본과 포트폴리오의 `main`을 각각 읽어 기준을 구분했습니다.
- 실행 안내와 API 경로를 실제 설정·컨트롤러의 선언과 대조했습니다.
- 문서 안의 상대 경로·이미지·제목 연결을 점검했습니다.
- 기존 수상 증빙, 대표 기여 커밋, 개인·팀 기여 구분을 유지했습니다.
- 수정 범위는 포트폴리오의 문서입니다. 원본 팀 저장소 코드와 AWS 운영 설정은 변경하지 않았습니다.
- 자동 테스트·전체 빌드·현재 AWS 연결을 실행 검증했다고 표시하지 않았습니다.
