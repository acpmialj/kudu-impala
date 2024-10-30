# kudu-impala
## Lanzamiento del clúster Kudu
Usar el fichero compose quickstart.yml

## Lanzamiento de Impala
Ejecutar el script launch_impala.sh

El contenedor expone estos puertos

* 21000:21000: puerto para recibir consultas (protocolo propio)
* 21050:21050: puerto para recibir consultas (protocolo hive2) -- es el que se usa para conectar a Superset
* 25000:25000: webUI impalad
* 25010:25010: webUI statestored
* 25020:25020: webUI catalogd

## Conexión a Superset
Superset debe tener instalado el driver "impyla".

El URI debe tener la forma impala://{hostname}:{port}/{database}. Para este caso particular:

impala://kudu-impala:21050/default

docker run -d --rm --network=kudu-impala_default -p 8080:8088 --name superset
