# 숙소별 예약 통계 시스템 v1.7

allmytour.com의 숙소별 예약 통계를 실시간으로 조회하고 엑셀로 추출할 수 있는 백오피스 시스템입니다.

## 📋 프로젝트 개요

### 목적
- 비개발자 관리자들이 직접 DB 쿼리 없이 웹 인터페이스로 숙소별 데이터 추출
- 특정 숙소에서 각 채널별로 얼마나 예약이 되었는지 확인
- 예: A숙소에서 Expedia, Hotelbeds, 다보 등 각 채널별 예약 현황 조회

### 주요 기능
- 📊 **날짜별 + 숙소별 + 채널별 예약 데이터 조회**
  - 구매일(예약일) 또는 이용일(체크인) 기준 조회
  - 다중 숙소 선택 지원 (최대 10개)
  - 각 숙소별로 채널별 통계 제공
- 🔍 **고급 숙소 검색 기능**
  - 숙소명 또는 숙소코드로 검색
  - 유사 키워드 검색 지원 (공백 제거, 대소문자 무시)
  - 자동완성 기능 (최대 15개 표시)
  - 이전 선택한 숙소 목록 저장 (최대 10개)
- 📈 **요약 통계 대시보드**
  - 채널별 통계와 동일한 2행 4컬럼 구조
  - 총 예약건수, 총 입금가, 총 실구매가, 총 수익
  - 총 객실수, 확정 객실 수, 취소 객실 수, 취소율
- 📋 **상세 데이터 조회**
  - 날짜별 + 숙소별 + 채널별 집계
  - 예약건수, 총객실수, 확정객실수, 취소객실수, 취소율
  - 총 입금가, 총 실구매가, 총 수익, 수익률
  - 상위 10개 미리보기 (전체 데이터는 엑셀 다운로드)
- 📥 **원클릭 엑셀 다운로드**
  - 채널별 통계와 동일한 형식
  - 파일명: `숙소별_예약통계_YYYYMMDD_HHMMSS.xlsx`
  - 요약 통계 및 상세 데이터 포함

## 🎯 UI 구조

### TAB 방식 통합
- **TAB 1**: 채널별 예약 통계 시스템 (기존 v1.6)
- **TAB 2**: 숙소별 예약 통계 시스템 (신규 v1.7)

### 숙소별 통계 화면 구성

#### 사이드바 (검색 조건)
1. **날짜유형 선택**
   - 이용일(체크인) 또는 구매일(예약일)
   - 채널별 통계와 동일한 로직

2. **날짜 범위 선택**
   - **구매일 기준**: 오늘 -1일 ~ 3개월 전까지
   - **이용일 기준**: 오늘 기준 앞뒤 90일
   - 채널별 통계와 동일한 범위

3. **숙소 검색**
   - 텍스트 입력 박스
   - 플레이스홀더: "숙소명 or 숙소코드를 입력해주세요"
   - 검색 시작 조건: 2자 이상 입력 시 검색 시작
   - 자동완성 드롭다운 (최대 15개)
   - 정렬: 1순위 `name_kr` 가나다순, 2순위 `idx` 내림차순

4. **이전 선택한 숙소 목록**
   - 최대 10개까지 저장
   - 태그 형태로 표시
   - X 버튼으로 삭제 가능
   - 클릭 시 재선택

5. **선택된 숙소 목록**
   - 최대 10개까지 선택 가능
   - '전체' 선택 불가 (서버 과부하 방지)
   - 반드시 1개 이상 선택 필수

6. **조회 및 초기화 버튼**
   - [조회] 버튼: 데이터 조회 실행
   - [초기화] 버튼: 모든 필터 초기화

#### 메인 영역
1. **요약 통계** (2행 4컬럼)
   - 1행: 총 예약건수 | 총 입금가 | 총 실구매가 | 총 수익
   - 2행: 총 객실수 | 확정 객실 수 | 취소 객실 수 | 취소율

2. **상세 데이터 테이블**
   - 컬럼: 구매일(or 이용일) | 숙소명 | 채널명 | 예약건수 | 총객실수 | 확정객실수 | 취소객실수 | 취소율 | 총입금가 | 총실구매가 | 총 수익 | 수익률(%)
   - 상위 10개만 표시 (전체는 엑셀 다운로드)
   - 정렬: 날짜 내림차순, 숙소명, 채널명

3. **엑셀 다운로드 버튼**
   - 전체 데이터 다운로드
   - 채널별 통계와 동일한 형식

## 🗄️ 데이터베이스 구조

### 추가 테이블

