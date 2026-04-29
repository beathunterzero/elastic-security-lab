## 1. Propósito de Filebeat en el Lab

**Filebeat** es el agente encargado de la **ingesta de logs** dentro del laboratorio.

Su función principal es:

- Leer archivos desde `datasets/`
    
- Parsear los datos (JSON o texto plano)
    
- Enriquecer eventos con metadata
    
- Enviar los eventos a Elasticsearch
    

En este laboratorio, Filebeat es crítico porque:

- No se usan integraciones automáticas
    
- Toda la ingesta es manual y controlada
    
- Permite entender completamente el flujo de datos
    

---

## 2. Estructura General del filebeat.yml

El archivo se divide en 4 bloques:

- Inputs → fuentes de datos
    
- Output → destino (Elasticsearch)
    
- Setup → conexión con Kibana
    
- Processors → enriquecimiento de eventos
    

---

## 3. Sección: Inputs (filebeat.inputs)

Define qué logs se van a leer y cómo procesarlos.

```yml
filebeat.inputs:
```

Cada bloque representa una fuente distinta de datos.

---

## 4. Input: Firewalls (Logs tipo Syslog)

```yml
- type: filestream  
  # Input moderno que reemplaza `log`, mejor manejo de rotación
   
  enabled: true  
  # Activa el input
  
  id: firewall-input  
  # Identificador único (evita conflictos internos)
  
  paths:
   - /datasets/firewall/*.log  
   # Define la ruta de logs
  
  fields:  
   log_type: firewall  
   # Etiqueta personalizada para filtrar en hunting
 
  fields_under_root: true  
  # Inserta los fields en el root del evento
  
  pipeline: "fortigate-kv"  
  # Aplica pipeline de parsing en Elasticsearch
  
  ignore_older: 0s  
  # Procesa todos los logs sin importar antigüedad
```

---

## 5. Input: Windows (Logs JSON)

 ```yml
 - type: filestream
   enabled: true
   id: windows-input
   paths:
   - /datasets/windows/*.json
 ``` 

### parsers:

```yml
parsers:
- ndjson  
  # Procesa logs JSON línea por línea
  
   target: ""  
   # Inserta el JSON directamente en el root

   add_error_key: true  
   # Añade campo si falla el parseo

   overwrite_keys: true  
   # Sobrescribe campos duplicados
```

---

### fields:

```yml
fields:
  log_type: windows
fields_under_root: true
ignore_older: 0s
```

---

## 6. Input: Linux (Syslog)

```yml
- type: filestream
    enabled: true
    id: linux-input
    paths:
      - /datasets/linux/*.log
    fields:
      log_type: linux
    fields_under_root: true
    ignore_older: 0s
```

---

## 7. Input: Azure (JSON)

```yml
- type: filestream
    enabled: true
    id: azure-input
    paths:
      - /datasets/azure/activity/*.json
      - /datasets/azure/signin/*.json
      - /datasets/azure/audit/*.json
      - /datasets/azure/firewall/*.json
```

### parsers:

```yml
parsers:
 - ndjson:
   target: ""
   add_error_key: true
   overwrite_keys: true
```

### fields:

```yml
fields:
      log_type: azure
    fields_under_root: true
    ignore_older: 0s
```

---

## 8. Input: AWS (JSON)

```yml
- type: filestream
    enabled: true
    id: aws-json-input
    paths:
      - /datasets/aws/cloudtrail/*.json
      - /datasets/aws/guardduty/*.json
```

### parsers:

```yml
parsers:
   - ndjson:
       target: ""
       add_error_key: true
       overwrite_keys: true
```

### fields:

```yml
fields:
      log_type: aws
    fields_under_root: true
    ignore_older: 0s
```

---

## 9. Input: AWS (Logs Texto)

```yml
- type: filestream
    enabled: true
    id: aws-log-input
    paths:
      - /datasets/aws/vpcflow/*.log
      - /datasets/aws/s3access/*.log
    fields:
      log_type: aws
    fields_under_root: true
    ignore_older: 0s
```

---

## 10. Sección: Output (Elasticsearch)

```yml
output.elasticsearch:
  hosts: ["http://es01:9200"]
  username: "elastic"
  password: "changeme"
```

Credenciales de acceso al cluster

---

## 11. Sección: Setup Kibana

```yml
setup.kibana:
  host: "http://kibana:5601"
  username: "elastic"
  password: "changeme"
```

Permite que Filebeat configure dashboards si se requiere

---

## 12. Sección: Processors

Los processors enriquecen los eventos automáticamente.

```yml
processors:
  - add_host_metadata: ~
  - add_cloud_metadata: ~
  - add_docker_metadata: ~
  - add_kubernetes_metadata: ~
```

---

### add_host_metadata

Añade:

- hostname
    
- IP
    
- sistema operativo
    

---

### add_cloud_metadata

Detecta si el host está en:

- AWS
    
- Azure
    
- GCP
    

---

### add_docker_metadata

Añade:

- container ID
    
- imagen
    
- labels
    

Muy útil para debugging en Docker

---

### add_kubernetes_metadata

Añade metadata si corre en Kubernetes  
(No crítico en este lab, pero útil si escala)

---

## 13. Sección: Logging

```yml
logging.level: info
# Nivel de logs

logging.to_files: false
# No guarda logs en archivos

logging.to_stdout: true
# Envía logs a consola Docker
```

---

## 14. Flujo de Procesamiento en Filebeat

1. Detecta archivos en datasets
    
2. Lee eventos línea por línea
    
3. Parsea (JSON o texto)
    
4. Añade metadata
    
5. Envía a Elasticsearch
    

---

## 15. Decisiones de Diseño del Lab

Este filebeat.yml está diseñado para:

- Control total de ingesta
    
- Separación clara por tipo de log
    
- Facilidad de hunting mediante fields
    
- Compatibilidad con múltiples fuentes
    

---

## 16. Buenas Prácticas Aplicadas

- Uso de IDs únicos por input
    
- Uso de fields para clasificación
    
- Uso de ndjson para logs estructurados
    
- Uso de read-only volumes
    

---

## 17. Posibles Mejoras Futuras

- Integración con pipelines ECS completos
    
- Uso de ingest pipelines personalizados
    
- Normalización avanzada (ECS full mapping)
    
- Integración con Logstash
    

---

## 18. Consideraciones de Laboratorio

- No está optimizado para alto volumen
    
- No incluye parsing avanzado completo
    
- Prioriza visibilidad sobre performance
    
