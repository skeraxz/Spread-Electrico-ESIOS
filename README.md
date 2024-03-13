# Evolución del Spread del PVPc

El mercado eléctrico español es uno de los más eficientes de Europa, si
atendemos a su ajuste entre la oferta y la demanda de energía. Sin embargo, no
son pocos los desafíos a los que se enfrenta.

Una de las críticas más comunes está relacionada con su método para calcular
el Precio voluntario para el pequeño consumidor (PVPc), pues este precio se
indexa al resultado del precio SPOT diario, sin tener en cuenta sus previsiones
a futuro. El resultado es una mayor volatilidad en el precio que pagan la mayoría
de los consumidores. Una volatilidad muy por encima de la media europea.

Conocedor de este hecho, el gobierno español ha publicado una nueva
metodología de cálculo de dicho precio, con una progresiva adaptación hacia un
modelo donde se tienen en cuenta las previsiones a largo plazo. Esta fórmula
entró en vigor el 1 de enero de 2024 y sus resultados son todavía ambiguos.

Este proyecto tiene dos objetivos: realizar una comparación entre el spread de precios de 2023 con lo que llevamos de año y explorar las facilidades que nos ofrecen
las nuevas plataformas de Cloud Computing, como lo es Microsoft Fabric.

# Contenido

Este repositorio está compuesto de tres carpetas:

La primera de ellas, denominada "Dashboard", contiene el archivo de Power BI con el reporte final. Al trabajar con una versión de prueba de Microsoft Fabric, la exportación de datos fuera de la plataforma es
bastante limitada.

A continuación, la carpeta "Images" contiene capturas de la herramienta Fabric por dentro. A través de ellas explicaré la ETL planteada en nuestro análisis. También contiene una foto del dashboard de Power BI.

Finalmente, la carpeta "Notebooks" contiene el código de Spark y Python que hemos utilizado en este trabajo. Uno de ellos extrae los datos de la API de ESIOS, mientras que el otro se corresponde con las transformaciones y carga de tablas realizadas sobre nuestros datos.

# Agradecimientos

Quisiera agradecer a @Rogarui por su guía para el uso de la API de ESIOS. Sin él saberlo, me ha servido de gran ayuda para la extracción de información de REE.

Aquí dejo su perfil de Linkedin: https://www.linkedin.com/in/rogarui/

Aquí dejo su guía de uso: https://github.com/rogarui/ESIOS

# Metodología

Los datos se extraen desde la API de ESIOS ofrecida por REE. Para su extracción es necesario obtener un token de acceso por parte de REE. Se puede conseguir mandando un correo a [consultasios@ree.es](mailto:consultasios@ree.es).
Los detalles sobre el código se encuentran directamente sobre el notebook "Obtencion_Datos".

Los datos corresponden al valor del PVPc para cada hora del día desde el 1 de Enero de 2023. La ETL se ejecuta cada día, de manera que los datos se actualizan con una frecuencia diaria.

Una vez extraídos los datos de la API de ESIOS, es necesario importarlos en nuestro entorno cloud. Para ello, debemos importar la url generada con el notebook anterior.

Para entender el proceso, fijémonos en la imagen "Proceso_ETL". Todos los objetos de la imagen pertenecen a un mismo espacio de trabajo. En la parte superior derecha encontramos el pipeline "API_Indicators". Este objeto actúa como orquestador de procesos, puesto que importa los datos de la API y ejecuta el notebook "Transformacion_Datos".

En la esquina superior izquierda encontramos nuestro Lakehouse. Este tipo de repositorio permite almacenar tanto datos estructurados como no estructurados. Además, permite trabajar directamente con notebooks de Spark o Python. Sin embargo, su Endpoint para SQL no permite crear nuevas tablas. Estan deben crearse a través de "Delta Tables" usando Spark.

El Endpoint "REE_Lakehouse" recibe los datos directamente desde el Lakehouse anterior. Como veremos más adelante, apenas hacemos uso de su capacidad SQL. Sin embargo, permite generar un modelo semántico sobre el que seleccionar las tablas que queremos incluir en el informe de Power BI.

Queda así establecido el siguiente orden: el pipeline importa los datos a través de la url obtenida anteriormente. A su vez, este objeto carga los datos sobre el Lakehouse bajo la tabla "Id_1001_2023_NoEnd_AVG". Posteriormente ejecuta el notebook "Transformación_Datos" y guarda las tablas "Spread_PVPc" y "Bar_Spread_PVPc" sobre el mismo Lakehouse, que ahora contiene 3 tablas. El mismo pipeline permite programar una hora de ejecución. En nuestro caso ejecutamos el proceso diariamente a las 23:50, de manera que obtengamos todos los datos del día anterior.
Una vez cargadas las 3 tablas sobre el Lakehouse, hacemos una breve comprobación de calidad con SQL para verificar que la fecha del último dato disponible se corresponde con la fecha de ejecución (imagen Endpoint). Finalmente, seleccionamos las tablas desde el modelo semántico, también con actualización diaria, y generamos el informe.

El Spread de precio se calcula como la diferencia entre el precio máximo y el precio mínimo del PVPc para cada día. Este análisis se realiza directamente sobre Microsoft Fabric. Como se observa en el notebook "Transformacion_Datos", la carga y lectura de las tablas se realizan con Spark. Después, se pueden trabajar con Python - Pandas.

### Nota

Tanto en Notebook "ID_1001_Pruebas" como el modelo semántico "REE_Lakehouse" no se usan en este trabajo. Sin embargo he decidido mantenerlo por su ventaja organizativa que me ofrece de cara a otros proyectos.

# Comentarios sobre imágenes

La imagen "Pipeline" muestra el interior de este objeto. Podemos observar tres ventanas: la ventana superior izquierda muestra la conexión con la API de ESIOS y su trasnformación con Spark y Python. La ventana derecha muestra la capacidad de programar su ejecución y su ventana inferior muestra la posibilidad de modificar el nombre y el tipo de las variables, similar a como se haria con Power Query.

La imagen "Endpoint" muestra 3 sencillas querys de SQL. Su finalidad es verificar que la fecha del último dato obtenido corresponde con la fecha de ejecución del pipeline.

La imagen "Spread_PVPc" muestra una captura del informe de Power BI. Se incluye esta imagen por los posibles errores que genere el archivo pbix al descargarse, puesto que estamos trabajando con una versión de prueba.

# Conclusiones

Es complicado aislar el impacto de la nueva metodología estatal del resto de factores que influyen en el precio de la electricidad. Sin embargo, si tomamos el mismo periodo de 2023 y de 2024, podemos observar una reducción en el spread del PVPc del 23%.

La línea discontinua naranja muestra la media del periodo en el año 2023, mientras que la línea verde muestra la media para 2024. Observamos como, para la mayoría de los días, los valores de 2024 se encuentran por debajo de la media de 2023.

El valor máximo del periodo lo encontramos en enero de 2023, mientras que el valor mínimo fue registrado a finales de febrero de este año.

Recordemos que el informe se actualiza de manera diaria, por lo que los resultados están en continua evolución.

Cualquier investigación bien realizada siempre marca el inicio de nuevas preguntas:

¿Qué ha ocurrido con el precio PVPc desde la entrada en vigor del nuevo método de cálculo?

¿Qué podemos esperar que ocurra en 2024 con respecto al PVPc?

### Dónde encontrarme

Linkedin: [www.linkedin.com/in/alvaro-munoz-gismero](http://www.linkedin.com/in/alvaro-munoz-gismero)
