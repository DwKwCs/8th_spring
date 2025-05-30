## 추가 공부

RDBMS의 성능을 높이려면 어떻게 해야하는지에 대해서 궁금해졌다.

**데이터베이스에서 성능을 결정짓는 요인들**

- ## 인덱스

관계형 데이터베이스에서 인덱스(Index)는 데이터 검색 성능을 향상시키는 자료구조다.

테이블에서 특정 행을 빠르게 찾을 수 있도록 추가적인 데이터 구조(B- Tree, Hash 등)를 사용한다.

### 1. 인덱스의 개념

- 테이블의 특정 컬럼을 기준으로 정렬된 별도의 데이터 구조.
- 검색 속도를 향상시키지만, **삽입(INSERT), 수정(UPDATE), 삭제(DELETE) 성능은 약간 저하됨**.
- 일반적으로 **자주 조회되는 컬럼**에 적용.

### 2. 인덱스의 동작 원리

- 인덱스의 검색 원리
- 인덱스 없는 검색(Full Table Scan): 테이블의 모든 행을 확인해야 하므로 느리다.

        SELECT * FROM users WHERE name = 'Alice';

        - 인덱스 사용 검색(Indexed Search): 컬럼에 인덱스를 추가하면 빠르게 찾을 수 있다.

        CREATE INDEX idx_users_name ON users(name);
        SELECT * FROM users WHERE name = 'Alice';

### 3. 인덱스의 종류
    
- **기본 인덱스(Primary Index):** Primary Key가 자동으로 생성하는 클러스터형 인덱스(Clustered Index). ****데이터가 인덱스에 따라 정렬되어 저장됨.
- **보조 인덱스(Secondary Index):** Primary Key가 아닌 컬럼에 생성하는 인덱스. B- Tree 구조 사용.
- **유니크 인덱스(Unique Index):** 중복을 허용하지 않는 인덱스.
- **복합 인덱스(Composite Index):** 여러 개의 컬럼을 묶어서 생성하는 인덱스. 컬럼의 선언 순서가 중요하다.
- **FULLTEXT 인덱스(전문 검색 인덱스):** LIKE 검색이 아닌 자연어 검색을 지원.
- **공간 인덱스(Spatial Index):** GIS(지리 정보 시스템)에서 사용.
    
### 4. 인덱스의 장점과 단점
    
**장점**
    
- 검색 속도 향상 (`WHERE`, `ORDER BY` 최적화)
- 조인 성능 향상 (`JOIN` 시 인덱스가 있으면 속도가 빨라짐)
- 고유성(Unique) 보장 가능 (`UNIQUE INDEX`)
    
**단점**
    
- 삽입/수정/삭제 성능 저하 (인덱스 갱신 비용 발생)
- 메모리 사용 증가 (추가적인 저장 공간 필요)
- 과도한 인덱스 사용 시 역효과 (쿼리 최적화 실패 가능)


