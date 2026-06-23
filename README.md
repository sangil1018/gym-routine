# 🏋️ FLUXION GYM — 스마트 루틴 트래커

개인 헬스 루틴을 관리하는 단일 HTML 파일 앱입니다.  
GitHub Pages로 호스팅하고 Google Apps Script로 기기 간 데이터를 동기화합니다.

---

## 주요 기능

- **2주 반복 루틴** — Week 1 / Week 2 탭으로 구분, 월·화·목·금 4일 운동
- **PT 연동 모드** — PT 요일·부위를 주차별로 자유 설정, 자체운동일 자동 구성
- **주말 야외 러닝** — 토·일 러닝 카드 자동 추가 (워밍업·쿨다운 포함)
- **4부위 배분 트래커** — 등·어깨·가슴·하체가 2주 PT에 고르게 배분됐는지 실시간 표시
- **TODAY 감지** — 오늘 요일에 해당하는 카드 자동 강조
- **진행률 바** — 전체·주차별 완료율 실시간 표시
- **완료 컨페티** — 하루 루틴 완료 시 축하 효과
- **연속 달성 스트릭** — 주 단위 연속 달성 횟수 기록
- **☁ 클라우드 동기화** — Google Apps Script로 모든 기기에서 데이터 공유
- **완전 오프라인 지원** — localStorage로 로컬 백업 병행

---

## 파일 구성

```
gym-routine/
├── index.html          # 앱 전체 (HTML + CSS + JS 단일 파일)
└── README.md           # 이 파일
```

---

## 배포 가이드

### 1단계 — Google Apps Script 설정 (클라우드 동기화용)

