## 1. Introducción

En un entorno SIEM profesional, la segregación de funciones es crítica. El usuario `elastic` (Superuser) solo debe emplearse para la configuración inicial y el mantenimiento del stack. Para las actividades diarias de **Threat Hunting**, es necesario crear usuarios con privilegios limitados que reduzcan el riesgo de cambios accidentales en la infraestructura.

---

## 2. Acceso Administrativo

Antes de crear nuevos perfiles, se debe acceder con la cuenta raíz generada en el despliegue:

- **URL:** `http://localhost:5601`
    
- **Usuario:** `elastic`
    
- **Contraseña:** `changeme` (o la configurada en tu `docker-compose.yml`).
    

## 3. Configuración del Usuario de Análisis

Sigue esta ruta en la interfaz de Kibana para gestionar la seguridad:

1. Abre el menú lateral → **Stack Management**.
    
2. Busca la sección **Security** → **Users**.
    
3. Haz clic en el botón: **Create user**.
    

### 3.1 Definición del Perfil "Hunter"

Completa el formulario con los siguientes parámetros técnicos:

- **Username:** `hunter` (Nombre de usuario para el analista).
    
- **Password:** Define una credencial robusta.
    
- **Full name:** `Threat Hunter` (Identificador para logs de auditoría).
    

### 3.2 Asignación de Roles (RBAC)

Para un analista de CTH, se recomienda el principio de menor privilegio asignando los siguientes roles específicos:

|**Rol**|**Capacidad Técnica**|**Propósito en el Lab**|
|---|---|---|
|**`kibana_admin`**|Acceso total a las funciones de gestión de Kibana.|Crear Data Views y Dashboards.|
|**`monitoring_user`**|Permiso para ver el estado de salud del stack.|Verificar que ES y Filebeat están estables.|
|**`viewer`**|Lectura de datos en los índices.|Consultar logs en **Discover** y **Lens**.|

> **Nota de Seguridad:** Estos roles permiten realizar investigaciones completas sin otorgar permisos de `superuser`, protegiendo la integridad de los índices y la configuración del cluster.

---

## 4. Validación de Acceso

Tras guardar el usuario, es mandatorio realizar una prueba de _login_ para confirmar la propagación de permisos:

1. Cierra la sesión actual de `elastic`.
    
2. Ingresa con las credenciales de `hunter`.
    
3. Verifica el acceso a **Discover** y valida que puedes visualizar los logs de las plataformas (AWS, Azure, etc.) configuradas en el Data View.
    

## 5. Buenas Prácticas de Gestión de Identidades

- **Aislamiento de Cuentas:** No compartas el usuario `hunter`. Crea una identidad única para cada analista para mantener la trazabilidad de las investigaciones.
    
- **Usuarios de Sistema:** Nunca intentes loguearte manualmente con `kibana_system`, `logstash_system` o `beats_system`; estos son usuarios de servicio para comunicación interna.
    
- **Auditoría:** Revisa periódicamente la lista de usuarios activos y elimina aquellos que ya no sean necesarios tras finalizar un ejercicio de laboratorio.
    

