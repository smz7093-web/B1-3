# 프로젝트 1 비교 분석 보고서

## 1. 과제 개요

- **과제명**: 동일한 자동화 워크플로우를 서로 다른 2개의 자동화 도구로 구현하고 비교 분석
- **작성자**: 장상민
- **작성일**: 2026-07-25
- **비교 대상 도구**:
  - 도구 A: Make
  - 도구 B: Zapier

---

## 2. 구현한 자동화 워크플로우 설명

### 2-1. 워크플로우 주제
> Google Form으로 요청을 입력받고, 응답이 Google Sheets에 자동 저장되면 우선순위에 따라 이메일 알림을 보내는 자동화

- **주제**: Google Form 응답 기반 우선순위별 이메일 알림 자동화
- **자동화 목적**:
  - 사용자가 Google Form에 요청을 제출하면 자동으로 접수되도록 한다.
  - 응답이 Google Sheets에 저장되면 우선순위(`긴급`, `일반`)에 따라 다른 이메일 알림을 전송한다.
  - 처리 결과를 다시 스프레드시트에 기록하여 실행 여부를 쉽게 확인할 수 있도록 한다.
- **사용한 외부 서비스**:
  - Google Forms
  - Google Sheets
  - Gmail
  - Make / Zapier

### 2-2. 전체 흐름
1. 사용자가 Google Form에 요청 내용을 입력하고 제출한다.
2. Google Form 응답이 연결된 Google Sheets에 자동으로 저장된다.
3. 자동화 도구가 응답 시트의 새 행을 감지한다.
4. `Priority` 값이 `긴급`인지 `일반`인지 조건 분기한다.
5. 긴급이면 `[긴급]` 제목의 이메일을 전송한다.
6. 일반이면 `[일반]` 제목의 이메일을 전송한다.
7. 처리 완료 후 `Status`, `Processed At` 값을 스프레드시트에 기록한다.

---

## 3. 사전 준비

### 3-1. Google Form 구성
자동화 테스트를 위해 요청 접수용 Google Form을 만들었다.

- **폼 이름**: `시스템 오류 접수 양식`

### 3-2. Google Form 질문 항목

| 항목 | 설명 |
|---|---|
| Request Title | 요청 제목 |
| Priority | 우선순위 (`긴급` / `일반`) |
| Requester Email | 요청자 이메일 |
| Details | 요청 내용 |

### 3-3. Google Sheets 구조
Google Form 응답을 자동 저장하도록 스프레드시트를 연결하였다.

- **스프레드시트 이름**: `Automation_Assignment_Project1`
- **시트 이름**: `Form Responses 1`

### 3-4. 응답 시트 컬럼 구성

| 컬럼명 | 설명 |
|---|---|
| Timestamp | 폼 제출 시간 |
| Request Title | 요청 제목 |
| Priority | 우선순위 |
| Requester Email | 요청자 이메일 |
| Details | 요청 내용 |
| Status | 처리 상태 |
| Processed At | 처리 시간 |

### 3-5. 테스트 데이터 예시

| Timestamp | Request Title | Priority | Requester Email | Details | Status | Processed At |
|---|---|---|---|---|---|---|
| 2026-07-25 14:10 | 로그인 오류 문의 | 긴급 | user1@example.com | 로그인 불가 문제 발생 |  |  |
| 2026-07-25 14:15 | 배송 일정 확인 | 일반 | user2@example.com | 배송 일정 문의 |  |  |

- 첫 번째 응답은 긴급 분기 테스트용
- 두 번째 응답은 일반 분기 테스트용

---

## 4. 도구별 구현 내용

## 4-1. 도구 A 구현

- **도구명**: Make
- **구현 방식**: Google Form 응답이 저장되는 Google Sheets의 새 행을 감지하는 시나리오 구성
- **사용한 Trigger**: Google Sheets - Watch New Rows
- **사용한 Action**:
  - Action 1: Gmail - Send an Email
  - Action 2: Google Sheets - Update a Row
- **사용한 조건 분기(Router)**:
  - Router 사용
  - 분기 A: `Priority = 긴급`
  - 분기 B: `Priority = 일반`

### 구현 과정 요약
1. Google Form을 Google Sheets와 연결하여 응답이 자동 저장되도록 설정했다.
2. Make에서 응답 시트(`Form Responses 1`)의 새 행을 감지하도록 Trigger를 설정했다.
3. Router를 추가해 `Priority` 값이 `긴급`인지 `일반`인지에 따라 경로를 분기했다.
4. 각 경로에서 Gmail 모듈로 서로 다른 제목의 이메일을 전송하도록 설정했다.
5. 이메일 전송 후 응답 시트의 해당 행에 `Status`와 `Processed At` 값을 업데이트했다.

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