#### 1-1. Google Sheets 열기
[sheets.google.com](https://sheets.google.com) → 새 스프레드시트 만들기

#### 1-2. Apps Script 에디터 열기
상단 메뉴 → **확장 프로그램** → **Apps Script**

#### 1-3. 코드 붙여넣기
에디터의 기존 코드를 전체 삭제하고 아래 코드를 붙여넣기:

```javascript
const ss = SpreadsheetApp.getActiveSpreadsheet();

function doGet(e) {
  const sheet = ss.getSheetByName('sync') || ss.insertSheet('sync');
  if (e.parameter.action === 'save') {
    sheet.getRange('A1').setValue(e.parameter.data);
    return ContentService.createTextOutput('saved');
  }
  const data = sheet.getRange('A1').getValue();
  return ContentService.createTextOutput(data || '{}')
    .setMimeType(ContentService.MimeType.JSON);
}
```

#### 1-4. 저장 및 배포
1. `Ctrl + S` 로 저장 (프로젝트 이름: `gym-sync` 등 원하는 이름)
2. 우측 상단 **배포** 버튼 클릭 → **새 배포**
3. 유형 선택: ⚙️ **웹 앱**
4. 설정:
   - 설명: `gym sync`
   - 다음 사용자로 실행: **나 (본인 계정)**
   - 액세스 권한: **모든 사용자**
5. **배포** 클릭 → Google 계정 권한 허용
6. 생성된 **웹 앱 URL** 복사 (형식: `https://script.google.com/macros/s/...../exec`)

> ⚠️ URL은 외부에 공유하지 마세요. 이 URL이 곧 데이터 접근 키입니다.

---

### 2단계 — index.html에 URL 연결

`index.html` 파일을 텍스트 에디터(메모장, VS Code 등)로 열고  
`const SYNC_URL=` 으로 검색하여 URL을 확인합니다.

```javascript
// 이미 설정된 경우:
const SYNC_URL='https://script.google.com/macros/s/여기에URL/exec';
```

URL이 비어 있다면 위 형식으로 채워넣고 저장합니다.

---

### 3단계 — GitHub Pages 배포

#### 3-1. GitHub 저장소 만들기
1. [github.com](https://github.com) 로그인 (계정 없으면 무료 가입)
2. 우측 상단 **+** → **New repository**
3. 설정:
   - Repository name: `gym-routine` (원하는 이름)
   - Public 선택 (Pages 무료 사용 조건)
   - **Create repository** 클릭

#### 3-2. 파일 업로드
1. 저장소 메인 페이지 → **Add file** → **Upload files**
2. `index.html` 파일 드래그 앤 드롭
3. 하단 **Commit changes** 클릭

#### 3-3. GitHub Pages 활성화
1. 저장소 상단 **Settings** 탭
2. 좌측 메뉴 → **Pages**
3. Source 섹션: **Branch → main** 선택 → **Save**
4. 1~2분 후 상단에 배포 URL 표시됨

```
https://[GitHub아이디].github.io/gym-routine
```

#### 3-4. 접속 확인
브라우저에서 위 URL 접속 → 앱이 정상 로딩되면 완료 🎉

---

### 4단계 — 다기기 동기화 사용법

| 상황 | 동작 |
|------|------|
| 페이지 열기 | 자동으로 클라우드 데이터 로드 (더 최신 데이터 우선) |
| 체크/설정 변경 | 1.8초 후 자동으로 Google Sheets에 저장 |
| 오프라인 상태 | localStorage에 저장, 온라인 복귀 시 다음 변경 때 동기화 |

화면 우하단 뱃지로 동기화 상태를 확인할 수 있습니다:
- 🟡 `동기화 중...` — 저장 진행 중
- 🟢 `☁ 저장됨 14:32` — 마지막 저장 시각
- 🔴 `동기화 오류` — 네트워크 또는 스크립트 오류

---

## PT 연동 설정 방법

1. 상단 **PT 연동 설정** 패널 → 토글 ON
2. WEEK 1 / WEEK 2 각각:
   - **PT 세션 ①** — 요일 선택 (월/화/목/금) + 운동 부위 선택
   - **PT 세션 ②** — 요일 선택 + 운동 부위 선택
3. 하단 **4부위 배분 현황**에서 등·어깨·가슴·하체가 고르게 배분됐는지 확인
4. **자체운동 자동 구성** 미리보기에서 화/금 자동 생성 내용 확인

> PT 요일로 지정된 날: 트레드밀 15분(워밍업) → PT 세션 → 스트레칭  
> 나머지 운동일: 트레드밀 25분 → PT 보완 웨이트 → 코어 → 스트레칭

---

## 주말 러닝 설정 방법

1. **주말 야외 러닝** 패널 → 토글 ON
2. 토요일 / 일요일 중 원하는 요일 선택 (복수 선택 가능)
3. 해당 요일 카드 자동 추가: 동적 워밍업 5분 → 야외 러닝 1시간 → 정적 쿨다운 10분

---

## 기술 스택

| 항목 | 내용 |
|------|------|
| 프론트엔드 | 순수 HTML / CSS / JavaScript (프레임워크 없음) |
| 로컬 저장소 | `localStorage` (키: `fluxion_v5`) |
| 클라우드 동기화 | Google Apps Script Web App + Google Sheets |
| 호스팅 | GitHub Pages (무료) |
| 외부 의존성 | 없음 (완전 자급자족 단일 파일) |

---

## 문제 해결

**동기화 오류가 표시될 때**
- Apps Script 배포 URL이 올바른지 확인
- Apps Script 액세스 권한이 "모든 사용자"인지 확인
- Google 계정 권한 재허용: Apps Script → 배포 → 배포 관리 → 새 버전 배포

**데이터가 기기 간에 다를 때**
- 두 기기 모두 온라인 상태에서 페이지 새로고침
- 타임스탬프 기반으로 더 최신 데이터가 자동 적용됨

**새 달 초기화**
- 매월 1일 접속 시 초기화 배너 자동 표시
- "초기화" 버튼 클릭 시 체크 기록만 삭제 (설정값 유지)
- 2주 완주 기록은 스트릭으로 누적 보존

---

## 라이선스

개인 사용 목적으로 자유롭게 수정 가능합니다.
