
# Pronóstico de demanda de urgencias y buffer de capacidad

Proyecto de análisis de datos con Python para estimar las atenciones semanales de urgencias no programadas y proponer un margen de capacidad ante aumentos inesperados de demanda.

## Problema

La demanda de urgencias varía con el tiempo. Subestimar las atenciones puede dificultar la planeación de recursos, mientras que sobreestimarlas puede llevar a reservar capacidad que no se utiliza.

Este proyecto comparará métodos de pronóstico y calculará un buffer basado en los errores históricos para explorar escenarios de capacidad semanal.

## Objetivos

- Analizar el comportamiento histórico de las atenciones.
- Identificar tendencias y posibles patrones estacionales.
- Comparar métodos de pronóstico.
- Evaluar los errores mediante MD, MAD, MAPE y RMSE.
- Estimar un buffer de capacidad.
- Evaluar cuánto ayuda ese buffer a cubrir la demanda observada.

## Base de datos

Se utiliza la base pública **Weekly A&E Activity and Waiting Times**, publicada por Public Health Scotland.

Fuente:
https://www.opendata.nhs.scot/uk/dataset/weekly-accident-and-emergency-activity-and-waiting-times

Archivo: `weekly_ae_activity_20260920.csv`

El archivo contiene 42,915 filas y 14 columnas, con información semanal de servicios de urgencias de Escocia desde 2015.

Cada fila representa una combinación de semana, ubicación, tipo de departamento y categoría de atención. Las filas no equivalen a pacientes individuales.

Los datos se publican bajo la UK Open Government Licence. Public Health Scotland se reconoce como fuente de los datos.

## Variables principales

| Variable | Descripción |
|---|---|
| WeekEndingDate | Fecha de cierre de la semana |
| TreatmentLocation | Código de la ubicación de atención |
| DepartmentType | Tipo de departamento |
| AttendanceCategory | Categoría de atención |
| NumberOfAttendancesEpisode | Número de atenciones registradas |
| NumberOver4HoursEpisode | Atenciones con espera superior a cuatro horas |

La variable que se pronosticará será el número de atenciones por semana.

## Preparación de los datos

1. Revisar tipos de datos, valores faltantes y posibles duplicados.
2. Convertir las fechas al formato adecuado.
3. Seleccionar la categoría `Unplanned`, correspondiente a atenciones no programadas.
4. Definir las ubicaciones incluidas en el análisis.
5. Agrupar las atenciones por semana.
6. Ordenar las fechas y verificar la continuidad semanal.

La base también contiene las categorías `All` y `New planned`. Se seleccionará una sola categoría para evitar contar dos veces las mismas atenciones.

Las semanas faltantes se investigarán antes de decidir cómo tratarlas; no se asumirán automáticamente como semanas con cero demanda.

## Análisis de la demanda

Se estudiarán los siguientes componentes:

- **Nivel:** cantidad habitual de atenciones semanales.
- **Tendencia:** aumentos o disminuciones sostenidos.
- **Estacionalidad:** patrones que se repiten en ciertos periodos del año.
- **Variabilidad:** cambios en la demanda entre semanas.

El análisis incluirá gráficas históricas y estadísticas descriptivas para identificar qué modelos tiene sentido comparar.

## Métodos de pronóstico

| Método | Descripción |
|---|---|
| Ingenuo | Utiliza la demanda de la última semana como pronóstico de la siguiente. |
| Ingenuo estacional | Utiliza la demanda de un periodo equivalente anterior. |
| Promedio móvil | Promedia las últimas semanas para suavizar variaciones. |
| Suavización exponencial simple | Da mayor peso a observaciones recientes y modela el nivel. |
| Holt | Modela el nivel y la tendencia. |
| Holt-Winters | Modela el nivel, la tendencia y la estacionalidad. |

Holt y Holt-Winters son extensiones de la suavización exponencial.

Los métodos estacionales se utilizarán si los datos muestran patrones repetitivos y existe suficiente historia para estimarlos.

Los métodos ingenuos servirán como referencia para evaluar si los modelos más elaborados mejoran el pronóstico.

## Parámetros de suavización

