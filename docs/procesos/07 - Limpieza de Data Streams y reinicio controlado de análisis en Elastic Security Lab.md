## 1. Introducción y Propósito

Durante actividades de Threat Hunting en laboratorios controlados, es común trabajar múltiples escenarios consecutivos:

- Ransomware
    
- C2
    
- PowerShell
    
- Movimiento lateral
    
- Persistencia
    
- Exfiltración
    

Si los eventos anteriores permanecen indexados en Elasticsearch, el entorno comienza a contaminarse con telemetría residual.

Esto puede generar:

- Ruido en Discover
    
- Resultados mezclados entre escenarios
    
- Dashboards inconsistentes
    
- Falsos positivos durante el hunting
    
- Dificultad para validar hipótesis específicas
    

Por esta razón, en el laboratorio se implementa un procedimiento de limpieza controlada utilizando `DELETE _data_stream`.

Este proceso permite reiniciar el entorno analítico sin destruir completamente el stack de Docker.

---

## 2. Alcance del Procedimiento

Este procedimiento aplica únicamente para el laboratorio local:

- Elastic Security Lab
    
- Filebeat
    
- Datasets manuales
    
- Ingesta controlada
    
- Análisis por escenarios
    

No está diseñado para producción.

En producción, eliminar Data Streams puede destruir evidencia, afectar retención, romper dashboards o impactar monitoreo continuo.

---

## 3. Concepto Técnico: ¿Qué es un Data Stream?

En Elastic Stack 8.x, Filebeat puede escribir eventos en **Data Streams**, que son estructuras modernas orientadas a datos de tipo time-series y telemetría continua.

Un Data Stream actúa como una capa lógica sobre índices internos.

En este laboratorio, el Data Stream principal es:

`filebeat-8.17.10`

Este Data Stream almacena eventos provenientes de:

- Windows
    
- Linux
    
- Firewall
    
- Azure
    
- AWS
    
- Metadata agregada por Filebeat
    

---

## 4. Problema Operativo Detectado

Durante el análisis, aunque los archivos originales fueran eliminados del directorio `datasets/`, los eventos seguían apareciendo en Kibana.

Esto ocurre porque:

- Elasticsearch ya indexó los eventos
    
- El Data Stream sigue existiendo
    
- Kibana consulta los datos almacenados en Elasticsearch
    
- El Data View sigue apuntando a esos datos
    

Resultado:

- Discover muestra eventos antiguos
    
- Las queries devuelven resultados contaminados
    
- El análisis pierde limpieza
    
- Las hipótesis pueden validarse con datos que no pertenecen al escenario actual
    

---

## 5. Separación de Capas

El laboratorio debe entenderse en tres capas:

|Capa|Descripción|¿Se elimina con DELETE _data_stream?|
|---|---|---|
|`datasets/`|Logs físicos en disco|No|
|Filebeat|Agente que lee y envía logs|No|
|Elasticsearch Data Stream|Logs indexados|Sí|

El comando `DELETE _data_stream/filebeat-8.17.10` solo elimina los datos indexados en Elasticsearch.

No elimina:

- Archivos locales
    
- Configuración de Filebeat
    
- Configuración de Kibana
    
- Reglas
    
- Dashboards
    
- Data Views
    

---

## 6. Estrategia de Limpieza Controlada

La estrategia validada para el laboratorio es:

1. Eliminar el dataset físico utilizado
    
2. Acceder a Kibana con la cuenta administrativa `elastic`
    
3. Entrar a Dev Tools
    
4. Eliminar el Data Stream de Filebeat
    
5. Cargar únicamente el nuevo dataset a analizar
    

Esto permite:

- Mantener Docker funcionando
    
- Evitar reinicios completos innecesarios
    
- Reiniciar escenarios rápidamente
    
- Analizar datasets aislados
    
- Trabajar hunts reproducibles
    

---

## 7. Procedimiento Operativo Completo

### Paso 1: Eliminar dataset anterior

Ejemplo para logs Windows:

`rm datasets/windows/*`

O eliminar un archivo específico:

`rm datasets/windows/windows-ransomware.json`

