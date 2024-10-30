# kudu-impala
Nos aseguramos de estar en el directorio ~/kudu-impala

## Lanzamiento del clúster Kudu
Usar el fichero compose quickstart.yml. Es necesaria una inicialización de variable:
```
export KUDU_QUICKSTART_IP=$(ifconfig | grep "inet " | grep -Fv 127.0.0.1 |  awk '{print $2}' | tail -1)
docker compose -f quickstart.yml up -d
```
Para eliminar el clúster es necesario que la variable anterior siga teniendo validez. Si no, se ejecuta el "export" de nuevo. Tras ello,
```
docker compose -f quickstart.yml down
```
Quedan creados una serie de volúmenes que habría que eliminar para una limpieza total. 

## Lanzamiento de Impala
Ejecutar el script launch_impala.sh
```
sh launch_impala.sh
```

El contenedor expone estos puertos

* 21000:21000: puerto para recibir consultas (protocolo propio)
* 21050:21050: puerto para recibir consultas (protocolo hive2) -- es el que se usa para conectar a Superset
* 25000:25000: webUI impalad
* 25010:25010: webUI statestored
* 25020:25020: webUI catalogd

Tras esto nos conectamos al CLI de Impala:
```
docker exec -it kudu-impala impala-shell
```
Creamos una base de datos almacenada en Kudu e insertamos algunos datos:
```
CREATE TABLE my_first_table
(
  id BIGINT,
  name STRING,
  PRIMARY KEY(id)
)
PARTITION BY HASH PARTITIONS 4
STORED AS KUDU;

DESCRIBE my_first_table;

INSERT INTO my_first_table VALUES (99, "sarah");
INSERT INTO my_first_table VALUES (1, "john"), (2, "jane"), (3, "jim");
SELECT * FROM my_first_table;

UPDATE my_first_table SET name="bob" where id = 3;
SELECT * FROM my_first_table;

DELETE FROM my_first_table WHERE id = 99;
SELECT * FROM my_first_table;

DELETE FROM my_first_table WHERE id < 3;
SELECT * FROM my_first_table;

CREATE EXTERNAL TABLE my_second_table
STORED AS KUDU
TBLPROPERTIES('kudu.table_name' = 'impala::default.my_first_table');

DESCRIBE my_second_table;
DESCRIBE EXTENDED my_second_table;

DROP TABLE my_second_table;
DROP TABLE my_first_table;

exit;
```
Podemos eliminar el contenedor con "docker stop" y, si es necesario, con "docker rm".


## Conexión a Superset
Superset debe tener instalado el driver "impyla". Usamos una versión tuneada e inicializada (hecha con el Dockerfile de este repositorio).

docker run --rm --network=kudu-impala_default -p 8080:8088 --name superset acpmialj/ipmd:ssuperset

El SQLALCHEMY URI debe tener la forma impala://{hostname}:{port}/{database}. Para este caso particular:

impala://kudu-impala:21050/default