- **Alfa:** controla la actualización del nivel.
- **Beta:** controla la actualización de la tendencia en Holt y Holt-Winters.
- **Gamma:** controla la actualización de la estacionalidad en Holt-Winters.

Los parámetros y el tamaño de la ventana del promedio móvil se elegirán usando los datos de entrenamiento y validación.

## Métricas de error

Se definirá el error como:

**Error = demanda real − pronóstico**

Con esta convención:
- Un error positivo indica subestimación.
- Un error negativo indica sobreestimación.

### MD: desviación media o sesgo

**MD = suma de los errores / número de pronósticos**

También se conoce como error medio o ME.

Permite identificar si el modelo tiende a pronosticar por debajo o por encima de la demanda.

Un MD cercano a cero no garantiza precisión: los errores positivos y negativos pueden compensarse.

### MAD: desviación absoluta media

**MAD = suma de los valores absolutos de los errores / número de pronósticos**

En este proyecto equivale al MAE.

Indica cuántas atenciones nos equivocamos en promedio, sin que los errores positivos y negativos se cancelen.

Será la métrica principal por su interpretación directa en atenciones.

### MAPE: error absoluto porcentual medio

**MAPE = promedio de |error / demanda real| × 100**

Expresa el error promedio en porcentaje.

No se calculará directamente para observaciones con demanda real igual a cero. También se revisará su interpretación cuando existan valores muy pequeños.

### RMSE: raíz del error cuadrático medio

**RMSE = raíz cuadrada del promedio de los errores al cuadrado**

Penaliza más los errores grandes y se expresa en atenciones.

Complementará al MAD para identificar modelos que presentan fallas importantes en determinadas semanas.

### Criterio de selección

Se buscará un MAD bajo, revisando también MAPE, RMSE y MD.

La selección no dependerá únicamente de un sesgo cercano a cero, porque un modelo puede tener errores grandes que se compensan.

## Validación temporal

Los datos se dividirán respetando el orden de las fechas:

1. **Entrenamiento:** ajustar los modelos.
2. **Validación:** comparar métodos, seleccionar parámetros y calcular el buffer.
3. **Prueba:** evaluar el desempeño final en semanas posteriores.

Se simularán pronósticos de la siguiente semana utilizando solamente información disponible hasta ese momento.

El método de pronóstico, sus parámetros y la regla del buffer se definirán antes de evaluar el conjunto de prueba.

## Aplicación a logística hospitalaria

El proyecto conecta el pronóstico con la planeación de demanda y capacidad.

| Concepto | Aplicación |
|---|---|
| Planeación de demanda | Estimar las atenciones de la próxima semana. |
| Subestimación | Identificar atenciones reales superiores al pronóstico. |
| Sobreestimación | Identificar pronósticos superiores a las atenciones reales. |
| Buffer de capacidad | Añadir un margen para absorber demanda inesperada. |
| Capacidad objetivo | Establecer un escenario de cobertura en atenciones semanales. |
| Cobertura | Medir el porcentaje de semanas cuya demanda queda dentro de la capacidad objetivo. |

## Buffer de capacidad

El buffer será un margen adicional al pronóstico.

Se propone calcularlo usando el percentil 95 de los errores de validación, definidos como demanda real menos pronóstico, con un mínimo de cero.

**Buffer = máximo entre cero y el percentil 95 de los errores de validación**

**Capacidad objetivo = pronóstico + buffer**

El percentil 95 representa un nivel de cobertura propuesto para el escenario. Su desempeño se comprobará en semanas posteriores; no garantiza una cobertura futura del 95%.

El buffer se calculará con errores del mismo horizonte que se desea cubrir: pronósticos de una semana hacia adelante.

### Ejemplo ilustrativo

Si el pronóstico es de 1,000 atenciones y el buffer calculado es de 120:

**Capacidad objetivo = 1,000 + 120 = 1,120 atenciones semanales**

Este ejemplo muestra el procedimiento y no representa un resultado del proyecto.

## Evaluación del escenario de capacidad

En el conjunto de prueba se calcularán:

- **Cobertura:** porcentaje de semanas en que la demanda real no supera la capacidad objetivo.
- **Déficit de capacidad:** máximo entre cero y demanda real menos capacidad objetivo.
- **Margen no utilizado:** máximo entre cero y capacidad objetivo menos demanda real.

