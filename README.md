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

## Lanzamiento de Impala
Ejecutar el script launch_impala.sh

El contenedor expone estos puertos

* 21000:21000: puerto para recibir consultas (protocolo propio)
* 21050:21050: puerto para recibir consultas (protocolo hive2) -- es el que se usa para conectar a Superset
* 25000:25000: webUI impalad
* 25010:25010: webUI statestored
* 25020:25020: webUI catalogd

## Conexión a Superset
Superset debe tener instalado el driver "impyla". Usamos una versión tuneada e inicializada:

docker run --rm --network=kudu-impala_default -p 8080:8088 --name superset acpmialj/ipmd:ssuperset

El SQLALCHEMY URI debe tener la forma impala://{hostname}:{port}/{database}. Para este caso particular:

impala://kudu-impala:21050/default

