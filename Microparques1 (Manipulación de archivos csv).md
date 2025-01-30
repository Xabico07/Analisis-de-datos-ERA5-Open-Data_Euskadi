Los parámetros atmosféricos necesarios para el análisis del factor de capacidad de los microparques se encuentran en archivos csv. La biblioteca Pandas (importada normalmente como pd), 
permite leer y manipular este tipo de archivos cómodamente. El procedimiento realizado para el microparque de la Facultad de Farmacia de Vitoria y la Escuela de Ingeniería de Portugalete es idéntico.

```python

import csv
import pandas as pd

# Se asocia una variable a la ruta al archivo CSV
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
Los archivos contienen información que no resulta de interés en este caso. Para determinar qué columnas de datos se requieren, se puede utilizar el siguiente procedimiento:

```python

print("Nombres de las columnas:", df2013.columns)

```

```bash

Nombres de las columnas: Index(['Date', 'Hour', 'D.vien (grados)', 'H (%)', 'P (mBar)', 'Tº (ºC)',
       'V.vien (m/s)'],
      dtype='object')

```

Para el caso de los microparques, los parámetros con los que se quiere trabajar son dos: la dirección del viento 'D.vien (grados)' y su velocidad 'V.vien (m/s)'. Se quiere trabajar solo con dichas columnas.

```python

# Se especifica el nombre de la columna que se desea leer

D = 'D.vien (grados)' 
V = 'V.vien (m/s)'

# Se crean las listas de datos

# Para las direcciones

d2013 = df2013[D].tolist()
d2014 = df2014[D].tolist()
d2015 = df2015[D].tolist()
d2016 = df2016[D].tolist()
d2017 = df2017[D].tolist()
d2018 = df2018[D].tolist()
d2019 = df2019[D].tolist()
d2020 = df2020[D].tolist()
d2021 = df2021[D].tolist()
d2022 = df2022[D].tolist()

# Para las velocidades

v2013 = df2013[V].tolist()
v2014 = df2014[V].tolist()
v2015 = df2015[V].tolist()
v2016 = df2016[V].tolist()
v2017 = df2017[V].tolist()
v2018 = df2018[V].tolist()
v2019 = df2019[V].tolist()
v2020 = df2020[V].tolist()
v2021 = df2021[V].tolist()
v2022 = df2022[V].tolist()

# Se unen en una única lista 

# Para las direcciones
Total_d = d2013+d2014+d2015+d2016+d2017+d2018+d2019+d2020+d2021+d2022

# Para las velocidades
Total_v = v2013+v2014+v2015+v2016+v2017+v2018+v2019+v2020+v2021+v2022

```
Los elementos de la lista generada para las velocidades y direcciones son de clase 'str' y 'float', respectivamente (esto se puede observar imprimiendo cualquier objeto df20XX). Para un posible análisis numérico, ambos deben ser de clase 'float'. Para ello:

```python

Total_v2 = [float(elemento.replace(',', '.')) if isinstance(elemento, str) and elemento is not None else None for elemento in Total_v]

```
Se obtiene una nueva lista Total_v2, donde los elementos que eran cadenas con números decimales representados con comas se convierten en objetos de tipo 'float', mientras que los otros elementos quedan igual. 
