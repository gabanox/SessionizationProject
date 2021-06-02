# Creando sesiones de flujo de clics en tiempo real y ejecutando análisis con Amazon Kinesis Data Analytics, AWS Glue y Amazon Athena 


#### I: Implementación automatizada con AWS CloudFormation

Todos los pasos de esta solución de un extremo a otro se incluyen en una plantilla de AWS CloudFormation . Encienda la plantilla, agregue el código en su servidor web y listo, obtendrá la sesionización en tiempo real.

Esta plantilla de AWS CloudFormation está diseñada para implementarse solo en la región us-east-1.

#### II: Creando la pila

1. Para comenzar, inicie sesión en la [Consola de administración de AWS](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/create/template?stackName=blog-sessionization&templateURL=https://s3.amazonaws.com/aws-bigdata-blog/artifacts/realtime-clickstream-sessions-analytics-kinesis-glue-athena/streaming-stagger-window.template) y luego abra la plantilla de ventana escalonada.

![GitHub](Cloud2/CloudSprint1.jpg)

2. En la consola de **AWS CloudFormation**, elija **Siguiente** y complete los parámetros de AWS CloudFormation:
    - **Nombre de pila** : El nombre de la pila ( el blog-sesionización o sesiones-blog )
    - **StreamName** : sesionesblog
    - **Recuento de fragmentos de transmisión** : 1 o 2 (1 MB / s) por fragmento.
    - **Nombre del depósito** : cambie a un nombre único, por ejemplo, session-n-bucket-hhug123121
    - **Intervalo de búfer** : sugerencia de almacenamiento en búfer de 60 a 900 segundos para Kinesis Data Firehose antes de que los datos se envíen a Amazon S3 desde Kinesis Data Firehose.
    - **Tamaño del búfer** : 1–128 MB por archivo, si el intervalo no se alcanza primero.
    - **Prefijo de destino** : agregado (carpeta interna del depósito para guardar datos agregados).
    - **Base de sesiones en segundos o minutos** : elija cuál desea (los minutos comenzarán con 1 minuto, los segundos comenzarán con 30 segundos).

![GitHub](Cloud2/CloudSprint2.jpg)

![GitHub](Cloud2/CloudSprint3.jpg)

![GitHub](Cloud2/CloudSprint4.jpg)

![GitHub](Cloud2/CloudSprint5.jpg)

3. Compruebe si el lanzamiento se ha completado y, si no lo ha hecho, compruebe si hay errores.

***Nota: el error más común es cuando apunta a un bucket de Amazon S3 que ya existe.***

![GitHub](Cloud2/CloudSprint6.jpg)

#### III: Procesar los datos

1. Después de la implementación, navegue hasta la solución en la consola de **Amazon Kinesis**

![GitHub](Cloud2/CloudSprint7.jpg)

2. Vaya a la página de aplicaciones de Kinesis Analytics y elija ***AnalyticsApp-blog-sessionizationXXXXX*** , de la siguiente manera.

![GitHub](Cloud2/CloudSprint8.jpg)

3. Elija **Ejecutar** aplicación para iniciar la aplicación.

![GitHub](Cloud2/CloudSprint9.jpg)

4. Espere unos segundos hasta que la aplicación esté disponible y luego elija **Detalles de la aplicación**.

![GitHub](Cloud2/CloudSprint10.jpg)

5. En la página de detalles de la aplicación, elija **Ir a resultados de SQL**

![GitHub](Cloud2/CloudSprint11.jpg)

6. Examine el código **SQL** y ``` SOURCE_SQL_STREAM ```, y cambie el **INTERVALO** si lo desea.

![GitHub](Cloud2/CloudSprint12.jpg)

7. Elija la   pestaña **Análisis en tiempo real** para verificar los resultados de ``` DESTINATION_SQL_STREAM ```

![GitHub](Cloud2/CloudSprint13.jpg)

8. Verifique la pestaña **Destino** para ver la función **AWS Lambda** como el destino de su agregación.

![GitHub](Cloud2/CloudSprint14.jpg)

9. Compruebe el panel de control **CloudWatch** en tiempo real.

![GitHub](Cloud2/CloudSprint15.jpg)

10. Abra el panel **Sessionization - < el nombre de su pila de formación en la nube >**

***Nota: Verifique el número de "eventos" durante las sesiones y el comportamiento de la "duración de la sesión" de un período de tiempo. Luego, puede tomar decisiones, como si necesita revertir un nuevo diseño de sitio o nuevas funciones de su aplicación.***

![GitHub](Cloud2/CloudSprint16.jpg)

11. Abra la consola de **AWS Glue** y ejecute el rastreador que la plantilla de AWS CloudFormation que creó para usted.

12. Elija el trabajo del rastreador y luego elija *Ejecutar rastreador*.

![GitHub](Cloud2/CloudSprint17.jpg)

#### IV: Analizar los datos

1. Una vez finalizado el trabajo, abra la consola de **Amazon Athena** y explore los datos.

2. En la consola de Athena, elija la **base de datos de sesionización en la lista**. Debería ver dos tablas creadas en función de los datos de Amazon S3: datos sin procesar y agregados.

![GitHub](Cloud2/CloudSprint18.jpg)

3. Elija la **elipsis vertical (tres puntos)** en el lado derecho para explorar cada una de las tablas, como se muestra en las siguientes capturas de pantalla: 

![GitHub](Cloud2/CloudSprint19.jpg)

![GitHub](Cloud2/CloudSprint20.jpg)

4. Cree una vista en la consola de Athena para consultar ***solo los datos de hoy*** de su tabla agregada, de la siguiente manera:

```
CREATE OR REPLACE VIEW clicks_today AS
SELECT 
*
FROM "aggregated" 
WHERE
cast(partition_0 as integer)=year(current_date) and
cast(partition_1 as integer)=month(current_date) and
cast(partition_2 as integer)=day(current_date) ;
```

5. La consulta exitosa aparece en la consola de la siguiente manera:

![GitHub](Cloud2/CloudSprint21.jpg)

6. Cree una vista para consultar ***solo los datos del mes actual*** de su tabla agregada, como en el siguiente ejemplo:

```
CREATE OR REPLACE VIEW clicks_month AS
SELECT 
*
FROM "aggregated" 
WHERE
cast(partition_0 as integer)=year(current_date) and
cast(partition_1 as integer)=month(current_date) ;
```

7. La consulta exitosa aparece de la siguiente manera:

![GitHub](Cloud2/CloudSprint22.jpg)

8. Consultar datos con las ***sesiones agrupadas por la duración de la sesión ordenadas por sesiones***, de la siguiente manera:

```
SELECT duration_sec, count(1) sessions 
FROM "clicks_today"
where duration_sec>0
group by duration_sec
order by sessions desc;
```
9. Los resultados de la consulta aparecen de la siguiente manera:

![GitHub](Cloud2/CloudSprint23.jpg)

#### V: Visualiza los datos

1. Abra la consola de **Amazon QuickSight**

***Nota: Si nunca ha utilizado Amazon QuickSight, [primero realice esta configuración.](https://docs.aws.amazon.com/es_es/quicksight/latest/user/signing-up.html)***

2. Configure los ajustes de la cuenta de Amazon QuickSight para acceder a **Athena** y su **bucket de S3**.

3. Primero, seleccione la **casilla de verificación Amazon Athena**. Seleccione la **casilla de verificación Amazon S3** para editar el acceso de Amazon QuickSight a sus buckets S3.

![GitHub](Cloud2/CloudSprint33.jpg)

4. Elija los depósitos que desea que estén disponibles y, luego, seleccione **Seleccionar depósitos**.

![GitHub](Cloud2/CloudSprint34.jpg)

5. Elija **Administrar datos**
6. Luego elija **NUEVO CONJUNTO DE DATOS**

![GitHub](Cloud2/CloudSprint24.jpg)

7. En la lista de fuentes de datos, elija **Athena**.

![GitHub](Cloud2/CloudSprint25.jpg)

8. Ingrese **daily_session** como su nombre de fuente de datos.

![GitHub](Cloud2/CloudSprint26.jpg)

9. Elija la vista que creó para las sesiones diarias y elija **Seleccionar**

![GitHub](Cloud2/CloudSprint27.jpg)

10. Luego, puede optar por utilizar **SPICE (caché)** o el acceso directo a consultas.

![GitHub](Cloud2/CloudSprint28.jpg)

11. Elija **beginnavigation** y **duration_sec** como métricas.

![GitHub](Cloud2/CloudSprint29.jpg)

12. Elija **+ Agregar** para agregar una nueva visualización.

![GitHub](Cloud2/CloudSprint30.jpg)

13. En tipos visuales , elija el **tipo de gráfico de mapa de árbol**.

![GitHub](Cloud2/CloudSprint32.jpg)

14. Para Agrupar por , elija **device_id** ; para Tamaño , elija **duration_sec (Suma)** ; y para Color , elija **eventos (Suma)**.

![GitHub](Cloud2/CloudSprint31.jpg)

#### Resumen
En esta práctica, aprendió cómo realizar la sesionización de eventos de flujo de clics y analizarlos en una arquitectura sin servidor. El uso de una ventana escalonada de Kinesis Data Analytics hace que el código SQL sea breve y fácil de escribir y comprender. La integración entre los servicios permite un flujo de datos completo con una codificación mínima.

También aprendió formas de explorar y visualizar estos datos con Amazon Athena, AWS Glue y Amazon QuickSight.













