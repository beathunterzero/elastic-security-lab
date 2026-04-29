## 1. Introducción al Data View

Un **Data View** (anteriormente _Index Pattern_) es el componente que permite a Kibana **interpretar, consultar y visualizar los datos almacenados en Elasticsearch**.

Sin un Data View:

- No se pueden ejecutar queries en Discover
    
- No se pueden construir dashboards
    
- No se puede hacer Threat Hunting operativo
    

Es, en la práctica, el **punto de entrada al análisis de datos** dentro del SIEM.

---

## 2. Contexto dentro del Laboratorio

En este laboratorio:

- Filebeat envía datos a índices con prefijo `filebeat-*`
    
- Cada tipo de log (windows, firewall, aws, etc.) queda dentro de esos índices
    
- Kibana necesita una referencia para agruparlos
    

El Data View cumple ese rol de abstracción.

---

## 3. Configuración Paso a Paso

### 3.1 Acceso Inicial

Acceder a Kibana desde el navegador:

[http://localhost:5601](http://localhost:5601/)

Credenciales:

- Usuario: elastic (o usuario creado)
    
- Password: definido en docker-compose
    

---

### 3.2 Navegación

Ruta dentro de Kibana:

1. Ir a **Stack Management**
    
2. Sección **Kibana**
    
3. Seleccionar **Data Views**
    
4. Click en **Create data view**
    

---

### 3.3 Definición del Data View

Completar los siguientes campos:

---

#### Name

elastic-security-lab

Propósito:

- Nombre lógico para identificar el dataset
    
- No afecta a Elasticsearch
    

---

#### Index pattern

filebeat-*

Explicación técnica:

- `filebeat-*` indica que Kibana leerá todos los índices que empiecen con "filebeat-"
    
- Permite centralizar múltiples fuentes de datos
    

Ejemplos incluidos:

- filebeat-firewall
    
- filebeat-windows
    
- filebeat-linux
    
- filebeat-azure
    
- filebeat-aws
    

---

#### Timestamp field

@timestamp

Explicación:

- Campo estándar generado por Filebeat
    
- Representa el momento exacto del evento
    
- Es obligatorio para análisis temporal
    

Impacto en Threat Hunting:

- Permite correlación temporal
    
- Permite detectar patrones (ej. beaconing, bursts)
    

---

## 4. Verificación de Ingesta

Una vez creado el Data View:

1. Ir a **Discover**
    
2. Seleccionar `elastic-security-lab`
    
3. Ajustar rango de tiempo (últimos 15 min / 1 hora / 24 horas)
    

---

### Resultado esperado

- Histograma de eventos visible
    
- Logs listados en tiempo real
    
- Campos disponibles para filtrado
    

---

## 5. Validación mediante Queries

Usar filtros rápidos para validar fuentes de datos:

log_type : "windows"  
log_type : "firewall"  
log_type : "aws"  
log_type : "azure"

Propósito:

- Confirmar que Filebeat está etiquetando correctamente
    
- Validar separación lógica de datasets
    

---

## 6. Interpretación Operativa

Si los datos aparecen correctamente:

- La ingesta está funcionando
    
- Elasticsearch está indexando correctamente
    
- Kibana puede consultar los datos
    

Esto valida todo el pipeline:

datasets → filebeat → elasticsearch → kibana

---

## 7. Troubleshooting (Resolución de Problemas)

Si no aparecen datos, validar lo siguiente:

---

### 7.1 Contenedores

Comando:

docker ps

Verificar:

- es01 → running
    
- kibana → running
    
- filebeat → running
    

---

### 7.2 Datasets

Comando:

ls -l datasets/

Verificar:

- Existencia de archivos `.log` o `.json`
    
- Rutas correctas
    

---

### 7.3 Salud de Elasticsearch

Abrir en navegador:

[http://localhost:9200/_cluster/health](http://localhost:9200/_cluster/health)

Estado esperado:

- yellow o green
    

---

### 7.4 Verificación de índices

[http://localhost:9200/_cat/indices?v](http://localhost:9200/_cat/indices?v)

Debe mostrar índices `filebeat-*`

---

### 7.5 Logs de Filebeat

docker logs filebeat

Buscar:

- errores de parsing
    
- errores de conexión
    

---

### 7.6 Rango de tiempo en Kibana

Problema común:

- Kibana filtra por tiempo reciente
    
- Logs antiguos no aparecen
    

Solución:

- Cambiar a "Last 24 hours" o "Last 7 days"
    

---

## 8. Buenas Prácticas

- Usar siempre `@timestamp` como campo temporal
    
- Validar ingestión antes de crear dashboards
    
- Utilizar fields como `log_type` para segmentar
    
- Confirmar índices antes de crear queries
    

---

## 9. Consideraciones de Laboratorio

- No hay separación por índices por tipo (todo entra en filebeat-*)
    
- No hay ILM (Index Lifecycle Management)
    
- No hay control de retención
    

Esto es intencional para:

- Simplificar el laboratorio
    
- Priorizar aprendizaje sobre optimización
    

---

## 10. Evolución Recomendada

A futuro se puede:

- Separar índices por tipo de log
    
- Implementar ECS completo
    
- Crear múltiples Data Views por dominio
    
- Integrar dashboards por caso de uso
    
