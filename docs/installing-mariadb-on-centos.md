## MariaDB

- https://mariadb.org/

### Installing MariaDB

- https://mariadb.com/
    - Home > Download
    - apt/yum tab

```bash
curl -LsS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup | sudo bash
yum install mariadb-server
systemctl start mariadb.service
# 비밀번호 변경
mysqladmin -uroot password <비밀번호 지정>
```

### DBGEN 설치

- http://tpc.org/tpch/

```bash
wget https://github.com/electrum/tpch-dbgen/archive/master.zip
unzip master.zip
mv tpch-dbgen-master dbgen

# makefile 수정
CC = gcc
DATABASE = MYSQL
MACHINE = LINUX

make

# 데이터 생성
dbgen -s 1 # 1G 데이터 셋 생성

# 스키마 스크립트 변경
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

#### 방화벽 추가

```bash
firewall-cmd --permanent --zone=public --add-port=3306/tcp
```