긴급 요청이 Google Form을 통해 접수되었습니다.

요청 제목: {{Request Title}}
요청자: {{Requester Email}}
요청 내용: {{Details}}
우선순위: {{Priority}}
제출 시간: {{Timestamp}}

#### 일반 분기 이메일
- 제목: `[일반] 새로운 요청이 접수되었습니다: {{Request Title}}`
- 본문:

일반 요청이 Google Form을 통해 접수되었습니다.

요청 제목: {{Request Title}}
요청자: {{Requester Email}}
요청 내용: {{Details}}
우선순위: {{Priority}}
제출 시간: {{Timestamp}}


### 실행 결과 요약
- **분기 A 결과**:
- 사용자가 Google Form에 `Priority = 긴급` 요청을 제출하면 긴급 이메일이 정상 발송되었다.
- 응답 시트에 `Status = 긴급 알림 완료`, `Processed At = 실행 시간`이 기록되었다.
- **분기 B 결과**:
- 사용자가 Google Form에 `Priority = 일반` 요청을 제출하면 일반 이메일이 정상 발송되었다.
- 응답 시트에 `Status = 일반 알림 완료`, `Processed At = 실행 시간`이 기록되었다.

---

## 4-2. 도구 B 구현

- **도구명**: Zapier
- **구현 방식**: Google Form 응답이 저장되는 Google Sheets 행을 기준으로 Zap 실행
- **사용한 Trigger**: Google Sheets - New Spreadsheet Row
- **사용한 Action**:
- Action 1: Paths by Zapier
- Action 2: Gmail - Send Email
- Action 3: Google Sheets - Update Spreadsheet Row
- **사용한 조건 분기(Paths)**:
- Path A: `Priority = 긴급`
- Path B: `Priority = 일반`

### 구현 과정 요약
1. Google Form 응답이 저장되는 Google Sheets를 Zapier와 연결했다.
2. 새 행이 추가되면 Zap이 실행되도록 Trigger를 설정했다.
3. `Paths by Zapier`를 사용해 `Priority` 값에 따라 긴급/일반 경로를 분기했다.
4. 각 경로에서 Gmail 액션으로 서로 다른 제목의 이메일을 전송했다.

### Zap 구성
1. **Google Sheets - New Spreadsheet Row**
2. **Paths by Zapier**
 - Path A: Priority = 긴급
   - Gmail - Send Email
 - Path B: Priority = 일반
   - Gmail - Send Email

### Zapier에서 사용한 메시지 예시

#### 긴급 분기 이메일
- 제목: `[긴급] 새로운 폼 요청이 접수되었습니다: {{Request Title}}`

#### 일반 분기 이메일
- 제목: `[일반] 새로운 폼 요청이 접수되었습니다: {{Request Title}}`

### 실행 결과 요약
- **분기 A 결과**:
- Google Form에 긴급 요청 제출 시 Path A가 실행되었고 긴급 이메일 전송이 성공했다.
- 응답 시트 상태값이 `긴급 알림 완료`로 업데이트되었다.
- **분기 B 결과**:
- Google Form에 일반 요청 제출 시 Path B가 실행되었고 일반 이메일 전송이 성공했다.
- 응답 시트 상태값이 `일반 알림 완료`로 업데이트되었다.

---

## 5. 실행 결과 확인

### 5-1. 조건 분기 실행 확인

| 항목 | Make | Zapier |
|---|---|---|
| Google Form 제출 확인 | 확인 완료 | 확인 완료 |
| Google Sheets 자동 저장 확인 | 확인 완료 | 확인 완료 |
| 분기 A 실행 확인 (`긴급`) | 확인 완료 | 확인 완료 |
| 분기 B 실행 확인 (`일반`) | 확인 완료 | 확인 완료 |
| Action 2개 이상 실행 확인 | 확인 완료 | 확인 완료 |
| 자동 실행 확인 | 확인 완료 | 확인 완료 |

### 5-2. 실제 테스트 내용

