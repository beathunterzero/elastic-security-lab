# Creación de la estructura inicial del proyecto

Primero se creó el directorio raíz del laboratorio:

```bash
mkdir elastic-security-lab
cd elastic-security-lab
```

Luego se creó el directorio donde se almacenará la arquitectura del proyecto:

```bash
mkdir -p docs/architecture
```

En este directorio se diseñó el diagrama de arquitectura (`elastic-security-lab.excalidraw`) y se exportó como imagen (`elastic-security-lab.png`).

<img width="892" height="593" alt="image" src="https://github.com/user-attachments/assets/f93adac7-9e21-4dbf-bae4-16fa99be76e7" />


# Creación del resto de directorios

Se creó la estructura de datasets, organizada por plataforma:

```bash
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
```

También se creó el directorio para Filebeat:

```bash
mkdir filebeat
```

# Creación del archivo `docker-compose.yml`

A continuación se creó el archivo principal del stack:

```code
elastic-security-lab/docker-compose.yml
```

Y se añadió la siguiente configuración:

## Explicación línea por línea del `docker-compose.yml`

### Servicio: Elasticsearch (`es01`)

```yaml
es01:
  image: docker.elastic.co/elasticsearch/elasticsearch:8.17.10
```

Usa la imagen oficial de Elasticsearch versión 8.17.10, dicha imagen es LTS y es la mas estable en Abril 2026 para funcionamiento con WSL, las versiones 9.x.x tienen a tener problemas de compatibilidad, se hará una actualización del laboratorio cuando salga una nueva versión LTS de Elasticsearch


```yaml
  container_name: es01
```

Nombre del contenedor.


```yaml
  restart: unless-stopped
```

El contenedor se reinicia automáticamente salvo que se detenga manualmente.


```yaml
  environment:
    - node.name=es01
```

Nombre del nodo dentro del cluster.


```yaml
    - discovery.type=single-node
```

Modo de un solo nodo (ideal para laboratorios).


```yaml
    - xpack.security.enabled=true
```

Activa la seguridad (usuarios, contraseñas, roles).


```yaml
    - ELASTIC_PASSWORD=changeme
```

Contraseña inicial del usuario `elastic`.


```yaml
    - xpack.security.http.ssl.enabled=false
    - xpack.security.transport.ssl.enabled=false
```

Desactiva SSL para simplificar el laboratorio.


```yaml
    - ES_JAVA_OPTS=-Xms2g -Xmx2g
```

Asigna 2GB de RAM a Elasticsearch.


```yaml
  ulimits:
    memlock:
      soft: -1
      hard: -1
```

Evita problemas de bloqueo de memoria.


```yaml
  volumes:
    - esdata:/usr/share/elasticsearch/data
```

Persistencia de datos.


```yaml
  ports:
    - "9200:9200"
```

Expone Elasticsearch en localhost:9200.


```yaml
  networks:
    - soc-lab-net
```

Conecta el servicio a la red interna del lab.


```yaml
  healthcheck:
    test: ["CMD-SHELL", "curl -s -u elastic:changeme http://localhost:9200/_cluster/health | grep -E '\"status\":\"(yellow|green)\"'"]
```

Comprueba que Elasticsearch esté saludable antes de iniciar Kibana.

### Servicio: Kibana

```yaml
kibana:
  image: docker.elastic.co/kibana/kibana:8.17.10
```

Imagen oficial de Kibana.


```yaml
  depends_on:
    es01:
      condition: service_healthy
```

Kibana solo inicia cuando Elasticsearch está listo.


```yaml
  environment:
    - ELASTICSEARCH_HOSTS=http://es01:9200
```

Conexión hacia Elasticsearch.


```yaml
    - ELASTICSEARCH_USERNAME=kibana_system
    - ELASTICSEARCH_PASSWORD=dQ_WGNvbbT+agbHBXPbj
```

Credenciales del usuario interno de Kibana.


```yaml
    - SERVER_PUBLICBASEURL=http://localhost:5601
```

URL pública de Kibana.


```yaml
    - XPACK_ENCRYPTEDSAVEDOBJECTS_ENCRYPTIONKEY=acadaab8c49b160a7f2fad480683a25b
```

Clave necesaria para cifrar objetos guardados.


```yaml
  ports:
    - "5601:5601"
```

Expone Kibana en localhost:5601.

### Servicio: Filebeat

```yaml
filebeat:
  image: docker.elastic.co/beats/filebeat:8.17.10
```

Imagen oficial de Filebeat.


```yaml
  user: root
```

Permite leer archivos montados.


```yaml
  command: ["-e", "--strict.perms=false"]
```

Evita errores de permisos en entornos Docker.


```yaml
  volumes:
    - ./filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
    - ./datasets:/datasets:ro
```

Montaje del archivo de configuración y datasets.

# Creación del archivo `filebeat.yml`

Ubicado en:

```code
elastic-security-lab/filebeat/filebeat.yml
```

## Explicación línea por línea del `filebeat.yml`

### Inputs por tipo de log

Cada bloque:

```yaml
- type: filestream
  enabled: true
  id: firewall-input
  paths:
    - /datasets/firewall/*.log
```

Significa:
- `type: filestream` → Filebeat leerá archivos línea por línea.
- `paths:` → ruta dentro del contenedor.
- `fields:` → agrega metadatos personalizados.
- `fields_under_root: true` → los campos se agregan al nivel raíz del documento.
- `ignore_older: 0s` → nunca ignora archivos viejos.

Se repite para:
- firewall
- windows
- linux
- azure
- aws

Cada uno con sus rutas específicas.

### Output hacia Elasticsearch

```yaml
output.elasticsearch:
  hosts: ["http://es01:9200"]
  username: "elastic"
  password: "changeme"
```

Filebeat enviará los logs directamente a Elasticsearch.

### Configuración de Kibana

```yaml
setup.kibana:
  host: "http://kibana:5601"
  username: "elastic"
  password: "changeme"
```

Permite que Filebeat configure dashboards automáticamente.

### Processors

```yaml
processors:
  - add_host_metadata: ~
  - add_cloud_metadata: ~
  - add_docker_metadata: ~
  - add_kubernetes_metadata: ~
```

Agregan metadatos útiles para análisis.

# Levantar el laboratorio

```bash
docker-compose up -d
```

# Cambiar la contraseña del usuario `kibana_system`

```bash
docker exec -it es01 bin/elasticsearch-reset-password -u kibana_system
```

Copiar la contraseña generada y actualizarla en:

```code
docker-compose.yml
```

# Reiniciar todo

```bash
docker-compose down
docker-compose up -d
```

# Acceder a Kibana

```code
http://localhost:5601
```


