Los datos atmosféricos necesarios para el análisis del factor de capacidad de los microparques se encuentran en archivos csv. La biblioteca Pandas (importada normalmente como pd), 
permite leer y manipular este tipo de archivos cómodamente.

```python

import csv
import pandas as pd

# Ruta al archivo CSV
ruta_archivo2013 = r'\Datos microparque farmacia\FARMACIA_2013.csv'
ruta_archivo2014 = r'\Datos microparque farmacia\FARMACIA_2014.csv'
ruta_archivo2015 = r'\Datos microparque farmacia\FARMACIA_2015.csv'
ruta_archivo2016 = r'\Datos microparque farmacia\FARMACIA_2016.csv'
ruta_archivo2017 = r'\Datos microparque farmacia\FARMACIA_2017.csv'
ruta_archivo2018 = r'\Datos microparque farmacia\FARMACIA_2018.csv'
ruta_archivo2019 = r'\Datos microparque farmacia\FARMACIA_2019.csv'
ruta_archivo2020 = r'\Datos microparque farmacia\FARMACIA_2020.csv'
ruta_archivo2021 = r'\Datos microparque farmacia\FARMACIA_2021.csv'
ruta_archivo2022 = r'\Datos microparque farmacia\FARMACIA_2022.csv'

# Lee el archivo CSV con pandas
df2013 = pd.read_csv(ruta_archivo2013, delimiter=';')
df2014 = pd.read_csv(ruta_archivo2014, delimiter=';')
df2015 = pd.read_csv(ruta_archivo2015, delimiter=';')
df2016 = pd.read_csv(ruta_archivo2016, delimiter=';')
df2017 = pd.read_csv(ruta_archivo2017, delimiter=';')
df2018 = pd.read_csv(ruta_archivo2018, delimiter=';')
df2019 = pd.read_csv(ruta_archivo2019, delimiter=';')
df2020 = pd.read_csv(ruta_archivo2020, delimiter=';')
df2021 = pd.read_csv(ruta_archivo2021, delimiter=';')
df2022 = pd.read_csv(ruta_archivo2022, delimiter=';')
```