#### product 테이블
- **용도**: 숙소 마스터 데이터
- **주요 컬럼**:
  - `idx`: 숙소 ID (PK)
  - `product_code`: 숙소코드
  - `name_kr`: 숙소 한글명
  - `reg_date`: 등록일 (신규 호텔 필터링에 사용)
- **레코드 수**: 약 270만개
- **인덱스**: BTREE 인덱스 존재 (`name_kr`, `product_code`)

#### 테이블 관계
```
order_product (N) ──< (1) product
    │
    └── (1) ──< (1) order_pay
    │
    └── (N) ──< (1) common_code
    │
    └── (1) ──< (N) order_item
```

### 데이터 일관성
- `order_product.product_idx`는 NULL 값 없음 (필수 컬럼)
- `order_product.product_name`과 `product.name_kr`는 무조건 같음
- `order_product.product_idx` = `product.idx`로 조인

### 쿼리 구조 (예시)

```sql
SELECT 
    DATE(op.create_date) as booking_date,  -- 날짜별
    p.name_kr as hotel_name,               -- 숙소별
    COALESCE(cc.code_name, op.order_type, ...) as channel_name,  -- 채널별
    COUNT(DISTINCT op.order_num) as booking_count,
    COUNT(DISTINCT p.idx) as hotel_count,
    SUM(COALESCE(op.terms, 1) * COALESCE(op.room_cnt, 0)) as total_rooms,
    -- ... 기타 지표들
FROM order_product op
LEFT JOIN product p ON op.product_idx = p.idx
LEFT JOIN order_pay opay ON op.order_pay_idx = opay.idx
LEFT JOIN common_code cc ON cc.code_id = op.order_channel_idx AND cc.parent_idx = 1
WHERE op.create_date >= :start_date
  AND op.create_date <= :end_date
  AND op.create_date < CURDATE()
  AND op.product_idx IN (:selected_hotel_ids)
  -- 예약상태 조건 (전체)
GROUP BY booking_date, hotel_name, channel_name
ORDER BY booking_date DESC, hotel_name, channel_name
```

## 🔍 숙소 검색 기능 상세

### 검색 로직

#### 1. 검색 시작 조건
- **최소 입력 글자 수**: 2자 이상
- 2자 미만 입력 시 검색하지 않음 (270만개 데이터 보호)

#### 2. 검색 방식
- **숙소명 검색**: `product.name_kr LIKE '%검색어%'`
- **숙소코드 검색**: `product.product_code LIKE '%검색어%'`
- **유사 검색**: 공백 제거 후 검색
  - 예: "힐튼 호텔" → "힐튼호텔"로도 검색 가능
  - 예: "호텔 힐튼" → "호텔힐튼"로도 검색 가능

#### 3. 검색 결과 정렬
- **1순위**: 예약 있는 호텔 우선 표시 (`has_recent_booking DESC`)
- **2순위**: `name_kr` 가나다순 (오름차순)
- **3순위**: `idx` 내림차순
- **최대 표시**: 15개

#### 4. 성능 최적화
- **인덱스 활용**: `name_kr`, `product_code`에 BTREE 인덱스 존재
- **검색 범위 제한**: 
  - 최근 예약이 있는 숙소 또는 신규 등록 숙소만 검색 대상으로 제한 (JOIN 활용)
  - **구매일 기준**: 최근 180일 (6개월) - 성능 최적화
  - **이용일 기준**: 오늘 기준 앞뒤 180일
  - **신규 호텔**: 최근 90일 이내 등록된 호텔 (`product.reg_date` 기준)
- **캐싱**: 검색 결과 1시간 캐싱 (`@st.cache_data(ttl=3600)`)

### 검색 쿼리 (최적화)

```sql
-- 최근 예약이 있는 숙소 또는 신규 등록 숙소 검색 (성능 최적화)
SELECT DISTINCT 
    p.idx, 
    p.product_code, 
    p.name_kr,
    CASE WHEN op.idx IS NOT NULL THEN 1 ELSE 0 END as has_recent_booking
FROM product p
LEFT JOIN order_product op ON p.idx = op.product_idx
    AND (
        -- 구매일 기준: 최근 180일 (6개월)
        op.create_date >= DATE_SUB(CURDATE(), INTERVAL 180 DAY)
        -- 또는 이용일 기준: 오늘 기준 앞뒤 180일
        OR (
            op.checkin_date >= DATE_SUB(CURDATE(), INTERVAL 180 DAY)
            AND op.checkin_date <= DATE_ADD(CURDATE(), INTERVAL 180 DAY)
        )
    )
WHERE (
    p.name_kr LIKE '%검색어%' 
    OR p.product_code LIKE '%검색어%'
    OR REPLACE(p.name_kr, ' ', '') LIKE '%검색어%'  -- 공백 제거 검색
)
-- 최근 예약이 있거나, 신규 등록 호텔도 검색
AND (
    op.idx IS NOT NULL  -- 최근 예약이 있는 호텔
    OR p.reg_date >= DATE_SUB(CURDATE(), INTERVAL 90 DAY)  -- 최근 90일 이내 등록 (신규 호텔)
)
GROUP BY p.idx, p.product_code, p.name_kr
ORDER BY 
    has_recent_booking DESC,  -- 예약 있는 호텔 우선 표시
    p.name_kr ASC, 
    p.idx DESC
LIMIT 15;
```

