# 채용 플랫폼 로그 분석 (Job Platform Log Analysis)

채용 플랫폼의 원본 접근 로그(`com_2022`, `com_2023`)를 바탕으로 AARRR 프레임워크 기준
**가입 → 이력서 작성 → 지원 완료** 퍼널을 정의하고, 전환율 · 리텐션 · 이탈 지점을 분석한 프로젝트.

---

## 1. 데이터 개요

| 항목 | 내용 |
|---|---|
| 기간 | 2022-01-01 ~ 2023-12-31 |
| 로그 행 수 | 16,468,531건 |
| 고유 유저 수 | 21,340명 |
| 원본 테이블 | `com_2022`, `com_2023` (URL, timestamp, response_code 등), `application`, `company`, `job` 등 |
| 필터링 조건 | `response_code IN ('200', '302')` |

> 원본 로그 행(`df.head()` 등)과 DB 계정 정보는 레포에 포함하지 않음. 전부 집계/스키마 수준 출력만 남김.

## 2. 퍼널 정의

| 단계 | 이벤트 기준 |
|---|---|
| Acquisition (가입 완료) | `signup/step3/done`, `complete/github` (소셜 가입 포함) |
| Activation ① (이력서 작성) | `resume/step1`, `resume/step2` |
| Activation ② (지원) | `jobs/id/apply/step1~4`, `jobs/id/apply/complete` |

- 각 단계는 유저별 최초 발생 시각(`min timestamp`) 기준으로 집계
- 직전 단계보다 **이후 시점**에 발생한 경우에만 유효 전환으로 인정 (역행 전환 배제)

## 3. 주요 분석

1. **월별 가입자 추이** — 2022-01(317명) → 2023-12(31명)로 지속 감소
2. **가입 → 이력서 → 지원 단계별 전환율**
3. **전환까지 걸린 시간 분포** (가입 → 지원 완료)
4. **코호트 기반 리텐션 히트맵**
5. **Classic / Range(누적) / Rolling 리텐션 비교** — Day 1/3/7/14/30/60/90/180/365 기준
   - 가입 유저 4,746명 기준, Day 1 리텐션 38.5% → Day 30 15.1% → Day 365 1.7%
6. **원클릭 지원 기능 효과** 검증
7. **미전환 유저의 마지막 행동 분석** — URL을 기능 카테고리(가입/이력서/지원/공고탐색/기업탐색/검색/프로필/알림/설정)로 분류해 이탈 지점 파악

## 4. 폴더 구조

```
.
├── 01_eda.ipynb            # DB 연결, 테이블/스키마 확인, 기간·유저 수·URL 패턴 탐색
├── 02_preprocessing.ipynb  # 퍼널 이벤트 정의, 유저 단위 퍼널 테이블 생성, URL 카테고리 분류 함수
├── 03_analysis.ipynb       # 전환율·리텐션·코호트·이탈 분석, 인사이트 정리
├── .env.example             # DB 접속 정보 템플릿 (.env는 커밋 금지)
└── .gitignore
```

## 5. 실행 방법

```bash
pip install pandas sqlalchemy pymysql python-dotenv koreanize-matplotlib matplotlib seaborn

cp .env.example .env
# .env에 실제 DB_USER / DB_PASSWORD 입력 (로컬 전용, 커밋 금지)
```

노트북 실행 순서: `01_eda.ipynb` → `02_preprocessing.ipynb` → `03_analysis.ipynb`
(같은 커널 세션에서 이어서 실행 권장. 새 세션이면 `01_eda`의 라이브러리/DB 연결 셀부터 다시 실행)

## 6. 기술 스택

`Python (pandas)` · `MySQL` · `SQLAlchemy` · `Matplotlib / Seaborn` · `Tableau`

## 7. 개인정보 및 계정 정보 처리

- DB 계정 정보는 `python-dotenv` + `.env`로 분리, 레포에는 `.env.example`만 포함
- 원본 로그 행 단위 출력(개별 `user_uuid` 노출 등)은 전부 제거하고 집계 결과만 남김
- 원본 사내 파일명, 개인 이름 등 식별 정보는 제거
