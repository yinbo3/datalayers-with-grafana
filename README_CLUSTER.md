# DataLayers with Grafana(cluster)
Visualize the data stored in DataLayers using Grafana.

## How to use

1. Clone the code repository. Please make sure you have installed the `git`, and then running the following commands to start the demo:

  ```bash
  git clone https://github.com/datalayers-io/datalayers-with-grafana.git
  ```
  
2. Please run the following script first.

```bash
cd datalayers-with-grafana
```

``` bash
./init.sh  
```

3. Please make sure you have installed the [docker](https://www.docker.com/), and then running the following commands to start the demo:

``` bash
docker pull datalayers/datalayers:nightly
```

``` bash
docker compose -f cluster.yaml up -d
```

4. Running the following commands to see the message from DataLayers: (If you don't care about logs, skip it.)

``` bash
docker compose -f cluster.yaml logs datalayers
```

5. Connect to DataLayers using the command-line tool:

```bash
docker compose -f cluster.yaml exec -it datalayers dlsql -u admin -p public
```

6. Create a database using the command-line tool:

``` bash
docker compose -f cluster.yaml exec -it datalayers dlsql -u admin -p public
```

```bash
create database demo;
```

![docker-compose create](./static/images/create_database_cluster.gif)

7. Create tables:

``` bash
use demo;
```

``` bash
CREATE TABLE test(
  ts TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP,
  sn int NOT NULL,
  speed float NOT NULL,
  temperature float,
  timestamp KEY (ts))
PARTITION BY HASH(sn) PARTITIONS 2
ENGINE=TimeSeries;
```
You can use `exit` command to exit `command-line tool` session.

6. Use the following script to write data:

``` bash
while true
do
  speed=$((RANDOM % 21 + 100))
  temperature=$((RANDOM % 11 + 10))
  timestamp=$(date +%s%9N) # ns
  code="insert into demo.test(sn,speed,temperature) values(10000, ${temperature}, ${speed})"
  echo "$code"
  curl -u"admin:public" -X POST http://127.0.0.1:18361/api/v1/sql?db=demo -H 'Content-Type: application/binary' -d "$code" -s -o /dev/null
  sleep 1
done
```

7. Query data through the command line:

``` bash
docker compose -f cluster.yaml exec -it datalayers dlsql -u admin -p public
```

``` bash
select * from demo.test limit 10
```

8. Visualize data using Grafana:

Visit: [http://localhost:13000/](http://localhost:13000/)

> Username: admin <br> Password: admin


Try to add dashboard by `Menu - Dashboards` page.

![add dashboard](./static/images/dashboard.jpg)

For example:

``` sql
SELECT date_bin('5 seconds', ts) as timepoint, avg(speed) FROM demo.test group by timepoint;

```
As always, you can use [SQL functions](https://docs.datalayers.cn/datalayers/latest/sql-reference/sql-functions.html) in the sentence.



## End the experience

You can use below command to exit processes.

``` bash
docker compose -f cluster.yaml down
```


## License

[Apache License 2.0](./LICENSE)

## Standalone-mode

Click to [Standalone-mode](./README.md) documentation.

> If you want to quick start with standalone-mode. Please use `docker compose -f cluster.yaml down` to stop the services first.