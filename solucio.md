# Base de dades no estructurades

```hbase
create_namespace 'bdne'
create 'bdne:accidentes','lugar','fecha_evento','detalles'
```
```bash
hbase org.apache.hadoop.hbase.mapreduce.ImportTsv \
  -Dimporttsv.separator=';' \
  -Dimporttsv.columns="HBASE_ROW_KEY,lugar:distrito,lugar:barrio,fecha_evento:nombre_dia,fecha_evento:año,fecha_evento:mes,fecha_evento:nombre_mes,fecha_evento:dia,fecha_evento:hora,detalles:turno,detalles:tipo_accidente" \
  'bdne:accidentes' /files/accidentes_bcn_2018.csv
```
```bash
echo lugar:{distrito,barrio} fecha_evento:{nombre_dia,año,mes,nombre_mes,dia,hora} detalles:{turno,tipo_accidente}
```

# 2)
```hbase
hbase(main):002:0> count 'bdne:accidentes'
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

# 3)
```hbase
scan 'bdne:accidentes',
```

# 4)
```hbase
Celda por celda
delete 'table','row'
deleteall 'tabla','row'
```

# 9)
```hbase
put 'table','row','columnFamily:column','value'
```


# Miselanius
```hbase
get 'table','row'
scan 'table'

crud 11, abansat
scan 'bdne:accidentes'

neteja
truncate 'table'

scan 'users',
{
 COLUMNS => ['contact_infromation:name', 'contact_information:city']
 LIMIT -> 1,
 STARTROW -> "2",
 FILTER =>
  RowFilter.new(
   CompareFilter::CompareOp.valueOf('EQUAL'),
   BinaryComparator.new(
    Bytes.toBytes('2')
   )
  )
}
```
