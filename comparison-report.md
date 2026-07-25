# 프로젝트 1 비교 분석 보고서

## 1. 과제 개요

- **과제명**: 동일한 자동화 워크플로우를 서로 다른 2개의 자동화 도구로 구현하고 비교 분석
- **작성자**: 홍길동
- **작성일**: 2026-07-25
- **비교 대상 도구**:
  - 도구 A: Make
  - 도구 B: Zapier

---

## 2. 구현한 자동화 워크플로우 설명

### 2-1. 워크플로우 주제
> Google Sheets에 새 행이 추가되면 우선순위에 따라 다른 알림을 보내고, 처리 결과를 다시 시트에 기록하는 자동화

- **주제**: Google Sheets 새 행 추가 시 우선순위별 알림 보내기
- **자동화 목적**:
  - 새 요청이 들어왔을 때 사람이 직접 확인하지 않아도 자동으로 알림을 보낼 수 있도록 한다.
  - `긴급` 요청과 `일반` 요청을 구분하여 서로 다른 방식으로 처리한다.
  - 처리 결과를 시트에 남겨 실행 여부를 쉽게 확인한다.
- **사용한 외부 서비스**:
  - Google Sheets
  - Gmail
  - Make / Zapier

### 2-2. 전체 흐름
1. Trigger 발생: Google Sheets에 새 행 추가
2. 데이터 조회/수집: 새로 들어온 요청의 제목, 내용, 우선순위, 요청자 이메일 확인
3. 조건 분기: `우선순위` 값이 `긴급`인지 확인
4. 분기 A 실행: 긴급이면 제목 앞에 `[긴급]`을 붙여 즉시 이메일 알림 전송
5. 분기 B 실행: 일반이면 제목 앞에 `[일반]`을 붙여 일반 알림 이메일 전송
6. 결과 저장: 처리 완료 시간과 처리 상태를 Google Sheets에 기록

---

## 3. 사전 준비

### 3-1. Google Sheets 구조
자동화 테스트를 위해 아래와 같은 시트를 만들었다.

- **스프레드시트 이름**: `Automation_Assignment_Project1`
- **시트 이름**: `Requests`

### 3-2. 컬럼 구성

| 컬럼명 | 설명 |
|---|---|
| ID | 요청 번호 |
| Request Title | 요청 제목 |
| Priority | 우선순위 (`긴급` 또는 `일반`) |
| Requester Email | 요청자 이메일 |
| Details | 요청 내용 |
| Status | 처리 상태 |
| Processed At | 처리 시간 |

### 3-3. 테스트 데이터 예시

| ID | Request Title | Priority | Requester Email | Details | Status | Processed At |
|---|---|---|---|---|---|---|
| 1 | 로그인 오류 문의 | 긴급 | user1@example.com | 로그인 불가 문제 발생 |  |  |
| 2 | 배송 일정 확인 | 일반 | user2@example.com | 배송 일정 문의 |  |  |

- **1번 행**은 긴급 분기 테스트용
- **2번 행**은 일반 분기 테스트용

---

## 4. 도구별 구현 내용

## 4-1. 도구 A 구현

- **도구명**: Make
- **구현 링크 또는 참고 정보**: Make Scenario 사용
- **사용한 Trigger**: Google Sheets - Watch New Rows
- **사용한 Action**:
  - Action 1: Gmail - Send an Email
  - Action 2: Google Sheets - Update a Row
- **사용한 조건 분기(Filter/Router)**:
  - Router 사용
  - 분기 A: `Priority = 긴급`
  - 분기 B: `Priority = 일반`

### 구현 과정 요약
1. Google Sheets의 `Requests` 시트에 새 행이 추가되면 Trigger가 실행되도록 설정했다.
2. Router를 추가하여 `Priority` 값이 `긴급`인지 `일반`인지에 따라 경로를 나누었다.
3. 각 경로에서 Gmail 모듈로 다른 제목의 이메일을 전송하도록 설정했다.
4. 이메일 전송 후 Google Sheets의 해당 행을 업데이트하여 `Status`와 `Processed At` 값을 기록했다.

### Make 시나리오 구성
1. **Google Sheets - Watch New Rows**
2. **Router**
   - 경로 1: Priority = 긴급
     - Gmail - Send an Email
     - Google Sheets - Update a Row
   - 경로 2: Priority = 일반
     - Gmail - Send an Email
     - Google Sheets - Update a Row

### Make에서 사용한 메시지 예시

#### 긴급 분기 이메일
- 제목: `[긴급] 새로운 요청이 접수되었습니다: {{Request Title}}`
- 본문:
- 긴급 요청이 접수되었습니다.

요청 제목: {{Request Title}}
요청자: {{Requester Email}}
내용: {{Details}}
우선순위: {{Priority}}


#### 일반 분기 이메일
- 제목: `[일반] 새로운 요청이 접수되었습니다: {{Request Title}}`
- 본문:
- 일반 요청이 접수되었습니다.

요청 제목: {{Request Title}}
요청자: {{Requester Email}}
내용: {{Details}}
우선순위: {{Priority}}


