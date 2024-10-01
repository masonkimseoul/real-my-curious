# 10장 실행 계획

날짜: 2024-10-02

범위: 10.1 ~ 10.2(p.394~416)

# 서론

많은 데이터를 안전하게 저장 및 관리하고, 사용자가 원하는 데이터를 빠르게 조회하려면 옵티마이저가 사용자의 쿼리를 최적으로 실행될 수 있도록 실행 계획을 수립해야 한다.

하지만 옵티마이저가 항상 좋은 실행 계획을 만들어 내는 것이 아니기 때문에 `EXPLAIN` 등을 통해 수립된 실행 계획을 확인할 수 있다.

이를 이해하려면 데이터를 처리하는 로직에 대한 이해가 필요하다.

# 10.1 통계 정보

MySQL 5.7 버전까지는 테이블과 인덱스에 대한 개괄적인 정보를 가지고 실행 계획을 수립했음.

이는 실제 데이터 분포 등에 대한 정보가 없어 정확도가 떨어짐.

MySQL 8.0 부터 인덱스되지 않은 칼럼들에 대해 분포도를 수집해 저장하는 히스토그램 정보가 도입됨.

히스토그램과 인덱스의 통계 정보를 통해 실행 계획을 수립하게 됨.

## 10.1.1 테이블 및 인덱스 통계 정보

비용 기반 최적화에 제일 중요한 것은 통계 정보.

타 DBMS 와 같이 비용 기반 최적화를 사용하지만 다른 DBMS 에 비해 통계 정보의 정확도가 높지 않고 휘발성이 강해 MySQL 서버에서 쿼리의 실행 계획을 수립할 때에 실제 테이블의 데이터를 잉ㄹ부 분석해 통계 정보를 보완하여 사용함.

### MySQL 서버의 통계 정보

MySQL 5.5 까진 각 테이블의 통계 정보가 메모리에 관리되었고, `SHOW INDEX` 를 통해서만 인덱스 칼럼의 분포도를 볼 수 있었음.

MySQL 5.6 부터 InnoDB 스토리지 엔진을 사용하는 테이블에 대해 `mysql` 테이블의 `innodb_index_stats`, `innodb_table_stats` 테이블로 영구적으로 관리됨.

참고: 테이블 생성 시 `STATS_PERSISTENT` 옵션을 통해 영구적으로 해당 테이블의 통계 정보를 보관할지 여부를 결정할 수 있음(0 -> 5.5 이전의 방식, 1 -> 5.6 이후의 방식, DEFAULT -> `innodb_stats_persistent` 시스템 변수의 값대로(default = On(1))). 테이블 통계 정보 저장 방식 수정은 `ALTER TABLE {table} STATS_PERSISTENT={value}` 를 통해 수정할 수 있음.

통계 정보 테이블의 칼럼:
- `innodb_index_stats.stat_name='n_diff_pfx%'`: 인덱스가 가진 유니크한 값의 개수
- `innodb_index_stats.stat_name='n_leaf_pages'`: 인덱스의 리프 노드 페이지 개수
- `innodb_index_stats.stat_name='size'`: 인덱스 트리의 전체 페이지 개수
- `innodb_table_stats.n_rows`: 테이블의 전체 레코드 건수
- `innodb_table_stats.clustered_index_size`: primary key 의 크기(InnoDB 페이지 개수)
- `innodb_table_stats.sum_of_other_index_sizes`: primary key 를 제외한 인덱스의 크기(InnoDB 페이지 개수)

MySQL 5.5 까진 영구적이지 않았기 때문에 특정 이벤트를 통해 통계 정보가 갱신되었음.
- 새로운 테이블 오픈, 테이블의 레코드 대량 변경(1/16 정도), `ANALYZE TABLE` 명령 실행, `SHOW TABLE STATUS`, `SHOW INDEX FROM` 명령 실행, InnoDB 모니터 활성화, `innodb_stats_on_metadata` 가 ON 인 상태에서 `SHOW TABLE STATUS` 명령 실행

