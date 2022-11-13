# Protecto 2
# Creación de Mapas de Calor

## Sistemas de Información Geográficos
### Diego Muñoz y Francisco Villanueva 

## Semestre II
## 2022
___

### Introducción
<br>
Para el segundo proyecto del curso, se nos solicitó realizar unos mapas de calor con respecto a una base de datos en Costa Rica. Para ello se nos otorgo temas sobre los cuales se harian estos mapas. El tema asignado de estre proyecto fue la proyección de la población costarricense en el 2021 y el 2022. Además, era necesario ubicar las escuelas y los hospitales en las areas de mayor población. Para ello fue necesario tomar unas bases de datos de los distritos de Costa Rica y mediante el uso de SQL hacer una conexión entre las tablas de los distritos y las tablas de la población proyectada, los métodos se explicaran mas adelante.  


<br>

### Metodología
<br>
Primeramente, una vez estaba el tema seleccionado fue necesario analizar las tablas e incluso editarlas para lograr que estas tuvieran una conexión y de esta forma se pudieran unir. En este caso particular la tabla que nos fue otorgada con la proyección proyectada estaba dividida en provincias, cantones y distritos, por lo que hubo que eliminar las provincias y acomodar los cantones para que cada distrito tuviera su cantón correspondiente. Además, fue necesario quitar algunas filas que estaban de mas, ya que, la tabla estaba divida en población femenina, masculina y la que nos interesaba era el total. De igual manera, se tuvo que hacer ciertas correcciones en algunos nombres de los distritos, ya que, estaban escritos diferentes. 

<br>
Luego de tener las tablas y hacer que tuvieran columnas iguales para lograr hacer una conexión, se utilizó la libreria Spatialite de SQLite para lograr importar los shape files y las tablas en formato csv para poder unir las mismas. Este fue el comando que se utilizó para hacer esta conexión: 

<br>

SELECT c1.ncanton, c1.ndistrito, c2.DISTRITO, c2.CANTON, c1.geometry, c2.TOTAL
FROM geo_distritos AS c1, poblacionCantidadSinTildes32022 AS c2
WHERE c1.ncanton = c2.CANTON and c1.ndistrito = c2.DISTRITO

<br>
Una vez se tenían las tablas unidas, esta se exportaba en formato shapefile, para poder hacer el trabajo de mapas en grass. Lo que primero que se realizo fue importar el archivo con las referencias empaciales que contenía el archivo. Una vez el archivo estaba en grass, se realizo una interpolación mediante el uso de distancia inversa ponderada y de esta manera tener un raster donde se pudieran observar las zonas mas pobladas. Esto se realizo mediante el siguente comando

<br>

```
g.region vect="Nombre_Del_Archivo"

v.surf.idw input="Nombre_Del_Archivo" output=idw npoints=6 column=TOTAL
```
<br>
Algunos de los parámetros del comando original fueron modificados, el primero de ellos es el parámetro de npoints, este lo hicimos con seis puntos para que hiciera una interpolación mayor y se modificó el column para que el valor que se guardara fueran la cantidad de habitantes en el distrito. Este paso se tuvo que repetir con las datos de polación del 2021 y 2022. Una vez, se obtenía el raster, la primera parte del proyecto estaba terminada, ya que, se podía oberservar el mapa de calor en los diferentes distritos de Costa Rica. 

<br>
Para la segunda parte solicitada del proyecto, que es ver cuales escuelas se encuentran en los distritos de mayor población. Para ello, de igual manera se importó el archivo de escuelas en vectorial y se hizo la conversión a raster mediante el siguente comando:

<br>

```
g.region vect="Nombre_Del_Archivo"

v.to.rast in="Nombre_Del_Archivo" out=rasterEscuelas use=attr column=codigopres
```
<br>

Igual que el comando anterior, fue necesario modificar algunos parámetros. Se cambio el parámetro use y se igualó a attr, ya que, era necesario que el raster guarda datos que son atributos y se agregó el parámetro column=codigopres, que es la columna de los atributos que necesitaban ser guardados. La columna codigopres funciona como un ID que identifica las diferentes escuelas.  
<br>
Se continuó haciendo una especie de filtro que nos permitiera crear un raster apartir de un algebra de mapas, solo con los distritos que tuvieran en este caso mas de 50 mil habitantes. Esto se hizo mediante el siguente comando: 

```
r.mapcalc expression="resultado=("raster_de_los distritos">=50000)"
```
<br>
A partir de este raster se puede comparar cuales escuelas se encuentrar adentro de estos distritos. Sin embargo, por la necesidad de que los puntos vectoriales conservaran los datos que tenian de la escuela, se tuvo que realizar la tabla desde cero con las escuelas que se encotraban en los distritos de mas de 50 mil habitantes. Luego se hizo una conexión entre la tabla de las escuelas que cumplían la condición y la tabla de la polación del 2021. Esto para exportarse como shapefile y de esta manera ya tener las escuelas resultantes. Cabe recalcar que ese paso se repitió con los hospitales, el único cambio que se realizó fue que en vez de compararlo con 50 mil habitantes, se hizo con 30 mil habitantes en el distrito. 

<br>

### Digitalizar el Mapa
<br>
Por último, era necesario que este mapa fuera publicado en la web. Para ello se utilizó la pagina web "OpenStreetMap" para obtener la plantilla sobre la que se colocaría las imagenes raster. Mediante la librería leaflet se realizó todo el procedimiento para crear los mapas. Para mostrar las imaganes raster se uso la funcion de imageOverlay y se le colocaron coordenas a la imagen para que se coincidieran con Costa Rica. Además, para poder colocar los hospitales y escuelas estos fuero exportados de grass como GeoJSON para que en leaflet fueran ubicados como puntos. Posteriormente mediante el uso de Netlify, se pubilicaron estos mapas en la web para el uso gratituo. 
