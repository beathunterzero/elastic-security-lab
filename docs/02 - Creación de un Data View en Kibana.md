## 1. Introducción al Data View

Un **Data View** (anteriormente conocido como _Index Pattern_) es el puente que permite a Kibana interpretar y visualizar los datos almacenados en Elasticsearch. Sin este paso, las herramientas de análisis como **Discover**, **Dashboards** o **Lens** no pueden realizar consultas de Threat Hunting.

---

## 2. Configuración Paso a Paso

### 2.1 Acceso Inicial

Con el laboratorio de Docker levantado, accede a la interfaz gráfica:

- **URL:** `http://localhost:5601`
    
- **Credenciales:** Usa tu usuario de analista (ej. `elastic` o `hunter`).
    

### 2.2 Navegación en el Stack Management

Para registrar la fuente de datos, sigue esta ruta en el menú lateral:

1. Entra a **Stack Management** (sección de administración).
    
2. Bajo el bloque de **Kibana**, selecciona **Data Views**.
    
3. Haz clic en el botón azul: **Create data view**.
    

### 2.3 Definición del Patrón de Índice

En el formulario de configuración, completa los campos clave con la siguiente lógica:

- **Name:** `elastic-security-lab`
    
    - _Explicación:_ Es el nombre amigable que verás en la interfaz.
        
- **Index pattern:** `filebeat-*`
    
    - _Explicación:_ El uso del comodín (`*`) es crítico. Indica a Kibana que debe agrupar todos los índices generados por Filebeat, permitiendo analizar de forma centralizada logs de:
        
        - `filebeat-firewall`
            
        - `filebeat-windows`
            
        - `filebeat-linux`
            
        - `filebeat-azure`
            
        - `filebeat-aws`
            
- **Timestamp field:** `@timestamp`
    
    - _Explicación:_ Selecciona el campo estándar que Filebeat utiliza para marcar la fecha y hora exacta de cada evento. Es el eje temporal para todas las gráficas y líneas de tiempo.
        

---

## 3. Verificación de Ingesta

Una vez creado el Data View, el siguiente paso es validar que los logs están fluyendo correctamente hacia el SIEM.

1. Ve a la sección **Discover** en el menú principal.
    
2. Asegúrate de seleccionar el Data View `elastic-security-lab` en el selector superior.
    
3. Si la configuración es correcta, verás el histograma de eventos y la lista de logs en tiempo real.
    

### 3.1 Filtros Rápidos de Auditoría

Puedes usar la barra de búsqueda para segmentar la vista por plataforma:

- `log_type : "windows"`
    
- `log_type : "firewall"`
    
- `log_type : "aws"`
    

---

## 4. Resolución de Problemas (Troubleshooting)

Si al entrar a **Discover** no aparecen datos ("No results match your search"), realiza el siguiente checklist técnico:

|**Componente**|**Acción de Verificación**|**Comando / URL**|
|---|---|---|
|**Contenedores**|Verificar que Filebeat y ES están activos.|`docker ps`|
|**Datasets**|Confirmar que hay archivos `.log` en las rutas.|`ls -l datasets/`|
|**Salud del Cluster**|Validar que Elasticsearch no esté bloqueado.|`http://localhost:9200/_cluster/health`|
|**Permisos**|Revisar que el usuario tenga roles de lectura.|Check en **Stack Management > Users**|

