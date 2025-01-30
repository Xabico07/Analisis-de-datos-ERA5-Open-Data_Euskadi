Una vez descargados los ficheros NetCDF4 con datos horarios de presión, temperatura y ambas componentes de velocidad del viento para un periodo de diez años, 
en Python se le puede asignar una variable a cada uno para trabajar cómodamente con ellos durante el resto del programa.
Cabe destacar que los archivos tardan mucho tiempo en descargarse desde ERA5 debido a la cantidad de datos que se quieren formatear.
Por ello, lo más asequible, si no se quiere dejar la computadora trabajando excesivamente seguido, es realizar cinco descargas bianuales por separado y agruparlos después.
Las variables se pueden definir utilizando la librería 'netCDF4' (importada habitualmente como 'nc'), que consta del módulo 'Dataset' encargado de abrir el archivo tipo NetCDF deseado. 
El comando 'nc.Dataset()' abre dicho fichero, cuya ruta completa es la especificada dentro del argumento principal.

```python
import netCDF4 as nc
import xarray as xr

x=nc.Dataset('/Datos de viento ERA5/Datos_2013-2014.nc')
y=nc.Dataset('/Datos de viento ERA5/Datos_2015-2016.nc')
z=nc.Dataset('/Datos de viento ERA5/Datos_2017-2018.nc')
k=nc.Dataset('/Datos de viento ERA5/Datos_2019-2020.nc')
j=nc.Dataset('/Datos de viento ERA5/Datos_2021-2022.nc')

```
El objeto 'Dataset' permite explorar las dimensiones y variables de dicho archivo, además
de poder acceder a todos los datos almacenados en él para poder extraerlos
y analizarlos. Por ejemplo:

```python

print(x)

```

```bash
<class 'netCDF4._netCDF4.Dataset'>
root group (NETCDF3_64BIT_OFFSET data model, file format NETCDF3):
    Conventions: CF-1.6
    history: 2023-12-19 20:26:02 GMT by grib_to_netcdf-2.25.1: /opt/ecmwf/mars-client/bin/grib_to_netcdf.bin -S param -o /cache/data9/adaptor.mars.internal-1703016880.5709424-25829-12-5b1210de-b465-42db-904f-bf649f09ab68.nc /cache/tmp/5b1210de-b465-42db-904f-bf649f09ab68-adaptor.mars.internal-1703013763.4203577-25829-17-tmp.grib
    dimensions(sizes): longitude(8), latitude(5), time(17520)
    variables(dimensions): float32 longitude(longitude), float32 latitude(latitude), int32 time(time), int16 u100(time, latitude, longitude), int16 v100(time, latitude, longitude), int16 t2m(time, latitude, longitude), int16 sp(time, latitude, longitude)
    groups: 
```
Para trabajar con un solo archivo, conviene agrupar los cinco ficheros descargados:

```python
# Se agrupan las cinco descargas
archivos = ['/Datos de viento ERA5/Datos_2013-2014.nc', 'Datos de viento ERA5/Datos_2015-2016.nc', '/Datos de viento ERA5/Datos_2017-2018.nc',
 '/Datos de viento ERA5/Datos_2019-2020.nc', '/Datos de viento ERA5/Datos_2021-2022.nc']

# Se nombra el nuevo archivo
archivo_combinado = 'Datos_2013-2022x.nc'

# Se abre cada archivo descargado con xarray
datasets = [xr.open_dataset(archivo) for archivo in archivos]

# Se combinan los conjuntos de datos usando la función concat
dataset_combinado = xr.concat(datasets, dim='time')  

# Se guarda el conjunto de datos combinado en un nuevo archivo NetCDF4
dataset_combinado.to_netcdf(archivo_combinado)

# Se le asigna una variable Dataset
x2=nc.Dataset('/Datos de viento ERA5/Datos_2013-2022x.nc')

```

Puede llamarse a cada una de las variables para mostrar sus datos de forma
completa o para unas dimensiones específicas. Lo más cómodo si se quiere
trabajar con ellos numéricamente es almacenar dichos datos en un arreglo numérico. La
librería numpy (importada habitualmente como np), entre otras cosas, permite
generar dicha lista mediante el módulo 'array'.

```python

import numpy as np

Tpm=np.array(x2.variables["T2m"][:,4,3],'d')

```

El código descrito extrae los datos numéricos de la temperatura (t2m) para todas la horas (por ello el ”:”),
el tercer elemento en la dimensión de longitud y el quinto en la dimensión de
latitud, y ordena cada uno de los valores en una lista a la que se le ha llamado
Tpm. Esto ofrece una gran flexibilidad a la hora de escoger los datos necesarios
para los posteriores cálculos. El orden en el que se especifican las dimensiones
(tiempo, longitud, latitud) se basa en las convenciones COARDS. La ’d’, por
último, indica que el tipo de dato del arreglo creado es de doble precisión,
float64. Las listas generadas son realmente útiles ya que permiten realizar arreglos
matemáticos para una gran cantidad de datos a la vez.

