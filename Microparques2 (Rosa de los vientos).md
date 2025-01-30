Dado el diseño del aerogenerador ROSBI, debe establecerse para qué direcciones se maximiza la eficiencia energética de los micropárques. Este tipo de análisis se lleva a cabo 
generando una rosa de los vientos, mediante la cual se puede observar la distribución
angular en función de varias variables, en este caso el tiempo que ha soplado el
viento para cada dirección. 

```python

from windrose import WindroseAxes 

ax = WindroseAxes.from_ax()
ax.bar(Total_d, Total_v2, normed=True, opening=0.8, edgecolor="white")
ax.set_legend()

```
Añadir la lista de velocidades 'Total_v2' permite observar cómo se distribuyen las velocidades del viento para cada dirección. Así, una rosa de los vientos puede verse como sigue:

![Gráfico de datos](Screenshots/Captura de pantalla 2024-05-05 213636.png)
