## 1. Propósito del docker-compose en el Lab

El archivo `docker-compose.yml` es el **orquestador del laboratorio**. Define cómo se crean, configuran y conectan los servicios:

- Elasticsearch → almacenamiento e indexación
    
- Kibana → visualización
    
- Filebeat → ingesta de logs
    

Permite levantar todo el entorno con un solo comando, asegurando consistencia y reproducibilidad.

---

## 2. Estructura General del Archivo

El archivo está dividido en 3 bloques principales:

- services → define los contenedores
    
- networks → define la red interna
    
- volumes → define almacenamiento persistente
    

---

## 3. Servicio: Elasticsearch (es01)

Este es el componente más crítico del laboratorio.

```yml
services:  
es01:
```

---

### image

```yml
image: docker.elastic.co/elasticsearch/elasticsearch:8.17.10
```

Define la imagen oficial de Elasticsearch en versión LTS.

---

### container_name

```yml
container_name: es01
```

Asigna un nombre fijo al contenedor.  
Permite que otros servicios lo referencien fácilmente (ej. `http://es01:9200`).

---

### restart

```yml
restart: unless-stopped
```

El contenedor se reinicia automáticamente si falla, excepto si se detiene manualmente.

---

### environment

Define variables internas de configuración:

```yml
enviroment:
  - node.name=es01
  # Nombre del nodo dentro del cluster
  
  - discovery.type=single-node  
  # Ejecuta Elasticsearch en modo standalone (sin cluster)
 
  - xpack.security.enabled=true  
  # Activa autenticación y control de acceso
   
  - ELASTIC_PASSWORD=changeme  
  # Define la contraseña del usuario `elastic`
 
  - xpack.security.http.ssl.enabled=false  
  # Desactiva HTTPS (solo para laboratorio)
 
  - xpack.security.transport.ssl.enabled=false  
  # Desactiva cifrado interno entre nodos (no relevante en single-node)
   
  - ES_JAVA_OPTS=-Xms2g -Xmx2g  
  # Define memoria fija de 2GB para la JVM  
  # Evita consumo dinámico que degrade el host
```


---

### ulimits

```yml
ulimits:  
  memlock:  
   soft: -1  
   hard: -1
```

Permite bloquear memoria en RAM.  
Evita uso de swap → mejora rendimiento crítico en Elasticsearch.

---

### volumes

```yml
volumes:
  - esdata:/usr/share/elasticsearch/data
```

Define almacenamiento persistente.  
Los datos sobreviven aunque el contenedor se elimine.

---

### ports

```yml
ports:
  - "9200:9200"
```

Expone Elasticsearch en el host.  
Acceso vía: [http://localhost:9200](http://localhost:9200/)

---

### networks

```yml
networks:
  - soc-lab-net
```

Conecta el contenedor a la red interna del laboratorio.

---

### healthcheck

```yml
healthcheck:  
  test: ["CMD-SHELL", "curl -s -u elastic:changeme http://localhost:9200/_cluster/health | grep -E '"status":"(yellow|green)"'"]  
  interval: 10s  
  timeout: 10s  
  retries: 10
```

Valida que Elasticsearch esté operativo.

- Usa curl para consultar el estado del cluster
    
- Espera estado "yellow" o "green"
    
- Permite a Kibana saber cuándo puede iniciar
    

---

## 4. Servicio: Kibana

Interfaz visual del SIEM.

```yml
kibana:
```

---

### image

```yml
image: docker.elastic.co/kibana/kibana:8.17.10
```

Define la versión compatible con Elasticsearch.

---

### container_name

```yml
container_name: kibana
```

Nombre fijo del contenedor.

---

### restart

```yml
restart: unless-stopped
```

Reinicio automático en caso de fallo.

---

### depends_on

```yml
depends_on:  
  es01:  
    condition: service_healthy
```

Kibana no inicia hasta que Elasticsearch esté listo.  
Evita errores de conexión al arranque.

---

### environment

```yml
environment:
   - ELASTICSEARCH_HOSTS=[http://es01:9200](http://es01:9200/)  
   # Define el endpoint interno de Elasticsearch
   
   - ELASTICSEARCH_USERNAME=kibana_system  
   # Usuario técnico de Kibana
   
   - ELASTICSEARCH_PASSWORD=***  
   # Contraseña del usuario
   
   - SERVER_PUBLICBASEURL=[http://localhost:5601](http://localhost:5601/)  
   # URL pública de acceso
   
   - XPACK_ENCRYPTEDSAVEDOBJECTS_ENCRYPTIONKEY=***  
   # Clave para cifrar objetos internos (dashboards, creds)
```

---

### ports

```yml
ports:
  - "5601:5601"
```

Expone Kibana en el host.  
Acceso vía navegador.

---

### networks

```yml
networks:
  - soc-lab-net
```

Permite comunicación con Elasticsearch.

---

## 5. Servicio: Filebeat

Agente de ingesta de logs.

```yml
filebeat:
```

---

### image

```yml
image: docker.elastic.co/beats/filebeat:8.17.10
```

Versión compatible con el stack.

---

### container_name

```yml
container_name: filebeat
```

Nombre del contenedor.

---

### user

```yml
user: root
```

Ejecuta con privilegios elevados.  
Necesario para leer todos los archivos montados.

---

### restart

```yml
restart: unless-stopped
```

Reinicio automático.

---

### depends_on

```yml
depends_on:
  - es01
```

Asegura que Elasticsearch esté disponible.

---

### command

```yml
command: ["-e", "--strict.perms=false"]
```

- -e → logs en consola (útil para debugging)
    
- --strict.perms=false → ignora permisos estrictos en WSL
    

---

### volumes

```yml
volumes:
  - ./filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
  - ./datasets:/datasets:ro
```

Montajes clave:

- Configuración de Filebeat
    
- Dataset de logs
    

`:ro` → solo lectura → protege integridad de logs

---

### networks

```yml
networks:
  - soc-lab-net
```

Permite envío de datos a Elasticsearch.

---

## 6. Redes

```yml
networks:  
  soc-lab-net:  
   driver: bridge
```

Define una red interna Docker.

Propósito:

- Comunicación entre contenedores
    
- Aislamiento del host
    

---

## 7. Volúmenes

```yml
volumes:  
  esdata:
```

Define almacenamiento persistente para Elasticsearch.

Evita pérdida de datos al reiniciar contenedores.

---

## 8. Flujo Operativo del docker-compose

1. Se levanta Elasticsearch
    
2. Se valida con healthcheck
    
3. Kibana inicia cuando Elasticsearch está listo
    
4. Filebeat comienza a enviar logs
    

---

## 9. Consideraciones de Laboratorio

Este diseño prioriza:

- Simplicidad
    
- Visibilidad total del flujo
    
- Control manual de ingesta
    

No incluye:

- TLS
    
- Hardening
    
- Multi-node
    

---

## 10. Posibles Mejoras Futuras

- Habilitar HTTPS (xpack SSL)
    
- Separar nodos Elasticsearch
    
- Integrar Logstash para parsing avanzado
    
- Automatizar pipelines
    
