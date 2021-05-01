# Base de dades no estructurades


# 1) Importa los datos del fichero csv en una tabla llamada **accidentes** y que tenga la siguiente estructura:

desde hbase:
```hbase
create_namespace 'bdne'
create 'bdne:accidentes','lugar','fecha_evento','detalles'
```

desde bash:
```bash
hbase org.apache.hadoop.hbase.mapreduce.ImportTsv \
  -Dimporttsv.separator=';' \
  -Dimporttsv.columns="HBASE_ROW_KEY,lugar:distrito,lugar:barrio,fecha_evento:nombre_dia,fecha_evento:año,fecha_evento:mes,fecha_evento:nombre_mes,fecha_evento:dia,fecha_evento:hora,detalles:turno,detalles:tipo_accidente" \
  'bdne:accidentes' /files/accidentes_bcn_2018.csv
```
<!--
```bash
echo lugar:{distrito,barrio} fecha_evento:{nombre_dia,año,mes,nombre_mes,dia,hora} detalles:{turno,tipo_accidente}
```
-->

# 2) Verifica que hayas importado 9927 registros.
```hbase
count 'bdne:accidentes'
```

output:
```output
Current count: 1000, row: 2018S001009
Current count: 2000, row: 2018S002016
Current count: 3000, row: 2018S003020
Current count: 4000, row: 2018S004030
Current count: 5000, row: 2018S005035
Current count: 6000, row: 2018S006039
Current count: 7000, row: 2018S007040
Current count: 8000, row: 2018S008043
Current count: 9000, row: 2018S009046
9927 row(s) in 1.1650 seconds

=> 9927
```
Verificado, se ha importado un total de 9927 registros.

# 3) Muestra todos los registros cuya Row Key no empiece por 2018
```hbase
scan 'bdne:accidentes', { FILTER => "RowFilter(!=, 'regexstring:^2018')" }
```

output:
```output
ROW                                                  COLUMN+CELL
 C\xC3\xB3digo Expediente                            column=detalles:tipo_accidente, timestamp=1619639152103, value=Tipo de Accidente
 C\xC3\xB3digo Expediente                            column=detalles:turno, timestamp=1619639152103, value=Turno
 C\xC3\xB3digo Expediente                            column=fecha_evento:a\xC3\xB1o, timestamp=1619639152103, value=A\xC3\xB1o
 C\xC3\xB3digo Expediente                            column=fecha_evento:dia, timestamp=1619639152103, value=D\xC3\xADa del Mes
 C\xC3\xB3digo Expediente                            column=fecha_evento:hora, timestamp=1619639152103, value=Hora del D\xC3\xADa
 C\xC3\xB3digo Expediente                            column=fecha_evento:mes, timestamp=1619639152103, value=Mes
 C\xC3\xB3digo Expediente                            column=fecha_evento:nombre_dia, timestamp=1619639152103, value=D\xC3\xADa de la Semana
 C\xC3\xB3digo Expediente                            column=fecha_evento:nombre_mes, timestamp=1619639152103, value=Nombre Mes
 C\xC3\xB3digo Expediente                            column=lugar:barrio, timestamp=1619639152103, value=Barrio
 C\xC3\xB3digo Expediente                            column=lugar:distrito, timestamp=1619639152103, value=Distrito
1 row(s) in 0.0640 seconds
```
Solo hay una row que no empieze por 2018

# 4) Al importar, hemos importado como un registro más la cabecera del csv, elimina ese registro.
```hbase
deleteall 'bdne:accidentes','Código Expediente'
```

Se hace estraño, porque el output no acaba de parecer coherente:
```output
0 row(s) in 0.0300 seconds
```

Entonces las comprobaciones de que realmente ha funcionado:
```check
hbase(main):004:0> scan 'bdne:accidentes', { FILTER => "RowFilter(!=, 'regexstring:^2018')" }
ROW                                                  COLUMN+CELL
0 row(s) in 0.0970 seconds

count 'bdne:accidentes'
=> 9926
```

Si, se ha eliminado correctamente la fila de más.

# 5) Muestra los 5 primeros registros de la tabla. Usando LIMIT
```hbase
scan 'bdne:accidentes', { LIMIT => 5 }
```

# 6) Muestra los 5 primeros registros de la tabla. Sin usar LIMIT, busca un Filter que te permita hacerlo.
## Investigació
```hbase
show_filters
```