**참고사항**:
- 검색 범위는 관리자 요청에 따라 조정 가능
- 구매일 기준은 성능 최적화를 위해 180일로 설정
- 신규 호텔 정의(90일)도 관리자 요청에 따라 변경 가능

### 이전 선택한 숙소 목록

#### 저장 방식
- Streamlit `session_state`에 저장
- 브라우저별로 독립적으로 저장
- 최대 10개까지 저장

#### 표시 방식
- 태그 형태로 표시
- 각 태그에 X 버튼으로 삭제 가능
- 클릭 시 검색 대상으로 자동 선택

#### 저장 로직
```python
# 세션 상태에 저장
if 'recent_hotels' not in st.session_state:
    st.session_state.recent_hotels = []

# 숙소 선택 시 저장
if selected_hotel not in st.session_state.recent_hotels:
    st.session_state.recent_hotels.append(selected_hotel)
    # 최대 10개 유지
    if len(st.session_state.recent_hotels) > 10:
        st.session_state.recent_hotels.pop(0)
```

## 📊 결과 데이터 구조

### 엑셀 파일 컬럼 구조
| 컬럼명 | 설명 | 데이터 타입 |
|--------|------|------------|
| 구매일(or 이용일) | 날짜유형에 따라 표시 | 날짜 |
| 숙소명 | product.name_kr | 문자열 |
| 채널명 | common_code.code_name 또는 order_type | 문자열 |
| 예약건수 | COUNT(DISTINCT order_num) | 정수 |
| 총객실수 | SUM(terms * room_cnt) | 정수 |
| 확정객실수 | 확정 상태의 terms * room_cnt 합계 | 정수 |
| 취소객실수 | 취소 상태의 terms * room_cnt 합계 | 정수 |
| 취소율 | (취소객실수 / 총객실수) * 100 | 소수점 1자리 (%) |
| 총입금가 | SUM(order_item.due_price) * room_cnt | 정수 |
| 총실구매가 | SUM(order_pay.total_amount) | 정수 |
| 총 수익 | 총실구매가 - 총입금가 | 정수 |
| 수익률 (%) | (총 수익 / 총입금가) * 100 | 소수점 1자리 (%) |

### 집계 기준
- **날짜별**: DATE(create_date) 또는 DATE(checkin_date)
- **숙소별**: product.idx (product.name_kr로 표시)
- **채널별**: order_channel_idx (common_code.code_name으로 표시)

## 🚀 구현 계획

### 파일 구조

```
통계프로그램/
├── app_v1.7_total.py          # 메인 애플리케이션 (TAB 방식)
│   ├── TAB 1: 채널별 통계 (기존 app_v1.6.py 코드)
│   └── TAB 2: 숙소별 통계 (신규)
├── utils/
│   ├── hotel_search.py        # 숙소 검색 기능 (신규)
│   ├── query_builder_hotel.py # 숙소별 통계 쿼리 생성 (신규)
│   ├── data_fetcher_hotel.py # 숙소별 데이터 조회 (신규)
│   └── excel_handler_hotel.py # 숙소별 엑셀 생성 (신규)
└── docs/
    └── 숙소별 예약 통계 시스템.md  # 이 문서
```

### 구현 단계

#### 1단계: 기본 구조 생성
- [ ] `app_v1.7_total.py` 파일 생성
- [ ] TAB 구조 구현
- [ ] 기존 채널별 통계 코드 통합

#### 2단계: 숙소 검색 기능
- [ ] `utils/hotel_search.py` 모듈 생성
- [ ] 검색 쿼리 최적화
- [ ] 자동완성 UI 구현
- [ ] 이전 선택한 숙소 목록 기능

#### 3단계: 데이터 조회 기능
- [ ] `utils/query_builder_hotel.py` 모듈 생성
- [ ] 날짜별 + 숙소별 + 채널별 집계 쿼리 작성
- [ ] `utils/data_fetcher_hotel.py` 모듈 생성
- [ ] 입금가 계산 로직 (order_item.due_price 사용)

