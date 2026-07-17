# 마음의 날씨

> 말하지 못한 감정을 기록하면 AI가 감정을 분석해 날씨와 오브젝트로 표현하고, 여러 사람의 기록을 하나의 시각적 결과물로 완성하는 감정 기록·공유 서비스입니다.

## 주요 성과

- 2026년 7월 16일 **SW 잡브릿지-DAY 통합프로젝트 발표회**에서 `마음의 날씨` 프로젝트로 **2026 K-디지털트레이닝 벤처·스타트업 유형 우수상**을 수상했습니다.
- 관련 과정: **고용노동부 K-디지털트레이닝(벤처·스타트업 유형)**
- 주관·수여: **(사)한국경영혁신중소기업협회(MAINBiz)**
- 4인 팀 프로젝트로 기획, 개발, 배포 및 발표를 완료했습니다.
- 김준영은 팀에서 **Java 백엔드 개발, GitHub Actions·YAML·환경변수 구성과 AWS 배포**를 담당했습니다.
- 광장 참여 데이터가 완료 조건을 충족하면 AI 최종 이미지를 생성하고 참여자의 편지함으로 전달하는 흐름을 구현했습니다.

> 수상은 개인 단독 수상이 아닌 4인 팀 프로젝트의 성과입니다.

## 프로젝트 개요

`마음의 날씨`는 감정을 텍스트 목록으로만 남기는 대신, AI 분석 결과를 날씨와 오브젝트로 시각화하는 서비스입니다.

- **개인 방**: 자신의 감정을 기록하고, 추천받은 날씨와 오브젝트를 배치해 날짜별로 돌아봅니다.
- **광장**: 여러 사용자의 감정과 오브젝트가 하나의 공간에 쌓입니다.
- **광장 완성**: 설정된 참여 조건을 충족하거나 방장이 종료하면 참여 데이터를 모아 AI 최종 이미지를 생성합니다.
- **편지함**: 완성 이미지와 참여 당시 기록을 각 참여자에게 전달합니다.

공개 반응 수치를 중심으로 하는 SNS보다, 함께 만든 공간의 분위기와 시각적 표현을 통해 감정을 공유하는 데 초점을 맞췄습니다.

## 프로젝트 정보

| 항목 | 내용 |
| --- | --- |
| 프로젝트명 | 마음의 날씨 (Weather of the Heart) |
| 프로젝트 형태 | 4인 팀 프로젝트 |
| 개발 기간 | 2026.05 ~ 2026.07 |
| 김준영 담당 | Java 백엔드 개발, 광장 핵심 로직, AI 이미지 생성 연동, AWS 이미지 저장, GitHub Actions·YAML·환경변수 구성 및 백엔드 배포 |
| 수상 내역 | SW 잡브릿지-DAY 통합프로젝트 발표회, 2026 K-디지털트레이닝 벤처·스타트업 유형 우수상 (고용노동부 관련 과정, 한국경영혁신중소기업협회 주관·수여, 2026.07.16) |
| Frontend | React 19, TypeScript, Vite, React Router, Tailwind CSS |
| Backend | Java 21, Spring Boot 4, Spring Web MVC, Spring Data JPA |
| Database | MySQL, AWS RDS for MySQL(운영 구성) |
| AI | OpenAI API, LangChain4j, WebClient |
| Infrastructure | AWS Elastic Beanstalk, AWS S3, AWS RDS |
| CI/CD | GitHub Actions, Gradle |
| 협업 도구 | Git, GitHub |

## 기획 배경과 해결하려는 문제

감정 기록은 꾸준히 이어가기 어렵고, 텍스트만 쌓이면 과거의 분위기를 직관적으로 되짚기 어렵습니다. 반대로 일반 SNS는 공개 반응과 비교에 초점이 맞춰져 있어 말하기 어려운 감정을 편안하게 남기기에는 부담이 될 수 있습니다.

이 프로젝트는 다음 방식으로 문제를 풀었습니다.

