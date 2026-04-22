# 매장 매출 관리 (Boss-Cloth-Sales) — 프로젝트 메모

## 파일 위치
- `Boss-Cloth-Sales/index.html` (서버 없이 브라우저로 바로 열기)

## 원본과의 차이점
| 항목 | 원본 (Cloth-Sales-Report) | 신규 (Boss-Cloth-Sales) |
|---|---|---|
| 페이지 타이틀 | 🧥 의류 매출 관리 | 🏪 매장 매출 관리 |
| 지점 | 부평 / 영등포 | **하남 / 영등포** |
| 추가 변동지출 | — | **💜 주 결제 (금요일에만 표시)** |
| 비밀번호 | 1225 | **0925** |
| Firestore 컬렉션 | `cloth-sales` | `boss-cloth-sales` |
| localStorage 키 | `cloth-sales-data` | `boss-cloth-sales-data` |
| 인증 해시 salt | `cloth-sales-secure-v2` | `boss-sales-secure-v2` |
| 세션 키 | `cloth-auth` | `boss-auth` |

Firebase 프로젝트는 원본과 **동일**하지만 컬렉션이 분리되어 있어 데이터가 섞이지 않습니다.

## 기능
- 월별 캘린더 뷰 (← → 화살표로 월 이동)
- 날짜 클릭 → 카드/이체/현금 매출 + 매입/관리비/기타 변동지출 입력
- **금요일**을 클릭하면 입력 모달에 `💜 주 결제` 항목이 자동 표시됨 (다른 요일엔 숨김)
- 지점 탭 (하남 / 영등포) — 지점별 독립 데이터
- 일 합계 / 주간 / 월간 합계 실시간 자동 계산
- 고정지출 (월세 / 급여 / 세금 / 기타) 월별 입력
- 모바일 반응형 + 터치 스크롤 최적화

## 페이지 비밀번호 (클라이언트 잠금)
- 현재 비밀번호: **`0925`**
- 비밀번호는 평문이 아닌 **SHA-256 해시**로만 저장됩니다.
- **Rate limiting**: 5회 실패 시 15분 잠금 (localStorage 기록).
- **세션 만료**: 로그인 후 4시간 지나면 자동 잠금.
- **탭 복귀 감지**: 백그라운드였다가 돌아왔을 때 만료 재검사.
- 비밀번호 변경:
  ```bash
  echo -n "boss-sales-secure-v2:새비밀번호" | sha256sum
  ```
  출력된 64자 해시를 `index.html`의 `PW_HASH` 값에 붙여넣으세요.

## 데이터 형식 (Firestore & localStorage 동일)
```
컬렉션: boss-cloth-sales

일별 매출 문서 ID: "YYYY-MM-DD_지점명"
  예) "2026-04-24_하남"
  필드: { card, transfer, cash, purchase, mgmt, etcVar, weekpay? }
  (weekpay는 금요일에만 저장됨)

고정지출 문서 ID: "fixed_YYYY-MM_지점명"
  예) "fixed_2026-04_하남"
  필드: { rent, salary, vat, etc }
```

## Firestore 보안 규칙 (원본과 같은 프로젝트 기준)
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /cloth-sales/{document=**}       { allow read, write: if true; }
    match /boss-cloth-sales/{document=**}  { allow read, write: if true; }
  }
}
```
⚠️ API 키만 알면 누구나 접근 가능합니다. 실사용에서는 Firebase Auth 적용 권장:
```
allow read, write: if request.auth != null;
```

## GitHub 배포 순서 (GitHub Desktop 기준)
1. **GitHub 웹**에서 새 저장소 생성: `Boss-Cloth-Sales` (Public)
2. **GitHub Desktop**: File → Clone repository → `Boss-Cloth-Sales` 선택 → 로컬 경로 지정
3. 클론된 폴더에 `index.html` 복사 (이 폴더 전체를 통째로 넣어도 OK)
4. GitHub Desktop에서 commit (예: "Initial release") → Push origin
5. GitHub 웹 → 저장소 → Settings → Pages
   - Source: `Deploy from a branch`
   - Branch: `main` / `/(root)` → Save
6. 몇 분 후 `https://rani326.github.io/Boss-Cloth-Sales/` 로 접속

## 나중에 하고 싶은 것
- Firebase Auth 적용 (익명 or 이메일)
- 월별 / 지점별 비교 차트
- 원본과 신규 앱 간 지점 이동/분리 도구
