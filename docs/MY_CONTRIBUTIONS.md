# 김준영 담당 기능 상세

이 문서는 `마음의 날씨` 4인 팀 프로젝트에서 김준영이 담당한 Java 백엔드, Elastic Beanstalk 백엔드 배포와 AWS Amplify 프론트엔드 배포 작업을 코드·커밋 기록과 사용자 제공 정보를 기준으로 정리합니다.

- 원본 저장소: [guddlrdl123/WeatherOfTheHeart-](https://github.com/guddlrdl123/WeatherOfTheHeart-)
- 분석 기준 브랜치: `main`
- 분석 기준일: 2026년 7월 17일
- 이전 GitHub 사용자 이름: `bamkkayo`
- 확인된 Git 작성자: `bamkkayo <junyeong2107@gmail.com>`

## 근거와 작성 원칙

담당 범위는 다음 세 종류의 근거를 구분해 판단했습니다.

| 근거 | 사용 방법 |
| --- | --- |
| Git 작성자·커미터 | `bamkkayo <junyeong2107@gmail.com>`로 확인되는 직접 커밋을 1차 근거로 사용 |
| 코드 이력 | 파일별 `git log --follow`, 현재 코드의 `git blame`, 커밋별 변경 파일을 대조 |
| 사용자 제공 정보 | GitHub Actions, YAML 설정과 환경변수 구성은 김준영이 작업했다는 정보를 담당 근거로 사용 |

환경변수는 **구성 방식과 변수 이름의 역할만 설명**하며, 실제 API 키·비밀번호·AWS 키·DB 주소·버킷의 민감한 값은 기록하지 않습니다.

## 담당 범위 요약

1. 광장 참여 데이터 엔티티와 JPA 조회 구조
2. 참여 글 저장 이후 광장 완료 여부를 확인하는 이벤트
3. 완료 조건 검사와 AI 최종 이미지 생성·편지 발송의 비동기 처리
4. 광장 오브젝트·감정·위치를 이미지 생성 프롬프트로 변환
5. AI 결과 이미지의 AWS S3 영속 저장
6. GitHub Actions 기반 AWS Elastic Beanstalk 백엔드 배포
7. AWS Amplify 프론트엔드 배포
8. `application.yaml`, 워크플로 YAML과 환경변수 구성
9. 서버 포트, Procfile, Actuator와 헬스 체크 구성
10. AWS RDS MySQL 연결을 위한 배포 환경변수와 운영 연결 점검

## 현재 코드에서 확인되는 기여 흔적

아래 숫자는 2026년 7월 17일 원본 `main`의 `git blame` 기준으로 `bamkkayo` 작성자가 남아 있는 줄 수입니다. 작업량을 단순 비교하는 지표가 아니라, 현재 코드에 직접 작성 흔적이 남아 있음을 확인하는 참고 자료입니다.

| 파일 | `bamkkayo` 작성 줄 | 의미 |
| --- | ---: | --- |
| [`PlazaEntry.java`](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/main/backend/src/main/java/com/woth/backend/plaza/PlazaEntry.java) | 75 | 광장 참여 데이터 모델 |
| [`PlazaRepository.java`](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/main/backend/src/main/java/com/woth/backend/plaza/PlazaRepository.java) | 27 | 광장 조회·완료 상태 저장 기반 |
| [`PlazaEntryRepository.java`](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/main/backend/src/main/java/com/woth/backend/plaza/PlazaEntryRepository.java) | 17 | 광장 참여·중복 검사 조회 기반 |
| [`PlazaEntryCreatedEvent.java`](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/main/backend/src/main/java/com/woth/backend/plaza/PlazaEntryCreatedEvent.java) | 4 | 저장 이후 완료 검사 이벤트 |
| [`PlazaCompletionService.java`](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/main/backend/src/main/java/com/woth/backend/plaza/PlazaCompletionService.java) | 149 | 완료 조건·AI 이미지·편지 처리 핵심 |
| [`PlazaImagePromptBuilder.java`](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/main/backend/src/main/java/com/woth/backend/plaza/PlazaImagePromptBuilder.java) | 231 | 광장 이미지 프롬프트의 중심 구현 |
| [`S3ImageStorageService.java`](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/main/backend/src/main/java/com/woth/backend/storage/S3ImageStorageService.java) | 68 | S3 이미지 업로드·다운로드 기반 |
| [`S3Config.java`](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/main/backend/src/main/java/com/woth/backend/config/S3Config.java) | 20 | AWS SDK S3 클라이언트 구성 |
| [`HealthCheckController.java`](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/main/backend/src/main/java/com/woth/backend/global/health/HealthCheckController.java) | 23 | `/`, `/health` 상태 응답 |
| [`deploy-backend.yml`](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/main/.github/workflows/deploy-backend.yml) | 56 | 전체 백엔드 자동 배포 워크플로 |

YAML·환경변수 설정과 AWS Amplify 프론트엔드 배포는 사용자 제공 정보상 김준영 담당입니다. `application.yaml`은 이후 팀 기능이 추가되면서 여러 작성자의 변경이 섞였지만, 배포를 위한 서버 포트, DB·OpenAI·메일·AWS 설정의 환경변수화와 워크플로 구성은 김준영 담당으로 정리했습니다. 전체 Git 이력에는 Amplify 설정 파일이 없어 프론트엔드 배포의 세부 자동화 방식은 단정하지 않습니다.

## 1. 광장 참여 데이터 처리

### 목적

사용자가 특정 광장에 남긴 감정 기록과 선택한 오브젝트를 하나의 참여 단위로 저장하고, 광장별 조회와 중복 참여 제한에 활용합니다.

### 관련 코드

- `backend/.../plaza/PlazaEntry.java`
- `backend/.../plaza/PlazaRepository.java`
- `backend/.../plaza/PlazaEntryRepository.java`
- `backend/.../plaza/PlazaService.java`

### 데이터 구조

`PlazaEntry`는 다음 정보를 보존합니다.

- 참여한 광장과 작성자
- 제목과 감정 기록 본문
- 감정 키와 날씨 키
- 오브젝트 키와 슬롯 키
- 오브젝트 X·Y 위치와 레이어 순서
- 생성·수정 시각

### 요청 처리 흐름

1. 광장과 사용자가 실제로 존재하는지 확인합니다.
2. 완료된 광장인지 검사합니다.
3. 일반 사용자의 광장 중복 참여 여부를 검사합니다.
4. 광장 설정에 따라 같은 오브젝트의 중복 사용 여부를 검사합니다.
5. 현재 참여 수가 광장의 `maxObjects`에 도달했는지 검사합니다.
6. 감정·날씨·오브젝트·위치 정보를 `PlazaEntry`로 저장합니다.
7. 저장 트랜잭션 안에서 완료 검사 이벤트를 발행합니다.

현재 서비스의 핵심 검증은 다음과 같습니다.

```java
if (!adminEntryOwner
        && plazaEntryRepository.existsByPlazaIdAndOwnerId(plazaId, owner.getId())) {
    throw new CustomException(ErrorCode.PLAZA_ALREADY_JOINED);
}

if (!adminEntryOwner
        && !plaza.getAllowDuplicateObjects()
        && plazaEntryRepository.existsByPlazaIdAndObjectKey(plazaId, request.objectKey())) {
    throw new CustomException(ErrorCode.PLAZA_DUPLICATE_OBJECT);
}

List<PlazaEntry> entries = plazaEntryRepository.findByPlazaId(plazaId);
if (entries.size() >= plaza.getMaxObjects()) {
    throw new CustomException(ErrorCode.PLAZA_COMPLETE);
}
```

### 구현 결과

광장 참여 데이터가 완료 조건 검사와 AI 프롬프트 생성에 동일한 원천 데이터로 사용되도록 연결했습니다.

## 2. 광장 완료 조건 검사

광장의 완료 기준은 고정된 하나의 숫자가 아니라 각 광장의 `maxObjects` 값입니다. `Plaza` 엔티티의 기본값은 8이며, 실제 광장 생성 요청에서 다른 값이 저장될 수 있습니다.

완료 서비스는 다음 조건을 검사합니다.

- 광장이 존재하는가
- 이미 완료된 광장이 아닌가
- 참여 글 수가 `maxObjects` 이상인가
- 방장이 직접 종료한 경우 `forceComplete`로 같은 완료 흐름을 실행하는가
- `imageGenerating` 상태로 같은 이미지 생성 작업이 중복 실행되지 않는가

`PlazaCompletionService`는 완료 시점에 광장 정보, 참여 데이터, 프롬프트와 편지 수신자 정보를 하나의 `CompletionSnapshot`으로 확정합니다. 지연 로딩 대상이 비동기 작업 밖에서 뒤늦게 조회되는 문제를 피하기 위해 필요한 정보를 트랜잭션 안에서 읽습니다.

## 3. 트랜잭션 완료 이후 이벤트 처리

### 문제

이벤트를 발행했다고 해서 참여 데이터의 DB 커밋까지 끝난 것은 아닙니다. 완료 검사가 먼저 실행되면 방금 저장한 참여 글이 조회되지 않아 완료 조건을 잘못 판단할 수 있습니다.

### 구현

```java
@Async
@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
public void completeIfReady(PlazaEntryCreatedEvent event) {
    CompletionSnapshot snapshot = loadCompletionSnapshot(
            event.plazaId(),
            event.forceComplete()
    );

    if (snapshot == null) {
        return;
    }

    // AI 생성, S3 저장, 편지 발송
}
```

- `@TransactionalEventListener(phase = AFTER_COMMIT)`: 참여 글 저장 커밋 이후 실행
- `@Async`: 외부 AI 호출이 사용자 저장 요청을 오래 점유하지 않게 분리
- `TransactionTemplate` + `PROPAGATION_REQUIRES_NEW`: 완료 스냅샷 조회와 상태 변경을 별도 트랜잭션으로 실행
- `@EnableAsync`: Spring Boot 애플리케이션에서 비동기 실행 활성화

### 결과

핵심 데이터 저장의 정합성을 먼저 확보하고, 시간이 오래 걸리거나 실패할 수 있는 AI·스토리지 작업을 후속 흐름으로 분리했습니다.

## 4. AI 최종 이미지 생성

### 입력 데이터

- 광장 제목과 주제
- 배경 유형, 색상과 날씨 키
- 참여 글의 감정·날씨·오브젝트
- 오브젝트 X·Y 위치
- 감정 기록의 요약 텍스트

### 프롬프트 구성 규칙

- `MAX_OBJECTS_FOR_PROMPT = 30`으로 입력 크기를 제한합니다.
- 0~1 비율, 0~100 백분율, 픽셀 좌표를 모두 받아 대략적인 상·중·하/좌·중·우 위치로 변환합니다.
- 사용자 이름, 얼굴, 로고, UI, 워터마크와 읽을 수 있는 텍스트 생성을 제한합니다.
- 숫자·영문 흔적이 이미지에 생기는 문제를 줄이기 위해 제목·주제·메모를 정규화합니다.
- 동물 오브젝트의 신체가 왜곡되거나 다른 오브젝트와 합쳐지지 않도록 프롬프트 규칙을 보강했습니다.

현재 코드에는 오브젝트를 `MAIN`과 `SUPPORTING`으로 분리하는 상수나 구조가 없습니다. 확인되지 않은 분류는 담당 내용에 포함하지 않았습니다.

### 대표 개선 이력

`bamkkayo` 명의로 2026년 6월 16~17일 사이 `PlazaImagePromptBuilder`를 반복 개선한 커밋이 13개 확인됩니다. 글자·숫자 흔적 감소, 위치 반영, 날씨 충돌 완화와 표현 강도 조정 같은 실험을 수행한 이력입니다.

## 5. 편지함 결과 전달

완료 서비스는 광장 참여자 목록을 수집해 다음 정보를 편지함 서비스로 전달합니다.

- 광장 ID와 제목
- 광장 생성·완료 시각
- 참여 인원
- 참여자별 오브젝트 키, 제목과 감정 기록
- AI 결과 이미지 data URL 또는 S3 URL

`LetterRepository.existsByReceiverIdAndPlazaId`로 같은 광장의 결과가 같은 참여자에게 중복 발송되는 것을 막습니다. `isRead` 상태는 편지 조회와 별도로 변경합니다.

초기 완료·발송 흐름은 김준영 커밋에서 구현됐고, 현재 편지 데이터의 세부 정보와 다운로드 기능에는 팀원의 후속 개선이 함께 반영돼 있습니다.

## 6. AWS S3 이미지 저장

### 선택 이유

Elastic Beanstalk 인스턴스의 로컬 디스크는 재배포나 인스턴스 교체 시 영속성을 보장하지 않습니다. 사용자가 나중에도 편지함에서 이미지를 확인하려면 애플리케이션 인스턴스 밖에 저장해야 했습니다.

### 처리 흐름

1. OpenAI 응답의 이미지 data URL을 MIME 타입과 Base64 본문으로 분리합니다.
2. PNG·JPEG·WebP만 허용합니다.
3. `plazas/{plazaId}/{UUID}.{ext}` 키를 생성합니다.
4. AWS SDK의 `S3Client.putObject`로 업로드합니다.
5. 환경변수에서 읽은 공개 기준 URL과 키를 조합합니다.
6. 결과 URL을 광장 완료 편지 데이터에 연결합니다.

### 설정 원칙

- AWS 리전, 버킷과 공개 기준 URL은 YAML에서 환경변수로 참조합니다.
- SDK는 기본 자격 증명 공급자 체인을 사용하도록 정적 키를 코드에 넣지 않았습니다.
- GitHub Actions의 AWS 자격 증명은 GitHub Secrets에서 주입합니다.
- 실제 버킷명, 액세스 키와 계정 번호는 이 포트폴리오에 노출하지 않습니다.

## 7. AWS 배포

김준영은 GitHub Actions, YAML, 환경변수 구성과 백엔드·프론트엔드 배포를 담당했습니다.

### 백엔드: AWS Elastic Beanstalk

### 배포 산출물

- Gradle 실행 JAR을 `application.jar`로 복사
- `backend/Procfile`의 `web: java -jar application.jar` 사용
- JAR과 Procfile을 `deploy.zip`으로 패키징
- Elastic Beanstalk 애플리케이션 버전 생성 및 환경 배포

### 운영 설정

- Spring Boot 기본 포트: `5000`
- DB·OpenAI·메일·OAuth·AWS 설정: 환경변수 참조
- 민감값: GitHub Secrets 또는 Elastic Beanstalk 환경 속성에서 관리
- 서버 상태 확인: `GET /`, `GET /health`

### 프론트엔드: AWS Amplify

- React 프론트엔드의 운영 배포에 AWS Amplify 사용
- 사용자 제공 정보상 김준영 담당
- Amplify는 AWS 콘솔에서 저장소와 빌드 설정을 연결할 수 있어 별도 YAML이 꼭 필요하지 않음
- 전체 Git 이력에는 Amplify 설정 파일이 없어 서비스 선택과 담당 사실만 기록
- 프로젝트 종료 후 유지 비용 절감을 위해 AWS 배포 리소스 정리

## 8. GitHub Actions 백엔드 자동 배포

현재 `.github/workflows/deploy-backend.yml`의 동작은 다음과 같습니다.

| 항목 | 설정 |
| --- | --- |
| 실행 조건 | `main` 브랜치 push, 수동 `workflow_dispatch` |
| 경로 조건 | `backend/**`, `.github/workflows/**` |
| 실행 환경 | `ubuntu-latest` |
| Java | Amazon Corretto 21 |
| 빌드 | `./gradlew clean build -x test` |
| 패키지 | `application.jar`, `Procfile`, `deploy.zip` |
| 배포 | `einaregilsson/beanstalk-deploy@v21` |
| 비밀값 | `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION` GitHub Secrets |

워크플로는 초기에 추가한 뒤 Node.js Actions 호환성, JAR 탐색, 패키지 경로, 버전 라벨과 재배포 동작을 여러 커밋으로 조정했습니다.

## 9. AWS RDS MySQL 연결과 환경변수

코드는 JDBC 연결을 `DB_URL`, `DB_USERNAME`, `DB_PASSWORD`로 받습니다. 같은 Spring Boot 애플리케이션이 로컬 MySQL과 AWS RDS에서 동작하도록 실제 호스트·계정·비밀번호를 저장소 밖으로 분리했습니다.

배포 환경에서 확인해야 했던 항목은 다음과 같습니다.

- JDBC URL과 MySQL 드라이버
- RDS 보안 그룹의 3306 접근 범위
- Elastic Beanstalk에서 RDS까지의 네트워크 연결
- 환경변수 이름과 애플리케이션 YAML 매핑
- 자격 증명 누락 또는 잘못된 값으로 인한 시작 실패

실제 호스트 주소와 계정 정보는 공개하지 않습니다.

## 10. 헬스 체크

`HealthCheckController`는 `GET /`와 `GET /health`에서 문자열 `OK`와 HTTP 200을 반환합니다.

이 엔드포인트는 다음 용도로 사용됩니다.

- Elastic Beanstalk·ELB가 애플리케이션 프로세스의 HTTP 응답을 확인
- 배포 직후 서버가 5000 포트에서 정상 기동했는지 확인
- GitHub Actions 배포 후 기본 상태 점검

현재 컨트롤러는 DB나 OpenAI·S3의 상태까지 검사하는 심층 헬스 체크가 아니라, 웹 애플리케이션이 요청에 응답하는지를 확인하는 가벼운 체크입니다.

## 11. 관리자 신고와 사용자 제재 기능

신고, 블라인드, 경고 편지와 사용자 정지 기능은 프로젝트 전체 기능으로 확인됩니다. 그러나 대표 구현 커밋은 `Change03`와 `eunjung3` 작성자로 기록돼 있어 김준영의 직접 구현 기능으로 표시하지 않았습니다.

김준영이 만든 `PlazaEntry`, 광장 조회와 완료 흐름은 해당 기능이 사용하는 도메인 기반 중 일부지만, 이를 근거로 관리자 기능 전체를 개인 기여로 확대하지 않습니다.

## 문제 해결 정리

| 문제 | 원인 | 해결 | 결과 |
| --- | --- | --- | --- |
| 참여 요청이 AI 생성 시간만큼 지연 | 데이터 저장과 외부 API 호출이 같은 흐름 | AFTER_COMMIT 이벤트와 비동기 처리 | 핵심 저장과 부가 작업 분리 |
| 커밋 전 완료 검사 가능성 | 이벤트 발행과 DB 커밋 시점 차이 | 트랜잭션 이벤트와 새 트랜잭션 스냅샷 | 최신 참여 데이터를 기준으로 완료 판단 |
| 재배포 시 로컬 이미지 유실 | EB 인스턴스 디스크의 비영속성 | S3 업로드와 URL 저장 | 애플리케이션과 이미지 수명 분리 |
| 배포 환경에서 서버 상태 불명확 | 포트·패키지·환경변수·외부 연결 차이 | Procfile, port 5000, 환경변수화, 헬스 체크 | 반복 가능한 배포와 상태 확인 지점 확보 |
| 생성 이미지에 글자·위치 왜곡 | 생성 모델의 프롬프트 해석 편차 | 텍스트 정규화, 위치 힌트, 금지 규칙 반복 개선 | 프로젝트 의도에 가까운 결과를 얻기 위한 제어 강화 |

## 대표 직접 커밋

| 커밋 | 작성자 | 변경 내용 |
| --- | --- | --- |
| [8d5467f](https://github.com/guddlrdl123/WeatherOfTheHeart-/commit/8d5467fe6c70086af266b41afc9ba099c232a756) | bamkkayo | 광장 참여 데이터 엔티티 추가 |
| [eb37a1a](https://github.com/guddlrdl123/WeatherOfTheHeart-/commit/eb37a1ab99d849a7dde68d91e42dd554e99ce3a2) | bamkkayo | 광장별 조회·중복 참여 검사 저장소 추가 |
| [90cb2be](https://github.com/guddlrdl123/WeatherOfTheHeart-/commit/90cb2be9b58229a305c98503acf333329037317c) | bamkkayo | 저장 커밋 이후 완료 검사 이벤트 추가 |
| [0aa3629](https://github.com/guddlrdl123/WeatherOfTheHeart-/commit/0aa36292477c776b455e74a57a92fb9a875e92a6) | bamkkayo | 광장 참여 데이터 기반 이미지 프롬프트 추가 |
| [31e75ae](https://github.com/guddlrdl123/WeatherOfTheHeart-/commit/31e75ae2b16eb22a966e8bd8ffa33ed6919d8932) | bamkkayo | 완료 조건·AI 이미지·발송 비동기 로직 추가 |
| [6695eed](https://github.com/guddlrdl123/WeatherOfTheHeart-/commit/6695eed8084033ac4d012ce77f4d21c1d7188b54) | bamkkayo | Elastic Beanstalk 배포 워크플로 추가 |
| [098643b](https://github.com/guddlrdl123/WeatherOfTheHeart-/commit/098643bcedde1b4d63c45f85cdd08ecb2896ebd5) | bamkkayo | Spring Boot 서버 포트 5000 설정 |
| [2e23cf1](https://github.com/guddlrdl123/WeatherOfTheHeart-/commit/2e23cf18935c7294275b866c177eb4528411068c) | bamkkayo | S3 클라이언트와 이미지 저장 서비스 추가 |
| [1f39b44](https://github.com/guddlrdl123/WeatherOfTheHeart-/commit/1f39b4498936033e7c88e2769ba3a21e4c0cb609) | bamkkayo | 광장 완료 이미지의 S3 저장 연동 |
| [30d7d38](https://github.com/guddlrdl123/WeatherOfTheHeart-/commit/30d7d386b531b657ac05bea964a812b4440050e6) | bamkkayo | `/`·`/health` 헬스 체크 추가 |

## 커밋 귀속과 팀 후속 개선

사용자 요청에 따라 `bamkkayo` 이름만 검색하지 않고 전체 335개 커밋을 검사했습니다.

### 전체 검사 결과

- `main`과 `--all`의 도달 가능한 커밋 수가 모두 335개로, 별도 원격 브랜치에만 숨은 커밋은 없었습니다.
- Git 작성자 식별자는 `bamkkayo`, `guddlrdl123`, `Change03`, `eunjung3` 네 종류입니다.
- `Co-authored-by` 트레일러는 9개 확인됐으며 모두 `Claude Opus 4.8 <noreply@anthropic.com>`입니다. 김준영을 공동 작성자로 표시한 트레일러는 없어 개인 기여 커밋 수에 더하지 않았습니다.
- `bamkkayo <junyeong2107@gmail.com>` 직접 작성 커밋은 63개입니다.
- 그중 3개는 GitHub 웹에서 반영돼 커미터가 `GitHub <noreply@github.com>`로 기록됐지만 작성자는 여전히 `bamkkayo`입니다.
- 다른 세 작성자 이름의 커밋을 김준영 개인 작업이라고 증명할 메타데이터는 없어 직접 기여로 귀속하지 않았습니다.

### 같은 기능선의 팀 후속 커밋

아래 커밋은 김준영이 만든 핵심 파일을 이후 팀원이 수정한 기록입니다. 현재 최종 코드 설명에는 반영하되, 김준영의 대표 직접 커밋 목록에는 포함하지 않았습니다.

| 커밋 | 기록된 작성자 | 후속 변경 |
| --- | --- | --- |
| [55944b8](https://github.com/guddlrdl123/WeatherOfTheHeart-/commit/55944b8) | guddlrdl123 | 광장 트랜잭션 흐름 보완 |
| [89efa7e](https://github.com/guddlrdl123/WeatherOfTheHeart-/commit/89efa7e) | guddlrdl123 | 방장 임의 종료 이미지 발송 추가 |
| [d79c72f](https://github.com/guddlrdl123/WeatherOfTheHeart-/commit/d79c72f) | eunjung3 | 광장과 편지함 알림 연동 개선 |
| [9325091](https://github.com/guddlrdl123/WeatherOfTheHeart-/commit/9325091) | eunjung3 | 완성 편지 상세 정보 개선 |
| [e578782](https://github.com/guddlrdl123/WeatherOfTheHeart-/commit/e578782) | Change03 | 광장 삭제·이미지 생성 잠금 흐름 보완 |
| [f59284d](https://github.com/guddlrdl123/WeatherOfTheHeart-/commit/f59284d) | Change03 | AI 이미지 스타일 프롬프트 보완 |
| [f25528b](https://github.com/guddlrdl123/WeatherOfTheHeart-/commit/f25528b) | guddlrdl123 | 최종 이미지 프롬프트 추가 조정 |

이 구분을 통해 현재 코드의 팀 공동 발전 과정은 설명하면서도, 확인되지 않은 타 계정 커밋을 개인 성과로 과장하지 않았습니다.