- ## 파티셔닝

  ### **1. 파티셔닝**

  파티셔닝(Partitioning)은 **큰 테이블을 여러 개의 작은 논리적 단위(파티션)로 나누어 저장하는 방법**이다.

  이를 통해 **쿼리 성능을 최적화하고, 관리 효율성을 높이며, I/O 부하를 분산**할 수 있다.

  **주요 목적**

    - **성능 향상**: 특정 파티션만 검색하여 쿼리 속도 증가
    - **관리 용이**: 오래된 데이터 파티션을 쉽게 삭제 또는 아카이브
    - **병렬 처리 최적화**: 여러 파티션에서 동시에 작업 가능
    - **부하 분산**: I/O와 CPU 부하를 분산하여 데이터베이스 성능 유지

  ### 2. 파티셔닝의 종류

    - **범위 파티셔닝(Range Partitioning):** 특정 범위(Range)를 기준으로 데이터를 나눔. 날짜, 숫자 등의 연속적인 값에 적합.
    - **리스트 파티셔닝(List Partitioning):** 특정 값(리스트)를 기준으로 데이터를 나눔. 지역, 카테고리, 상태 값 같은 특정 그룹이 있는 경우 유용.
    - **해시 파티셔닝(Hash Partitioning):** 특정 칼럼 값을 해싱(Hash)하여 균등하게 분산. 데이터가 균이랗게 분포되지 않을 때 유용.
    - **조합(복합) 파티셔닝(Composite Partitioning):** 여러 개의 파티셔닝 기법을 결합하여 사용. RANGE + HASH, LIST + HASH 조합이 일반적.

  ### 3. 파티셔닝의 장단점

  **장점**

    - **성능 향상:** 특정 파티션만 검색 가능→쿼리 속도 증가
    - **데이터 관리 용이:** 오랜된 파티션만 삭제 가능
    - **부하 분산:** I/O 및 CPU 사용량 분산
    - **백업 및 복구 용이:** 파티션 단위로 백업 가능

  **단점**

    - **INSERT/UPDATE 성능 저하**
    - **인덱스 관리 복잡**
    - **추가적인 스토리지 필요**
    - **잘못된 파티셔닝 시 성능 저하**
    - **테이블 설계가 복잡해짐**

  ### 4. sql에서 PARTION 활용

  테이블을 생성할 때 `PARTITION BY` 키워드를 사용하여 특정 컬럼 기준으로 데이터를 나눈다.

    ```sql
    CREATE TABLE 테이블명 (
        컬럼1 데이터타입,
        컬럼2 데이터타입,
        ...
    )
    PARTITION BY [방식] (기준컬럼) (
        PARTITION 파티션명 VALUES ...
    );
    ```

  파티션 추가, 삭제, 조회

    ```sql
    **파티션 추가**
    ALTER TABLE sales ADD PARTITION (PARTITION p_2023 VALUES LESS THAN (2024));
    
    **파티션 삭제**
    ALTER TABLE sales DROP PARTITION p_2020;
    
    **파티션 데이터 조회**
    SELECT * FROM sales PARTITION (p_2021) WHERE amount > 1000;
    ```


- ## JOIN(최적화)

  ### 1. JOIN 개념

  **JOIN**은 관계형 데이터베이스에서 **두 개 이상의 테이블을 연결하여 원하는 데이터를 조회**할 때 사용된다. 보통 **공통된 키(Primary Key & Foreign Key)를 기준으로 데이터를 결합**한다.

  ### 2. JOIN 종류

    - **INNER JOIN:** 두 테이블에서 공통된 값이 있는 행만 반환
    - **LEFT JOIN (LEFT OUTER JOIN):** 왼쪽 테이블의 모든 데이터 + 오른쪽 테이블과 일치하는 데이터 반환 (오른쪽 테이블에 없는 데이터는 NULL)
    - **RIGHT JOIN (RIGHT OUTER JOIN):** 오른쪽 테이블의 모든 데이터 + 왼쪽 테이블과 일치하는 데이터 반환(왼쪽 테이블에 없는 데이터는 NULL)
    - **FULL JOIN (FULL OUTER JOIN):** 두 테이블의 모든 데이터 반환(일치하지 않는 데이터는 NULL 처리)
    - **CROSS JOIN:** 두 테이블의 모든 행을 곱집합(카테시안 곱)으로 반환

  ### 3. JOIN 최적화 팁

    - **JOIN 키에 인덱스 생성** → 성능 향상
    - **필요한 컬럼만 SELECT** → 불필요한 데이터 조회 방지
    - **JOIN 순서 최적화** → 작은 테이블을 먼저 결합하면 효율적

  **INNER JOIN이 가장 빠르고, OUTER JOIN은 상대적으로 느림**

  **데이터 특성에 맞는 JOIN을 선택하면 성능을 향상할 수 있음**

  ### JOIN 최적화

  | 최적화 방법 | 설명 |
      | --- | --- |
  | **1. 인덱스 활용** | `JOIN` 키에 인덱스 생성 (`PRIMARY KEY, FOREIGN KEY`) |
  | **2. 불필요한 컬럼 제거** | `SELECT *` 대신 필요한 컬럼만 조회 |
  | **3. 조인 순서 최적화** | 작은 테이블을 먼저 `JOIN` |
  | **4. WHERE 절 조기 필터링** | `JOIN` 전에 `WHERE`로 데이터 축소 |
  | **5. EXISTS vs IN 최적 활용** | 작은 서브쿼리는 `IN`, 큰 서브쿼리는 `EXISTS` |
  | **6. JOIN 방식 선택** | `Nested Loop` vs `Hash Join` 분석 (`EXPLAIN` 활용) |
  | **7. PARTITION 활용** | 대량 데이터 테이블에 `PARTITION` 적용 |
  | **8. ORDER BY & LIMIT 최적화** | `ORDER BY` 컬럼에 인덱스 생성 |
  | **9. JOIN 대신 UNION ALL 사용** | 단순 데이터 결합 시 `UNION ALL` 활용 |