#### 4단계: UI 구현
- [ ] 사이드바 검색 조건 UI
- [ ] 요약 통계 표시
- [ ] 상세 데이터 테이블
- [ ] 엑셀 다운로드 버튼

#### 5단계: 엑셀 다운로드
- [ ] `utils/excel_handler_hotel.py` 모듈 생성
- [ ] 채널별 통계와 동일한 형식으로 엑셀 생성
- [ ] 파일명: `숙소별_예약통계_YYYYMMDD_HHMMSS.xlsx`

#### 6단계: 테스트 및 최적화
- [ ] 검색 성능 테스트
- [ ] 대량 데이터 조회 테스트
- [ ] UI/UX 개선

## ⚡ 성능 최적화 방안

### 1. 검색 성능
- **인덱스 활용**: `name_kr`, `product_code`에 BTREE 인덱스 사용
- **검색 범위 제한**: 
  - 최근 예약이 있는 숙소 또는 신규 등록 숙소만 검색
  - 구매일 기준: 최근 180일 (6개월) - 성능 최적화
  - 이용일 기준: 오늘 기준 앞뒤 180일
  - 신규 호텔: 최근 90일 이내 등록 (`reg_date` 기준)
- **캐싱**: 검색 결과 1시간 캐싱
- **검색 시작 조건**: 2자 이상 입력 시에만 검색
- **정렬 우선순위**: 예약 있는 호텔 우선 표시, 그 다음 가나다순

### 2. 쿼리 최적화
- **JOIN 최적화**: 필요한 테이블만 JOIN
- **인덱스 활용**: `product_idx`, `order_channel_idx` 인덱스 활용
- **서브쿼리 최적화**: 입금가 계산은 서브쿼리로 처리 (중복 방지)

### 3. 데이터 조회 최적화
- **선택 제한**: 최대 10개 숙소만 선택 가능
- **날짜 범위 제한**: 최대 3개월 (90일)
- **페이징**: 상세 데이터는 상위 10개만 표시

## 🔒 보안 및 제약사항

### 제약사항
- **'전체' 선택 불가**: 서버 과부하 방지를 위해 반드시 1개 이상의 숙소 선택 필수
- **최대 선택 개수**: 10개까지 선택 가능
- **날짜 범위 제한**: 최대 90일 (3개월)
- **구매일 기준**: 당일 데이터 조회 불가 (D-1까지만 조회 가능)

### 데이터 정확성
- `order_product.product_idx`는 NULL 없음 (필수 컬럼)
- `order_product.product_name`과 `product.name_kr`는 항상 일치
- `booking_master_offer` 테이블은 사용하지 않음 (제외)

## 📝 주요 차이점 (채널별 vs 숙소별)

| 항목 | 채널별 통계 | 숙소별 통계 |
|------|------------|------------|
| 집계 기준 | 날짜별 + 채널별 | 날짜별 + 숙소별 + 채널별 |
| 필터 | 채널 선택 (전체 가능) | 숙소 선택 (전체 불가, 최대 10개) |
| 검색 기능 | 채널 목록 (드롭다운) | 숙소 검색 (자동완성) |
| 데이터 소스 | order_product | order_product + product (JOIN) |
| 테이블 | 채널별 집계 | 숙소별 + 채널별 집계 |

## 🎯 사용 시나리오

### 예시 1: 특정 숙소의 채널별 예약 현황 확인
1. TAB 2 (숙소별 통계) 선택
2. 날짜유형: 구매일 선택
3. 날짜 범위: 2025-01-01 ~ 2025-01-31
4. 숙소 검색: "서울 그랜드 호텔" 입력
5. 검색 결과에서 숙소 선택
6. [조회] 버튼 클릭
7. 결과 확인: 서울 그랜드 호텔에서 Expedia, Hotelbeds 등 각 채널별 예약 현황 확인

### 예시 2: 여러 숙소 비교
1. TAB 2 선택
2. 날짜 범위 설정
3. 숙소 검색으로 여러 숙소 선택 (최대 10개)
4. [조회] 버튼 클릭
5. 각 숙소별 채널별 예약 현황 비교

## 📚 참고 문서

- [채널별 예약 통계 시스템](./#%20채널별%20예약%20통계%20시스템.md): 기존 채널별 통계 시스템 문서
- [README.md](../README.md): 프로젝트 전체 문서

## 📞 문의

프로젝트 관련 문의는 GitHub Issues를 통해 등록해주세요.

---

*Last Updated: 2025.01.16*
*Version: v1.7 (Planning)*