output
```output
DependentColumnFilter
KeyOnlyFilter
ColumnCountGetFilter
SingleColumnValueFilter
PrefixFilter
SingleColumnValueExcludeFilter
FirstKeyOnlyFilter
ColumnRangeFilter
TimestampsFilter
FamilyFilter
QualifierFilter
ColumnPrefixFilter
RowFilter
MultipleColumnPrefixFilter
InclusiveStopFilter
PageFilter
ValueFilter
ColumnPaginationFilter
```

## Solució
```hbase
scan 'bdne:accidentes', { FILTER => "PageFilter(5)" }
```

# 7) Muestra todas las celdas que tengan algún campo como valor "Desconegut"
```hbase
scan 'bdne:accidentes', { FILTER => "ValueFilter(=, 'binary:Desconegut')" }
```

# 8) Muestra todos los accidentes de "Atropellament" que hayan sido en Lunes por la Noche
## Atropellament
```hbase
scan 'bdne:accidentes', { FILTER => "SingleColumnValueFilter('detalles','tipo_accidente',= ,'binary:Atropellament')" }
```

## Lunes
```hbase
scan 'bdne:accidentes', { FILTER => "SingleColumnValueFilter('fecha_evento','nombre_dia',= ,'binary:Lunes')" }
```

## Noche
```hbase
scan 'bdne:accidentes', { FILTER => "SingleColumnValueFilter('detalles','turno',= ,'binary:Noche')" }
```

## Solució
```hbase
scan 'bdne:accidentes', { FILTER =>
    "SingleColumnValueFilter('detalles','tipo_accidente',=,'binary:Atropellament') \
 AND SingleColumnValueFilter('fecha_evento','nombre_dia',= ,'binary:Lunes') \
 AND SingleColumnValueFilter('detalles','turno',=,'binary:Noche')"
}
```

# 9) Actualiza la fecha del accidente con código "2018S008673", la fecha correcta es Martes 13/11/2018
Comprovamos que valores tiene correctos
```hbase
get 'bdne:accidentes','2018S008673'
hbase(main):017:0> get 'bdne:accidentes','2018S008673'
COLUMN                                               CELL
 detalles:tipo_accidente                             timestamp=1619851241949, value=Abast
 detalles:turno                                      timestamp=1619851241949, value=Tarde
 fecha_evento:a\xC3\xB1o                             timestamp=1619851241949, value=2018
 fecha_evento:dia                                    timestamp=1619851241949, value=12
 fecha_evento:hora                                   timestamp=1619851241949, value=18
 fecha_evento:mes                                    timestamp=1619851241949, value=11
 fecha_evento:nombre_dia                             timestamp=1619851241949, value=Lunes
 fecha_evento:nombre_mes                             timestamp=1619851241949, value=Noviembre
 lugar:barrio                                        timestamp=1619851241949, value=les Corts
 lugar:distrito                                      timestamp=1619851241949, value=Les Corts
10 row(s) in 0.0200 seconds
```

Los valores que tenemos que mantener es el `fecha_evento:año` y `fecha_evento:mes` como `fecha_evento:nombre_mes`.
Tanto el `fecha_evento:nombre_dia` como `fecha_evento:dia` se tienen que actualizar.
Al parecer solo hay un error de 1 día.

## Solució
```hbase
put 'bdne:accidentes','2018S008673','fecha_evento:nombre_dia','Martes'
put 'bdne:accidentes','2018S008673','fecha_evento:dia',13
```

## Comprovació
```hbase
get 'bdne:accidentes','2018S008673'
COLUMN                                               CELL
 detalles:tipo_accidente                             timestamp=1619851241949, value=Abast
 detalles:turno                                      timestamp=1619851241949, value=Tarde
 fecha_evento:a\xC3\xB1o                             timestamp=1619851241949, value=2018
 fecha_evento:dia                                    timestamp=1619852229251, value=13
 fecha_evento:hora                                   timestamp=1619851241949, value=18
 fecha_evento:mes                                    timestamp=1619851241949, value=11
 fecha_evento:nombre_dia                             timestamp=1619852219440, value=Martes
 fecha_evento:nombre_mes                             timestamp=1619851241949, value=Noviembre
 lugar:barrio                                        timestamp=1619851241949, value=les Corts
 lugar:distrito                                      timestamp=1619851241949, value=Les Corts
10 row(s) in 0.0100 seconds
```
