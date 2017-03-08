# 설치

* bat 파일 만들어서 설치
```
C:\Users\Administrator\Downloads\postgresql-9.6.2-2-windows.exe  --serverport 5432 --servicename postgres_service --locale C --superaccount admin --superpassword shpa$$w0rd12 --unattendedmodeui minimal --debuglevel 2 --mode unattended
```
* 외부 접속 허용 설정
pg_hba.conf 파일에 아래 내용 추가
```
host    all             all             192.168.1.0/24            md5
```

* 방화벽에서 TCP port 5432 허용

* SSL 설정

* Database 및 Table 생성 스크립트 실행

* samsVX Lite 서버에서 접속할 사용자 생성 및 설정

**위 항목들을 설치 시 자동화 할 수 있는 방법 필요**

<br/>
# C# 에서 연결

* Nuget에서 Npgsql 설치
```cs
using Npgsql;
```

* 연결
```cs
using (var conn = new NpgsqlConnection("Host=192.168.10.158;Username=admin;Password=shpa$$w0rd12;Database=testdb"))
{
  conn.Open();
}
```

<br/>
# CRUD 테스트

* SELECT
```cs
using (var conn = new NpgsqlConnection("Host=192.168.10.158;Username=admin;Password=shpa$$w0rd12;Database=testdb"))
{
    conn.Open();
    using (var cmd = new NpgsqlCommand())
    {
        cmd.Connection = conn;

        cmd.CommandText = "SELECT * FROM addresses";

        using (var reader = cmd.ExecuteReader())
        {
            while (reader.Read())
            {
                var data = new string[] { reader["id"].ToString(),
                    reader["name"].ToString()};

                foreach (var x in data)
                {
                    Console.Write(x);
                    Console.Write(" -- ");
                }
                Console.WriteLine();
            }
        }
    }
}
```

* INSERT
```cs
using (var conn = new NpgsqlConnection("Host=192.168.10.158;Username=admin;Password=shpa$$w0rd12;Database=testdb"))
{
    conn.Open();
    using (var cmd = new NpgsqlCommand())
    {
        cmd.Connection = conn;

        cmd.CommandText = "INSERT INTO addresses VALUES(@id, @name)";

        cmd.Parameters.AddWithValue("@id", 3);
        cmd.Parameters.AddWithValue("@name", "tester");

        cmd.ExecuteNonQuery();
    }
}

```

* UPDATE
```cs
using (var conn = new NpgsqlConnection("Host=192.168.10.158;Username=admin;Password=shpa$$w0rd12;Database=testdb"))
{
    conn.Open();
    using (var cmd = new NpgsqlCommand())
    {
        cmd.Connection = conn;

        cmd.CommandText = "UPDATE addresses SET name = @name WHERE id = @id";

        cmd.Parameters.AddWithValue("@name", "Steve");
        cmd.Parameters.AddWithValue("@id", 3);

        cmd.ExecuteNonQuery();
    }
}
```

* DELETE
```cs
using (var conn = new NpgsqlConnection("Host=192.168.10.158;Username=admin;Password=shpa$$w0rd12;Database=testdb"))
{
    conn.Open();
    using (var cmd = new NpgsqlCommand())
    {
        cmd.Connection = conn;

        cmd.CommandText = "DELETE FROM addresses WHERE id = @id";

        cmd.Parameters.AddWithValue("@id", 3);

        cmd.ExecuteNonQuery();
    }
}
```

<br/>
# Advanced Features

## Views

별 특징 없음

```sql
CREATE VIEW myview AS
SELECT city, temp_lo, temp_hi, prcp, date, location
FROM weather, cities
WHERE city = name;
SELECT * FROM myview;
```

<br/>
## Foreign Keys

지정된 컬럼이 테이블을 참조할때, 그 값이 테이블에 존재할때만 사용한다는 제약 조건

아래와 같이 두 개의 테이블이 있을 때

```sql
CREATE TABLE cities (
city varchar(80) primary key,
location point
);

CREATE TABLE weather (
city varchar(80) references cities(city),
temp_lo int,
temp_hi int,
prcp real,
date date
);
```

다음과 같이 레코드 삽입을 시도 하면 에러가 발생

```sql
INSERT INTO weather VALUES (’Berkeley’, 45, 53, 0.0, ’1994-11-28’);
```

```
ERROR: insert or update on table "weather" violates foreign key constraint "weather_city_fkey"
DETAIL: Key (city)=(Berkeley) is not present in table "cities".
```
<br/>
## Transactions

완벽하게 모든 단계가 완료 되었거나, 전혀 발생하지 않아거나

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100.00
WHERE name = ’Alice’;
-- etc etc
COMMIT;
```

PostgreSQL은 실제로 모든 SQL 문이 트랜잭션 내에서 실행되는 것으로 취급된다.
Begin 명령이 없을 경우 각각의 개별 문구는 앞뒤에 암시적으로 BEGIN 및 COMMIT 문을 갖는다.

SAVEPOINT를 사용하여 저장지점을 정의 한 후, ROLLBACK TO 지점까지 롤백 할 수 있다.

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100.00
WHERE name = ’Alice’;
SAVEPOINT my_savepoint;
UPDATE accounts SET balance = balance + 100.00
WHERE name = ’Bob’;
-- oops ... forget that and use Wally’s account
ROLLBACK TO my_savepoint;
UPDATE accounts SET balance = balance + 100.00
WHERE name = ’Wally’;
COMMIT;
```
<br/>
## Window Functions

테이블에서 행 집합을 대상으로 계산하는 함수

다음 예제는 각 부서별 평균 임금과 각 직원별 급여를 비교하는 것이다.

```sql
SELECT depname, empno, salary, avg(salary) OVER (PARTITION BY depname) FROM empsalary;
```

depname  | empno | salary |          avg
------------ | ------------- | ------------ | ------------
develop   |    11 |   5200 | 5020.0000000000000000
develop   |     7 |   4200 | 5020.0000000000000000
develop   |     9 |   4500 | 5020.0000000000000000
develop   |     8 |   6000 | 5020.0000000000000000
develop   |    10 |   5200 | 5020.0000000000000000
personnel |     5 |   3500 | 3700.0000000000000000
personnel |     2 |   3900 | 3700.0000000000000000
sales     |     3 |   4800 | 4866.6666666666666667
sales     |     1 |   5000 | 4866.6666666666666667
sales     |     4 |   4800 | 4866.6666666666666667

<br/>
## Inheritance

테이블 상속 개념 제공

cities (도시) 테이블과 capitals (수도) 테이블 두개가 있을때, 전통적인 개념의 자료 구조

```sql
CREATE TABLE capitals (
  name       text,
  population real,
  altitude   int,    -- (in ft)
  state      char(2)
);

CREATE TABLE non_capitals (
  name       text,
  population real,
  altitude   int     -- (in ft)
);

CREATE VIEW cities AS
  SELECT name, population, altitude FROM capitals
    UNION
  SELECT name, population, altitude FROM non_capitals;
```
<br/>
상속 개념으로 변환 하면
```sql
CREATE TABLE cities (
  name       text,
  population real,
  altitude   int     -- (in ft)
);

CREATE TABLE capitals (
  state      char(2)
) INHERITS (cities);
```

inherits 예약어를 사용함으로써, cities 테이블의 모든 컬럼을 상속 받음. 또한 테이블의 자료 조회는 하위 테이블의 모든 자료를 포함해서 조회 한다.

예를 들어, 고도가 500 ft. 보다 큰 도시들을 모두 찾을 때, 물론 이때 수도도 포함하고자 할 때는 다음과 같은 쿼리를 이용.
```sql
SELECT name, altitude
  FROM cities
  WHERE altitude > 500;
```
name    | altitude
---------- | ----------
Las Vegas |     2174
Mariposa  |     1953
Madison   |      845

<br/>
수도를 빼고 검색하려면 다음과 같이 ONLY 예약어 사용
```sql
SELECT name, altitude
    FROM ONLY cities
    WHERE altitude > 500;
```
name    | altitude
----------- | ----------
Las Vegas |     2174
Mariposa  |     1953

<br/>
# Data Type

integer, char, date 형은 다른 DBMS와 대체적으로 비슷

<br/>
### Enumerated Types

Enum 타입은 CREATE TYPE 명령을 사용해서 생성 된다.
```sql
CREATE TYPE mood AS ENUM ('sad', 'ok', 'happy');
```
<br/>
한번 생성되면, 다른 타입처럼 사용 가능
```sql
CREATE TYPE mood AS ENUM ('sad', 'ok', 'happy');
CREATE TABLE person (
    name text,
    current_mood mood
);
INSERT INTO person VALUES ('Moe', 'happy');
SELECT * FROM person WHERE current_mood = 'happy';
```
name | current_mood
------ | --------------
Moe  | happy

<br/>
### Network Address Type

IPv4, IPv6, Mac 주소 전용 타입 지원

Name | Storage Size | Description
------ | ------- | -------
cidr |	7 or 19 bytes	| IPv4 and IPv6 networks
inet	| 7 or 19 bytes	| IPv4 and IPv6 hosts and networks
macaddr	| 6 bytes	| MAC addresses

<br/>
### UUID Type

8자리-4자리-4자리-4자리-12자리, 총 32자리의 16진수로 이루어진다. (128비트)

```
550e8400-e29b-41d4-a716-446655440000
```

<br/>
### XML Type

xmlparse 함수를 이용하여 xml 데이터를 생성

```xml
XMLPARSE (DOCUMENT '<?xml version="1.0"?>
<tutorial>
<title>PostgreSQL Tutorial </title>
   <topics>...</topics>
</tutorial>')

XMLPARSE (CONTENT 'xyz<foo>bar</foo><bar>foo</bar>')
```

<br/>
### JSON Type

JSON 타입은 저장된 값이 JSON 타입에 유효한지 확인 한다.
다음은 JSON Data Type에 유용한 함수이다.
Example	| Example Result
--------- | -----------
array_to_json('{{1,5},{99,100}}'::int[])	| [[1,5],[99,100]]
row_to_json(row(1,'foo'))	| {"f1":1,"f2":"foo"}

<br/>
### Array Type

배열에는 기본, 열거, 사용자 정의 타입등 대부분 타입을 지정할 수 있다.

**배열 선언**
```sql
CREATE TABLE monthly_savings (
   name text,
   saving_per_quarter integer[],
   scheme text[][]
);
```
다음과 같이 "ARRAY" 키워드를 사용할 수도 있다.
```sql
CREATE TABLE monthly_savings (
   name text,
   saving_per_quarter integer ARRAY[4],
   scheme text[][]
);
```

<br/>
**Insert 배열**
```sql
INSERT INTO monthly_savings
VALUES ('Manisha',
'{20000, 14600, 23500, 13250}',
'{{"FD", "MF"}, {"FD", "Property"}}');
```

<br/>
**Select 배열**
```sql
SELECT name FROM monhly_savings WHERE saving_per_quarter[2] > saving_per_quarter[4];
```
<br/>
**Update 배열**
```sql
UPDATE monthly_savings SET saving_per_quarter = '{25000,25000,27000,27000}'
WHERE name = 'Manisha';

또는

UPDATE monthly_savings SET saving_per_quarter = ARRAY[25000,25000,27000,27000]
WHERE name = 'Manisha';
```

<br/>
**Where 배열**
```sql
SELECT * FROM monthly_savings WHERE saving_per_quarter[1] = 10000 OR
saving_per_quarter[2] = 10000 OR
saving_per_quarter[3] = 10000 OR
saving_per_quarter[4] = 10000;
```

배열 사이즈를 모를땐 다음과 같이 사용
```sql
SELECT * FROM monthly_savings WHERE 10000 = ANY (saving_per_quarter);
```

<br/>
### Composite Types

**컴포지트 타입 선언**
```sql
CREATE TYPE inventory_item AS (
   name text,
   supplier_id integer,
   price numeric
);
```
```sql
CREATE TABLE on_hand (
   item inventory_item,
   count integer
);
```

<br/>
**Insert 컴포지트 타입**
```sql
INSERT INTO on_hand VALUES (ROW('fuzzy dice', 42, 1.99), 1000);
```

<br/>
**Select 컴포지트 타입**
```sql
SELECT (item).name FROM on_hand WHERE (item).price > 9.99;
```
```sql
SELECT (on_hand.item).name FROM on_hand WHERE (on_hand.item).price > 9.99;
```

<br/>
### Range Types

범위를 값으로 가지는 타입

* int4range - Range of integer
* int8range - Range of bigint
* Range of numeric
* Range of timestamp without time zone
* tstzrange - Range of timestamp with time zone
* daterange - Range of date

<br/>
# Indexes

인덱스 생성
```sql
CREATE INDEX index_name ON table_name;
```

PostgreSQL은 몇가지 인덱스 타입들을 지원한다.(B-tree, Hash, GiST, SP-GiST, GIN)

CREATE INDEX 명령은 기본으로 B-tree 인덱스로 생성

<br/>
### Single-Column Indexes
```sql
CREATE INDEX index_name
ON table_name (column_name);
```

<br/>
### Multicolumn Indexes
```sql
CREATE INDEX index_name
ON table_name (column1_name, column2_name);
```

<br/>
### Unique Indexes
Unique 인덱스는 성능 뿐만 아니라 데이터 무결성을 위해서 사용 된다. Unique 인덱스는 중복 값 Insert를 허용하지 않는다.

```sql
CREATE UNIQUE INDEX index_name
on table_name (column_name);
```

<br/>
### Partial Indexes
테이블의 일부분만을 대상으로 생성된 인덱스. 일부분이라는 것은 조건문에 의해서 정의 된다.
```sql
CREATE INDEX index_name
on table_name (conditional_expression);
```

<br/>
### Drop Index
```sql
DROP INDEX index_name;
```

<br/>
### 인덱스를 사용하지 말아야 할 경우
* 테이블 크기가 작을 때
* 테이블이 잦은 주기로, 그리고 많은 batch Update 또는 Insert 명령이 실행 될 때
* 컬럼이 Null 인 경우가 대부분일 때
* 컬럼의 값이 자주 변경 될 때

<br/>
# Concurrency Control
내부적으로, 데이터 일관성은 MVCC(Multiversion Concurrency Control)을 사용하여 유지 된다. 즉, 데이터베이스를 쿼리하는 동안 각 트랜잭션은 기본 데이터의 현재 상태와 관계없이 이전 데이터의 스냅 샷을 볼 수 있다. MVCC는 전통적인 데이터베이스 시스템의 잠금 방법론을 피함으로써, 잠금 경합을 최소화한다.

MVCC가 잠금모델과 비교하여 가장 큰 이점은, 읽기 블럭과 쓰기 블럭이 충돌하지 않는다는 점이다.

PostgreSQL은 테이블 및 행 수준의 잠금 기능을 사용할 수 있다.

<br/>
### 트랜잭션 격리

트랜잭션 4가지 격리 수준

Isolation Level |	Dirty Read	| Nonrepeatable Read	| Phantom Read
------- | --------- | --------- | ---------
Read uncommitted	| Possible	| Possible	| Possible
Read committed	| Not possible	| Possible	| Possible
Repeatable read	| Not possible	| Not possible	| Possible
Serializable	| Not possible	| Not possible	| Not possible

**dirty read** : 커밋 되지 않은 트랜잭션이 수정한 데이터를 다른 트랜잭션이 읽는다.

**nonrepeatable read** : 한 트랜잭션 내에서 같은 쿼리를 두번 수행 할 때 그 사이에 다른 트랜잭션이 값을 수정 또는 삭제함으로써 두 쿼리의 결과가 상이하게 나타나는 비일관성이 발생하는 것을 말함

 **phantom read** : 한 트랜잭션 안에서 일정 범위의 레코드를 두번 이상 읽을 때, 첫번째 쿼리에서 없던 레코드가 두번째 쿼리에서 나타나는 현상. 이는 트랜잭션 도중 새로운 레코드가 삽입되는 것을 허용하기 때문에 나타나는 현상임.