Se compararán escenarios con y sin buffer para observar el intercambio entre mayor cobertura y mayor capacidad reservada.

Estas cantidades serán escenarios calculados, no mediciones de recursos disponibles en los hospitales.

## Alcance y limitaciones

El proyecto pronosticará atenciones semanales. No permitirá estimar directamente la demanda por día o por hora.

La base contiene datos de servicios de urgencias de Escocia. Para aplicar el análisis a un hospital de México sería necesario utilizar sus registros y evaluar nuevamente los modelos.

La capacidad objetivo se expresará en atenciones semanales. Para convertirla en necesidades de médicos, enfermeros, camas o insumos se requerirían datos sobre:
- Recursos y turnos disponibles.
- Duración de las atenciones.
- Complejidad de los casos.
- Productividad y restricciones operativas.

La cobertura semanal no garantiza capacidad suficiente durante los picos dentro de una misma semana.

Los cambios extraordinarios y las modificaciones en el registro de datos pueden afectar los pronósticos y la utilidad del buffer.

## Herramientas previstas

- Python
- Pandas
- NumPy
- Matplotlib
- Statsmodels
- Jupyter Notebook
- Git y GitHub

## Resultados esperados

- Serie semanal preparada para el análisis.
- Gráficas del comportamiento histórico.
- Comparación de métodos mediante MD, MAD, MAPE y RMSE.
- Selección de un modelo mediante validación temporal.
- Pronósticos de demanda semanal.
- Escenario de capacidad con buffer.
- Evaluación de cobertura, déficit y margen no utilizado.
- Conclusiones sobre la utilidad y las limitaciones del análisis.

## Referencias

- Public Health Scotland. Weekly A&E Activity and Waiting Times.
  https://www.opendata.nhs.scot/uk/dataset/weekly-accident-and-emergency-activity-and-waiting-times

- Hyndman y Athanasopoulos. Forecasting: Principles and Practice.
  https://otexts.com/fpp3/

## Estado del proyecto

Proyecto completado: preparación de datos, análisis exploratorio, comparación de pronósticos y evaluación de un buffer de capacidad.

## Resultados del pronóstico

La serie contiene 605 semanas. Se utilizaron 501 para entrenamiento, 52 para validación y 52 para prueba, respetando el orden temporal.

Se compararon Naive, Naive estacional, promedio móvil de cuatro semanas, suavizamiento exponencial simple, Holt y Holt-Winters.

Holt-Winters obtuvo el menor MAD en validación: 628.23 atenciones semanales. Su ventaja frente al suavizamiento exponencial simple fue pequeña, aproximadamente 0.85%.

En prueba, Holt-Winters obtuvo:

| Métrica | Resultado |
|---|---:|
| MD | 23.07 |
| MAD | 761.22 |
| MAPE | 2.88% |
| RMSE | 1,011.94 |

Cada pronóstico semanal se calculó usando únicamente información disponible de semanas anteriores.

## Buffer de capacidad

Se calculó un buffer de 1,234.82 atenciones semanales usando el percentil 95 de los errores de validación, definidos como demanda real menos pronóstico.

La capacidad propuesta se calculó como:

Capacidad = pronóstico + buffer

| Indicador en prueba | Sin buffer | Con buffer |
|---|---:|---:|
| Semanas con capacidad suficiente | 51.92% | 86.54% |
| Semanas con faltante | 25 | 7 |
| Faltante acumulado | 20,391.45 | 3,819.74 |
| Excedente acumulado | 19,191.80 | 66,830.58 |

El buffer redujo los faltantes, pero aumentó el excedente de capacidad. La cobertura observada fue de 86.54%; el percentil 95 de validación no garantiza una cobertura de 95% en prueba.

Estas cantidades representan diferencias entre demanda y capacidad propuesta. Para convertirlas en personal, camas o turnos se requieren datos operativos adicionales.

## Valor para el hospital y sus pacientes

Este análisis permite anticipar la demanda semanal y evaluar cuánta capacidad adicional considerar. Su utilidad consiste en apoyar la planificación de recursos antes de que aumenten las atenciones.

