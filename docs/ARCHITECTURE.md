# 시스템 및 배포 구조

이 문서는 `마음의 날씨`의 전체 시스템과 김준영 담당 영역, 광장 완료 처리와 AWS 배포 흐름을 분리해 설명합니다.

- 원본 저장소: [guddlrdl123/WeatherOfTheHeart-](https://github.com/guddlrdl123/WeatherOfTheHeart-)
- 전체 데이터 모델: [원본 저장소 DB ERD](https://github.com/guddlrdl123/WeatherOfTheHeart-/blob/main/docs/db-erd.md)
- 분석 기준: 2026년 7월 17일 `main`
- 프로젝트 형태: 4인 팀 프로젝트
- 김준영 담당: Java 백엔드, 광장 완료·AI 이미지 흐름, S3 저장, GitHub Actions·YAML·환경변수 구성, AWS 백엔드 배포

## 1. 전체 시스템 구조

```mermaid
flowchart LR
    User["사용자"] --> Web["React 19 + TypeScript"]
    Web -->|"JSON REST API"| API["Spring Boot 4 Backend"]

    subgraph BackendDomains["Backend Domains"]
        Auth["인증·OAuth·이메일"]
        Memory["개인 방·기억"]
        Plaza["광장·참여 글"]
        Completion["AI 완성 이미지"]
        Mailbox["편지함"]
        Admin["공지·문의·신고·제재"]
    end

    API --> Auth
    API --> Memory
    API --> Plaza
    Plaza --> Completion
    Completion --> Mailbox
    API --> Admin

    Auth --> OAuth["Google · Kakao · Naver OAuth"]
    Auth --> SMTP["SMTP Mail"]
    Memory --> DB["MySQL / AWS RDS"]
    Plaza --> DB
    Mailbox --> DB
    Admin --> DB
    Completion --> OpenAI["OpenAI API"]
    Completion --> S3["AWS S3"]
```

### 구성 요소

| 계층 | 기술 | 역할 |
| --- | --- | --- |
| Client | React, TypeScript, Vite | 화면 렌더링, 사용자 입력, 오브젝트 배치와 API 호출 |
| API | Java 21, Spring Boot, Spring Web MVC | REST API, 인증과 도메인 로직 |
| Persistence | Spring Data JPA, MySQL | 사용자·기억·광장·편지·운영 데이터 저장 |
| AI | WebClient, OpenAI API, LangChain4j | 감정 분석과 광장 완성 이미지 생성 |
| Object storage | AWS SDK for Java, S3 | AI 결과와 서비스 이미지 저장 |
| Infrastructure | Elastic Beanstalk, RDS | 백엔드 프로세스와 운영 DB |
| Delivery | GitHub Actions, Gradle, Procfile | 자동 빌드, 패키징과 배포 |

## 2. 사용자 공간과 데이터 흐름

```mermaid
flowchart TB
    Login["사용자 로그인"] --> Choice{"기록 공간 선택"}

    Choice -->|"개인 방"| PrivateInput["개인 감정 기록"]
    PrivateInput --> Analyze["AI 감정·날씨·오브젝트 분석"]
    Analyze --> PrivateMemory["PrivateMemory 저장"]
    PrivateMemory --> PrivateRoom["날짜별 개인 방 시각화"]

    Choice -->|"광장"| PlazaInput["광장 감정·오브젝트 등록"]
    PlazaInput --> PlazaEntry["PlazaEntry 저장"]
    PlazaEntry --> CompletionCheck["완료 조건 확인"]
    CompletionCheck --> FinalImage["AI 최종 이미지"]
    FinalImage --> ImageStorage["S3 저장"]
    ImageStorage --> Letters["참여자별 Letter 저장"]
    Letters --> Mailbox["편지함 조회"]
```

개인 방은 한 사용자의 기록을 날짜별로 보존하고, 광장은 여러 참여자의 데이터를 하나의 결과 이미지로 결합합니다.

## 3. 주요 도메인 관계

```mermaid
erDiagram
    USER ||--o{ PRIVATE_ROOM : owns
    PRIVATE_ROOM ||--o{ PRIVATE_MEMORY : contains
    USER ||--o{ PLAZA : creates
    PLAZA ||--o{ PLAZA_ENTRY : contains
    USER ||--o{ PLAZA_ENTRY : writes
    PLAZA_ENTRY ||--o{ OBJECT_LIKE : receives
    USER ||--o{ OBJECT_LIKE : creates
    PLAZA_ENTRY ||--o{ PLAZA_ENTRY_REPORT : reported_by
    USER ||--o{ LETTER : receives
    PLAZA ||--o{ LETTER : result_of

    USER {
        bigint id PK
        string email
        string nickname
    }
    PLAZA {
        bigint id PK
        bigint owner_id FK
        int max_objects
        datetime completed_at
        boolean image_generating
    }
    PLAZA_ENTRY {
        bigint id PK
        bigint plaza_id FK
        bigint owner_id FK
        string mood_key
        string weather_key
        string object_key
        int position_x
        int position_y
    }
    LETTER {
        bigint id PK
        bigint receiver_id FK
        bigint plaza_id
        string generated_image_data
        boolean is_read
    }
```

`Letter.plazaId`와 오브젝트 키 일부는 코드상 논리 참조이며 모든 관계가 DB 외래 키로 직접 매핑된 것은 아닙니다.

## 4. 김준영 담당 구조

```mermaid
flowchart TB
    Request["POST /api/plazas/{id}/entries"] --> Service["PlazaService"]
    Service --> Validate["참여·중복·정원 검증"]
    Validate --> EntryRepo["PlazaEntryRepository"]
    EntryRepo --> DB["MySQL / AWS RDS"]
    Service --> Event["PlazaEntryCreatedEvent"]

    Event -->|"AFTER_COMMIT + @Async"| Completion["PlazaCompletionService"]
    Completion --> Snapshot["완료 조건·스냅샷"]
    Snapshot --> Prompt["PlazaImagePromptBuilder"]
    Prompt --> AI["OpenAI API"]
    AI --> Storage["S3ImageStorageService"]
    Storage --> S3["AWS S3"]
    Storage --> LetterService["MailboxService"]
    LetterService --> DB

    Source["GitHub main"] --> Actions["deploy-backend.yml"]
    Actions --> Build["Gradle + JDK 21"]
    Build --> Package["application.jar + Procfile"]
    Package --> EB["Elastic Beanstalk"]
    EB --> Health["/ · /health"]
```

### 담당 경계

| 포함 | 제외 또는 팀 전체 |
| --- | --- |
| 광장 참여 데이터의 백엔드 기반 | React 화면과 사용자 상호작용 구현 |
| 완료 조건·커밋 이후 이벤트 | 인증·개인 방·관리자 기능 전체를 단독 구현했다는 표현 |
| AI 이미지 프롬프트와 완료 생성 흐름 | 다른 팀원 명의의 후속 통합 커밋을 개인 커밋으로 귀속 |
| S3 이미지 저장과 URL 연결 | 프론트엔드 배포 구조(저장소에서 확인되지 않음) |
| GitHub Actions·YAML·환경변수 구성 | 서비스 전체를 혼자 설계·개발했다는 표현 |
| Elastic Beanstalk 배포와 헬스 체크 | 수상 결과를 개인 단독 수상으로 표현 |

## 5. 광장 참여 요청 시퀀스

```mermaid
sequenceDiagram
    actor User as 사용자
    participant FE as Frontend
    participant PC as PlazaController
    participant PS as PlazaService
    participant PR as PlazaEntryRepository
    participant DB as MySQL
    participant EP as EventPublisher

    User->>FE: 감정·오브젝트 입력
    FE->>PC: POST 광장 참여
    PC->>PS: createEntry
    PS->>DB: 광장·사용자 확인
    PS->>PR: 중복 참여·오브젝트 검사
    PS->>PR: 현재 참여 수 확인
    PS->>PR: PlazaEntry 저장
    PR->>DB: INSERT
    PS->>EP: PlazaEntryCreatedEvent 발행
    DB-->>PS: 트랜잭션 커밋
    PS-->>PC: 저장 결과
    PC-->>FE: API 응답
```

이벤트는 트랜잭션 안에서 발행되지만 완료 리스너가 `AFTER_COMMIT` 단계에 연결돼, AI 작업은 DB 커밋 이후 시작합니다.

## 6. 광장 완료 시퀀스

```mermaid
sequenceDiagram
    participant Event as AFTER_COMMIT Event
    participant PCS as PlazaCompletionService
    participant DB as MySQL
    participant PB as PromptBuilder
    participant AI as OpenAI API
    participant S3 as AWS S3
    participant MB as MailboxService

    Event-->>PCS: plazaId, forceComplete
    PCS->>DB: 새 트랜잭션에서 Plaza·Entry 조회
    PCS->>PCS: entryCount와 maxObjects 비교
    alt 완료 조건 미충족
        PCS-->>Event: 종료
    else 완료 조건 충족 또는 방장 종료
        PCS->>PB: 프롬프트 구성
        PCS->>DB: imageGenerating 잠금
        PCS->>AI: 이미지 생성 요청
        AI-->>PCS: image data URL
        PCS->>S3: plazas/{id}/{uuid} 업로드
        S3-->>PCS: 이미지 URL
        PCS->>MB: 참여자별 완성 편지 요청
        MB->>DB: 중복 검사 후 Letter 저장
        PCS->>DB: imageGenerating 해제
    end
```

### 완료 조건

- 자동 완료: `entries.size() >= plaza.maxObjects`
- 방장 완료: `forceComplete = true`
- 엔티티 기본 정원: 8
- 이미지 프롬프트 입력 상한: 30개
- 중복 실행 제어: `imageGenerating`
- 중복 편지 제어: `receiverId + plazaId` 존재 검사

## 7. AI 이미지 처리 구조

```mermaid
flowchart LR
    Entries["PlazaEntry 목록"] --> Limit["최대 30개 선택"]
    Limit --> Normalize["텍스트·좌표 정규화"]
    Normalize --> Prompt["장면 프롬프트"]
    Prompt --> API["OpenAI Image API"]
    API --> DataUrl["Image data URL"]
    DataUrl --> Decode["MIME·Base64 디코딩"]
    Decode --> Key["plazas/{plazaId}/{UUID}.{ext}"]
    Key --> Upload["S3 putObject"]
    Upload --> URL["Public base URL + key"]
    URL --> Letter["Letter.generatedImageData"]
```

### 실패 경계

- AI 생성 실패 시 예외를 완료 서비스가 잡고 이미지 없는 편지 흐름을 허용합니다.
- S3 업로드 실패 시 원본 data URL을 편지 데이터에 전달하는 폴백이 있습니다.
- 이미지 생성 잠금은 `finally`에서 해제됩니다.
- 편지 저장은 `REQUIRES_NEW` 트랜잭션에서 수신자별 중복을 검사합니다.

## 8. 배포 구조

```mermaid
flowchart LR
    Dev["개발자 push"] --> Main["GitHub main"]
    Main -->|"backend/** 또는 workflow 변경"| Runner["GitHub Actions · ubuntu-latest"]
    Runner --> Checkout["actions/checkout@v4"]
    Checkout --> Java["Amazon Corretto 21"]
    Java --> Gradle["./gradlew clean build -x test"]
    Gradle --> Jar["실행 JAR → application.jar"]
    Jar --> Zip["deploy.zip + Procfile"]
    Zip --> Action["beanstalk-deploy@v21"]
    Action --> EB["AWS Elastic Beanstalk"]
    EB --> Spring["Spring Boot · port 5000"]
    Spring --> RDS["AWS RDS MySQL"]
    Spring --> S3["AWS S3"]
    Spring --> OpenAI["OpenAI API"]
    Spring --> Mail["SMTP"]
    EB --> Health["GET / 또는 /health"]
```

### 워크플로 설정

| 설정 | 값 |
| --- | --- |
| 트리거 | `push` to `main`, `workflow_dispatch` |
| 경로 필터 | `backend/**`, `.github/workflows/**` |
| Java | 21, Corretto |
| 빌드 | `./gradlew clean build -x test` |
| 배포 파일 | `application.jar`, `Procfile`, `deploy.zip` |
| 버전 라벨 | GitHub run ID와 run attempt 조합 |
| AWS 인증 | GitHub Secrets |

프론트엔드의 실제 운영 배포 워크플로는 원본 저장소에서 확인되지 않아 위 자동 배포 구조에는 포함하지 않았습니다.

## 9. 설정과 환경변수 경계

GitHub Actions, `application.yaml`을 포함한 YAML, 환경변수 구성은 김준영 담당입니다. 설정 파일은 값을 저장하지 않고 실행 환경이 주입할 이름과 기본 동작을 정의합니다.

```mermaid
flowchart TB
    Local["로컬 .env"] --> Config["application.yaml"]
    EBEnv["Elastic Beanstalk Environment Properties"] --> Config
    GHSecrets["GitHub Actions Secrets"] --> Workflow["deploy-backend.yml"]
    Workflow --> EB["Elastic Beanstalk Deploy"]

    Config --> DB["DB_URL · DB_USERNAME · DB_PASSWORD"]
    Config --> AI["OPENAI_API_KEY"]
    Config --> AWS["AWS_REGION · S3 설정"]
    Config --> Mail["MAIL 설정"]
    Config --> OAuth["OAuth Client 설정"]
```

### 공개 문서에 포함하지 않는 값

- AWS Access Key와 Secret Key
- AWS 계정 번호와 내부 리소스 식별자
- 실제 RDS 호스트·사용자·비밀번호
- 실제 OpenAI API 키
- SMTP 앱 비밀번호
- OAuth Client Secret
- 개인정보가 포함된 허용 URL 또는 운영 로그

## 10. 헬스 체크와 관측

| 엔드포인트 | 응답 | 의미 |
| --- | --- | --- |
| `GET /` | `200 OK`, `OK` | 루트 경로의 기본 생존 확인 |
| `GET /health` | `200 OK`, `OK` | Elastic Beanstalk·ELB 상태 확인용 |

현재 구현은 애플리케이션 프로세스가 HTTP 요청에 응답하는지 확인하는 얕은 헬스 체크입니다. DB·S3·OpenAI 연결까지 검사하지 않기 때문에, 외부 서비스 문제는 애플리케이션 로그와 각 AWS 서비스 상태를 함께 봐야 합니다.

## 11. 운영 상태와 확인된 한계

- 원본 설정의 서비스 도메인은 2026년 7월 17일 기준 DNS 응답이 없어 데모 운영 종료로 표시했습니다.
- RDS 보안 그룹, IAM 정책과 Elastic Beanstalk 환경 속성 자체는 저장소에 포함되지 않으므로 코드의 환경변수 참조와 사용자 제공 정보를 기준으로 설명했습니다.
- `MAIN`, `SUPPORTING` 오브젝트 역할 구분은 현재 코드에 없어 구조도에 넣지 않았습니다.
- 관리자 신고·경고·정지 기능은 프로젝트 전체 구조에 포함하지만 김준영의 직접 구현 영역으로 표시하지 않았습니다.
- 백엔드 테스트 파일은 컨텍스트 로드와 인증 회귀 테스트 각 1건이며, 배포 워크플로는 테스트를 제외하므로 자동 검증 범위가 좁습니다.
- `@Async` 완료 처리는 애플리케이션 프로세스 안에서 실행되며, 영속 메시지 큐나 자동 재시도 정책은 확인되지 않았습니다.
- S3 결과 URL은 공개 기준 URL을 조합하는 방식이므로, 민감한 결과 이미지에는 비공개 버킷과 서명 URL 방식이 더 적합합니다.
- 우수상 증빙은 추가됐고, 실제 서비스 화면과 시연 영상은 개인정보 확인 후 [images 안내](../images/README.md)에 따라 추가해야 합니다.
