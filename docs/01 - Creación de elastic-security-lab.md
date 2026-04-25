## 1. Introducción y Propósito

El **Elastic Security Lab** es un entorno SIEM (_Security Information and Event Management_) diseñado para centralizar, analizar y visualizar logs. Usamos la versión **8.17.10 (LTS)** por su estabilidad en **WSL2**.

## 2. Estructura de Directorios (FileSystem)

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

La organización es jerárquica para facilitar la escalabilidad del laboratorio:

Bash

```bash
elastic-security-lab/
├── datasets/            # Repositorio de logs crudos
│   ├── aws/             # Cloudtrail, Guardduty, S3, VPC
│   ├── azure/           # Activity, Audit, Firewall, Signin
│   ├── linux/           # Logs de sistemas Linux
│   ├── windows/         # Eventos de Windows
│   └── firewall/        # Tráfico de red
├── docs/                # Documentación y Diagramas
│   └── architecture/    # Diagramas Excalidraw y PNG
├── filebeat/            # Configuración del agente recolector
│   └── filebeat.yml
└── docker-compose.yml   # Orquestador del Stack
```


- `datasets/`: Repositorio organizado por plataforma (AWS, Azure, Linux, Windows, Firewall).
    
- `docs/architecture/`: Diagramas de diseño (`.excalidraw` y `.png`).
    
- `filebeat/`: Archivos de configuración del agente de ingesta.
    
- `docker-compose.yml`: El manifiesto que orquesta todo el stack.
    

---

## 3. Desglose Técnico: docker-compose.yml

A continuación, se explica la función de cada directiva configurada en el orquestador:

### **Servicio: Elasticsearch (es01)**

Es el corazón del stack; encargado del almacenamiento y el indexado de los datos.

- **`image: ...:8.17.10`**: Utiliza la imagen oficial de Elastic. La versión 8.x incluye el motor de seguridad habilitado por defecto.
    
- **`container_name: es01`**: Asigna un nombre estático al contenedor para facilitar la resolución DNS interna.
    
- **`environment`**:
    
    - **`node.name=es01`**: Define el nombre del nodo dentro del cluster.
        
    - **`discovery.type=single-node`**: Configura Elasticsearch para operar de forma independiente, eliminando las comprobaciones de quorum de un cluster multi-nodo.
        
    - **`xpack.security.enabled=true`**: Habilita el control de acceso basado en roles (RBAC).
        
    - **`ELASTIC_PASSWORD=changeme`**: Establece la contraseña inicial para el usuario `elastic`.
        
    - **`xpack.security.http.ssl.enabled=false`**: Desactiva el cifrado SSL para las conexiones HTTP. **Nota:** Solo se recomienda en entornos de laboratorio para simplificar la ingesta inicial.
        
    - **`ES_JAVA_OPTS=-Xms2g -Xmx2g`**: Asigna 2GB de memoria RAM mínima y máxima a la JVM. Esto evita que Java consuma memoria dinámicamente y degrade el host.
        
- **`ulimits.memlock`**: Establece límites `soft` y `hard` en `-1` (ilimitado). Esto permite que Elasticsearch bloquee su memoria en RAM y evita que el sistema operativo la mueva al área de intercambio (swap) en disco, lo que causaría latencias críticas.
    
- **`healthcheck`**: Ejecuta un comando `curl` cada 10 segundos. Si el cluster devuelve un estado `yellow` o `green`, el contenedor se marca como saludable. Esto es vital para que Kibana sepa cuándo conectarse.
    

### **Servicio: Kibana**

Interfaz visual para la interacción con los datos y la gestión del SIEM.

- **`depends_on.es01.condition: service_healthy`**: Directiva fundamental que impide que Kibana intente arrancar antes de que la base de datos esté lista para recibir peticiones.
    
- **`environment`**:
    
    - **`ELASTICSEARCH_HOSTS=http://es01:9200`**: Apunta al servicio interno de base de datos.
        
    - **`ELASTICSEARCH_USERNAME=kibana_system`**: Define el usuario de sistema necesario para la comunicación técnica.
        
    - **`XPACK_ENCRYPTEDSAVEDOBJECTS_ENCRYPTIONKEY`**: Llave de 32 caracteres necesaria para cifrar objetos guardados (como dashboards o credenciales) en la base de datos interna.
        
- **`ports: "5601:5601"`**: Mapea el puerto del contenedor al puerto local para acceso web vía navegador.
    

### **Servicio: Filebeat**

Agente ligero encargado de recolectar y enviar logs.

- **`user: root`**: Ejecuta el proceso con privilegios máximos para asegurar la lectura de cualquier log montado desde el host.
    
- **`command: ["-e", "--strict.perms=false"]`**:
    
    - `-e`: Envía la salida de logs a la consola de Docker.
        
    - `--strict.perms=false`: Ignora la verificación estricta de permisos en el archivo de configuración (necesario cuando se trabaja con sistemas de archivos montados desde Windows/WSL).
        
- **`volumes`**: Monta el archivo `filebeat.yml` y la carpeta `datasets` en modo Solo Lectura (`:ro`) para proteger la integridad de los logs crudos.
    

---

## 4. Desglose Técnico: filebeat.yml

Este archivo define la lógica de ingesta y normalización:

### **Sección: Inputs (Entradas de Datos)**

Cada bloque define un `id` único para evitar colisiones en los metadatos de Filebeat.

- **`type: filestream`**: Es el input moderno que reemplaza al antiguo `log`. Permite un mejor manejo de la rotación de archivos y el estado de lectura.
    
- **`paths`**: Utiliza comodines (`*.log`, `*.json`) para monitorizar dinámicamente nuevos archivos en los subdirectorios.
    
- **`parsers.ndjson`**:
    
    - **`target: ""`**: Indica que el JSON debe parsearse en la raíz del documento de Elastic, no dentro de un campo anidado.
        
    - **`overwrite_keys: true`**: Si hay campos duplicados en el JSON original, los valores del log prevalecen sobre los de Filebeat.
        
- **`fields`**: Añade etiquetas personalizadas (`log_type`, `platform`). Esto es crítico en CTH para filtrar rápidamente por origen de datos sin necesidad de realizar búsquedas de texto completo pesadas.
    
- **`ignore_older: 0s`**: Obliga a Filebeat a leer todos los archivos del directorio, sin importar cuándo fueron creados o modificados por última vez.
    

### **Sección: Output y Procesadores**

- **`output.elasticsearch`**: Configura el destino de los datos procesados, incluyendo las credenciales de acceso.
    
- **`processors`**:
    
    - **`add_host_metadata`**: Registra nombre de host, IP y OS del recolector.
        
    - **`add_cloud_metadata`**: Detecta automáticamente si el agente corre en una instancia de AWS/Azure/GCP.
        
    - **`add_docker_metadata`**: Añade ID del contenedor e imagen, fundamental para depurar fallos en el lab.
    

---

## 5. Guía de Ejecución y Mantenimiento

1. **Levantar:** `docker-compose up -d`.
    
2. **Seguridad:** Resetear la contraseña de Kibana para que los servicios se hablen correctamente:
    
    `docker exec -it es01 bin/elasticsearch-reset-password -u kibana_system`
    
3. **Persistencia:** Si cambias la contraseña, actualízala en el archivo `.yml` y reinicia con `docker-compose down && docker-compose up -d`.
    
4. **Acceso:** `http://localhost:5601`.
    
5. **Detener:** `docker-compose stop`

