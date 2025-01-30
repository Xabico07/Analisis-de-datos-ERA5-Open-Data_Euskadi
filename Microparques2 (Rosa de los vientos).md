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
Añadir la lista de velocidades 'Total_v2' permite observar cómo se distribuyen las velocidades del viento para cada dirección. Un ejemplo se muestra en la misma carpeta.

De la rosa de los vientos se determinan las velocidades que sí se deben tener en
cuenta a la hora de calcular cada factor de capacidad de las microturbinas. Para filtrar los datos que sí interesan de los que no, se puede emplear el siguiente código:

```python

# Se filtran las direcciones que cumplen con las condiciones determinadas para cada caso (X e Y son los grados que delimitan el filtro)
direcciones_filtradas = [dir for dir in Total_d if (X <= dir <= Y)]

# Se obtienen las velocidades correspondientes a las direcciones filtradas
velocidades_filtradas = [Total_v2[i] for i, dir in enumerate(Total_d) if (X <= dir <= Y)]

```
La lista de datos 'velocidades_filtradas' es la adecuada para el posterior cálculo del factor de capacidad. La visualización de datos se realiza de forma idéntica al caso del macroparque de Azazeta.