1. 사용자가 작성한 감정을 AI가 감정·날씨·오브젝트 정보로 구조화합니다.
2. 개인 기록을 오브젝트와 날씨가 있는 방으로 시각화합니다.
3. 광장에서는 여러 사용자의 기록을 하나의 공간에 축적합니다.
4. 광장이 완료되면 참여 데이터를 하나의 이미지로 재해석해 공동 결과물로 남깁니다.
5. 완성 결과를 참여자의 편지함으로 전달해 기록을 다시 확인할 수 있게 합니다.

## 주요 사용자 흐름

```mermaid
flowchart LR
    subgraph PrivateRoom["개인 방 흐름"]
        P1["로그인"] --> P2["감정 기록 작성"]
        P2 --> P3["AI 감정 분석"]
        P3 --> P4["날씨·오브젝트 추천"]
        P4 --> P5["개인 방에 오브젝트 배치"]
        P5 --> P6["날짜별 기록 조회"]
    end

    subgraph PlazaFlow["광장 흐름"]
        G1["광장 생성 또는 입장"] --> G2["감정·오브젝트 등록"]
        G2 --> G3["여러 사용자의 기록 축적"]
        G3 --> G4["완료 조건 확인"]
        G4 --> G5["AI 최종 이미지 생성"]
        G5 --> G6["AWS S3 저장"]
        G6 --> G7["참여자 편지함으로 전달"]
    end
```

## 주요 기능

| 영역 | 기능 |
| --- | --- |
| 인증 | 이메일 회원가입·인증, 로그인, 비밀번호 재설정, Google·Kakao·Naver 소셜 로그인 |
| 개인 방 | 기억 작성·조회·수정·삭제, 감정 분석, 날씨·오브젝트 추천 및 배치, 날짜별 조회 |
| 광장 | 광장 생성·입장, 글 작성·수정·삭제, 위치 저장, 좋아요, 신고, 완료 처리 |
| AI 결과 | 광장 감정·오브젝트 수집, 프롬프트 생성, 최종 이미지 생성, S3 저장 |
| 편지함 | 광장 완성 결과 전달, 읽음 상태, 전체 읽음, 이미지 다운로드 |
| 사용자 | 마이페이지, 프로필 수정, 활동 내역, 회원 탈퇴 |
| 운영 | 공지사항, 1:1 문의, 신고 내역 확인, 글 블라인드, 경고, 사용자 정지 |

## 전체 시스템 구조

```mermaid
flowchart LR
    User["사용자"] --> Frontend["React + TypeScript Frontend"]
    Frontend -->|"REST API"| Backend["Spring Boot Backend"]

    Backend --> Database["MySQL / AWS RDS"]
    Backend --> OpenAI["OpenAI API"]
    Backend --> S3["AWS S3"]
    Backend --> Mail["SMTP 이메일 발송"]
    Backend --> OAuth["Google · Kakao · Naver OAuth"]

    GitHub["GitHub main branch"] --> Actions["GitHub Actions"]
    Actions --> EB["AWS Elastic Beanstalk"]
    EB --> Backend
```

## 구성 요소별 역할

| 구성 요소 | 역할 |
| --- | --- |
| React Frontend | 개인 방·광장·편지함·관리 화면과 사용자 상호작용 처리 |
| Spring Boot Backend | 인증, 기억, 광장, AI 연동, 편지함, 관리자 도메인의 REST API와 비즈니스 로직 처리 |
| MySQL / AWS RDS | 사용자, 기억, 광장, 참여 글, 편지 데이터 저장 |
| OpenAI API | 감정 분석 및 광장 최종 이미지 생성 |
| AWS S3 | AI 결과 이미지와 서비스 이미지의 영속 저장 |
| AWS Elastic Beanstalk | Spring Boot 백엔드 운영 환경 |
| GitHub Actions | main 브랜치의 백엔드 변경을 빌드하고 Elastic Beanstalk에 배포 |
| SMTP / OAuth 제공자 | 이메일 인증·재설정 메일과 소셜 로그인 처리 |