### Beneficios potenciales

| Área | Qué podría mejorar | Cómo comprobarlo |
|---|---|---|
| Operación | Preparación de personal y espacios para periodos de mayor demanda. | Comparar saturación y tiempos de espera antes y después de implementar la propuesta. |
| Finanzas | Menor necesidad de horas extra imprevistas y mejor uso del presupuesto. | Comparar los costos de la planificación actual con los de la propuesta. |
| Experiencia del paciente | Una atención más organizada, si el pronóstico se traduce en recursos adecuados. | Medir tiempos de espera, satisfacción y quejas. |
| Reputación | Mayor confianza en el hospital si mejora la experiencia de atención de forma sostenida. | Evaluar satisfacción, recomendaciones y percepción del servicio. |

### Ejemplo de aplicación en Suiza

En Suiza, la Oficina Federal de Salud Pública publica datos operativos e indicadores de calidad hospitalaria. En este contexto, adaptar el análisis podría apoyar una gestión basada en evidencia y complementar la evaluación del servicio.

Un hospital suizo podría utilizar sus registros de urgencias para comparar modelos, calcular un buffer local y evaluar si la planificación propuesta mejora la disponibilidad de recursos. Después debería medir su efecto sobre costos, tiempos de espera y experiencia del paciente.

Este es un ejemplo de adaptación. El proyecto actual utiliza datos de Escocia y no ha sido validado en un hospital suizo.

