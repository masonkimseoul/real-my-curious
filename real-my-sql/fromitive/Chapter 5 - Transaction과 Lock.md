
### Transaction

#### 개념

논리적인 작업의 완정성을 보장하는 것을 목표로 두는 개념. 따라서 작업 중간에 오류가 발생해도 트렌젝션 범위를 설정하면 부분적으로 완료[^1] 되지 않음을 보장

[^1]: INSERT를 3개를 한다고 가정할 때, 3번째 삽입이 실패하게 되면 1, 2도 삽입이 되는 현상이다. Trasaction이 적용되면 이러한 부분적으로 완료 되지 않는다. 즉, 1,2 번째가 성공적으로 삽입이 되었더라도 3번째에서 삽입 오류가 발생할 때 1, 2삽입이 없는 일이 되버린다.

#### 사용해야 하는 이유

트렌젝션 범위를 설정하게 되면 한 쿼리 뿐만 아니라 여러쿼리를 조합해서 한 작업단위로 묶을 수 있기 때문에 작업이 예측 가능해 진다. 즉, 여러 DML을 통해 작업을 해야하는 상황에서 트렌젝션은 예측 가능하도록 통제할 수 있는 수단이 된다.

#### 주의 사항

트렌젝션으로 묶을 때 연결과 같이 `작업 단위를 최대한 짧게 만들어야 한다.` 다음은 트렌젝션을 길게 만드는 원인이 되는 요인이다.

```
1. 다른 시스템의 통신(FTP, external API)과 묶여 있는 경우
2. 데이터가 영향을 받지 않는 조회와 같은 쿼리가 섞여있는 경우
3. 전체 트렌젝션을 봤을 때 연관없는 DML 쿼리가 섞여있는 경우
```

### Lock

데이터 정합성을 위해 자원에 대한 접근을 차단하는 것

#### MySQL Engine Lock

##### 글로벌 락

데이터베이스 전체를 백업해야 할 때 발생하는 Lock이며 Lock 이 걸리는 중 DDL, DML를 사용할 때대기가 걸린다.

**백업 락**

MySQL 8.0에서 자주 백업이 일어나는 needs로 생긴 Lock. 좀 더 가벼운 Lock이라고 한다.

##### 테이블 락

테이블 단위로 적용할 수 있는 Lock. DDL은 Lock이 걸리지만 DML은 무시된다.

##### 네임드 락

락에 이름을 붙여서 사용하는 lock. batch 작업을 할 때 사용할 것 같다.

##### 메타데이터 락

테이블 이름 변경 시 발생하는 Lock이며 대상 테이블 쿼리 실행 도중 테이블 이름을 변경할 수 있게 도와준다.

#### InnoDB Storage Engine Lock

##### 레코드 락

레코드를 잠글 때 사용하는 Lock이며 다른 상용 DB와의 차이점은, 인덱스에 잠긴다는 점이다.

>[!tip] 인덱스 단위로 잠그면 어떤 이점이 있나요?
>
> 여러 범위를 한번에 `Update`하는 쿼리를 실행할 때 Index가 걸려있는 Column만 있을 경우 보다 빠르게 업데이트 수행이 가능한 반면, 걸려있지 않은 컬럼을 Update를 하게 되면 연관된 레코드가 전부 잠기게 된다
>

##### 갭 락

레코드와 레코드 사이에 데이터를 추가할 예정일 때 해당 빈 공간에 락을 거는 방식을 갭 락이라고 한다.
##### 넥스트 키 락

데이터 삽입 및 변경 시 갭 락과 레코드 락이 발생학게 되는데 이를 합친 락이 넥스트 키 락이다. 

##### 자동 증가(Autoincrement Lock) 락

Autoincrement가 걸려있는 Column을 삽입이나 대치(Replace) 할 때 해당 레코드에 락이 걸리게 된다. 명시적으로 획득하는 게 아니라 MySQL 내부에서 처리하는 Lock이라 통제할 수 없다.

자동증가 락의 종류

mode 0
모든 INSERT문 발생 시 Lock이 자동으로 걸림. 연속성을 보장

mode 1
예측이 가능한 삽입이 일어날 때 Lock을 걸지않고 처리함. 대부분 자동 증가할 때 연속성 보장하지만, 한번에 많은 INSERT가 일어나는 상황에 자동 증가 값을 사용하지 않을 때는 비어있는 값이 있을 수 있음

