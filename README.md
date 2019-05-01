[![CircleCI](https://circleci.com/gh/eloirobe/mdas_hbase.svg?style=svg)](https://circleci.com/gh/eloirobe/mdas_hbase)

# MDAS Bases de datos no estructuradas - Hbase <img src="https://hbase.apache.org/images/hbase_logo_with_orca_large.png" width="100">
Bienvenidos a la imagen de Hbase para a la asignatura de Bases de datos no estructuradas del master Máster en Desarrollo y Arquitectura de Software (MDAS)

Para arrancar Hbase y empezar a utilizar la imagen de docker realizar lo siguiente:

*Requisito previo tener instalado docker*

1) Clonar el repositorio
```bash
git clone https://github.com/eloirobe/mdas_hbase.git
```
2) Arrancar la imagen de docker
```bash
cd mdas_hbase
## En modo stdout
docker-compose -f docker-compose-standalone.yml up
## En modo silencioso
docker-compose -f docker-compose-standalone.yml up -d
```
3) Una vez haya arrancado loguearse en la console
```bash
docker-compose exec hbase bash
```
4) En el directorio files se pueden copiar los datasets para introducir los datos

## Contributors
This is a fork from [https://github.com/big-data-europe/docker-hbase](https://github.com/big-data-europe/docker-hbase)

