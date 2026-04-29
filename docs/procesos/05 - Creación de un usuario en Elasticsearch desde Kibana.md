## 1. Introducción

En un entorno SIEM, el control de accesos es un componente crítico. El usuario `elastic` posee privilegios de **superusuario**, por lo que su uso debe limitarse a:

- Configuración inicial
    
- Administración del stack
    
- Mantenimiento
    

Para actividades de **Threat Hunting**, se deben crear usuarios con privilegios controlados siguiendo el principio de **mínimo privilegio (Least Privilege)**.

---

## 2. Objetivo del Proceso

Crear un usuario operativo (`hunter`) que permita:

- Consultar logs
    
- Ejecutar hunting en Discover
    
- Crear visualizaciones básicas
    

Sin exponer la integridad del cluster.

---

## 3. Acceso Administrativo

Acceder a Kibana con el usuario administrador:

URL:  
[http://localhost:5601](http://localhost:5601/)

Credenciales:

- Usuario: elastic
    
- Password: definido en docker-compose
    

---

## 4. Navegación en Kibana

Ruta para gestión de usuarios:

1. Ir a **Stack Management**
    
2. Sección **Security**
    
3. Seleccionar **Users**
    
4. Click en **Create user**
    

---

## 5. Creación del Usuario "hunter"

### 5.1 Campos de configuración

Completar:

- Username: hunter  
    Identificador del analista
    
- Password:  
    Definir contraseña robusta
    
- Full name: Threat Hunter  
    Útil para auditoría
    

---

## 6. Asignación de Roles (RBAC)

Asignar los siguientes roles:

---

### kibana_admin

Capacidad:

- Acceso completo a funcionalidades de Kibana
    

Uso en el lab:

- Crear Data Views
    
- Crear dashboards
    
- Navegación completa
    

---

### monitoring_user

Capacidad:

- Visualizar estado del cluster
    

Uso:

- Validar salud de Elasticsearch
    
- Revisar métricas
    

---

### viewer

Capacidad:

- Lectura de índices
    

Uso:

- Ejecutar queries en Discover
    
- Analizar logs
    

---

## 7. Análisis de Seguridad de Roles

Este conjunto de roles permite:

- Realizar Threat Hunting completo
    
- Evitar modificaciones críticas
    
- Proteger la configuración del cluster
    

No incluye:

- Gestión de índices
    
- Creación de roles
    
- Configuración de seguridad
    

---

## 8. Validación de Acceso

Procedimiento:

1. Cerrar sesión de elastic
    
2. Iniciar sesión con hunter
    
3. Verificar acceso a:
    

- Discover
    
- Data Views
    
- Dashboards
    

---

### Validación técnica

Confirmar:

- Visualización de logs
    
- Ejecución de filtros
    
- Acceso sin errores de permisos
    

---

## 9. Problemas Comunes

### No aparecen logs

Causa:

- Falta de permisos de lectura
    

Solución:

- Verificar rol viewer
    

---

### No se puede acceder a Discover

Causa:

- Falta de permisos Kibana
    

Solución:

- Verificar rol kibana_admin
    

---

### Error de autenticación

Causa:

- Password incorrecto
    
- Usuario no creado correctamente
    

Solución:

- Reconfigurar usuario desde elastic
    

---

## 10. Buenas Prácticas

- No usar `elastic` para operaciones diarias
    
- Crear un usuario por analista
    
- No compartir credenciales
    
- Revisar usuarios periódicamente
    
- Eliminar usuarios no utilizados
    

---

## 11. Usuarios del Sistema (Importante)

Usuarios como:

- kibana_system
    
- logstash_system
    
- beats_system
    

No deben usarse para login manual.

Propósito:

- Comunicación interna entre servicios
    

---

## 12. Consideraciones de Laboratorio

Este entorno:

- No tiene integración con LDAP/AD
    
- No usa MFA
    
- No tiene auditoría avanzada
    

Esto es aceptable porque:

- Es un entorno de aprendizaje
    
- Se prioriza simplicidad
    

---

## 13. Evolución Recomendada

A futuro se puede implementar:

- Roles personalizados por índice
    
- Separación por equipos
    
- Integración con Active Directory
    
- Auditoría de accesos
    
- MFA
    
