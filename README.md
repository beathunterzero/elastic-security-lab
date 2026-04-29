# elastic-security-lab

![Elasticsearch](https://img.shields.io/badge/Elasticsearch-8.17-blue)
![Kibana](https://img.shields.io/badge/Kibana-8.17-yellow)
![Filebeat](https://img.shields.io/badge/Filebeat-8.17-green)
![License](https://img.shields.io/badge/License-MIT-purple)

Elastic Security Lab es un laboratorio orientado a analistas de seguridad (SOC / Threat Hunters) para practicar ingestión de logs, análisis y detección usando Elasticsearch, Kibana y Filebeat.

Incluye una arquitectura funcional, datasets de entrenamiento y un pipeline de ingestión listo para uso local.

---

## Requisitos

- Docker  
- Docker Compose  
- WSL2 (solo en Windows)  
- Git  

---

## Quick Start

```bash
git clone https://github.com/beathunterzero/elastic-security-lab.git
cd elastic-security-lab
````

### 1. Configurar credenciales

Antes de iniciar, genera una nueva contraseña para `kibana_system`:

```bash
docker exec -it elasticsearch bin/elasticsearch-reset-password -u kibana_system
```

Luego actualiza la variable correspondiente en `docker-compose.yml`:

```
ELASTICSEARCH_PASSWORD=<password_generado>
```

---

### 2. Preparar estructura de datasets

```bash
mkdir -p datasets/windows datasets/linux datasets/aws datasets/azure datasets/firewall
```

---

### 3. Levantar el laboratorio

```bash
docker-compose up -d
```

Acceso por defecto:

```
http://localhost:5601
```

Credenciales iniciales:

```
username: elastic
password: changeme
```

---

## Uso del laboratorio

### Ingesta de logs

Coloca los archivos en las rutas correspondientes:

```
datasets/windows/
datasets/linux/
datasets/aws/
datasets/azure/
```

Filebeat procesará automáticamente los logs y los enviará a Elasticsearch.

---

### Data Views en Kibana

Crear un Data View con el patrón:

```
filebeat-*
```

Esto habilita el uso de:

- Discover
    
- Dashboards
    
- Lens
    

---

### Gestión de usuarios

Ruta en Kibana:

```
Stack Management → Security → Users
```

Roles recomendados:

- kibana_admin
    
- monitoring_user
    
- viewer
    

---

## Estructura del proyecto

```
elastic-security-lab/
│
├── datasets/
│         
├── filebeat/
│   └── filebeat.yml
│   
├── docs/                  
│   └── architecture/    
│   └── procesos/
│
├── docker-compose.yml
└── README.md
```

---

## Datasets

El laboratorio utiliza datasets públicos para entrenamiento:

- Windows Event Logs
    
- Linux auth logs
    
- AWS CloudTrail / GuardDuty
    
- Azure Activity / Sign-In
    
- Firewall logs
    

---

## Seguridad

Proyecto orientado a entorno local y fines educativos.  
No incluye datos sensibles ni configuraciones de producción.

---

## Licencia

MIT

---

## Autor

**beathunterzero**