통계 정보가 자주 갱신되면 인덱스 레인지 스캔으로 잘 처리하던 쿼리를 풀 레인지 스캔으로 처리하게 될 수도 있음.

`innodb_stats_auto_recalc` 시스템 변수 수정으로 자동 갱신을 막을 수 있음.

통계 정보 수집 시 몇 개의 InnoDB 테이블 블록을 샘플링할지 결정할 수 있음.
- `innodb_stats_transient_sample_pages`: 기본값 8, 자동으로 통계 정보 수집이 실행될 때 해당 변수만큼의 페이지만 임의로 샘플링해 분석하고 통계 정보로 활용
- `innodb_stats_persistent_sample_pages`: 기본값 20, `ANALYZE TABLE` 명령 실행 시 해당 변수만큼의 페이지만 분석하고 결과를 영구 저장 및 활용

## 10.1.2 히스토그램

### 히스토그램 정보 수집 및 삭제

히스토그램 정보는 칼럼 단위로 관리되며, 지동 수집되지 않고 `ANALYZE TABLE ... UPDATE HISTOGRAM` 을 통해서 수동으로 수집 및 관리됨.

수집된 히스토그램은 시스템 딕셔너리에 저장, MySQL 서버 시작 시 히스토그램 정보를 `information_schema` 데이터베이스의 `column_statistics` 테이블로 로드함.

이를 `SELECT` 하여 조회 가능.

히스토그램 타입:
- SingleTon: 칼럼값 개별로 레코드 건수를 관리, Value-Based 히스토그램 혹은 도수 분포라고 불림.
- Equi-Height: 칼럼값의 범위를 균등한 개수로 구분해서 관리, Height-Balanced 히스토그램으로 불림.

히스토그램은 버킷 단위로 구분되어 관리, 싱글톤의 경우 칼럼이 가지는 값 별로 버킷이 할당되고 높이 균형 히스토그램은 범위별로 하나의 버킷이 할당.

싱글톤은 각 버킷이 칼럼의 값, 발생 빈도의 비율을 가지고, 높이 균형은 범위 시작 값과 마지막 값, 발생 빈도율과 각 버킷에 포함된 유니크한 값의 개수를 가짐.

`information_schema.column_statistics` 테이블의 HISTOGRAM 칼럼은 다음 필드도 포함:
- sampling-rate: 정보 수집을 위해 스캔한 페이지 비율, `histogram_generation_max_mem_size`(default = 20MB) 에 지정된 메모리 크기에 맞게 적절히 샘플링.
- histogram-type: 히스토그램 종류 저장
- number-of-buckets-specified: 히스토그램 생성 시 설정한 버킷의 개수 저장. 기본 100개가 사용, 최대 1024까지 가능.

히스토그램 삭제(`ALTER TABLE {table} DROP HISTOGRAM ON {value};`)는 테이블의 데이터를 참고하는 것이 아니라 즉시 완료되며 성능에 영향이 없음, 단 삭제 시 실행 계획이 달라질 수 있음을 유의.

삭제하지 않고 사용하지 않으려면 `optimizer_switch` 시스템 변수를 global 로 변경하면 됨, 이 또한 다른 최적화 기능이 사용되지 않을 수 있음에 유의.

특정 커넥션 혹은 쿼리에서 사용하지 않으려면 `SET SESSION optimizer_switch='condition_fanout_filter=off';` 혹은 `SELECT /*+ SET_VAR(optimizer_switch='condition_fanout_filter=off') */ * FROM ...` 를 사용.

### 히스토그램의 용도

히스토그램이 없으면 옵티마이저는 데이터가 균등하게 분포할 것으로 예측.

이를 통해 조인에서 드라이빙 테이블을 결정하는 등 성능에 영향을 줄 수 있음.

### 히스토그램과 인덱스

인덱스 다이브(Index Dive): 실행 계획 수립 시 사용 가능 인덱스로부터 조건절에 일치하는 레코드 건수를 대략 파악하고 실행 계획 선택, 이때 건수 예측을 위해 실제 인덱스의 B-Tree 를 샘플링하는 과정.

