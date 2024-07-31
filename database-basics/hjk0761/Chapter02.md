## 무결성 제약조건

무결성 제약조건은 데이터의 삽입, 업데이트, 삭제 등에 있어 데이터 무결성에 영향을 주지 못하게 함을 보장하는 규칙들의 집합입니다.
이는 데이터의 정확성과 일관성을 유지하는 데이터베이스를 보장합니다.

### 무결성 제약조건 종류
1. domain constraint
2. not-null constraint
3. entity integrity constraint
4. key constraint
5. primary key constraint
6. referential integrity constraint

### 도메인 제약조건
도메인 제약조건은 attribute 로 올수 있는 값에 대한 정의입니다.
도메인의 데이터 타입은 string, char, time, integer, date, currency 등이 될 수 있습니다.
attribute 의 값은 비교 가능한 도메인에서 사용될 수 있어야 합니다.

### Null이 아닌 제약조건
해당 제약조건은 튜플에서 null 이 아닌 제약조건에 지정된 속성에 null 값이 들어갈 수 없는 조건입니다.

### 엔티티 무결성 제약
이는 기본 키에 null 값이 포함될 수 없다는 제약조건입니다.
relation 의 각 행을 정의하는 고유한 값이기 때문입니다.

### 키 제약조건
엔티티는 여러 키를 가질 수 있고, 그 중 하나가 primary key가 됩니다. primary key는 항상 고유하고 null 값을 포함하지 않습니다.

### Primary key 제약조건
이는 primary key가 고유하며 null 값을 가지지 않음을 말합니다.
다시말해, 각 relation 에서 primary key 는 null 을 값으로 가질 수 없으며 두 튜플의 primary key 의 값은 같을 수 없습니다.

### 참고 무결성 제약조건
두 테이블 사이의 관계를 정의하는 제약조건입니다.
한 테이블의 foreign key 가 다른 테이블의 primary key 를 참조하는 경우 foreign key 를 사용하는 테이블에서의 값이 모두 null 이거나 다른 테이블의 primary key 로 존재해야 합니다.

이런 제약조건을 통해 데이터베이스가 신뢰할 수 있고, 일관되며 정확함을 보장할 수 있습니다.

## 키
키(key)는 데이터베이스에서 조건을 만족하는 튜플을 찾거나 정렬할 때에 다른 튜플과 구별할 수 있는 기준이 되는 attribute 를 의미합니다.

### Candidate key

후보키는 relation 을 구성하는 attribute 중에서, 튜플을 유일하게 식별할 수 있는 attribute 들의 부분집합입니다.
모든 relation 은 반드시 하나 이상의 후보키를 가져야 하고, 후보키는 모든 튜플에 대해 유일성과 최소성을 만족해야 합니다.

### Primary key
기본키는 후보키 중 튜플을 유일하게 구별할 수 있는 attribute입니다.
Null 값을 가질 수 없으며 동일한 값이 중복될 수 없습니다.

### Alternate key
대체키는 후보키가 둘 이상인 경우에서 기본키를 제외한 나머지 후보키를 말합니다.

### Super key
슈퍼키는 한 relation 내의 attribute 들의 집합으로 구성된 키입니다.
이는 모든 튜플에 대해 유일성은 만족하지만, 최소성은 만족하지 못합니다.

### Foreign key
외래키는 연관 관계가 있는 두 relation 에서, 한 테이블의 기본키를 참조하는 키를 의미합니다.
참고 무결성 제약조건에 의해 외래키로 지정되면 참조되는 기본키에 없는 값이 들어갈 수 없습니다.

## 관계대수
관계 대수(relational algebra)는 질의를 어떻게 수행할 지를 명시하는 절차적인 언어입니다.
차후 SQL(Structured Query Language)의 기반이 됩니다.

관계 대수는 관계 연산자들을 이용해 기존의 relation들로부터 새로운 relation을 만들어냅니다.

### 관계 연산자
1. selection(σ): relation에서 조건에 부합하는 튜플을 추출하는 연산자입니다.
실행 결과 relation은 기존 relation과 차수는 동일하고 cardinality는 작거나 같습니다.
문법은 σ_<조건>(Relation)입니다.:

### projection(π): relation에서 조건에 부합하는 attribute를 추출합니다.
실행 결과는 차수는 작거나 같고, cardinality는 같습니다.
문법은 π_<조건>(Relation)입니다.

### union(∪): 두 relation 을 합칩니다.
단, 두 relation은 서로 같은 attribute 순서를 가져고 domain이 같아야 합니다.
해당 연산의 결과 relation 의 속성은 연산 시 첫 relation 의 attribute 이름을 가지게 됩니다.
문법은 R1∪R2입니다.

### difference(-): union 과 같은 조건에서, 연산 시 앞 relation 에 속하지만, 뒤 relation에 속하지 않는 튜플을 반환합니다.
문법은 R1-R2입니다.

### cartesian product(x): 두 relation 의 cartesian product를 수행합니다.
두 relation 을 하나로 합칩니다.
각각 n, m의 차수를 가지는 두 relation에 대해, cartesian product는 n+m의 차수를 가지고 n*m의 cardinality를 가집니다.
두 relation의 튜플들의 가능한 모든 조합을 가지며, 앞 relation의 attribute와 뒤 relation 의 attribute 를 순차적으로 모두 가집니다.
문법은 R1xR2입니다.

### intersection(∩): 두 relation 에서 공통으로 가지는 튜플을 반환합니다.
union과 조건을 같습니다.
문법은 R1∩R2입니다.

### join(⋈): cartesian product 이후에 selection 을 수행한 것과 같습니다.
join에 참여하는 attribute는 동일한 도메인으로 구성되어야 합니다.
비교 조건에 의해 selection 되면 theta join, 동등 조건이라면 equal join, 동등 조건에서 중복된 attribute 를 제거한다면 natural join, 자연 join 에서 한 쪽의 attribute 를 제거한다면 semi join, 자연 join 이후 각 방향으로의 모든 값을 추출한다면 outer join 이 됩니다.
문법은 R1⋈R2입니다.

### division(÷): 뒤 relation 에 모든 값을 포함하는 앞 relation 의 튜플을 추출합니다.
문법은 R1÷R2입니다.

## 뷰
뷰는 가상의 테이블입니다.
제한된 정보를 보여주기 위해 기존의 테이블로부터 유도됩니다.
테이블과 달리 물리적으로 존재하지 않고, 필요한 부분만 테이블로부터 추출하여 사용하기 때문에 관리가 용이합니다.
기본이 되는 테이블의 기본키를 포함해야 연산이 가능합니다.
`CREATE VIEW {VIEW_NAME}[({ATTRIBUTE_NAME[, ATTRIBUTE_NAME])] AS SELCET;`와 같이 정의하고 `DROP VIEW` 를 통해 삭제합니다.

뷰를 사용함으로서 데이터 독립성을 제공하고, 같은 데이터를 사용자에 따라 다른 형태로 제공할 수 있습니다.
또한 접근 제어를 통해 보안성이 강화됩니다.
다만, ALTER문이 사용 불가능해 뷰의 정의를 바꾸기 위해선 기존 뷰를 삭제하고 다시 만들어야만 합니다.

## 인덱스
인덱스는 추가적인 쓰기 작업과 저장 공간을 활용해 전체적인 검색 시간을 향상시키는 자료구조입니다.
인덱싱이 되지 않은 상태에선 SELECT, UPDATE, DELETE 등을 수행할 때에 전체를 순회하면서 해당하는 값을 찾아야 하는데, 인덱싱이 적용된다면 해당 명령 모두 성능을 향상시킬 수 있습니다.
다만, 인덱스는 최신 상태로 유지해 주어야 하기 때문에 추가적인 연산이 필요합니다.
예를 들어 새로운 값이 INSERT 된다면 그 값에 대한 인덱스를 추가해 주어야 하고, DELETE, UPDATE 시에도 변경되는 값에 대한 인덱스를 사용하지 않음으로 업데이트 해주어야 합니다.
이런 추가적인 작업 때문에 CREATE, DELETE, UPDATE 가 빈번한 attribute라면, 오히려 성능 저하를 가져올 수 있습니다.

### 인덱스 자료구조
Hash Table(linked-list), B+-tree 등 다양한 자료구조를 사용합니다.