## 팀 구성과 역할

4명이 함께 기획, 프론트엔드, 백엔드, 배포와 발표를 수행했습니다. 원본 저장소에서 팀원별 세부 역할을 모두 확정할 수 없어 이름별 역할은 임의로 작성하지 않았습니다.

| 구분 | 역할 |
| --- | --- |
| 팀 전체 | 서비스 기획, 기능 개발, 통합, 테스트, 배포, 발표 |
| 김준영 | Java 백엔드 개발, 광장 핵심 로직, AI 이미지 생성 연동, AWS 이미지 저장, 백엔드 배포 및 운영 오류 대응 |

## 김준영의 담당 영역

```mermaid
flowchart TB
    API["Spring Boot 광장 API"] --> Entry["광장 참여 데이터 저장"]
    Entry --> Event["트랜잭션 완료 이벤트"]
    Event --> Check["광장 완료 조건 검사"]
    Check --> Prompt["AI 이미지 프롬프트 구성"]
    Prompt --> Generate["OpenAI 이미지 생성 요청"]
    Generate --> S3["AWS S3 결과 저장"]
    S3 --> Letter["참여자 편지함 전달"]

    Actions["GitHub Actions"] --> Build["Gradle Build"]
    Build --> EB["AWS Elastic Beanstalk 배포"]
    EB --> Health["GET / · GET /health"]
    EB --> RDS["AWS RDS MySQL"]
```

- Java 백엔드 개발
- 광장 참여 데이터와 완료 조건 처리
- 트랜잭션 커밋 이후 비동기 완료 이벤트 처리
- 광장 데이터 기반 AI 최종 이미지 프롬프트 구성
- AI 결과 이미지의 AWS S3 저장 연동
- GitHub Actions 워크플로, 애플리케이션 YAML과 환경변수 구성
- GitHub Actions 기반 Elastic Beanstalk 자동 배포
- 서버 포트와 헬스 체크 구성, 배포 오류 대응

프론트엔드는 전체 시스템의 구성 요소이지만 김준영의 직접 담당 영역으로 표시하지 않았습니다.

## 김준영이 직접 구현한 기능

| 기능 | 구현 내용 | 코드·커밋 근거 |
| --- | --- | --- |
| 광장 참여 데이터 모델 | `PlazaEntry`에 사용자, 광장, 감정, 날씨, 오브젝트, 위치·레이어 정보를 저장 | `8d5467f`, `859be55` |
| 광장 JPA 조회 | 광장별 참여 글, 사용자별 참여 글, 중복 참여·오브젝트 검사 메서드 구성 | `6a78c2e`, `eb37a1a` |
| 완료 이벤트 | 참여 글 저장 트랜잭션 이후 완료 여부를 확인하는 이벤트 도입 | `90cb2be` |
| AI 최종 이미지 | 참여 오브젝트·감정·위치를 AI 요청용 프롬프트로 구성 | `0aa3629`, `faf0209` 등 |
| 비동기 완료 처리 | 완료 조건 검사, AI 이미지 생성, 참여자 편지 발송을 요청 흐름과 분리 | `31e75ae` |
| S3 저장 | AWS SDK 의존성, S3 클라이언트와 이미지 저장 서비스, 완료 흐름 연동 | `9701d3c`, `2e23cf1`, `1f39b44` |
| 배포 설정과 자동화 | GitHub Actions, 애플리케이션 YAML과 환경변수 구성; Java 21·Gradle 빌드, 배포 ZIP 생성, Elastic Beanstalk 배포 | `6695eed` 이후 배포 커밋 및 사용자 제공 담당 정보 |
| 헬스 체크 | 서버 포트 5000, Actuator 의존성, `/`·`/health` 200 응답 구성 | `098643b`, `624c0fd`, `30d7d38` |