Fuente: [Oficina Federal de Salud Pública de Suiza: datos de hospitales](https://www.bag.admin.ch/en/facts-figures-on-hospitals).

### Qué demuestra este proyecto

En el escenario de prueba, el buffer redujo las semanas con faltantes de 25 a 7, pero aumentó la capacidad sobrante.

Esto demuestra un intercambio entre cubrir más demanda y reservar más capacidad. Todavía no se han demostrado ahorros, ingresos adicionales, menores tiempos de espera ni mejoras de reputación.

Para evaluar esos beneficios se necesitan datos operativos y económicos del hospital, además de una comparación con su planificación actual.

## Cómo adaptar el proyecto a otro hospital

La metodología puede utilizarse con datos de otro hospital: preparar la demanda semanal, comparar pronósticos y evaluar un margen adicional de capacidad.

Es necesario volver a seleccionar el modelo y calcular el buffer con datos locales. Holt-Winters y el buffer de 1,234.82 atenciones corresponden al conjunto de datos analizado y no deben copiarse directamente a otro hospital.

### 1. Datos necesarios para comenzar

Para adaptar el pronóstico se necesita una tabla con:

| Dato | Utilidad |
|---|---|
| Fecha de cierre de la semana | Ordenar los datos y separar los periodos de evaluación. |
| Número de atenciones por semana | Construir y evaluar los pronósticos. |
| Identificador del hospital, si se incluyen varios | Analizar la demanda de cada centro. |
| Definición de las atenciones incluidas | Mantener un criterio consistente, por ejemplo, urgencias no programadas. |

Antes de modelar, se deben revisar duplicados, semanas ausentes y cambios en el registro. Una semana sin datos no debe interpretarse automáticamente como cero atenciones.

La configuración actual de Holt-Winters utiliza ciclos de 52 semanas y requiere al menos dos ciclos completos para inicializarse, además de datos posteriores para validación y prueba. Si el historial es limitado, se deben adaptar los periodos de evaluación y considerar métodos más simples.

### 2. Pasos para replicar el análisis

1. Obtener los registros del hospital y agrupar las atenciones por semana.
2. Adaptar la ruta del CSV, los nombres de las columnas y los filtros del notebook.
3. Construir `demanda_semanal`, con las fechas como índice y una columna llamada `Atenciones`.
4. Revisar la demanda, las semanas atípicas y los posibles patrones estacionales.
5. Separar entrenamiento, validación y prueba en orden cronológico.
6. Comparar los modelos con los datos locales y seleccionar uno utilizando el criterio de evaluación definido.
7. Calcular el buffer únicamente con los errores de validación.
8. Evaluar el pronóstico y el buffer en prueba, midiendo errores, cobertura, faltantes y sobrantes.
9. Incorporar nuevas observaciones y revisar periódicamente si el desempeño se mantiene.

En este proyecto se reservaron 52 semanas para validación y 52 para prueba. Esa división debe revisarse según la cantidad de datos disponibles en cada hospital.

Los pronósticos se realizan una semana adelante. Primero se estima la demanda y después se incorpora el resultado real. Para anticipar varias semanas sin conocer los resultados intermedios, se necesita una evaluación diferente.

### 3. Datos para convertir la demanda en recursos

El pronóstico estima atenciones semanales. Para traducirlo en personal, camas o turnos se necesitan datos operativos adicionales:

| Datos adicionales | Para qué sirven |
|---|---|
| Llegadas por día y hora | Identificar los turnos con mayor demanda. |
| Gravedad de los casos según el triaje | Distinguir diferentes necesidades de atención. |
| Tiempo de trabajo de médicos y enfermería por tipo de caso | Estimar las horas de personal necesarias. |
| Tiempo de ocupación de camas o espacios de urgencias | Estimar cuántos espacios se requieren simultáneamente. |
| Personal y espacios disponibles por turno | Comparar los recursos necesarios con los disponibles. |
| Pacientes que requieren hospitalización y su tiempo de espera | Evaluar la ocupación mientras esperan una cama hospitalaria. |
| Costos de personal, horas extra y capacidad adicional | Comparar el costo de diferentes escenarios. |

Por ejemplo, 1,000 atenciones adicionales por semana no equivalen a 1,000 camas. La necesidad de camas depende de cuántos pacientes coinciden y cuánto tiempo permanecen.

### 4. Mejoras para una siguiente versión

| Mejora | Motivo |
|---|---|
| Evaluar varias ventanas temporales | Comprobar si los resultados se mantienen en diferentes periodos. |
| Analizar cada hospital por separado | Detectar patrones locales ocultos en los datos agregados. |
| Comparar distintos niveles de buffer en validación | Medir el intercambio entre reducir faltantes y aumentar capacidad sobrante. |
| Incorporar costos y una meta de cobertura | Elegir un margen acorde con las necesidades y recursos del hospital. |
| Evaluar demanda diaria o por turno | Apoyar decisiones más específicas de programación de personal. |
| Explorar variables de calendario disponibles al pronosticar | Comprobar si mejoran la precisión frente a los métodos actuales. |
| Registrar versiones de Python y de las librerías | Facilitar la reproducción de los resultados. |

### 5. Alcance y limitaciones

El buffer alcanzó una cobertura de 86.54% en prueba, por debajo del 95% planteado. Se necesita evaluar su estabilidad en más periodos antes de utilizarlo para decisiones operativas.

Los faltantes y sobrantes representan diferencias entre demanda y capacidad propuesta. No son mediciones de pacientes sin atención ni de recursos realmente ociosos.

La aplicación en otro hospital requiere validación con sus propios datos y comparación con su capacidad real.

## Conclusiones

El pronóstico semanal permite anticipar la demanda y evaluar escenarios de capacidad. Holt-Winters presentó el menor MAD en validación, aunque su ventaja frente al suavizamiento exponencial simple fue pequeña.

Agregar un buffer redujo las semanas con faltante de 25 a 7 durante la prueba, a cambio de un mayor excedente de capacidad.

Los resultados corresponden a datos agregados de Escocia y no deben trasladarse directamente a un hospital mexicano sin una validación local.

### Instalar las librerías necesarias

Antes de ejecutar el notebook por primera vez, abre la terminal de VS Code y ejecuta:

```bash
pip install pandas numpy matplotlib statsmodels ipykernel
```

Este comando instala las herramientas que utiliza el proyecto:

- **pandas:** leer el CSV y organizar los datos.
- **numpy:** realizar cálculos numéricos.
- **matplotlib:** crear las gráficas.
- **statsmodels:** calcular los pronósticos de suavizamiento exponencial, Holt y Holt-Winters.
- **ipykernel:** ejecutar las celdas del notebook con Python.

Si estas librerías ya están instaladas en el entorno de Python seleccionado, puedes omitir este paso.