#### 테스트 1: 긴급 요청 폼 제출
- 입력값:
- Request Title: 로그인 오류 문의
- Priority: 긴급
- Requester Email: user1@example.com
- Details: 로그인 불가 문제 발생
- 기대 결과:
- Google Sheets 응답 행 자동 생성
- 긴급 이메일 발송
- 시트 상태값완료  `
- 실제 결과:
- Make: 정상 동작
- Zapier: 정상 동작

#### 테스트 2: 일반 요청 폼 제출
- 입력값:
- Request Title: 배송 일정 확인
- Priority: 일반
- Requester Email: user2@example.com
- Details: 배송 일정 문의
- 기대 결과:
- Google Sheets 응답 행 자동 생성
- 일반 이메일 발송
- 시트 상태값 `일반 알림 완료`
- 실제 결과:
- Make: 정상 동작
- Zapier: 정상 동작

### 5-3. 첨부한 캡처 목록
- [x] Google Form 입력 화면
- [x] Google Sheets 응답 저장 화면
- [x] 전체 워크플로우 화면
- [x] 조건 분기 설정 화면
- [x] Action 설정 화면

---

## 6. 비교 분석

| 비교 항목 | Make | Zapier | 비교 결과 요약 |
|---|---|---|---|
| UI/UX | 시나리오 연결 구조가 시각적이라 전체 흐름 파악이 쉬움 | 단계별 설정 화면이 단순하고 익숙함 | Make는 흐름 이해, Zapier는 빠른 설정에 유리 |
| 설정 난이도 | Form 응답 시트 기반 매핑과 Router 설정이 조금 더 세밀함 | 초보자가 따라가기 쉬운 편 | Zapier가 입문자에게 더 쉬움 |
| 무료 플랜 범위 | 과제 수준 테스트 가능 | 과제 수준 테스트 가능하나 제한 체감 있음 | Make가 약간 더 유연하게 느껴짐 |
| 조건 분기 편의성 | Router + Filter 조합이 시각적임 | Paths 설정이 명확함 | 분기 구조 이해는 Make가 쉬웠음 |
| 실행 로그 | 모듈별 데이터 확인이 편리함 | 실행 히스토리 확인은 가능하지만 흐름 추적은 상대적으로 단순함 | 디버깅은 Make가 유리 |
| Google 서비스 연동 | Form 응답 시트와 Gmail 연동이 안정적 | Google 서비스 연동도 쉬움 | 둘 다 무난함 |
| 초보자 적합성 | 자동화 개념을 알면 강력함 | 처음 시작할 때 더 편함 | Zapier 우세 |

---

## 7. 장점과 단점

## 7-1. Make

### 장점
- Google Form → Google Sheets → 이메일 흐름을 시각적으로 보기 좋다.
- Router로 긴급/일반 분기를 명확하게 구성할 수 있다.
- 실행 로그에서 어떤 응답 행이 어떻게 처리됐는지 확인하기 쉽다.

### 단점
- 처음 사용하는 경우 필드 매핑이 조금 어렵게 느껴질 수 있다.
- 응답 시트의 행 업데이트 시 Row Number 매핑을 정확히 해야 한다.

---

## 7-2. Zapier

### 장점
- 단계별 설정이 단순해서 빠르게 구현 가능하다.
- Google Sheets 기반 트리거 설정이 쉬운 편이다.
- 폼 응답 자동화를 입문자가 이해하기 좋다.

### 단점
- 분기가 늘어나면 경로 구조가 길어질 수 있다.
- 복잡한 자동화 확장 시 Make보다 답답할 수 있다.

---

## 8. 어떤 상황에서 적합한지

### Make가 적합한 경우
- Google Form 응답을 여러 조건으로 나누어 처리해야 할 때
- 실행 로그를 상세하게 보며 디버깅해야 할 때
- 시각적인 자동화 흐름 관리가 중요할 때

### Zapier가 적합한 경우
- Google Form 기반 자동화를 빠르게 만들고 싶을 때
- 자동화 도구를 처음 배우는 입문자일 때
- 간단한 SaaS 연동을 중심으로 사용할 때

---

## 9. 최종 의견

### 9-1. 더 사용하기 쉬웠던 도구
- Zapier

### 9-2. 더 확장성이 좋다고 생각한 도구
- Make

### 9-3. 실제 업무에 사용하고 싶은 도구
- Make

### 9-4. 선택 이유
이번 과제에서는 Google Form 입력을 시작점으로 하여 응답이 자동 저장된 뒤 분기 처리되는 구조를 두 도구로 모두 구현할 수 있었다. Zapier는 단계별 설정이 쉬워 빠르게 완성할 수 있었고, Make는 전체 흐름과 분기 구조를 시각적으로 관리하기 더 좋았다. 실제 업무에서 요청 유형이 늘어나거나 처리 단계가 복잡해질 가능성을 고려하면 Make가 더 적합하다고 판단했다.

---

## 10. 느낀 점

- **이번 과제를 통해 배운 점**:
- Google Form은 입력 인터페이스 역할을 하고, Google Sheets는 데이터 저장 및 후속 자동화의 기준이 될 수 있다는 점을 배웠다.
- 같은 폼 응답 기반 자동화라도 도구에 따라 조건 분기와 디버깅 방식이 다르다는 점을 확인했다.
- 입력, 저장, 분기, 알림, 기록까지 하나의 흐름으로 연결하면 반복 업무를 크게 줄일 수 있다는 점을 느꼈다.

- **어려웠던 점**:
  router를 이용한 분기 처리 방식의 이해에 어려움이 있었다.

