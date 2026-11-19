# Plan de validación del pronóstico

Este documento complementa el README y establece controles antes de interpretar resultados operativos.

## Integridad de datos

1. Registrar fecha de descarga y versión del archivo de Public Health Scotland.
2. Filtrar una sola categoría de asistencia (`Unplanned`) para evitar duplicar totales.
3. Fijar las ubicaciones y tipos de departamento incluidos; documentar cualquier cambio.
4. Revisar fechas repetidas, semanas ausentes y cambios de definición.
5. No tratar automáticamente las semanas faltantes como demanda igual a cero.

## Evaluación temporal

- Separar entrenamiento y prueba **cronológicamente**; no mezclar semanas futuras en el entrenamiento.
- Comparar contra un pronóstico ingenuo (última semana) y, cuando proceda, uno estacional.
- Usar validación de origen móvil (walk-forward) con horizontes equivalentes a la planeación real.
- Calcular MAE/MAD, RMSE y sesgo (MD) por ventana; documentar cómo se maneja MAPE si hay valores cercanos a cero.
- Construir cualquier buffer de capacidad usando únicamente errores históricos disponibles **antes** del periodo evaluado.
- Reportar la frecuencia real de insuficiencia de capacidad, además del error medio del pronóstico.

## Escenarios y límites

Un buffer no representa personal ni camas automáticamente disponibles. Antes de uso operativo deben considerarse tiempos de espera, capacidad efectiva, turnos, días festivos y cambios de política sanitaria.

Los datos de Escocia no justifican extrapolación directa a hospitales mexicanos sin revalidación local. Las estimaciones de este repositorio son exploratorias, no decisiones automáticas de asignación de recursos.
