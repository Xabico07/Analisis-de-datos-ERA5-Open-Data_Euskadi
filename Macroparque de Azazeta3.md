Con el objetivo de visualizar los resultados, la librería 'Matplotlib' permite crear gráficos y visualizaciones de datos. En
concreto, el módulo 'Matplotlib.hist' permite visualizar histogramas generadas mediante 'numpy'. Empleando las definiciones establecidas:

```python

import numpy as np
import matplotlib.pyplot as plt

Un=np.array(d.variables["Un"][:,4,3],'d')
# En el arreglo numérico generado se tienen en cuenta solo los datos asociados
# a la celda donde se encuentra el parque de Azazeta ([:,4,3])
np.histogram(Un, bins=12, range=(0,24), density=None, weights=None)
# Genera dos arreglos numéricos, un array con las frecuencias de cada bin (cantidad de elementos que
# caen en cada intervalo), y un array con los límites de los bins.


#Para ver el histograma, se emplea:

plt.hist(Un, bins=12, range=(0,24), density=None, color='blue', alpha=0.7)
plt.title('Distibrución de velocidad normalizada durante 10 años')
plt.xlabel('Un (m/s)')
plt.ylabel('t (h)')
plt.show()

```
