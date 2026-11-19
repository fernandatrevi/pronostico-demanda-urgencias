# Protocolo de validación del pronóstico y capacidad

Este documento propone criterios de evaluación. **No presenta resultados calculados ni pruebas ejecutadas.**

## Alcance
La unidad de análisis será el número de atenciones no programadas por semana para una selección documentada de ubicaciones y tipos de departamento. No se sumarán categorías agregadas (`All`) con sus subcategorías.

## Prevención de fuga temporal
1. Ordenar la serie por semana y documentar fechas de cobertura.
2. Separar un periodo final de prueba posterior a los datos usados para seleccionar métodos.
3. Comparar pronóstico ingenuo, ingenuo estacional y métodos de suavización con la misma ventana.
4. Ajustar hiperparámetros exclusivamente en ventanas históricas de validación progresiva.
5. Calcular MAD/MAE, sesgo MD y RMSE por horizonte; usar MAPE solo donde la demanda real sea distinta de cero.

## Buffer de capacidad
- Definir explícitamente el objetivo de cobertura (porcentaje de semanas con demanda cubierta).
- Estimar el buffer exclusivamente con errores de pronósticos fuera de muestra de entrenamiento/validación.
- Evaluar en prueba la cobertura empírica, exceso medio de capacidad y semanas con insuficiencia.
- Comparar contra una política sin buffer y contra una regla de referencia sencilla.
- Evitar afirmar ahorros, mejoras operativas o seguridad del paciente sin validación en contexto.

## Entregables de cierre
Código reproducible, fuente y licencia de datos, corte temporal, comparación tabulada, gráfica de observado vs. pronosticado, definición de buffer y limitaciones.
