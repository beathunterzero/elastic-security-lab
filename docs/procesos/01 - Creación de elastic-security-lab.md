## 1. Introducción y Propósito

El **Elastic Security Lab** es un entorno SIEM (Security Information and Event Management) diseñado para **centralizar, procesar y analizar logs desde múltiples fuentes** con fines de Threat Hunting.

Este laboratorio está construido sobre el **Elastic Stack (Elasticsearch + Kibana + Filebeat)** utilizando contenedores Docker en un entorno local (WSL2), priorizando:

- Simplicidad de despliegue
    
- Control total de la ingesta
    
- Flexibilidad para pruebas de hunting
    

La versión utilizada es **8.17.10 (LTS)** por su estabilidad y compatibilidad en entornos de laboratorio.

---

## 2. Objetivo Técnico del Laboratorio

El laboratorio no está diseñado como un SIEM productivo, sino como un entorno controlado para:

- Simular ingesta de logs reales (cloud, endpoints, red)
    
- Practicar consultas de hunting (KQL)
    
- Analizar comportamiento basado en TTPs (MITRE ATT&CK)
    
- Validar hipótesis de Threat Hunting
    

Este enfoque permite pasar de un modelo teórico a uno operativo.

---

## 3. Flujo de Datos del Laboratorio (Arquitectura Lógica)

El flujo de datos es el componente más importante del laboratorio:

datasets → filebeat → elasticsearch → kibana

### Descripción del flujo:

1. **datasets (logs crudos)**  
    Contiene archivos de logs organizados por fuente (AWS, Azure, Windows, etc.).
    
2. **Filebeat (ingesta)**  
    Lee los archivos, los parsea (JSON o texto) y agrega metadata útil para hunting.
    
3. **Elasticsearch (almacenamiento)**  
    Indexa los eventos y permite búsquedas rápidas y correlación.
    
4. **Kibana (visualización)**  
    Permite explorar, filtrar y analizar los datos mediante dashboards y queries.
    

---

## 4. Preparación Inicial (Creación de Directorios)

Antes de ejecutar el laboratorio por primera vez, es obligatorio crear la estructura de directorios.  
Esto garantiza que Filebeat tenga rutas válidas desde el inicio y evita errores de montaje en Docker.

Ejecutar los siguientes comandos:

````bash
mkdir -p datasets/aws/cloudtrail  
mkdir -p datasets/aws/guardduty  
mkdir -p datasets/aws/s3access  
mkdir -p datasets/aws/vpcflow

mkdir -p datasets/azure/activity  
mkdir -p datasets/azure/audit  
mkdir -p datasets/azure/firewall  
mkdir -p datasets/azure/signin

mkdir -p datasets/linux  
mkdir -p datasets/windows  
mkdir -p datasets/firewall

mkdir filebeat
`````


### Propósito técnico:

- Crear la estructura base del repositorio
    
- Permitir que Docker monte correctamente los volúmenes
    
- Evitar errores de Filebeat por rutas inexistentes
    
- Preparar el entorno para escalar datasets fácilmente
    

---

## 5. Estructura de Directorios (FileSystem)

Una vez creados los directorios, la estructura queda de la siguiente forma:

elastic-security-lab/  
├── datasets/  
│ ├── aws/  
│ │ ├── cloudtrail/  
│ │ ├── guardduty/  
│ │ ├── s3access/  
│ │ └── vpcflow/  
│ ├── azure/  
│ │ ├── activity/  
│ │ ├── audit/  
│ │ ├── firewall/  
│ │ └── signin/  
│ ├── linux/  
│ ├── windows/  
│ └── firewall/  
├── docs/  
│ └── architecture/  
├── filebeat/  
│ └── filebeat.yml  
└── docker-compose.yml

---

## 6. Explicación de Componentes

### datasets/

Repositorio de logs crudos.  
Cada subdirectorio representa una fuente de datos distinta.

Propósito en Threat Hunting:

- Simular múltiples superficies de ataque
    
- Permitir correlación entre fuentes
    
- Facilitar pruebas controladas
    

---

### filebeat/

Contiene la configuración del agente de ingesta.

Propósito:

- Leer logs desde datasets
    
- Parsear JSON o texto
    
- Enviar eventos a Elasticsearch
    

---

### docker-compose.yml

Archivo que orquesta todo el stack:

- Elasticsearch → almacenamiento
    
- Kibana → visualización
    
- Filebeat → ingesta
    

---

### docs/

Contiene documentación y diagramas del laboratorio.

Propósito:

- Documentar arquitectura
    
- Mantener trazabilidad
    
- Servir como base de conocimiento
    

---

## 7. Principios de Diseño del Lab

### 1. Separación de responsabilidades

Cada componente tiene un rol claro:

- Ingesta → Filebeat
    
- Procesamiento → Elasticsearch
    
- Visualización → Kibana
    

---

### 2. Control total de la data

A diferencia de entornos productivos:

- No se usan integraciones automáticas
    
- Toda la ingesta es manual y controlada
    

Esto permite entender completamente el flujo de datos.

---

### 3. Optimización para Threat Hunting

El diseño prioriza:

- Búsquedas rápidas por tipo de log
    
- Identificación clara del origen de eventos
    
- Capacidad de pivoting entre datasets
    

---

## 8. Casos de Uso dentro del Lab

Este entorno permite simular:

- Persistencia en endpoints (Windows/Linux)
    
- Ejecución de comandos sospechosos
    
- Beaconing y C2
    
- Exfiltración de datos
    
- Movimiento lateral
    

Todo mediante datasets controlados.

---

## 9. Guía de Ejecución

### Levantar el laboratorio

docker-compose up -d

---

### Acceso a Kibana

[http://localhost:5601](http://localhost:5601/)

---

### Detener el laboratorio

docker-compose stop

---

### Reiniciar completamente

docker-compose down && docker-compose up -d

---

## 10. Consideraciones Importantes

- Este entorno es **solo para laboratorio**
    
- No se recomienda en producción sin:
    
    - TLS habilitado
        
    - Gestión segura de credenciales
        
    - Arquitectura distribuida
        

---

## 11. Enfoque para Evolución

Este laboratorio está preparado para evolucionar hacia:

- Integración con EDR (ej. Velociraptor)
    
- Automatización de detecciones
    
- Creación de reglas SIEM
    
- Simulación de ataques reales
    