mode 2
절대 Lock을 걸지 않고 thread mutex lock 자동증가 값은 연속됨을 덜 보장하지만 고유한 값은 확실하게 보장하는 모드. 이 모드를 사용할 때 주의할 점은 replica 방식이 STATEMENT 포맷일 경우 원 서버와 레플리카의 id가 달라질 수 있음을 주의 

> [!info] MySQL 8.0의 기본 모드
> mode 2를 사용하고, replica 방식이 STATEMENT가 아닌 ROW이기 때문에 STATEMENT 변경 시 replica방식도 변경할 것을 권장 

중요한 건 **값이 한 번 증가하면 절대 줄어들지 않은 이유는 자동 증가 락을 최소화 하기 때문**

##### 레코드 수준의 잠금 수준 확인방법

MySQL의 레코드 수준의 잠금 정보를 담고 있는 메타정보를 performance_schema의 `data_locks`와 `data_locks_wait`를 통해  확인할 수 있다.

``` sql
SELECT 
	r.trx_id waiting_trx_id,
	r.trx_mysql_threrad_id waiting_threrad,
	r.trx_query waiting_query,
	b.trx_id blocking_trx_id,
	b.trx_mysql_thread_id blocking_thread,
	b.trx_query bllockking_query
FROM performance_schema.data_lock_waits w
INNER JOIN information_schema.innodb_trx b
	ON b.trx_id = w.blocking_engine.transaction_id
INNER JOIN information_schema.innodb_trx r
	ON r.trx_id = w.requesting_engine_trasaction_id;
```

#### MySQL의 격리 수준

크게 4가지로 구분된다.

| isolation type   | dirty read | non-repeatable read | phantom read |
| ---------------- | ---------- | ------------------- | ------------ |
| READ UNCOMMITTED | O          | O                   | O            |
| READ COMMITED    | X          | O                   | O            |
| REPEATABLE READ  | X          | X                   | O(InnoDB는 X) |
| SERIALIZABLE     | X          | X                   | X            |

##### 격리 수준 정리
##### READ UNCOMMITTED 
commit이 되지 않은 데이터를 다른 트렌젝션에서 참조할 수 있는 격리 수준이다. 이 격리 수준을 트렌젝션 격리라고 치지 않는다.

##### READ COMMITED

commit이 되지 않은 데이터를 다른 트렌젝션에서 참조할 수 없으며, commit이 되어야 참조가 가능하다. 가장 기본적인 트렌젝션 격리수준 다만, 한 트렌젝션이 여러 트렌젝션에 중첩이 되면 non-repeatable read 문제가 발생한다.
##### REPEATABLE READ

MySQL의 InnoDB에서 기본적으로 사용하는 트렌젝션 격리 수준, MVCC를 통해 언두로그에 수정 전 레코드를 임시로 저장하여 데이터 접근이 전부 끝날 때 레코드를 업데이트 한다. 따라서 한 트렌젝션이 레코드 수정이나 삽입을 하여 commit이 되더라도 해당 레코드를 참조하는 다른 트렌젝션이 있을경우 언두 로그에 수정 전 레코드를 저장하여 값이 변경되기 이전(commit되기 전)의 데이터를 보여준다.

#### SERIALIZABLE

트렌젝션 시 다른 트렌젝션 혹은 안쪽에서 중첩된 트렌젝션이라도 접근을 불가능하도록 하는 제일 큰 격리수준.  

##### 부정합 문제 정리
##### Dirty read

2중 조회(수정 전에 조회, 수정 후에 조회) 할 때 중간에 레코드가 변경되면 변경 전과 변경 후의 데이터가 달라지는 현상 

##### Non-repeatable read

트렌젝션 중첩이 발생하는 상황에서 수정 commit이 된 레코드 읽을 때 수정 전과 수정 후의 데이터가 달라지는 현상 - Transaction의 격리 수준으로 인정이 되지만 commit 된 이후에 쿼리 결과가 달라지게 되는 안되는 시스템(은행 도메인의 총 자산 관리 등)같은 경우엔..! 위험하다. 

##### Phantom read

트렌젝션 중첩이 발생하는 상황에서 삽입 commit이 된 레코드**들**을 읽을 때 수정 전과 수정 후의 결과가 달라지는 현상. 레코드 수준의 lock은 걸려서 undo로그에 조회가 가능하지만 새로 추가된 레코드에 대해선 Lock이 걸리지 않아 두번째 테이블 조회 시 나타나게 된다. (없었는데요 있었습니다~)