현재 코드에서 프롬프트 입력 오브젝트는 `MAX_OBJECTS_FOR_PROMPT = 30`으로 제한됩니다. 광장 자체의 완료 기준은 각 광장의 `maxObjects` 값이며 엔티티 기본값은 8입니다. 현재 구현에는 `MAIN`·`SUPPORTING` 분류 상수가 없으므로 해당 표현은 사용하지 않았습니다.

더 자세한 근거와 클래스별 처리 흐름은 [김준영 담당 기능 상세](docs/MY_CONTRIBUTIONS.md)에서 확인할 수 있습니다.

## 핵심 기술 구현 과정

### 1. 저장과 외부 AI 작업의 분리

`PlazaService`는 참여 글을 저장한 뒤 `PlazaEntryCreatedEvent`를 발행합니다. `PlazaCompletionService`는 `@TransactionalEventListener(phase = AFTER_COMMIT)`과 `@Async`로 이벤트를 받아, 저장 트랜잭션이 끝난 이후 완료 조건 검사와 외부 AI 호출을 수행합니다.

이 구조는 사용자 요청에서 반드시 성공해야 하는 데이터 저장과 응답 시간이 긴 AI 이미지 생성의 실패 범위를 분리합니다.

### 2. 광장 완료와 중복 실행 제어

완료 서비스는 새 트랜잭션에서 광장과 참여 글을 읽고 `entries.size() >= plaza.maxObjects`인지 확인합니다. `imageGenerating` 상태로 이미지 생성 작업을 잠그고, 방장 강제 종료 시에는 `forceComplete` 이벤트로 동일한 완료 흐름을 재사용합니다.

### 3. AI 프롬프트와 결과 저장

`PlazaImagePromptBuilder`는 참여자의 감정·날씨·오브젝트·좌표를 한 장의 장면을 위한 프롬프트로 변환합니다. 프롬프트 과대화를 막기 위해 최대 30개 항목만 사용하고, 좌표를 상·중·하와 좌·중·우의 대략적인 위치로도 변환합니다.

생성 결과는 data URL을 디코딩해 `plazas/{plazaId}/{UUID}.{ext}` 키로 S3에 업로드하고, 공개 기준 URL을 편지 데이터에 연결합니다. 버킷과 공개 기준 URL은 환경변수로 분리했습니다.

### 4. 배포 자동화

`main` 브랜치에서 `backend/**` 또는 워크플로가 바뀌면 Java 21 Corretto 환경에서 `./gradlew clean build -x test`를 실행합니다. 실행 JAR과 `Procfile`을 ZIP으로 묶고 GitHub Secrets를 사용해 Elastic Beanstalk 애플리케이션 버전으로 배포합니다.

구조와 시퀀스 다이어그램은 [시스템 및 배포 구조](docs/ARCHITECTURE.md)에 정리했습니다.

## 문제 해결 경험

### 1. AI 이미지 생성으로 참여 요청이 길어지는 문제

- **문제**: 참여 데이터 저장과 이미지 생성이 한 요청에 묶이면 최대 수십 초가 걸리는 외부 API 응답 때문에 사용자가 오래 기다릴 수 있었습니다.
- **원인 분석**: 핵심 데이터 저장과 부가 결과 생성의 처리 시간·실패 조건이 서로 달랐습니다.
- **해결 방법**: 참여 글 저장 후 이벤트를 발행하고, 트랜잭션 커밋 이후 별도 비동기 흐름에서 완료 검사와 이미지 생성을 실행했습니다.
- **결과**: 저장 트랜잭션과 외부 AI 작업의 실패 범위를 분리하고 요청 지연을 줄일 수 있는 구조를 만들었습니다.
- **배운 점**: 외부 API 작업은 핵심 쓰기 트랜잭션과 수명 주기를 분리해야 합니다.

### 2. 재배포 시 이미지가 사라질 수 있는 문제

