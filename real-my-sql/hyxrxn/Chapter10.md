# 10.1

## 테이블 및 인덱스 통계 정보

### MySQL 서버의 통계 정보
- 5.6부터 InnoDB 테이블에 대한 통계 정보를 영구적(Persistent)으로 관리할 수 있게 개선
- `mysql` 데이터베이스의 `innodb_index_stats`, `innodb_table_stats` 테이블로 관리
- `STATS_PERSISTENT` 옵션을 이용해 테이블 단위로 영구적인 정보를 저장할지 말지 결정
  - 0: 5.5 이전의 방식으로 관리
  - 1: 테이블의 통계 정보 저장
  - DEFAULT: `innodb_stats_persistent` 변수로 결정 (기본적으로 1)
- 저장 정보
  - `innodb_index_stats`
    - `stat_name='n_diff_pfx%'`: 인덱스가 가진 유니크한 값의 개수
    - `stat_name='n_leaf_pages'`: 인덱스의 리프 노드 페이지 개수
    - `stat_name='size'`: 인덱스 트리의 전체 페이지 개수
  - `innodb_table_stats`
    - `n_rows`: 테이블의 전체 레코드 건수
    - `clustered_index_size`: 프라이머리 키의 크기(InnoDB 페이지 개수)
    - `sum_of_other_index_sizes`: 프라이머리 키를 제외한 인덱스의 크기(InnoDB 페이지 개수)

### 통계 정보 자동 수집
- 5.5까지
  - 자동으로 통계 정보가 수집되는 경우
    - 테이블이 새로 오픈되는 경우
    - 테이블의 레코드가 대량으로 변경되는 경우(1/16 정도)
    - `ANALYZE TABLE` / `SHOW TABLE STATUS` / `SHOW INDEX FROM` 명령이 실행되는 경우
    - InnoDB 모니터가 활성화되는 경우
    - `innodb_stats_on_metadata` 시스템 설정이 `ON`인 상태에서 `SHOW TABLE STATUS` 명령이 실행되는 경우
- 5.6 이후
  - `STATS_AUTO_RECALC` 옵션
    - 1: 5.5 이전의 방식으로 자동 수집
    - 0: `ANALYZE TABLE` 명령을 실행할 때만 수행
    - DEFAULT: `innodb_stats_auto_recalc` 변수로 결정
  - `innodb_stats_transient_sample_pages`: 자동으로 통계 정보 수집이 실행될 때 몇 개의 페이지를 임의로 샘플링해서 분석할 것인지 (기본적으로 8)
  - `ìnnodb_stats_persistent_sample_pages`: `ANALYZE TABLE` 명령어 실행될 때 몇 개의 페이지를 임의로 샘플링해서 분석할 것이지 (기본적으로 20)

## 히스토그램(Histogram)
- 인덱스되지 않은 칼럼들에 대해서도 데이터 분포도를 수집해서 저장하는 것

### 정보 수집 및 삭제
- 정보는 칼럼 단위로 관리
- `ANALYZE TABLE ... UPDATE HISTOGRAM` 명령으로 수동으로 수집 및 관리
- 정보는 시스템 딕셔너리에 함께 저장
- `information_schema` 데이터베이스의 `column_statistics` 테이블에 로드
- 지원 타입
  - Singleton(싱글톤 히스토그램): 칼럼값 개별로 레코드 건수를 관리, Value-Based 히드토그램 또는 도수 분포라고도 불림
  - Equi-Height(높이 균형 히스토그램): 칼럼값의 개수를 균등한 개수로 구분해서 관리, Height-Balanced 히스토그램이라고도 불림
- `HISTOGRAM` 칼럼의 필드
  - `sampleing-rate`: 정보를 수집하기 위해 스캔한 페이지의 비율
  - `histogram-type`: 종류
  - `number-of-buckets-specified`: 생성 시 설정했던 버킷의 개수 (기본적으로 100개)
- 삭제
  - `ANALYZE TABLE ... DROP HISTOGRAM ON ...`
- 옵티마이저가 사용하지 않도록 설정
  - `SET GLOBAL optimazer_switch='condition_fanout_filter=off'`: 모든 쿼리에 적용
  - `SET SESSION optimazer_switch='condition_fanout_filter=off'`: 현재 커넥션의 퀴리에 적용
  - `SELECT /*+ SET_VAR(optimazer_switch='condition_fanout_filter=off') */ * FROM ...`: 현재 쿼리에만 적용

