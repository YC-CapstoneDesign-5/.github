# 시스템 아키텍처

> [← 조직 홈](https://github.com/YC-CapstoneDesign-5) · [아키텍처](ARCHITECTURE.md) · [기능 명세](FEATURES.md) · [로드맵](ROADMAP.md) · [협업 가이드](../CONTRIBUTING.md)

> **문서 상태**: 설계 단계. 기술 스택은 [추천 스택](#10-기술-스택-확정-전) 기준이며 팀 협의 후 확정합니다. 확정되면 이 문서를 먼저 갱신합니다.

## 목차
1. [한눈에 보기](#1-한눈에-보기)
2. [설계 원칙](#2-설계-원칙)
3. [사용자와 권한](#3-사용자와-권한)
4. [모듈 구성](#4-모듈-구성)
5. [시스템 구성도](#5-시스템-구성도)
6. [전체 업무 흐름](#6-전체-업무-흐름)
7. [핵심 파이프라인](#7-핵심-파이프라인)
8. [데이터 모델](#8-데이터-모델)
9. [업무 상태 모델](#9-업무-상태-모델)
10. [기술 스택 (확정 전)](#10-기술-스택-확정-전)
11. [보안 및 데이터 관리](#11-보안-및-데이터-관리)
12. [기술 제약 T1~T4](#12-기술-제약-t1t4)

---

## 1. 한눈에 보기

![행정 구조대 한눈에 보기](images/overview.svg)

약 60개 부서가 보내는 엑셀·스캔 출석부와 4대보험 고지 자료를 **한 곳에 모아 검증 → 계산 → 대조 → 서식 생성**까지 처리합니다.
부서 담당자는 제출 전에 **Pre-Check**로 스스로 오류를 고치고, 모든 단계에서 **AI 행정규정 챗봇**이 업무 지침을 근거와 함께 안내합니다.

---

## 2. 설계 원칙

모든 설계 결정은 아래 다섯 원칙을 따릅니다. 충돌하면 위에 있는 원칙이 우선합니다.

| # | 원칙 | 의미 | 적용 예 |
| :---: | :--- | :--- | :--- |
| 1 | **개인정보 최소 노출** | 주민번호 등 고유식별정보는 암호화 저장, 화면은 마스킹 | `880101-1******` |
| 2 | **결정론적 계산** | 금액은 규칙 엔진과 십진수(Decimal/NUMERIC)로만 계산. AI가 금액을 정하지 않음 | 급여·보험 공제 |
| 3 | **자동 확정 금지** | 근거가 불확실하면 `확인 필요`로 남기고 담당자가 확정 | 보험 매칭, OCR, 챗봇 |
| 4 | **원본 추적성** | 모든 값은 원본 파일·시트·행(페이지)까지 거슬러 올라갈 수 있음 | 오류 클릭 → 원본 위치 |
| 5 | **업무 상태 독립** | 계산·서식·지급·납부·신고 상태는 서로 자동으로 바꾸지 않음 | 다운로드 ≠ 신고 완료 |

---

## 3. 사용자와 권한

```mermaid
flowchart LR
    subgraph U["사용자"]
        A["총괄 관리자<br>(일자리창출과)"]
        B["사업 담당자"]
        C["부서 담당자<br>(약 60개 근무 부서)"]
    end
    subgraph S["접근 범위"]
        A1["전체 사업 열람·권한 부여<br>감사 로그·마감 통제"]
        B1["배정된 사업만<br>등록·수정·다운로드"]
        C1["자기 부서 자료만<br>업로드·Pre-Check·최종 제출"]
    end
    A --> A1
    B --> B1
    C --> C1
    A1 -. 포함 .-> B1
```

| 역할 | 할 수 있는 일 | 할 수 없는 일 |
| :--- | :--- | :--- |
| **총괄 관리자** | 전체 사업 조회, 권한 부여, 감사 로그, 마감 통제, 지침 문서 관리 | — |
| **사업 담당자** | 배정된 사업의 참여자·급여·보험·서식 처리 | 다른 사업 데이터 접근 (서버에서 403) |
| **부서 담당자** | 자기 부서 자료 업로드, Pre-Check 결과 확인·수정, 최종 제출, 챗봇 질문 | 급여 계산·보험 확정, 다른 부서 자료 열람 |

---

## 4. 모듈 구성

기능 번호는 [FEATURES.md](FEATURES.md)의 번호와 같습니다. 09·10은 [추가 기능 제안](FEATURES.md#추가-기능-제안)입니다.

```mermaid
flowchart TB
    subgraph UI["사용자 접점"]
        PRE["Pre-Check 포털 · 09"]
        DASH["현황 대시보드·알림 · 08"]
        BOT["AI 행정규정 챗봇 · 10"]
    end
    subgraph ENG["처리 엔진"]
        ING["엑셀 취합 · 02"]
        OCR["출석부 OCR 대조 · 06"]
        VAL["검증 룰 엔진 · 05"]
        PAY["인건비 계산 · 03"]
        INS["4대보험 귀속 매칭 · 04"]
        DOC["제출 서식 생성 · 07"]
    end
    MASTER[("참여자 마스터 · 01")]

    PRE --> ING & OCR
    ING & OCR --> VAL
    VAL --> PAY --> INS --> DOC
    MASTER --- VAL & PAY & INS
    DOC --> DASH
    VAL --> DASH
```

| 모듈 | 책임 | 입력 | 출력 | 기능 |
| :--- | :--- | :--- | :--- | :--- |
| 참여자 마스터 | 사람·사업 참여·부서 배치 이력 관리 | 기준 명단 | 참여자, 배치 이력 | 01 |
| 엑셀 취합 | 부서별 엑셀을 표준 스키마로 정규화 | 부서 엑셀 | 표준 행 + 원본 위치 | 02 |
| 출석부 OCR 대조 | 스캔 출석부 판독 후 엑셀과 비교 | PDF·이미지 | 일치/불일치/미판독 | 06 |
| 검증 룰 엔진 | 누락·중복·기간 외 근무·금액 차이 검사 | 표준 행 | `ValidationIssue` | 05 |
| 인건비 계산 | 근무시간·휴가·중도 입퇴사 반영 금액 산출 | 검증된 근태 | 월별 급여 | 03 |
| 4대보험 귀속 매칭 | 공단 고지 자료에서 참여자분 분리·대조 | 고지 자료 | 보험 내역, 확인 필요 | 04 |
| 제출 서식 생성 | 기관별 템플릿에 확정 데이터 주입 | 확정 데이터 | XLSX·EDI 파일 | 07 |
| 현황 대시보드·알림 | 집행률·미처리·마감 임박 표시 | 전 모듈 상태 | 화면, D-7/D-3/D-Day 알림 | 08 |
| Pre-Check 포털 | 부서가 제출 전 같은 검증 엔진으로 자가검증 | 부서 자료 | 정상/수정 필요/확인 필요, 제출 건 | 09 |
| AI 행정규정 챗봇 | 등록된 지침 문서를 근거로 질문에 답변 | 질문, 지침 문서 | 답변 + 출처 | 10 |

> **재사용 포인트**: Pre-Check(09)는 새 엔진이 아니라 02·05·06을 **부서 담당자가 제출 전에** 호출하는 것입니다. 검증 규칙은 한 곳에만 둡니다.

---

## 5. 시스템 구성도

```mermaid
flowchart LR
    U["브라우저<br>React SPA"] -->|HTTPS · REST| API["API 서버<br>Django REST Framework"]
    API --> DB[("PostgreSQL<br>업무 데이터 · 감사 로그")]
    API --> ST[("비공개 스토리지<br>원본 엑셀 · 스캔 · 생성 서식")]
    API -->|작업 등록| Q[["Redis<br>작업 큐"]]
    Q --> W["Celery 워커"]
    W --> OCRW["OCR<br>OpenCV + PaddleOCR"]
    W --> XL["서식 생성<br>openpyxl"]
    W --> DB
    W --> ST
    API --> RAG["RAG 서비스"]
    RAG --> VDB[("벡터 인덱스<br>지침 문서 조각")]
    RAG --> LLM["LLM API"]
```

| 구성 요소 | 왜 분리하나 |
| :--- | :--- |
| API 서버 | 권한 검사(RBAC)와 트랜잭션을 한 곳에서 처리 |
| 작업 큐 + 워커 | OCR·대량 서식 생성은 수십 초~수 분이 걸려 요청과 분리. 진행률 표시와 실패 재시도 |
| 비공개 스토리지 | 개인정보가 섞인 원본 파일을 DB와 분리해 접근 제한 |
| RAG 서비스 | 챗봇은 업무 데이터(급여·보험)에 접근하지 않고 **지침 문서만** 검색 |

---

## 6. 전체 업무 흐름

```mermaid
flowchart TD
    A["참여자 기준 명단 등록<br>(Person / Participation / Placement)"] --> P
    P["부서 담당자 Pre-Check<br>업로드 → 자가검증 → 수정 → 최종 제출"] --> C
    C["데이터 정규화 & 원본 위치 보존"] --> D
    D["교차 검증 & OCR 대조"] --> E
    E["월별 인건비 정밀 계산"] --> F
    F["4대보험 귀속 대조 & 공제 확정"] --> G
    G["오류 검토 & 관리자 확정"] --> H
    H["외부기관 제출 서식 생성"] --> I
    I["마감 알림 & 통합 대시보드"]
    BOT(["AI 행정규정 챗봇<br>모든 단계에서 지침 안내"]) -.-> P & G
```

---

## 7. 핵심 파이프라인

### 7.1 부서 제출과 Pre-Check (09)

```mermaid
sequenceDiagram
    actor D as 부서 담당자
    participant P as Pre-Check 포털
    participant V as 검증 엔진 (02·05·06)
    actor M as 중앙 담당자
    D->>P: 엑셀·스캔 출석부 업로드
    P->>V: 사전검사 요청
    V-->>P: 정상 / 수정 필요 / 확인 필요 + 원본 위치
    loop 수정 필요가 없어질 때까지
        D->>P: 수정본 재업로드
        P->>V: 재검사
        V-->>P: 결과
    end
    D->>P: 최종 제출
    P->>M: 제출 완료 알림
    M->>V: 후속 통합 검증·정산
```

- 결과는 **원본 파일·시트·행·페이지** 위치와 함께 보여줍니다.
- `확인 필요`(OCR 저신뢰, 서명·도장 판독 불확실)는 제출을 막지 않고 중앙 담당자에게 넘깁니다.
- MVP: 업로드 → 기본 검증 → 결과 확인 → 최종 제출. 공가 증빙 판독, 고도화된 서명·도장 판별은 OCR 실험 후 추가.

### 7.2 엑셀 취합 (02)

```mermaid
flowchart LR
    A["엑셀 업로드"] --> B["시트·셀 파싱<br>식별자는 문자열로"]
    B --> C["안내 문구·합계 행 제거"]
    C --> D["열 매핑 → 표준 스키마<br>날짜 YYYY-MM-DD, 금액 Decimal"]
    D --> E["원본 파일명·시트·행 번호 저장"]
    E --> F{"같은 파일<br>이미 반영?"}
    F -->|예| G["중복 반영 안 함 (멱등)"]
    F -->|아니오| H["기준 명단과 대조 → 검증 엔진"]
```

엑셀에 저장된 수식 계산값은 정답으로 믿지 않고 서버에서 다시 계산합니다 (T2).

### 7.3 인건비 계산 (03)

```mermaid
flowchart LR
    A["검증된 근태"] --> B["정상 근무 기본급"]
    B --> C["조퇴·결근 공제<br>유·무급 휴가<br>중도 입퇴사 일할"]
    C --> D["주휴·유급휴가 수당"]
    D --> E["총 지급액"]
    E --> F["근로자 4대보험 공제 차감<br>(04에서 확정된 값)"]
    F --> G["실지급액"]
```

$$\text{실지급액} = \sum \text{승인된 지급 항목} - \text{확정된 근로자 4대보험 공제액}$$

기관 부담 보험료는 근로자 공제액과 별도로 관리합니다. 모든 단계의 산출 근거를 저장합니다.

### 7.4 4대보험 귀속 매칭 (04)

```mermaid
flowchart TD
    A["공단 고지 자료 행"] --> B{"주민번호 일치?"}
    B -->|아니오| X["타 사업 / 대상 아님"]
    B -->|예| C{"사업장 기호 일치?"}
    C -->|아니오| X
    C -->|예| D{"귀속기간이 참여·배치 기간 안?"}
    D -->|예| OK["자동 연결 (확정)"]
    D -->|아니오| R{"퇴사자 과거분 환급?"}
    R -->|예| RF["미환급 원장에 연결<br>is_refund = true"]
    R -->|불명확| CK["확인 필요<br>+ 유사 후보 목록"]
```

주민번호 **단독** 매칭은 하지 않습니다.

### 7.5 출석부 OCR 대조 (06)

```mermaid
sequenceDiagram
    actor U as 담당자
    participant API as API 서버
    participant Q as 작업 큐
    participant W as OCR 워커
    U->>API: 스캔 출석부 업로드
    API->>Q: OCR 작업 등록
    API-->>U: 작업 ID (상태: 대기)
    Q->>W: 작업 전달
    W->>W: 기울기 보정 · 노이즈 제거 · 표 외곽선 검출
    W->>W: 글자·숫자·체크·서명 판독 (항목별 분리)
    W->>API: 엑셀과 비교 → 일치 / 불일치 / 미판독
    U->>API: 상태 조회 (진행 / 완료 / 실패)
    API-->>U: 원본 이미지 ↔ 엑셀 나란히 보기, 불일치 강조
```

신뢰도 점수만으로 자동 확정하지 않습니다 (T4).

### 7.6 AI 행정규정 챗봇 · RAG (10)

```mermaid
flowchart LR
    Q["질문"] --> S["관련 지침 검색<br>(벡터 인덱스)"]
    S --> E{"근거 문단 있음?<br>문서 간 충돌 없음?"}
    E -->|예| G["근거 문단으로 답변 생성"]
    G --> O["답변 + 출처<br>문서명 · 항목 · 버전/기준일"]
    E -->|아니오| N["담당자 확인 필요로 안내"]
    ADM["관리자: 지침 문서 등록·교체"] --> IDX["문서 조각화 · 버전 기록"] --> S
```

| 업무 | 처리 방식 | 원칙 |
| :--- | :--- | :--- |
| 급여 계산·보험 금액 반영 | 규칙 기반 엔진 | AI가 최종 금액을 정하지 않음 |
| 엑셀·출석부 오류 검증 | 규칙 + OCR | 오류·미판독은 담당자 검토 |
| 업무 지침 질문 | RAG 챗봇 | 등록된 근거 문서 중심으로 설명 |

- 지식 원본: 작성 시 유의사항, 기관 확정 업무 지침, FAQ, 확인된 관련 규정.
- MVP 제외: Text-to-SQL, 자동 의사결정. 챗봇은 업무 DB를 조회하지 않습니다.

---

## 8. 데이터 모델

사람과 업무 이력을 분리해, 부서 이동·재참여·퇴사 후에도 과거 급여·보험 내역이 끊기지 않게 합니다.

```mermaid
erDiagram
    PERSON ||--o{ PARTICIPATION : "참여"
    PARTICIPATION ||--o{ PLACEMENT_HISTORY : "배치 이력"
    PARTICIPATION ||--o{ MONTHLY_PAYROLL : "월별 급여"
    MONTHLY_PAYROLL ||--o{ INSURANCE_RECORD : "보험 공제"
    MONTHLY_PAYROLL ||--o{ VALIDATION_ISSUE : "검증 오류"
    DATA_SOURCE_FILE ||--o{ VALIDATION_ISSUE : "추적"
    SUBMISSION ||--o{ DATA_SOURCE_FILE : "제출 파일"
    GUIDE_DOCUMENT ||--o{ GUIDE_CHUNK : "조각"
    CHAT_LOG }o--o{ GUIDE_CHUNK : "근거"

    PERSON {
        uuid id PK
        string name "성명"
        string resident_num_encrypted "주민번호(AES-256)"
        string resident_num_hash "동일인 식별 해시"
        string resident_num_masked "주민번호(마스킹)"
    }
    PARTICIPATION {
        uuid id PK
        uuid person_id FK
        string project_code "사업 코드"
        date start_date "참여 시작일"
        date end_date "참여 종료일"
        string status "재직/퇴사/대기"
    }
    PLACEMENT_HISTORY {
        uuid id PK
        uuid participation_id FK
        string department_code "근무 부서"
        date placement_start "배치 시작일"
        date placement_end "배치 종료일"
    }
    MONTHLY_PAYROLL {
        uuid id PK
        uuid participation_id FK
        string work_month "YYYY-MM"
        numeric gross_pay "총지급액"
        numeric total_deductions "공제 합계"
        numeric net_pay "실지급액"
        string payment_status "예정/완료/미처리"
    }
    INSURANCE_RECORD {
        uuid id PK
        uuid monthly_payroll_id FK
        string insurance_type "국민/건강/장기/고용/산재"
        numeric worker_share "근로자 부담"
        numeric employer_share "기관 부담"
        string reconcile_status "확정/확인필요"
        boolean is_refund "환급 여부"
    }
    DATA_SOURCE_FILE {
        uuid id PK
        uuid submission_id FK
        string original_filename "원본 파일명"
        string file_type "출석부/급여/보험"
        string file_path "비공개 저장 경로"
    }
    VALIDATION_ISSUE {
        uuid id PK
        uuid monthly_payroll_id FK
        uuid source_file_id FK
        string sheet_name "시트명"
        int row_index "행 번호"
        string issue_type "오류 유형"
        string expected_val "기준값"
        string actual_val "입력값"
        boolean is_resolved "해결 여부"
    }
    SUBMISSION {
        uuid id PK
        string department_code "제출 부서"
        string work_month "YYYY-MM"
        string precheck_result "정상/수정필요/확인필요"
        string status "작성중/제출완료/반려"
        timestamp submitted_at "최종 제출 일시"
    }
    GUIDE_DOCUMENT {
        uuid id PK
        string title "문서명"
        string version "버전"
        date effective_date "기준일"
        boolean is_active "사용 여부"
    }
    GUIDE_CHUNK {
        uuid id PK
        uuid document_id FK
        string section "항목"
        string content "본문 조각"
    }
    CHAT_LOG {
        uuid id PK
        uuid user_id FK
        string question "질문"
        string answer "답변"
        boolean needs_confirmation "확인 필요 여부"
    }
```

| 도메인 | 핵심 엔티티 | 메모 |
| :--- | :--- | :--- |
| 사람·참여 | `PERSON`, `PARTICIPATION`, `PLACEMENT_HISTORY` | 재참여·사업 이동·부서 전보를 각각 독립 이력으로 |
| 급여 | `MONTHLY_PAYROLL` | 금액 컬럼은 전부 `NUMERIC` |
| 보험 | `INSURANCE_RECORD` | 근로자/기관 부담 분리, 환급 별도 추적 |
| 검증·추적 | `DATA_SOURCE_FILE`, `VALIDATION_ISSUE` | 오류마다 원본 위치 기록 |
| 제출 (09) | `SUBMISSION` | 부서·월 단위 제출 건, Pre-Check 결과 |
| 지식 (10) | `GUIDE_DOCUMENT`, `GUIDE_CHUNK`, `CHAT_LOG` | 답변에 쓴 문서 버전 기록. 업무 데이터와 분리 |

> 09·10 엔티티는 추가 기능 제안 기준의 초안입니다.

---

## 9. 업무 상태 모델

다섯 가지 상태는 **각자 따로** 움직입니다. 서식 파일을 내려받았다고 신고 완료로 바뀌지 않습니다.

```mermaid
flowchart LR
    subgraph SC["계산"]
        c1["미계산"] --> c2["계산 완료"]
    end
    subgraph SF["서식"]
        f1["미생성"] --> f2["서식 생성 완료"]
    end
    subgraph SP["지급"]
        p1["지급 예정"] --> p2["지급 완료"]
        p1 --> p3["미처리"]
    end
    subgraph SI["보험"]
        i1["부과"] --> i2["납부 완료"]
    end
    subgraph SR["신고"]
        r1["미신고"] --> r2["외부 신고 완료"]
    end
```

부서 제출 건(`SUBMISSION`)의 상태:

```mermaid
stateDiagram-v2
    state "작성 중" as Draft
    state "제출 완료" as Submitted
    state "반려" as Rejected
    state "중앙 검토 완료" as Reviewed
    [*] --> Draft
    Draft --> Draft: Pre-Check 재검사
    Draft --> Submitted: 최종 제출
    Submitted --> Rejected: 중앙 담당자 반려
    Rejected --> Draft: 부서 수정
    Submitted --> Reviewed: 후속 검증 완료
    Reviewed --> [*]
```

---

## 10. 기술 스택 (확정 전)

| 영역 | 추천 스택 | 이유 | 대안 |
| :--- | :--- | :--- | :--- |
| 프론트엔드 | React + TypeScript + MUI | 대용량 데이터 그리드, 타입 안정성 | Next.js, Vue 3 |
| 백엔드 | Python + Django REST Framework | 복잡한 도메인·룰 엔진, 세부 권한 검사 (T1) | FastAPI, Spring Boot |
| 데이터베이스 | PostgreSQL | 정확한 `NUMERIC` (T3), JSONB, 트랜잭션 | MySQL 8.0 |
| 엑셀 엔진 | pandas + openpyxl | 이종 엑셀 정규화, 원본 서식 보존 주입 (T2) | Apache POI, ExcelJS |
| OCR | OpenCV + PaddleOCR | 표 외곽선 검출, 한글 행정 서식 경량 모델 (T4) | Tesseract, Cloud OCR |
| 비동기 작업 | Celery + Redis | OCR·서식 생성 백그라운드, 재시도 | BullMQ |
| RAG | 벡터 인덱스 + LLM API | 지침 문서 근거 검색·답변, 출처 표시 | 미정 (실험 후 결정) |

---

## 11. 보안 및 데이터 관리

| 영역 | 규칙 |
| :--- | :--- |
| 접근 제어 (T1) | 역할·사업·부서 범위를 **서버에서** 검사. 범위 밖 요청은 403 |
| 고유식별정보 | 주민번호는 해시(동일인 식별)와 AES-256 암호문으로 분리 저장. 키는 환경 변수로 코드와 분리 |
| 화면 표시 | 생년월일·성별 외 뒷자리 마스킹 `880101-1******` |
| 원본 파일 | 타 사업 근로자가 섞인 공단 원본은 비공개 스토리지에 격리, 열람 권한 제한 |
| 트랜잭션 | 업로드·대량 계산은 트랜잭션으로 묶고 오류 시 롤백 |
| 감사 로그 | 모든 수정·확정에 수정자, 일시, 변경 전/후 값, 사유를 불변 기록 |
| 챗봇 | 지침 문서만 검색. 개인정보·급여 데이터는 LLM에 보내지 않음 |
| 개발 | 실제 개인정보·업무 파일은 저장소에 올리지 않음. 테스트는 가짜 데이터 |

---

## 12. 기술 제약 T1~T4

| 코드 | 제약 | 대응 |
| :---: | :--- | :--- |
| **T1** | 사업별 데이터 격리와 세부 권한 | 서버 측 RBAC, 객체 단위 권한 검사 |
| **T2** | 엑셀 파서는 수식을 계산하지 않음 | 캐시된 계산값을 믿지 않고 서버에서 재계산 |
| **T3** | 부동소수점 오차 | Decimal 연산 + DB `NUMERIC` |
| **T4** | OCR 판독 불확실성 | 인쇄 글자·숫자·체크·서명을 분리 검증, 신뢰도만으로 확정 금지 |