El problema que surge con este método es
que se pierde automáticamente toda la información sobre la dimensionalidad
de la nueva variable. Por ello, si se quieren determinar los valores para otros
elementos de tiempo, longitud o latitud se debe repetir de nuevo todo el proceso,
teniendo que llamar otra vez a las variables necesarias del archivo original, lo
cual resulta muy poco eficiente. La solución a este inconveniente es generar una
nueva variable e introducirla en un archivo NetCDF nuevo de acuerdo al resto
de variables y dimensiones. Lo más razonable es programar un módulo que haga
automáticamente ese trabajo. El módulo de Python ncstruct.py proporciona funciones para crear y copiar la estructura 
de archivos NetCDF que cumplen con las convenciones COARDS. Es,
por tanto, una excelente opción para introducir nuevas variables.

```python

# ncstruct.py

"""Copy the structure of a COARDS compliant netCDF file

"""
#
# 
# Copyright (C) 2000, Jon Saenz, Jesus Fernandez and Juan Zubillaga
# 
# This program is free software; you can redistribute it and/or
# modify it under the terms of the GNU General Public License
# as published by the Free Software Foundation, version 2.
# 
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
# 
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 59 Temple Place - Suite 330, Boston, MA  02111-1307, USA.
# 
# Added function create_bare_COARDS(), JS, June 2001



from netCDF4 import Dataset
import numpy
import string,sys
import time

# Return the global attributes of the netCDF dataset
def _global_attributes(inc):
  return inc.ncattrs()

# Return the keys of the variable attributes
def _variable_attributes(inc,varname):
  return inc.variables[varname].ncattrs()


# Copy the contents of a variable in the input file to the output variable
# MUST BE ALIGNED !!!, so that the copying of a record variable
# may be problematic. Not sure yet about if this works for multidimensional
# variables
def _copycontents(ovar,invar):
  # At least a dimension must have already been defined,
  # use it as a record variable and so on
  maxrec=invar.shape[0]
  for irec in range(maxrec):
    ovar[irec]=invar[irec]
  
def nccopystruct(name,inc,dims=None,vars=None,varcontents=None,myHist=None):
  """Duplicate the definition of the input netCDF file in the output file

  Arguments:

    'name' -- Name of the output file                                           

    'inc' -- Input netCDF object

  Optional arguments:

    'dims' -- Tuple (list) with the name of the dimensions to be copied.
              Defaults to 'None'.

    'vars' -- Tuple (list) with the name of the variables to be copied.
              Defaults to 'None'.

    'varcontents' -- Tuple (list) with the name of the variables whose          
                     contents are to be copied. They must have been previously  
                     defined in 'vars' or otherwise. Defaults to 'None'.
  """
  onc=Dataset(name,"w",format='NETCDF3_CLASSIC')
  # Copy the datasets global attributes and append a line to the
  # history attribute
  gAtts=inc.ncattrs()
  # print(gAtts)
  for key in gAtts:
    if key=="history":
      args=sys.argv
      if myHist is None:
          endwith="\n"+args[0]
          for arg in args[1:]:
            endwith=endwith+" "+arg
          endwith=endwith+" "+time.ctime(time.time())
      else:
          endwith=myHist
      setattr(onc,key,getattr(inc,key)+endwith)
    else:
      setattr(onc,key,getattr(inc,key))
  # Copy each of the dimensions in the input array
  if dims:
    for dim in dims:
      # print dim, inc.dimensions[dim],len(inc.dimensions[dim])
      if dim=="time" or inc.dimensions[dim].isunlimited():
        onc.createDimension(dim,None)
      else:
        onc.createDimension(dim,len(inc.dimensions[dim]))
  if vars:
    for var in vars:
      invar=inc.variables[var]
      ovar=onc.createVariable(var,invar.datatype,invar.dimensions)
      for key in _variable_attributes(inc,var):
        setattr(ovar,key,getattr(invar,key))
  if varcontents:
    for var in varcontents:
      invar=inc.variables[var]
      ovar=onc.variables[var]
      _copycontents(ovar,invar)
  return onc

def create_bare_COARDS(ncname,tdat=None,zdat=None,latdat=None,londat=None):
  """Create a bare COARDS compliant netCDF file with a minimum typing

  The created file may hold 4D variables as in the COARDS standard:
  'var[time,Z,lat,lon]', but the variables may hold less dimensions
  (like in a zonal average, for instance).

  Arguments:
  
    'ncname' -- is the name of the netCDF file to be created

  Optional arguments:

    'tdat' -- Tuple with first element the NumPy array of items to be held
              ('None' for a record variable) and with second element the 
              units attribute of the created time variable. If tdat itself is
              'None', no time variable will be created (This is the default).

    'zdat' -- Tuple with two elements. The first one is the NumPy array of 
              vertical levels. The last one is the units attribute of the 
              vertical levels. If 'zdat' is None, no vertical dimension is 
              created (Default).

    'latdat' -- NumPy array of latitudes. The units attribute is set
                to degrees_north. If 'None' (default), no latitudinal 
                dimension is created.

    'londat' -- NumPy array of longitudes. The units attribute 
                is set to degrees_east. If 'None' (default), no zonal dimension 
                is created.
  """
  onc=Dataset(ncname,"w")
  onc.Conventions="COARDS"
  onc.history="pyclimate.create_bare_COARDS"
  Float64='d'
  if (not tdat) and (not zdat) and (not latdat) and (not londat):
    return None
  if tdat:
    if not tdat[0]:
      onc.createDimension("time",tdat[0])
    else:
      onc.createDimension("time",len(tdat[0]))
    tvar=onc.createVariable("time",Float64,("time",))
    tvar.long_name="Time variable"
    tvar.units=tdat[1]
    if tdat[0]:
      tvar[:]=tdat[0].astype(Float64)
  if zdat:
    # Only time can be of "record type"
    onc.createDimension("level",len(zdat[0]))
    zvar=onc.createVariable("level",zdat[0].typecode(),("level",))
    zvar.long_name="Vertical coordinate"
    zvar.units=zdat[1]
    zvar[:]=zdat[0]
  if latdat:
    onc.createDimension("lat",len(latdat))
    latvar=onc.createVariable("lat",latdat.typecode(),("lat",))
    latvar.long_name="Meridional coordinate"
    latvar.units="degrees_north"
    latvar[:]=latdat[:]
  if londat:
    onc.createDimension("lon",len(londat))
    lonvar=onc.createVariable("lon",londat.typecode(),("lon",))
    lonvar.long_name="Zonal coordinate"
    lonvar.units="degrees_east"
    lonvar[:]=londat[:]
  return onc

```
La variable que interesa introducir mediante ncstruct.py en un nuevo fichero
es la velocidad normalizada, la cual ya tiene en cuenta el cambio de densidad
con la altitud. Se debe hacer uso del módulo ncstruct.py implementado y las relaciones físicas necesarias:

```python

from ncstruct import *
dims=("time","latitude","longitude")

# Del fichero de entrada (Datos_2013-2022x.nc) se copian dimensiones, variables coordenadas y algunas las llena
# Un=normalized wind speed

onc=nccopystruct("Un_Datos_2013-2022.nc",x2,dims,dims,dims)
Un=onc.createVariable("Un",'f',dims)
Un.units="m/s"
Un.long_name="Normalized wind speed"

onc.sync()

nrecords=len(times)
rho0=1.225
Rd=287.
gradT=0.0065
g=9.81
for irec in range(nrecords):
    # print(irec)
    sp=np.array(x2.variables["sp"][irec,:,:],'d') # Pa
    t2m=np.array(x2.variables["t2m"][irec,:,:],'d') # Kelvin
    u100=np.array(x2.variables["u100"][irec,:,:],'d') # m/s
    v100=np.array(x2.variables["v100"][irec,:,:],'d') # m/s
    V100=np.sqrt(u100*u100+v100*v100) # Módulo de la velocidad

    # Se calcula la densidad

    H=Rd*((2.*t2m-gradT*98.)/2.)/g # H es un campo en dos dimensiones
    P=sp*np.exp((g*98/(g*H))) # El valor numérico se obtiene calculando el geopotencial con su definición
    rho=P/(Rd*(t2m-gradT*98.))
       
    # La velocidad normalizada se define como sigue
  
    Un=np.power(rho/rho0,1./3.)*V100
      
    onc.variables["Un"][irec,:,:]=Un
    
onc.close()

```
El programa completo genera el nuevo archivo NetCDF (Un_Datos_2013-2022.nc) con todos los datos
de velocidad normalizada necesarios para el posterior cálculo del factor de capacidad.

```python

d=nc.Dataset('Datos de viento ERA5/Un_Datos_2013-2022.nc')
print(d)

```

```bash

<class 'netCDF4._netCDF4.Dataset'>
root group (NETCDF3_CLASSIC data model, file format NETCDF3):
    Conventions: CF-1.6
    history: 2023-12-19 20:26:02 GMT by grib_to_netcdf-2.25.1: /opt/ecmwf/mars-client/bin/grib_to_netcdf.bin -S param -o /cache/data9/adaptor.mars.internal-1703016880.5709424-25829-12-5b1210de-b465-42db-904f-bf649f09ab68.nc /cache/tmp/5b1210de-b465-42db-904f-bf649f09ab68-adaptor.mars.internal-1703013763.4203577-25829-17-tmp.grib
C:\Users\xabie\miniconda3\envs\py3\Lib\site-packages\ipykernel_launcher.py -f C:\Users\xabie\AppData\Roaming\jupyter\runtime\kernel-2307ccc8-5d3e-46f5-8943-dcdb277c7955.json Mon Apr  8 13:42:03 2024
    dimensions(sizes): time(17520), latitude(5), longitude(8)
    variables(dimensions): int32 time(time), float32 latitude(latitude), float32 longitude(longitude), float32 Un(time, latitude, longitude)
    groups:
```