- **문제**: Elastic Beanstalk 인스턴스의 로컬 디스크는 재배포·교체 시 영속성을 보장하지 않습니다.
- **원인 분석**: 애플리케이션 실행 환경과 사용자가 다시 조회해야 하는 이미지의 수명이 달랐습니다.
- **해결 방법**: AWS SDK for Java를 연결하고 AI 결과 이미지를 S3에 저장한 뒤 URL을 편지 데이터에 연결했습니다.
- **결과**: 백엔드 인스턴스 교체와 이미지 수명을 분리했습니다.
- **배운 점**: 운영 환경에서는 상태를 애플리케이션 인스턴스 밖에 보존해야 합니다.

### 3. 로컬과 AWS 배포 환경의 차이

- **문제**: 로컬에서 실행되던 서버가 Elastic Beanstalk에서는 빌드 산출물, 포트, 환경변수와 외부 리소스 권한 차이 때문에 정상화되지 않을 수 있었습니다.
- **원인 분석**: 배포 ZIP 구조, 실행 명령, 기본 포트, DB 연결, S3 권한과 헬스 체크가 함께 맞아야 했습니다.
- **해결 방법**: 실행 JAR과 `Procfile`을 명시적으로 패키징하고 서버 포트를 5000으로 맞췄으며, 환경별 값을 환경변수로 분리하고 `/`·`/health` 응답을 추가했습니다.
- **결과**: 빌드부터 배포·상태 확인까지 반복 가능한 흐름을 구성했습니다.
- **배운 점**: 운영 장애는 코드만이 아니라 패키징, 네트워크, 권한, 설정과 관측 지점을 함께 봐야 합니다.

### 4. 커밋 전 완료 처리가 실행될 수 있는 문제

- **문제**: 비동기 완료 검사가 참여 글 저장 커밋보다 먼저 최신 데이터를 조회하면 완료 조건을 놓칠 수 있었습니다.
- **원인 분석**: 이벤트 발행 시점과 트랜잭션 커밋 시점이 일치하지 않았습니다.
- **해결 방법**: `TransactionPhase.AFTER_COMMIT` 리스너와 별도 `REQUIRES_NEW` 트랜잭션으로 완료 스냅샷을 읽도록 구성했습니다.
- **결과**: 커밋된 참여 데이터를 기준으로 완료 여부를 판단하도록 실행 순서를 보장했습니다.
- **배운 점**: 비동기 이벤트는 어떤 트랜잭션 상태를 관찰해야 하는지 먼저 정의해야 합니다.

## 프로젝트 전체 기술 스택

| 구분 | 기술 |
| --- | --- |
| Frontend | React 19, TypeScript, Vite, React Router, Tailwind CSS, Lucide React, html-to-image |
| Backend | Java 21, Spring Boot 4.0.6, Spring Web MVC, Spring Data JPA, Spring Validation, Spring Mail, WebClient |
| Data | MySQL 8.4, AWS RDS for MySQL, JPA |
| AI | OpenAI API, LangChain4j |
| Infrastructure | AWS Elastic Beanstalk, AWS S3, AWS RDS |
| CI/CD | GitHub Actions, Gradle, Procfile |
| Local environment | Docker Compose(MySQL) |

## 김준영이 직접 사용한 기술

`Java 21` · `Spring Boot` · `Spring Data JPA` · `MySQL` · `Spring Transaction Event` · `@Async` · `OpenAI API` · `WebClient` · `AWS SDK for Java` · `AWS S3` · `AWS Elastic Beanstalk` · `AWS RDS` · `GitHub Actions` · `Gradle` · `Git`

프론트엔드 기술은 프로젝트 전체 기술에는 포함했지만, 김준영의 직접 개발 기술로 표시하지 않았습니다.

## 데이터 및 요청 처리 흐름

```mermaid
sequenceDiagram
    actor User as 사용자
    participant FE as React Frontend
    participant PS as PlazaService
    participant DB as MySQL
    participant PCS as PlazaCompletionService
    participant AI as OpenAI API
    participant S3 as AWS S3
    participant MB as MailboxService

    User->>FE: 감정·오브젝트 등록
    FE->>PS: 광장 참여 요청
    PS->>DB: PlazaEntry 저장
    DB-->>PS: 커밋
    PS-->>FE: 저장 결과 응답
    PS-->>PCS: 완료 이벤트
    PCS->>DB: 완료 조건과 참여 데이터 조회
    alt 완료 조건 충족
        PCS->>AI: 이미지 생성 프롬프트 요청
        AI-->>PCS: 이미지 data URL
        PCS->>S3: 이미지 업로드
        S3-->>PCS: 이미지 URL
        PCS->>MB: 참여자별 편지 생성
        MB->>DB: 편지 저장
    end
```

