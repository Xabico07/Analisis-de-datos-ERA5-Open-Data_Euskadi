Hasta ahora no se ha tenido en cuenta que todos los datos generados y analizados
tienen varias longitudes y latitudes asociadas. El parque eólico de Azazta solo
existe en una única celda, por lo que hay que averiguar en cual de las 54 posibles
se encuentra. En términos de Python, se quiere determinar qué elementos en
la dimensión de longitud y latitud representan la celda donde se encuentra el
macroparque. El módulo 'nearest.py' facilita dicho proceso. En él se define una
serie de funciones para localizar el punto más cercano en la superficie de una
esfera (aproximadamente el modelo de la Tierra) dado un par de coordenadas
(latitud y longitud).

```python

#nearest.py

import math as M
import numpy as np

def deg2rad(d):
    return d*M.acos(-1.)/180.

def magnitude(A):
    return M.sqrt(np.add.reduce(A*A))

def cdot(A,B):
    return np.add.reduce(A*B)

def Uvector(rlon,rlat):
    U=np.array([M.cos(rlat)*M.cos(rlon),
                M.cos(rlat)*M.sin(rlon),
                M.sin(rlat)],'d')
    return U/magnitude(U)

def cercano(rlon,rlat,rlons,rlats):
    Re=6370.0
    nlats=len(rlats)
    nlons=len(rlons)
    Rt=6370.
    Uref=Uvector(rlon,rlat)
    theCos=-2.0
    for ilon in range(nlons):
        rlonp=rlons[ilon]
        for ilat in range(nlats):
            rlatp=rlats[ilat]
            Upoint=Uvector(rlonp,rlatp)
            theC=cdot(Uref,Upoint)
            if theC>theCos:
                ILON=ilon
                ILAT=ilat
                theCos=theC
                if theCos>1.0:
                    theCos=1.0
    print('COORDENADA DE LA MALLA MÁS CERCANA A AZÁCETA')
    print(lons[ILON],lats[ILAT])
    return (ILON,ILAT,Re*M.acos(theCos))

```

En el módulo se definen primero varias funciones auxiliares, las cuales
permiten convertir grados en radianes, calcular la magnitud de un vector, el
producto escalar entre dos vectores y convertir coordenadas eféricas en vectores
unitarios (dependientes de la longitud y latitud, ya que el radio de la Tierra es
fijo). La función principal, haciendo uso de las anteriores, encuentra la posición
más cercana a un punto de referencia (longitud y latitud del parque de Azazeta)
en una cuadrícula definida en este caso por los 8 elementos longitudinales y los 5
latitudinales. Para ello, definido el radio de la Tierra, se itera sobre cada punto
de la cuadrícula y se convierte cada coordenada en un vector unitario. Se realiza
el producto vectorial entre cada uno de esos vectores y el vector unitario del
punto de referencia, para posteriormente determinar el coseno del ángulo entre
ambos. Durante la iteración, cada vez que el valor del nuevo coseno es mayor
que el previo, dicho valor máximo se actualiza y se almacenan las posiciones de
longitud y latitud de la cuadrícula asociada al caso iterado. El resultado del
módulo devuelve las coordenadas resultantes al final del proceso.

Para el caso estudiado, se deben definir los argumentos de la función principal:

```python

# Las coordenadas del parque son (Alto de Azazeta): 42.78499783697841, -2.542525562758741


# Las coordenadas de longitud y latitud del archivo NetCDF 'Un_Datos_2013-2022.nc' generado 

lons=np.array(d.variables["longitude"][:],'d')
lats=np.array(d.variables["latitude"][:],'d')


# En radianes

lat=deg2rad(42.7849978369784)
lon=deg2rad(-2.542525562758741)
lonss=deg2rad(lons)
latss=deg2rad(lats)

print('Coordenadas mallas en grados')
print(lons)
print(lats)


coordenada=cercano(lon,lat,lonss,latss)

print(coordenada)

```


```bash

Coordenadas mallas en grados
[-3.5  -3.25 -3.   -2.75 -2.5  -2.25 -2.   -1.75]
[43.5  43.25 43.   42.75 42.5 ]

COORDENADA DE LA MALLA MÁS CERCANA A AZÁCETA
-2.5 42.75
(4, 3, 5.2140376683177285)

```
Según los resultados arrojados, la celda con la que se debe trabajar es la definida con las coordenadas (42.75,-2.5), lo cual tiene sentido dadas las coordenadas reales del parque. 
Por lo tanto, los datos de velocidad normalizada que deben ser aplicados sobre la curva de potencia vienen definidos por el quinto elemento de la dimensión de longitud y el cuarto de la dimensión de latitud.

