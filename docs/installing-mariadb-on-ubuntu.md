# Installing MarinDB on Ubuntu

## MariaDB

- https://mariadb.org/

### Installing MariaDB

- https://mariadb.com/
    - Home > Download
    - Repo Setup > apt/yum tab


```bash
curl -LsS https://dlm.mariadb.com/3/mariadb/mariadb_repo_setup | sudo bash
sudo apt install mariadb-server
sudo apt install mariadb-client
sudo mysql_secure_installation # 계정과 보안 설정 수행 스크립트

mysql -uroot -p # 접속 테스트
```

### DBGEN 설치

- http://tpc.org/tpch/
  - 벤치마크 데이터 셋을 생성하는 DBGEN 프로그램을 제공

```bash
wget https://github.com/electrum/tpch-dbgen/archive/master.zip
unzip master.zip
mv tpch-dbgen-master dbgen

# makefile 수정
CC = gcc
DATABASE = MYSQL
MACHINE = LINUX

make dbgen

# 데이터 생성
dbgen -s 1 # 1G 데이터 셋 생성

# 스키마 스크립트 변경
# MySQL은 테이블 스키마의 대소문자를 구분하기 때문에 개발 편의성을 위해
# 스키마를 소문자로 변경
tr '[:upper:] [:lower:]' < dss.ddl > dss2.ddl
```

### MariaDB 작업

```bash
# hadoop 유저 생성
create user 'hadoop'@'localhost' identified by 'hadoop';
create user 'hadoop'@'%' identifned by 'hadoop';
# 데이터베이스 생성
create database tpch_1g;

grant all privileges on tpch_1g.* to 'hadoop'@'localhost';
grant all privileges on tpch_1g.* to 'hadoop'@'%';

# 테이블 생성
mysql -uroot -p tpch_1g < dss2.ddl 
# 데이터베이스 재 접속(hadoop)
mysql -uhadoop -p
show databases

use tpch_1g;
# 데이터 로드
LOAD DATA LOCAL INFILE '/home/hadoop/dbgen/nation.tbl' INTO TABLE nation FIELDS TERMINATED BY '|';
LOAD DATA LOCAL INFILE '/home/hadoop/dbgen/region.tbl' INTO TABLE region FIELDS TERMINATED BY '|';
LOAD DATA LOCAL INFILE '/home/hadoop/dbgen/part.tbl' INTO TABLE part FIELDS TERMINATED BY '|';
LOAD DATA LOCAL INFILE '/home/hadoop/dbgen/supplier.tbl' INTO TABLE supplier FIELDS TERMINATED BY '|';
LOAD DATA LOCAL INFILE '/home/hadoop/dbgen/partsupp.tbl' INTO TABLE partsupp FIELDS TERMINATED BY '|';
LOAD DATA LOCAL INFILE '/home/hadoop/dbgen/customer.tbl' INTO TABLE customer FIELDS TERMINATED BY '|';
LOAD DATA LOCAL INFILE '/home/hadoop/dbgen/orders.tbl' INTO TABLE orders FIELDS TERMINATED BY '|';
LOAD DATA LOCAL INFILE '/home/hadoop/dbgen/lineitem.tbl' INTO TABLE lineitem FIELDS TERMINATED BY '|';

# 내용의 확인
SELECT * FROM customer LIMIT 10;
```