---

### Paso 2: Acceder con usuario administrativo

Entrar a Kibana con el usuario:

`elastic`

Este usuario tiene permisos administrativos para ejecutar acciones de mantenimiento sobre Elasticsearch.

Nota:

No usar el usuario `hunter` para esta tarea, ya que la eliminación de Data Streams requiere privilegios elevados.

---

### Paso 3: Abrir Dev Tools

Ruta dentro de Kibana:

Management → Dev Tools

---

### Paso 4: Ejecutar limpieza del Data Stream

Ejecutar:

`DELETE _data_stream/filebeat-8.17.10`

Este comando elimina el Data Stream y sus datos asociados en Elasticsearch.

---

### Paso 5: Validar limpieza en Discover

Regresar a Discover y confirmar:

- No aparecen eventos antiguos
    
- No hay resultados residuales
    
- El entorno queda limpio para el siguiente dataset
    

---

### Paso 6: Cargar nuevo dataset

Copiar el nuevo log al directorio correspondiente.

Ejemplo:

`cp windows-cobaltstrike.json datasets/windows/`

Filebeat detectará el nuevo archivo y lo enviará a Elasticsearch.

---

## 8. Flujo Recomendado de Trabajo

### Inicio de análisis

1. Limpiar dataset anterior
    
2. Eliminar Data Stream
    
3. Cargar nuevo dataset
    
4. Validar ingesta en Discover
    
5. Ejecutar hunting
    

---

### Fin de análisis

1. Documentar hallazgos
    
2. Eliminar dataset usado
    
3. Eliminar Data Stream
    
4. Dejar el laboratorio limpio
    

---

## 9. Consideraciones Técnicas sobre Filebeat

Filebeat mantiene estado interno de lectura.

Ese estado puede incluir:

- archivos ya procesados
    
- offsets de lectura
    
- información del archivo
    
- posición hasta donde leyó
    

Por eso, si se reutiliza exactamente el mismo archivo con el mismo nombre, puede ocurrir que Filebeat no lo reprocese como se espera.

---

## 10. Recomendación de Nombres por Escenario

Para evitar problemas de reingesta, usar nombres únicos por escenario:

`windows-ransomware-01.json`

`windows-c2-01.json`

`windows-powershell-01.json`

`windows-lateral-movement-01.json`

Esto ayuda a que Filebeat trate cada archivo como un dataset nuevo y mejora la trazabilidad del laboratorio.

---

## 11. Ventajas del Método

Este procedimiento permite:

- Hunts reproducibles
    
- Escenarios limpios
    
- Reducción de ruido
    
- Validación precisa de reglas
    
- Testing de hipótesis aisladas
    
- Control total sobre qué datos existen en Kibana
    

También mejora el flujo de trabajo en:

- Threat Hunting
    
- Detection Engineering
    
- DFIR
    
- Purple Team Labs
    
- ATT&CK Emulation
    

---

## 12. Limitaciones del Método

Este método no reemplaza una limpieza completa del laboratorio.

Puede no limpiar:

- registry interno de Filebeat
    
- dashboards
    
- reglas
    
- usuarios
    
- Data Views
    
- configuraciones guardadas
    

Si se necesita un reinicio total del laboratorio, usar:

`docker compose down -v`

Ese comando elimina volúmenes persistentes, incluyendo datos de Elasticsearch.

---

## 13. Cuándo usar DELETE _data_stream

Usar cuando:

- Se quiere limpiar Discover
    
- Se quiere eliminar ruido de análisis anteriores
    
- Se cambia de escenario
    
- Se validan reglas con datasets pequeños
    
- Se trabaja con un laboratorio controlado
    

No usar cuando:

- Se necesita conservar evidencia
    
- Se trabaja en producción
    
- Se requiere trazabilidad histórica
    
- Se está midiendo retención de logs
    

---

## 14. Conclusión Técnica

La limpieza del Data Stream permite mantener el laboratorio en un estado controlado y reproducible.

El objetivo no es borrar por comodidad, sino preservar la calidad del análisis.

Principio operativo:

> Un hunt limpio requiere datos limpios.