## 배포 구조

```mermaid
flowchart LR
    Main["GitHub main"] -->|"backend 또는 workflow 변경"| Actions["GitHub Actions"]
    Actions --> JDK["Amazon Corretto JDK 21"]
    JDK --> Gradle["Gradle clean build -x test"]
    Gradle --> Zip["application.jar + Procfile → deploy.zip"]
    Zip --> EB["AWS Elastic Beanstalk"]
    EB --> App["Spring Boot · port 5000"]
    App --> RDS["AWS RDS MySQL"]
    App --> S3["AWS S3"]
    App --> AI["OpenAI API"]
    App --> Health["GET / · GET /health"]
```

원본 설정에 남아 있던 서비스 도메인은 2026년 7월 17일 기준 DNS 응답을 확인할 수 없어, 현재 상태는 **데모 운영 종료**로 표시합니다.

## 주요 화면

수상 증빙 이미지는 추가했으며, 서비스 화면은 개인정보 확인 후 추가할 예정입니다.

추가할 파일과 촬영 기준은 [images/README.md](images/README.md)에 정리했습니다.

- 랜딩·개인 방: `images/main.png`, `images/private-room.png`
- 광장 참여·완성: `images/plaza.png`, `images/plaza-result.png`
- 편지함·관리자: `images/mailbox.png`, `images/admin-report.png`
- 수상 증빙: [`images/award.jpg`](images/award.jpg)

아직 없는 서비스 화면은 깨진 이미지가 표시되지 않도록 파일 경로만 안내합니다.

## 주요 커밋

원본 Git 기록에서 `bamkkayo <junyeong2107@gmail.com>` 작성자로 확인되는 63개 커밋을 검토했습니다. 아래는 현재 GitHub 계정 `junyeong2107`의 이전 사용자 이름으로 제공된 `bamkkayo` 명의의 대표 커밋입니다.

