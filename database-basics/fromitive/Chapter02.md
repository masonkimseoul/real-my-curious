## 제약 조건

## (도메인) 무결성 제약조건
데이터베이스의 저장하는 데이터는 정확성과 일관성을 유지해야 한다.
입력될때 불필요한 오류와 부정확성을 방지하기 위해 사용됩니다.

### 고유성 제약 조건
~~특정 열에 있는 데이터는 모두 서로 다른 성질입니다.~~?
개체무결성, 참조 무결성

### 기본 키 제약 조건
테이블에서 각 행들을 고유하게 식별할 수 있는 제약조건입니다.

### 외래 키 제약 조건
한 테이블의 열이 다른 테이블의 기본키를 참조하도록 하는 조건

### 체크 제약 조건 (선택??)
각 열의 값이 비즈니스 로직을 만족해야 한다.

### NOT NULL 제약조건 (선택??)
null값을 허용하지 않음

## 키
### 기본키
각 행을 고유하게 식별하는 열

### 후보키
기본키가 될 수 있는 열들의 조합
슈퍼키에서 을 만족하는 거?

### 대체키
후보키 중 기본키로 선택되지 않은 나머지들

### 외래키
다른키를 참조할때 사용하는 키
`참조 무결성`을  위해 사용

### 슈퍼키
행을 유일하게 식별할 수 있는 키 인데, 조합을 해서 고유하게 만들어야 한다. 

#### 참고 : 참조 무결성
참조하는 외래키가 무조건 그 테이블의 기본키로 구성해야함
삭제 수정 규칙 외래키로 묶인 테이블을 삭제할때 처리할 수 있는 4가지 방법
* cascade - 연쇄 삭제
* set null - 외래 키 값을 null로 국성
* restrict - 삭제 수정 을 제한함

## 관계 대수
대수 수식: relation, tables, 

- **선택 (Selection, σ)**:  조건에 만족하는 행을 선택
    
- **투영 (Projection, π)**: 특정 열만 선택
    
- **합집합 (Union, ∪)**: 두 테이블을 이
    
- **차집합 (Difference, -)**: 첫 번째 테이블에서 두 번째 테이블에 있는 행을 제외한 결과를 반환합니다. 예를 들어, `Students - Graduates`는 Students 테이블에 있지만 Graduates 테이블에는 없는 학생을 반환합니다.
    
- **교집합 (Intersection, ∩)**: 두 테이블에 공통으로 있는 행을 반환합니다. 예를 들어, `Students ∩ Graduates`는 두 테이블에 모두 있는 학생을 반환합니다.
    
- **데카르트 곱 (Cartesian Product, ×)**: 두 테이블의 모든 행을 조합하여 가능한 모든 쌍을 반환합니다. 예를 들어, `Students × Courses`는 Students의 각 행과 Courses의 각 행을 조합한 결과를 반환합니다.
    
- **조인 (Join, ⋈)**: 두 테이블을 결합하여 공통 열을 기준으로 새로운 테이블을 만듭니다. 다양한 종류의 조인이 있으며, 가장 일반적인 것은 내부 조인입니다. 예를 들어, `Students ⋈ Courses`는 Students와 Courses의 공통 열을 기준으로 두 테이블을 결합합니다.

* **디비전 (÷)** : 우항의 릴레이션의 모든 값을 포함하는 좌항의 튜플을 추출

### 인덱스 (Index)

1. **검색 속도 향상**: 인덱스는 테이블의 열에 대한 검색 성능을 향상시키기 위해 사용됩니다. 인덱스는 특정 열의 데이터를 빠르게 검색할 수 있도록 데이터베이스 시스템에 별도의 구조로 저장됩니다. 예를 들어, 이름으로 사람을 검색할 때 인덱스가 있으면 더 빠르게 결과를 찾을 수 있습니다.
    
2. **고유성은 선택 사항**: 인덱스는 기본 키와 달리 고유성을 보장할 필요는 없습니다. 즉, 중복된 값이 있는 열에도 인덱스를 생성할 수 있습니다. 다만, 고유 인덱스(unique index)를 설정하면 해당 열의 데이터가 고유함을 보장할 수 있습니다.
    
3. **다양한 용도로 사용**: 인덱스는 검색, 정렬, 집계 등의 다양한 작업에서 성능을 향상시키기 위해 사용할 수 있습니다. 예를 들어, 특정 조건에 맞는 데이터의 빠른 필터링이 필요할 때 인덱스가 매우 유용합니다.
    
4. **여러 개의 인덱스**: 한 테이블에 여러 개의 인덱스를 만들 수 있습니다. 기본 키 외에도 검색 성능을 높이기 위해 여러 열에 대해 인덱스를 설정할 수 있습니다.

삽입 삭제는 좀 약하다. 
인덱스 20% ~ 25% 넘어가는 데이터에 설정하면, 탐색 알고리즘이 순차적으로 검색하는 것 보다 성능이 느려진다.