인덱스된 칼럼을 검색 조건으로 사용하는 경우 히스토그램을 사용한 통계보다 인덱스 다이브를 통한 정보가 더 정확함.

그러므로 주로 히스토그램은 인덱스되지 않은 카럶에 대한 데이터 분포도를 참조하는 용도로 사용.

## 10.1.3 코스트 모델

MySQL 서버가 쿼리를 처리하려면 다음과 같은 작업 필요:
    디스크로부터 데이터 페이지 읽기, 메모리로부터 데이터 페이지 읽기, 인덱스 키 비교, 레코드 평가, 메모리 및 데스크 임시 테이블 작업 등

전체 쿼리의 비용을 계산하는 데 필요한 단위 작업들의 비용을 코스트 모델이라고 정의, MySQL 5.7 이전까진 이들이 상수화.

MySQL 8.0 서버의 코스트 모델은 `mysql` DB 에 존재하는 두 테이블에 저장된 설정값 사용:
- server_cost: 인덱스를 찾고 레코드를 비교하고 임시 테이블 처리에 대한 비용 관리 테이블
- engine_cost: 레코드를 가진 데이터 페이지를 가져오는 데 필요한 비용 관리 테이블

이들은 공통으로 cost_name, default_value, cost_value, last_updated, comment 칼럼을 가지고, engine_cost 는 추가로 engine_name, device_type 칼럼을 가짐.


MySQL 8.0 의 코스트 모델이 지원하는 단위 작업은 다음과 같음.
- io_block_read_cost: 디스크 데이터 페이지 읽기(default = 1.00) -> 높히면 InnoDB 버퍼 풀에 데이터 페이지가 많이 적재된 인덱스를 사용하는 실행 계획 선택 확률 증가
- memory_block_read_cost: 메모리 데이터 페이지 읽기(default = 0.25) -> 높히면 InnoDB 버퍼 풀에 데이터 페이지가 상대적으로 적어도 선택 확률 증가
- disk_temtable_create_cost: 디스크 임시 테이블 생성(default = 20.00) -> 높히면 디스크에 임시 테이블을 만들지 않는 실행 계획 선택 확률 증가
- disk_temtable_row_cost: 디스크 임시 테이블의 레코드 읽기(default = 0.50) -> 위와 동일
- key_compare_cost: 인덱스 키 비교(default = 0.05) -> 높히면 가능하면 정렬을 수행하지 않는 방향의 실행 계획 선택 확률 증가
- memory_temtable_create_cost: 메모리 임시 테이블 생성(default = 1.00) -> 높히면 메모리에 임시 테이블을 만들지 않는 실행 계획 선택 확률 증가
- memory_temtable_row_cost: 메모리 임시 테이블의 레코드 읽기(default = 0.10) -> 위와 동일
- row_evaluate_cost: 레코드 비교(default = 0.10) -> 높히면 풀 스캔을 실행하는 쿼리의 비용 증가, 옵티마이저가 레인지 인덱스 스캔 선택 확률 증가


# 10.2 실행 계획 확인

## 실행 계획 출력 포멧

`EXPLAIN` 문법을 사용하려 출력, `FORMAT` 옵션으로 Json, Tree, 단순 테이블로 볼 지 결정할 수 있음.

## 실행 시간 확인

`SHOW PROFILE` 을 통해서도 가능하지만, 실행 계획의 단께별로 소요된 시간을 보여주진 않음.

이를 위해 `EXPLAIN ANALYZE` 기능이 추가됨, 항상 Tree 구조로 출력.

들여쓰기가 호출 순서이고, 같은 들여쓰기에선 상단이, 다른 들여쓰기에선 안쪽에 위치한 라인이 먼저 실행됨.

`EXPLAIN ANALYZE` 는 실제로 쿼리를 실행하고 결과를 보여주는 것, 실행 계획이 아주 나쁜 경우 `EXPLAIN`으로 계획만 확인하고 튜닝한 이후에 실행 시간 확인.

> 의문점: 왜 설명과 달리 D-F-E-C-B-A 인가?(p.415)