| 주제 | 커밋 | 설명 |
| --- | --- | --- |
| 광장 참여 엔티티 | [8d5467f](https://github.com/guddlrdl123/WeatherOfTheHeart-/commit/8d5467fe6c70086af266b41afc9ba099c232a756) | 광장별 사용자 기록과 감정·오브젝트·위치 정보를 저장하는 모델 추가 |
| 광장 JPA 조회 | [eb37a1a](https://github.com/guddlrdl123/WeatherOfTheHeart-/commit/eb37a1ab99d849a7dde68d91e42dd554e99ce3a2) | 광장 참여 글 조회와 사용자 중복 참여 검사 메서드 추가 |
| 완료 이벤트 | [90cb2be](https://github.com/guddlrdl123/WeatherOfTheHeart-/commit/90cb2be9b58229a305c98503acf333329037317c) | 저장 커밋 이후 완료 여부를 확인하는 이벤트 추가 |
| AI 프롬프트 | [0aa3629](https://github.com/guddlrdl123/WeatherOfTheHeart-/commit/0aa36292477c776b455e74a57a92fb9a875e92a6) | 광장 오브젝트와 감정을 최종 이미지 프롬프트로 구성 |
| 비동기 완료 처리 | [31e75ae](https://github.com/guddlrdl123/WeatherOfTheHeart-/commit/31e75ae2b16eb22a966e8bd8ffa33ed6919d8932) | 완료 조건, AI 이미지 생성과 참여자 편지 발송 흐름 구현 |
| Elastic Beanstalk 배포 | [6695eed](https://github.com/guddlrdl123/WeatherOfTheHeart-/commit/6695eed8084033ac4d012ce77f4d21c1d7188b54) | GitHub Actions 백엔드 배포 워크플로 추가 |
| S3 저장 서비스 | [2e23cf1](https://github.com/guddlrdl123/WeatherOfTheHeart-/commit/2e23cf18935c7294275b866c177eb4528411068c) | S3 클라이언트 설정과 이미지 저장 서비스 추가 |
| S3 완료 흐름 연동 | [1f39b44](https://github.com/guddlrdl123/WeatherOfTheHeart-/commit/1f39b4498936033e7c88e2769ba3a21e4c0cb609) | AI 결과 업로드와 편지 이미지 URL 연결 |
| 헬스 체크 | [30d7d38](https://github.com/guddlrdl123/WeatherOfTheHeart-/commit/30d7d386b531b657ac05bea964a812b4440050e6) | Elastic Beanstalk·ELB용 상태 확인 엔드포인트 추가 |

김준영이 만든 파일은 이후 팀원들의 통합·기능 개선 커밋으로 함께 발전했습니다. 작성자·커미터·공동 작성·병합 관계를 검토한 상세 결과는 [기여 상세 문서](docs/MY_CONTRIBUTIONS.md#커밋-귀속과-팀-후속-개선)에 구분해 두었습니다.

## 수상 및 증빙

### 2026 K-디지털트레이닝 벤처·스타트업 유형 우수상

- 행사명: SW 잡브릿지-DAY 통합프로젝트 발표회
- 수상일: 2026년 7월 16일
- 관련 과정: 고용노동부 K-디지털트레이닝(벤처·스타트업 유형)
- 주관·수여: (사)한국경영혁신중소기업협회(MAINBiz)
- 출품작: 마음의 날씨
- 형태: 4인 팀 프로젝트
- 김준영 담당: Java 백엔드 개발, GitHub Actions·YAML·환경변수 구성 및 AWS 배포

`마음의 날씨` 팀이 발표회에 참가해 받은 **우수상**이며, 개인 단독 수상이나 다른 등급의 상으로 확대해 표현하지 않았습니다.

![SW 잡브릿지-DAY 통합프로젝트 발표회 우수상 상장과 행사 안내](images/award.jpg)

상장과 행사 안내판을 함께 촬영한 실제 수상 증빙입니다. 상장에 표시된 팀원 이름은 원본 그대로이며 전화번호, 이메일, 생년월일이나 주소는 포함되어 있지 않습니다.

## 원본 저장소와 관련 링크

- [원본 팀 프로젝트 저장소](https://github.com/guddlrdl123/WeatherOfTheHeart-)
- [김준영 담당 기능 상세](docs/MY_CONTRIBUTIONS.md)
- [시스템 및 배포 구조](docs/ARCHITECTURE.md)
- [이미지 추가 안내](images/README.md)
- 프로젝트 형태: 4인 팀 프로젝트
- 김준영 담당: Java 백엔드 개발, GitHub Actions·YAML·환경변수 구성 및 AWS 배포
- 서비스 배포 주소: 데모 운영 종료

## 프로젝트를 통해 배운 점

- 커밋과 코드 리뷰를 바탕으로 여러 사람의 변경을 하나의 도메인 흐름으로 통합하는 법
- 광장 참여·완료 조건처럼 상태 전이가 있는 백엔드 도메인 로직을 모델링하는 법
- 데이터 저장과 외부 AI 작업을 트랜잭션 이벤트와 비동기 처리로 분리하는 기준
- 외부 API가 늦거나 실패해도 핵심 데이터의 정합성을 우선 지키는 설계 방식
- S3·RDS·Elastic Beanstalk의 권한, 네트워크와 환경변수를 함께 점검하는 운영 관점
- 로그와 헬스 체크를 이용해 로컬과 배포 환경의 차이를 추적하는 방법
- 구현 결과를 데모와 발표 자료로 전달하고, 팀 프로젝트가 외부 발표회에서 우수상을 받기까지의 협업 경험