- ## 쿼리 캐싱

  ### 1. 쿼리 캐싱(Query Caching)

  쿼리 캐싱은 자주 실행되는 SQL 쿼리의 결과를 저장하여 재사용함으로써 데이터베이스의 성능을 향상시키는 기술이다.

  일반적으로 같은 쿼리를 여러 번 실행할 때 데이터베이스에서 동일한 연산을 반복하지 않도록 함으로써 속도를 개선한다.

  ### 2. 쿼리 캐싱의 동작 원리

  쿼리 캐싱은 다음과 같은 방식으로 동작한다.

    1. 클라이언트가 SQL 쿼리를 요청한다.
    2. DBMS가 **쿼리 캐시(Query Cache)** 를 먼저 확인한다.
    3. **같은 쿼리의 결과가 캐시에 저장되어 있으면, 캐시에서 결과를 반환**한다.
    4. **캐시에 없으면 쿼리를 실행하고, 결과를 저장한 후 반환**한다.
    5. 이후 동일한 쿼리가 실행되면, 데이터베이스 연산 없이 캐시된 결과를 사용한다.

  **즉, 동일한 요청이 반복될 경우 DB 부하를 줄이고 응답 속도를 향상시킴!**

  ### 3. 쿼리 캐싱의 종류

    - **데이터베이스 레벨 캐싱(DB Query Cache)**
        - DBMS 자체적으로 쿼리 결과를 캐싱하여 재사용
        - 같은 SQL 문장이 다시 실행되면 캐싱된 결과를 반환
    - **애플리케이션 레벨 캐싱(Application Cache)**
        - 애플리케이션 단에서 쿼리 결과를 캐싱
        - DB 요청을 줄이기 위해 Redis, Memcached 같은 인메모리 캐시 사용
        - 웹 서버, API 서버에서 캐싱하면 DB 부하를 줄일 수 있음
        - 첫 요청 시 DB에서 조회하고, 이후에는 인메모리 캐시에서 조회하여 속도 개선
    - **프록시 레벨 캐싱(Proxy Cache)**
        - DB 프록시 캐시 활용
        - 로드 밸런싱 및 캐싱 가능을 동시에 제공하여 성능 개선 가능
    - **브라우저 및 CDN 캐싱**
        - 웹 애플리케이션의 경우 클라이언트 측에서 캐싱
        - REST API에서 동일한 요청이 반복될 경우, HTTP 캐시를 활용
        - 이미지, 정적 파일 등을 CDN(Content Delivery Network) 캐시로 저장하여 성능 향상

  ### 4. 쿼리 캐싱의 장점과 단점

  **장점**

    - **성능 향상:** 반복되는 쿼리 실행을 줄여 속도를 높임
    - **DB 부하 감소:** DB 연산을 최소화하여 트래픽 분산
    - **빠른 응답 속도:** 네트워크 및 디스크 I/O 비용 절감
    - **비용 절감:** 클라우드 DB 사용 시 쿼리 비용 감소

  **단점(주의할 점)**

    - 데이터 동기화 문제: 캐시된 데이터가 오래되념 캐시 무효화(Cache Invalidation) 필요
    - 메모리 사용 증가: 대량 데이터를 캐싱하면 메모리 부담 증가
    - 캐시 관리 필요: TTL(만료 시간) 설정, 데이터 갱신 등의 유지보수 필요

  ### 5. 쿼리 캐싱 최적화 전략

    - **변하지 않는 데이터(정적 데이터) 위주로 캐싱**
        - 예: 카테고리 목록, 국가 코드, 제품 리스트 등
    - **TTL 설정을 적절히 조정하여 메모리 낭비 방지**
    - **변경이 자주 일어나는 데이터는 캐싱보다 인덱스를 활용**
    - **Redis/Memcached 같은 인메모리 캐시 활용하여 속도 향상**
    - **데이터 변경 시 캐시를 무효화하여 동기화 유지**

이외에 여러가지 방법이 있지만 너무 많아서 나중에 찾아봐야갰다….

