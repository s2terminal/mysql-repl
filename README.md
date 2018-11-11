# mysql-repl
MySQL Replication sample for docker-compose.

## Requirements
- docker
- docker-compose

## Usage
```bash
$ docker-compose up
```

### Connecting MySQL
```bash
$ mysql -u root -h 127.0.0.1 -P 13306
$ mysql -u root -h 127.0.0.1 -P 23306
```

### Connecting Container
```bash
$ docker exec -it mysqlrepl_db-master_1 /bin/sh
$ docker exec -it mysqlrepl_db-slave_1 /bin/sh
```

### Start Replication
```bash
$ mysql -u root -h 127.0.0.1 -P 13306 -e 'FLUSH TABLES WITH READ LOCK'
$ mysqldump --all-databases -u root -h 127.0.0.1 -P 13306 --master-data --single-transaction --order-by-primary -r backup.sql
$ MASTER_LOG_FILE=`mysql -u root -h 127.0.0.1 -P 13306 -e 'SHOW MASTER STATUS\G' | grep File | awk '{ print $2 }'`
$ MASTER_LOG_POSITION=`mysql -u root -h 127.0.0.1 -P 13306 -e 'SHOW MASTER STATUS\G' | grep Position | awk '{ print $2 }'`
$ mysql -u root -h 127.0.0.1 -P 13306 -e 'UNLOCK TABLES'
```

```bash
$ mysql -u root -h 127.0.0.1 -P 23306 -e 'STOP SLAVE'
$ mysql -u root -h 127.0.0.1 -P 23306 -e 'SOURCE backup.sql'
$ mysql -u root -h 127.0.0.1 -P 23306 -e "CHANGE MASTER TO MASTER_HOST='db-master', MASTER_PORT=3306, MASTER_USER='root', MASTER_PASSWORD='', MASTER_LOG_FILE='${MASTER_LOG_FILE}', MASTER_LOG_POS=${MASTER_LOG_POSITION};"
$ mysql -u root -h 127.0.0.1 -P 23306 -e 'START SLAVE'
$ mysql -u root -h 127.0.0.1 -P 23306 -e 'SHOW SLAVE STATUS\G'
```

## Sample SQL
```sql
CREATE DATABASE sushi_ya;
USE sushi_ya;

CREATE TABLE sushi(
  id   INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(20),
  INDEX(id)
);

INSERT into sushi_ya.sushi (name) VALUES ('maguro');
INSERT into sushi_ya.sushi (name) VALUES ('tamago');
```

## Sources
- [Replication Between Aurora and MySQL or Between Aurora and Another Aurora DB Cluster \- Amazon Aurora](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Replication.MySQL.html)