### 실행 결과 요약
- **분기 A 결과**:
- `Priority = 긴급` 데이터 입력 시 긴급 이메일이 정상 발송되었다.
- Google Sheets에 `Status = 긴급 알림 완료`, `Processed At = 실행 시간`이 기록되었다.
- **분기 B 결과**:
- `Priority = 일반` 데이터 입력 시 일반 이메일이 정상 발송되었다.
- Google Sheets에 `Status = 일반 알림 완료`, `Processed At = 실행 시간`이 기록되었다.

---

## 4-2. 도구 B 구현

- **도구명**: Zapier
- **구현 링크 또는 참고 정보**: Zap 사용
- **사용한 Trigger**: Google Sheets - New Spreadsheet Row
- **사용한 Action**:
- Action 1: Filter by Zapier
- Action 2: Gmail - Send Email
- Action 3: Google Sheets - Update Spreadsheet Row
- **사용한 조건 분기(Filter/Paths)**:
- Paths by Zapier 사용
- Path A: `Priority = 긴급`
- Path B: `Priority = 일반`

### 구현 과정 요약
1. Google Sheets의 새 행을 감지하는 Trigger를 연결했다.
2. `Paths by Zapier`를 이용해 `Priority` 값에 따라 경로를 분기했다.
3. 각 경로에서 Gmail 액션을 사용해 서로 다른 제목의 이메일을 보내도록 설정했다.
4. 마지막으로 Google Sheets 행 업데이트 액션을 추가하여 처리 상태를 기록했다.

### Zap 구성
1. **Google Sheets - New Spreadsheet Row**
2. **Paths by Zapier**
 - Path A: Priority = 긴급
   - Gmail - Send Email
   - Google Sheets - Update Spreadsheet Row
 - Path B: Priority = 일반
   - Gmail - Send Email
   - Google Sheets - Update Spreadsheet Row

### Zapier에서 사용한 메시지 예시

#### 긴급 분기 이메일
- 제목: `[긴급] 새로운 요청이 접수되었습니다: {{Request Title}}`

#### 일반 분기 이메일
- 제목: `[일반] 새로운 요청이 접수되었습니다: {{Request Title}}`

### 실행 결과 요약
- **분기 A 결과**:
- 긴급 행 추가 시 Path A가 실행되었고, 긴급 이메일 전송이 성공했다.
- 상태값이 `긴급 알림 완료`로 업데이트되었다.
- **분기 B 결과**:
- 일반 행 추가 시 Path B가 실행되었고, 일반 이메일 전송이 성공했다.
- 상태값이 `일반 알림 완료`로 업데이트되었다.

---

## 5. 실행 결과 확인

### 5-1. 조건 분기 실행 확인

| 항목 | Make | Zapier |
|---|---|---|
| Trigger 실행 확인 | 확인 완료 | 확인 완료 |
| 분기 A 실행 확인 (`긴급`) | 확인 완료 | 확인 완료 |
| 분기 B 실행 확인 (`일반`) | 확인 완료 | 확인 완료 |
| Action 2개 이상 실행 확인 | 확인 완료 | 확인 완료 |
| 자동 실행 확인 | 확인 완료 | 확인 완료 |

### 5-2. 실제 테스트 내용

#### 테스트 1: 긴급 요청 입력
- 입력값:
- Request Title: 로그인 오류 문의
- Priority: 긴급
- Requester Email: user1@example.com
- 기대 결과:
- 긴급 이메일 발송
- 시트 상태값 `긴급 알림 완료`
- 실제 결과:
- Make: 정상 동작
- Zapier: 정상 동작

#### 테스트 2: 일반 요청 입력
- 입력값:
- Request Title: 배송 일정 확인
- Priority: 일반
- Requester Email: user2@example.com
- 기대 결과:
- 일반 이메일 발송
- 시트 상태값 `일반 알림 완료`
- 실제 결과:
- Make: 정상 동작
- Zapier: 정상 동작

### 5-3. 첨부한 캡처 목록
- [x] 전체 워크플로우 화면
- [x] Trigger 설정 화면
- [x] 조건 분기 설정 화면
- [x] Action 설정 화면
- [x] 실행 로그 화면
- [x] 분기 A 실행 결과 화면
- [x] 분기 B 실행 결과 화면

---

## 6. 비교 분석

아래 항목을 기준으로 Make와 Zapier를 비교하였다.