- ## **RDBMS의 구조**

  ### **RDBMS(Relational Database Management System) 구조**

  RDBMS는 관계형 데이터베이스를 관리하는 시스템으로, **데이터 저장, 관리, 검색, 보안 및 동시성 제어**를 담당한다.

  RDBMS는 내부적으로 여러 개의 **논리적 및 물리적 구성 요소**로 나뉜다.

  여기서는 물리적 구성을 알아보겠다.
    
  ---

  ## **1. 물리적 구조 (Physical Structure)**

    - **스토리지 엔진(Storage Engine)**
    - **데이터 파일(Data File)**
    - **로그 파일(Log File)**
    - **버퍼 풀(Buffer Pool)**
    - **캐시(Cache)**
    - **프로세스 관리자(Process Manager)**

  RDBMS 내부에서 데이터를 저장하고 관리하는 물리적인 구성 요소들.

  ### **(1) 스토리지 엔진(Storage Engine)**

    - 데이터를 **디스크에 저장하고 검색하는 방식**을 결정하는 엔진.
    - 대표적인 스토리지 엔진:
        - **MySQL**: InnoDB, MyISAM, MEMORY
        - **PostgreSQL**: MVCC 기반 스토리지
        - **Oracle**: Oracle Database Storage Engine

  ### **(2) 데이터 파일(Data File)**

    - 실제 데이터를 저장하는 파일.

  ### **(3) 로그 파일(Log File)**

    - **트랜잭션 로그, 변경 사항 로그를 저장**하는 파일.
    - 시스템 장애 발생 시, **데이터 복구(Redo, Undo Logs)**에 사용됨.

  **로그를 활용하여 장애 발생 시 데이터 복구 가능.**

  ### **(4) 버퍼 풀(Buffer Pool)**

    - 데이터베이스가 **자주 사용하는 데이터를 메모리에 저장**하는 영역.
    - **디스크 I/O를 최소화**하여 성능을 향상시킴.

  ### **(5) 캐시(Cache)**

    - **최근 실행된 쿼리 결과를 저장**하여 성능을 향상시킴.
    - `Query Cache`, `Index Cache`, `Table Cache` 등이 있음.

  ### **(6) 프로세스 관리자(Process Manager)**

    - 데이터베이스에서 **사용자의 요청을 처리하고, 세션을 관리**하는 역할.

  ## **2. RDBMS 아키텍처 요약**

  | 계층 | 주요 구성 요소 | 설명 |
      | --- | --- | --- |
  | **물리적 구조** | 스토리지 엔진 | 데이터를 저장하고 관리하는 핵심 엔진 |
  |  | 데이터 파일 | 실제 데이터를 저장하는 파일 |
  |  | 로그 파일 | 트랜잭션 로그 및 변경 사항 저장 |
  |  | 버퍼 풀 | 자주 사용되는 데이터를 메모리에 저장 |
  |  | 캐시 | 쿼리 및 인덱스 캐싱을 통한 성능 향상 |
  |  | 프로세스 관리자 | SQL 요청을 처리하고 세션을 관리 |

- ## **RDBMS에서 sql문을 처리하는 과정**

  RDBMS(Relational Database Management System)에서 SQL 문이 처리되는 과정은 **사용자의 SQL 문이 어떻게 해석되고 실행되며 최적화되는지**를 이해하는 데 중요하다.

  SQL 문이 실행될 때, **파싱 → 최적화 → 실행 → 결과 반환**의 단계를 거친다.
    
  ---

  ## **1. SQL 처리 과정 개요**

  SQL 문이 실행되는 과정은 다음과 같이 나눌 수 있다.

    1. **파싱(Parsing)**
        - SQL 문법 검증 및 구조 분석
    2. **옵티마이저(Optimizer) - 실행 계획 수립**
        - 최적의 실행 경로를 계산
    3. **실행(Execution)**
        - 데이터를 읽고, 쓰고, 조작
    4. **결과 반환(Fetch & Return Results)**
        - 결과를 사용자에게 반환

  아래에서 각 단계를 자세히 살펴보자.
    
  ---

  ## **2. SQL 문 처리 상세 과정**

  ### **① 파싱(Parsing)**

  SQL 문이 실행되면, 먼저 **SQL Parser(파서)**가 문법을 분석한다.

  ### **1) SQL 문법 검사 (Lexical & Syntax Analysis)**

    - SQL 문이 올바르게 작성되었는지 확인하는 과정.
    - SQL 문을 토큰(token)으로 분해한 후, **예약어, 테이블명, 연산자, 데이터 값 등을 구별**한다.
    - *구문 오류(Syntax Error)**가 발생하면 즉시 반환하고 실행이 중단됨.

  ### **2) 객체(테이블, 컬럼) 존재 여부 확인**

    - SQL에서 참조하는 테이블, 컬럼이 실제 데이터베이스에 존재하는지 확인.
    - 예를 들어 `SELECT * FROM non_existing_table;`과 같은 경우, 이 단계에서 오류 발생.

  ### **3) 내부 트리 구조(파스 트리, Parse Tree) 생성**

    - SQL을 **트리(Tree) 형태의 내부 구조**로 변환하여 처리 속도를 높임.

  🔹 **예제: `SELECT id, name FROM users WHERE age > 30;`**

    ```
    SELECT
     ├── Columns: id, name
     ├── FROM Table: users
     ├── WHERE age > 30
    
    ```

  **이 단계가 완료되면, SQL 문이 실행될 수 있는 기본 조건이 충족됨.**
    
  ---

  ### **② 옵티마이저(Optimizer) - 실행 계획 수립**

  옵티마이저(Optimizer)는 **SQL을 가장 효율적으로 실행할 방법을 찾는 역할**을 한다.

  ### **1) 실행 계획(Execution Plan) 생성**

    - SQL을 실행하는 여러 가지 방법 중에서 **가장 비용이 적게 드는 방법**을 찾는다.
    - 데이터 검색 시, **인덱스 사용 여부**와 **조인 방식(Nested Loop, Hash Join, Merge Join)** 등을 결정.

  ### **2) 비용 기반 최적화 (Cost-Based Optimization, CBO)**

    - **비용(cost) 계산**: 테이블 크기, 인덱스 여부, 통계 정보를 이용하여 실행 비용을 평가.
    - **예제: `SELECT * FROM users WHERE age > 30;`**
        - `users` 테이블에 `age` 컬럼에 인덱스가 있다면 → **인덱스를 사용하여 검색**.
        - `users` 테이블에 `age` 인덱스가 없다면 → **풀 테이블 스캔(Full Table Scan)**.

  ### **3) 쿼리 리라이트(Query Rewrite)**

    - SQL을 더 최적화된 형태로 변환할 수도 있음.
    - 예를 들어, **`OR`을 `UNION`으로 변환**하거나, 불필요한 `JOIN`을 제거하는 등의 최적화 수행.

  **이 과정에서 실행 계획을 생성하고, 실행할 준비를 마침.**
    
  ---

  ### **③ SQL 실행(Execution)**

  이제 SQL을 실제로 실행하는 단계이다.

  ### **1) 접근 방식 결정**

    - 옵티마이저에서 생성한 실행 계획을 기반으로 **데이터 검색**.
    - 선택된 접근 방식:
        - **인덱스 검색(Index Scan)**
        - **순차 검색(Sequential Scan)**
        - **조인 방식 선택 (Nested Loop, Hash Join, Merge Join 등)**

  ### **2) 데이터 조작**

    - `SELECT`: 데이터를 읽음.
    - `INSERT`: 데이터를 추가.
    - `UPDATE`: 데이터를 변경.
    - `DELETE`: 데이터를 삭제.

  **이 단계에서 실제 테이블 또는 인덱스에서 데이터를 가져오거나 변경 작업이 수행됨.**
    
  ---

  ### **④ 결과 반환(Fetch & Return Results)**

  마지막으로 실행된 SQL 문 결과를 사용자에게 반환한다.

  ### **1) 데이터 정렬(Sorting)**

    - `ORDER BY`, `GROUP BY`를 사용한 경우 **정렬(Sorting) 및 그룹핑(Grouping)** 작업 수행.

  ### **2) 페이징(Pagination)**

    - `LIMIT`, `OFFSET`을 사용하여 필요한 데이터만 반환.

  ### **3) 네트워크 전송**

    - 최종 결과를 **클라이언트 애플리케이션(웹 서버, API 등)으로 전송**.

  **최적화된 방식으로 데이터를 가져오고, 원하는 형식으로 반환하여 SQL 처리가 완료됨.**

  ## **4. SQL 처리 과정 요약**

    1. **파싱(Parsing)** → SQL 문법 검사 및 파스 트리 생성.
    2. **옵티마이저(Optimizer)** → 실행 계획을 수립하여 최적의 경로를 결정.
    3. **실행(Execution)** → 최적화된 방식으로 데이터 읽기/쓰기 수행.
    4. **결과 반환(Fetch & Return Results)** → 결과를 정렬, 페이징 후 클라이언트로 전달.

  **이 과정을 거쳐 SQL 문이 실행되며, 데이터베이스 성능을 최적화하려면 옵티마이저의 실행 계획을 분석하는 것이 중요하다.**