### 용도
- 기존
  - 전체 레코드 건수와 인덱스된 칼럼의 유니크한 값의 개수 정도 저장
  - 레코드가 1000건이고 유니크 칼럼의 개수가 100개라면 동등 비교 검색 시 대략 10개가 일치할 것으로 예상
  - 하지만 실제로는 균등하지 않은 분포를 가질 수 있음
- 현재
  - 각 범위(버킷)별 레코드의 건수와 유니크한 값의 개수 정보 저장
  - 훨씬 정확한 예측 가능

### 히스토그램과 인덱스
- 인덱스 다이브(Index Dive): 조건절에 일치하는 레코드 건수를 예측하기 위해 옵티마이저가 실제 인덱스의 B-Tree를 샘플링해서 살펴보는 것
- 인덱스된 칼럼을 검색 조건으로 사용하는 경우 히스토그램은 사용하지 않고 인덱스 다이브 수행
- 그래서 히스토그램은 주로 인덱스되지 않은 컬럼에 대한 데이터 분포를 참조하는 용도로 사용

## 코스트 모델(Cost Model)
- 전체 쿼리의 비용을 계산하는 데 필요한 단위 작업들의 비용
- 5.7 이전까지는 이 비용을 서버 소스 코드에 상수화해서 사용
- 하지만 서버가 사용하는 하드웨어에 따라 비용이 달라질 수 있음
- 5.7부터 비용을 DBMS 관리자가 조정 가능하도록 개선
- 사용하는 설정값
  - `server_cost`: 인덱스를 찾고 레코드를 비교하고 임시 테이블 처리에 대한 비용 관리
  - `engine_cost`: 레코드를 가진 데이터 페이지를 가져오는 데 필요한 비용 관리
- 공톨 칼럼
  - `cost_name`: 코스트 모델의 각 단위 작업
  - `dafault_value`: 각 단위 작업의 비용(서버 소스 코드에 설정된 기본값)
  - `cost_value`: DBMS 관리자가 설정한 값
  - `last_updated`: 단위 작업의 비용이 변경된 시점
  - `comment`: 비용에 대한 추가 설명
- `engine_cost` 테이블의 추가 칼럼
  - `engine_name`: 비용이 적용된 스토리지 엔진
  - `divece_type`: 디스크 타입
- 지원하는 단위 작업
  - `io_block_read_cost`: 디스크 데이터 페이지 읽기
  - `memory_block_read_cost`: 메모리 데이터 페이지 읽기
  - `disk_temptable_create_cost`: 디스크 임시 테이블 생성
  - `dist_temptable_row_cost`: 디스크 임시 테이블의 레코드 읽기
  - `key_compare_cost`: 인덱스 키 비교
  - `memory_temptable_create_cost`: 메모리 임시 테이블 생성
  - `memory_temptable_row_cost`: 메모리 임시 테이블의 레코드 읽기
  - `row_evaluate_cost`: 레코드 비교

# 10.2

## 실행 계획 출력 포맷
- 이전 버전까지는 `EXPLAIN EXTENDED` / `EXPLAIN PARTITIONS` 명령 구분
- 8.0부터 내용 통합, `EXTENDED` / `PARTITIONS` 옵션 제거
- `FORMAT` 옵션으로 실행 계획의 표시 방법을 JSON / TREE / 단순 테이블 형태로 선택 (예시는 p.412-413 참고)

## 퀴리의 실행 시간 확인

### EXPLAIN ANALYZE
- 8.0.18부터 도입
- 퀴리의 실행 계획과 단계별 소요된 시간 정보 확인 가능 
- 실제 쿼리를 실행하고 사용된 실행 계획과 소요된 시간을 보여줌
- 결과를 항상 TREE 포멧으로 표시하기 때문에 FORMAT 옵션 사용 불가 
- 들여쓰기는 호출 순서를 의미
    - 들여쓰기가 같은 레벨에서는 상단의 라인이 먼저 실행
    - 들여쓰긱 ㅏ다른 레벨에서는 가장 안쪽의 라인이 먼저 실행
- 단계별로 실제 소요된 시간(actual time)과 처리한 레코드 건수(rows), 반복 횟수(loops) 표시
