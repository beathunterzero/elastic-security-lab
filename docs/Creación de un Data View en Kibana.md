Un **Data View** en Kibana permite visualizar y consultar los datos que Filebeat envía a Elasticsearch. Sin este paso, no podrás usar **Discover**, **Dashboards**, **Lens** ni realizar análisis de Threat Hunting.

En este documento aprenderás a crear un Data View usando el patrón:

```code
filebeat-*
```

# Acceder a Kibana

Con el laboratorio levantado:

```code
http://localhost:5601
```

Inicia sesión con tu usuario (por ejemplo, `hunter`).

# Ir a la sección de Data Views

En el menú lateral izquierdo:

1. Entra a **Stack Management**
2. Selecciona **Data Views**
3. Haz clic en:

```code
Create data view
```

# Configurar el Data View

En el formulario, completa lo siguiente:
### Name

```code
elastic-security-lab
```

### Index pattern

```code
filebeat-*
```

Esto le indica a Kibana que debe incluir todos los índices generados por Filebeat, sin importar la plataforma:
- filebeat-firewall
- filebeat-windows
- filebeat-linux
- filebeat-azure
- filebeat-aws

Todos quedarán agrupados bajo este patrón.

# Seleccionar el campo de tiempo

En **Timestamp field**, selecciona:

```code
@timestamp
```

Este es el campo estándar que Filebeat usa para marcar la fecha y hora de cada evento.

# Crear el Data View

Haz clic en:

```code
Create data view
```

Kibana confirmará que el Data View fue creado correctamente.

# Verificar que los datos están llegando

Ahora ve a:

```code
Discover
```

Si todo está funcionando:
- verás eventos en tiempo real
- podrás filtrar por `log_type`
- podrás analizar logs de firewall, Windows, Linux, AWS y Azure

Ejemplos de filtros útiles:

```code
log_type : "windows"
log_type : "firewall"
log_type : "aws"
```

# ¿Qué pasa si no aparece ningún dato?

Revisa:
### Filebeat está corriendo

```code
docker ps
```

### Filebeat está leyendo los datasets

Los logs deben estar en:

```code
datasets/<plataforma>/
```

### Elasticsearch está saludable

```code
http://localhost:9200/_cluster/health
```

### El usuario tiene permisos

Debe tener al menos:
- viewer
- kibana_admin
- monitoring_user