| 비교 항목 | Make | Zapier | 비교 결과 요약 |
|---|---|---|---|
| UI/UX | 시나리오가 시각적으로 연결되어 흐름 파악이 쉬움 | 단계별 폼 입력 방식으로 깔끔함 | Make는 흐름 시각화에 강하고, Zapier는 단계별 설정이 단순함 |
| 설정 난이도 | 초반에는 Router와 필드 매핑이 약간 어렵게 느껴짐 | 초보자도 따라가기 쉬움 | Zapier가 처음 사용하는 입문자에게 더 쉬움 |
| 무료 플랜 범위 | 무료 플랜으로 기본 테스트 가능 | 무료 플랜도 가능하지만 기능 제한이 더 체감됨 | 간단한 과제 테스트는 둘 다 가능하나 Make가 조금 더 유연하게 느껴짐 |
| 연동 범위 | 매우 다양함 | 매우 다양하며 대표 서비스가 많음 | 둘 다 충분하지만 Zapier는 대중적 연동이 강점 |
| 조건 분기 구성 편의성 | Router + Filter 방식으로 직관적 | Paths 설정이 명확하지만 단계가 늘어남 | 분기 구조 자체는 Make가 더 시각적으로 이해하기 쉬움 |
| 실행 로그 확인 방식 | 각 모듈별 실행 결과 확인이 편리함 | Task 히스토리 중심으로 확인 가능 | 디버깅은 Make가 더 자세하게 보였음 |
| 디버깅 편의성 | 어느 모듈에서 실패했는지 찾기 쉬움 | 단계별 결과는 좋지만 전체 흐름 추적은 Make보다 약함 | Make 우세 |
| 초보자 적합성 | 자동화 개념을 알면 매우 좋음 | 완전 초보자가 시작하기 더 쉬움 | Zapier 우세 |

---

## 7. 장점과 단점

## 7-1. Make

### 장점
- 워크플로우 전체 구조를 한눈에 보기 쉽다.
- Router를 통해 조건 분기를 시각적으로 구성하기 편하다.
- 실행 로그가 상세해서 디버깅하기 좋다.

### 단점
- 처음 사용하는 경우 인터페이스가 다소 복잡하게 느껴질 수 있다.
- 필드 매핑 개념을 이해해야 설정이 편하다.
- 초보자는 모듈 연결 구조에 적응 시간이 필요하다.

---

## 7-2. Zapier

### 장점
- 단계별로 설정을 따라가기 쉬워 입문자에게 친숙하다.
- 대표적인 외부 서비스와의 연결이 편리하다.
- 테스트 버튼과 설정 흐름이 단순하다.

### 단점
- 분기가 추가되면 화면이 길어져 전체 흐름 파악이 어려워질 수 있다.
- 복잡한 로직은 Make보다 답답하게 느껴질 수 있다.
- 무료 플랜에서 제한을 더 크게 체감할 수 있다.

---

## 8. 어떤 상황에서 적합한지

### Make가 적합한 경우
- 여러 단계와 분기가 있는 자동화를 설계할 때
- 실행 로그를 자세히 보면서 디버깅해야 할 때
- 시각적인 흐름 중심으로 자동화를 관리하고 싶을 때

### Zapier가 적합한 경우
- 빠르게 간단한 자동화를 만들고 싶을 때
- 자동화 도구를 처음 배우는 입문자일 때
- 대표적인 SaaS 서비스 위주로 연결할 때

---

## 9. 최종 의견

### 9-1. 더 사용하기 쉬웠던 도구
- Zapier

### 9-2. 더 확장성이 좋다고 생각한 도구
- Make

### 9-3. 내가 실제 업무에 사용하고 싶은 도구
- Make

### 9-4. 선택 이유
이번 과제에서는 두 도구 모두 동일한 자동화를 구현할 수 있었지만, 전체 워크플로우를 한눈에 보고 조건 분기를 관리하는 측면에서는 Make가 더 편리했다. 반면 Zapier는 처음 설정할 때 더 쉽고 빠르게 작업할 수 있었다. 실제 업무에서 자동화가 점점 복잡해질 가능성을 고려하면 Make를 선택하는 것이 더 적합하다고 판단했다.

---

## 10. 느낀 점

- **이번 과제를 통해 배운 점**:
- 같은 자동화라도 도구에 따라 구성 방식과 사용 경험이 다를 수 있다는 점을 배웠다.
- Trigger, Action, 조건 분기를 조합하면 간단한 업무도 충분히 자동화할 수 있다는 점을 확인했다.
- 자동화 도구를 선택할 때는 기능뿐 아니라 디버깅 편의성과 확장성도 중요하다는 점을 느꼈다.

- **어려웠던 점**:
- 처음에는 각 도구에서 어떤 모듈이 Trigger이고 어떤 모듈이 Action인지 구분하는 것이 헷갈렸다.
- Google Sheets 행 업데이트 시 어떤 행을 다시 수정해야 하는지 매핑하는 과정이 조금 어려웠다.

- **해결 방법**:
- 테스트용 데이터를 2개 이상 넣어 분기별 실행 결과를 직접 확인했다.
- 실행 로그를 보고 Row ID 또는 Row Number 값을 기준으로 업데이트 대상을 정확히 지정했다.

---

## 11. 보안 및 민감정보 처리

- [x] API Key 마스킹 완료
- [x] 계정 정보 마스킹 완료
- [x] 이메일 주소 일부 마스킹 완료
- [x] 토큰/비밀번호 노출 없음

---

## 12. 첨부 파일/폴더 경로

- Make 캡처: `project1/screenshots/make/`
- Zapier 캡처: `project1/screenshots/zapier/`
- 비교 분석 보고서: `project1/comparison-report.md`

## 프로젝트1 폴더

project1/
├─ comparison-report.md
└─ screenshots/
   ├─ make/
   │  ├─ 01-overall-scenario.png
   │  �
