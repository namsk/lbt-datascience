## Hive

### About Hive

- 하둡의 MapReduce는 복잡도가 높은 프로그래밍 기법이 필요
- 프로그래밍 기술이 없는 업무 분석가 및 관리자들이 빅데이터에 접근하는 것에 어려움
- 이를 해결하기 위해 페이스북 주도로 SQL과 매우 유사한 방식으로 하둡 데이터에 접근성을 높인 **하이브(Hive)**를 개발하여 공개
- 공식 홈페이지: http://hive.apache.org/


### Hive 주요 구성 요소

- CLI : 사용자가 Hive 쿼리를 입력하고 실행할 수 있는 인터페이스
- JDBC/ODBC 드라이버 : 하이브의 쿼리를 다양한 데이터베이스와 연결하기 위한 드라이버
- Query Engine : 사용자가 입력한 하이브 쿼리를 분석, 실행계획을 수립하고 하이브QL을 맵리듀스 코드로 변환하여 실행

### Hive Architecture

![Hive Architecture](./images/hive-architecture.jpg)

- 하이브 클라이언트에서 작성한 QL이 맵리듀스 프로그램을 전환되어 실행
- MetaStore : 
  - 하이브에서 사용하는 테이블의 스키마 정보를 저장 및 관리
  - 기본적으로는 Derby DB가 사용되지만 MySQL, PostgreSQL 등 다른 DBMS로 변경 가능
- Thrift API를 제공, 클라이언트 프로그램이 다양한 하이브 액션을 외부에서도 실행할 수 있게 해 줌

### Hive: Installation

#### 프로그램 다운로드 및 원하는 위치로 이동

```bash
wget https://downloads.apache.org/hive/hive-2.3.7/apache-hive-2.3.7-bin.tar.gz
tar zxvf apache-hive-2.3.7-bin.tar.gz
sudo mv apache-hive-2.3.7-bin /usr/local/hadoop
```

#### 실행 스크립트 수정

- conf/hive-env.sh.template을 conf/hive-env.sh로 복사하여 사용

```bash
# in hive-env.sh
HADOOP_HOME=/usr/local/hadoop
```

#### 하이브 실행을 위한 환경 변수 설정

```bash
# in .bashrc
# Hive Setting
export HIVE_HOME=/usr/local/hive
export PATH=$HIVE_HOME/bin:$PATH
```

#### Hive 실행을 위한 필수 디렉터리를 HDFS에 생성 및 설정

- 하이브에서 업로드하는 데이터는 HDFS의 /user/hive/warehouse에 저장
- 하이브에서 실행하는 잡의 여유공간을 위한 /tmp/hive 디렉터리 필요

```bash
hdfs dfs -mkdir -p /user/hive/warehouse
hdfs dfs -mkdir -p /tmp/hive
hdfs dfs -chmod g+w /user/hive/warehouse
hdfs dfs -chmod 777 /tmp/hive
```

#### 메타스토어 초기화

```bash
# in HIVE_HOME
./bin/schematool -initSchema -dbType derby
# hive 실행
./bin/hive
hive> show databases;
hive> quit;
```

#### 데이터베이스 관리

```bash
# 데이터베이스의 생성과 관리
create database <데이터베이스명>;
use <데이터베이스명>;
# 데이터베이스 삭제
drop database <데이터베이스명>;
# 데이터베이스 내 테이블 확인
show tables;
```

#### 테이블 생성

- CREATE TABLE 문 사용
  - ROW FORMAT 절을 제외하면 SQL과 매우 유사
  - CREATE EXTERNAL TABLE을 사용하면 외부 테이블 생성 가능
    - 외부 테이블은 테이블을 DROP 해도 데이터가 보관된다는 장점

```sql
-- 기본 문법
CREATE EXTERNAL TABLE airline(
    YEAR INT,
    MONTH INT,
    DAY_OF_MONTH INT,
    DAY_OF_WEEK INT,
    FL_DATE DATE,
    UNIQUE_CARRIER STRING,
    TAIL_NUM STRING,
    FL_NUM STRING,
    ORIGIN_AIRPORT_ID STRING,
    ORIGIN STRING,
    ORIGIN_STATE_ABR STRING,
    DEST_AIRPORT_ID STRING,
    DEST STRING,
    DEST_STATE_ABR STRING,
    CRS_DEP_TIME INT,
    DEP_TIME FLOAT,
    DEP_DELAY FLOAT,
    DEP_DELAY_NEW FLOAT,
    DEP_DEL15 STRING,
    DEP_DELAY_GROUP STRING,
    TAXI_OUT INT,
    WHEELS_OFF INT,
    WHEELS_ON INT,
    TAXI_IN INT,
    CRS_ARR_TIME INT,
    ARR_TIME FLOAT,
    ARR_DELAY FLOAT,
    ARR_DELAY_NEW FLOAT,
    ARR_DEL15 STRING,
    ARR_DELAY_GROUP STRING,
    CANCELLED STRING,
    CANCELLATION_CODE STRING,
    DIVERTED STRING,
    CRS_ELAPSED_TIME INT,
    ACTUAL_ELAPSED_TIME INT,
    AIR_TIME INT,
    FLIGHTS INT, 
    DISTANCE INT,
    DISTANCE_GROUP STRING,
    CARRIER_DELAY INT,
    WEATHER_DELAY INT,
    NAS_DELAY INT,
    SECURITY_DELAY INT,
    LATE_AIRCRAFT_DELAY INT
)
COMMENT 'airline csv data'
ROW FORMAT DELIMITED
    FIELDS TERMINATED BY ','
    LINES TERMINATED BY '\n'
    STORED AS TEXTFILE
LOCATION '/user/hadoop/input';

-- 테이블 구조 확인
DESCRIBE airline;
SELECT * FROM airline limit 10;
-- 이 테이블의 문제는 무엇입니까?

-- SERDE를 이용한 고급 문법
CREATE EXTERNAL TABLE airline(
    YEAR INT,
    MONTH INT,
    DAY_OF_MONTH INT,
    DAY_OF_WEEK INT,
    FL_DATE DATE,
    UNIQUE_CARRIER STRING,
    TAIL_NUM STRING,
    FL_NUM STRING,
    ORIGIN_AIRPORT_ID STRING,
    ORIGIN STRING,
    ORIGIN_STATE_ABR STRING,
    DEST_AIRPORT_ID STRING,
    DEST STRING,
    DEST_STATE_ABR STRING,
    CRS_DEP_TIME INT,
    DEP_TIME FLOAT,
    DEP_DELAY FLOAT,
    DEP_DELAY_NEW FLOAT,
    DEP_DEL15 STRING,
    DEP_DELAY_GROUP STRING,
    TAXI_OUT INT,
    WHEELS_OFF INT,
    WHEELS_ON INT,
    TAXI_IN INT,
    CRS_ARR_TIME INT,
    ARR_TIME FLOAT,
    ARR_DELAY FLOAT,
    ARR_DELAY_NEW FLOAT,
    ARR_DEL15 STRING,
    ARR_DELAY_GROUP STRING,
    CANCELLED STRING,
    CANCELLATION_CODE STRING,
    DIVERTED STRING,
    CRS_ELAPSED_TIME INT,
    ACTUAL_ELAPSED_TIME INT,
    AIR_TIME INT,
    FLIGHTS INT, 
    DISTANCE INT,
    DISTANCE_GROUP STRING,
    CARRIER_DELAY INT,
    WEATHER_DELAY INT,
    NAS_DELAY INT,
    SECURITY_DELAY INT,
    LATE_AIRCRAFT_DELAY INT
)
COMMENT 'airline csv data'
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
WITH SERDEPROPERTIES(
    "separatorChar" = ",",
    "quoteChar" = "\""
)
LOCATION '/user/hadoop/input'
tblproperties('skip.header.line.count'='1');

-- 이 방식의 장점과 단점은 무엇입니까?

DESCRIBE airline;
SELECT * FROM airline LIMIT 10;
```

#### SELECT

- SELECT 문은 SQL과 거의 유사
- 질의를 실행하면 맵리듀스 잡을 실행
- 최근 버전의 하이브는 LIMIT 조건으로 테이블을 조회하면 파일을 직접 조회하여 출력(즉시 출력)
  
```SQL
-- 기본적인 SELECT
SELECT * FROM airline WHERE year=2010 LIMIT 10;
```

#### 집계 함수

- COUNT(1), COUNT(*) : 전체 데이터의 건수를 반환
- SUM(컬럼) : 컬럼 값의 합계
- AVG(컬럼) : 컬럼 값의 평균
- DISTINCT : 유일한 값 반환
- MIN(컬럼) : 최솟값
- MAX(컬럼) : 최댓값

```SQL
-- 2010년 항공 정보의 건수 조회
SELECT COUNT(1) 
FROM airline
WHERE year=2010
```

- GROUP BY, ORDER BY도 가능

```SQL
SELECT year, month, count(if(ARR_DELAY>0, "", NULL)) 
FROM airline 
GROUP BY year, month
ORDER BY year, month;
```

#### INSERT

```SQL
INSERT OVERWRITE DIRECTORY 'output/hive_dept_delay' 
  SELECT year, month, count(if(dep_delay>0, "", NULL)) 
  FROM airline_201x 
  GROUP BY year, month 
  ORDER BY year, month;
```