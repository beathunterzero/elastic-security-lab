## 1. Identificación del Caso (Ejemplo de Flujo de Trabajo)

- **ID de la Nota:** LAB-ELASTIC-REG-01.
    
- **Analista:** Rhodyn Ildefonso (@beathunterzero).
    
- **Entorno:** Elastic Security / Management.
    
- **Estado:** Operativo (Requiere aprobación del líder antes de activar).
    

---

## 2. Hipótesis y Objetivo Operativo

Una vez confirmada una hipótesis mediante **Threat Hunting**, el objetivo es automatizar la detección para liberar tiempo al analista.

- **Cyber Kill Chain:** Esta regla busca romper la cadena en la fase de **Persistencia (Persistence)** o **Instalación**, evitando que el atacante consolide su acceso mediante cuentas privilegiadas.
    
- **Modelo Diamante:** Protege el nodo **Víctima** monitoreando el abuso de la **Capacidad** de creación/modificación de cuentas locales.
    

---

## 3. Fuentes de Datos (Ingesta)

Para este caso, dependemos de la telemetría de Windows ingerida vía Filebeat:

- **Log Source:** `winlog` (Windows Event Logs).
    
- **ID Crítico:** `4732` (A member was added to a security-enabled local group).
    
- **Formato:** NDJSON parseado en el lab.
    

---

## 4. Desglose Técnico Profundo (La Regla)

### **A. Análisis Quirúrgico de la Query KQL**

La query de ejemplo utilizada para confirmar la hipótesis y crear la regla es:

`log_type : "windows" and winlog.event_id : "4732" and winlog.event_data.GroupName : "Administrators" and not user.name : ("SYSTEM" or "administrator" )`

- **`log_type : "windows"`**: Filtra el alcance de búsqueda solo a logs de Windows para optimizar el rendimiento del motor de búsqueda.
    
- **`winlog.event_id : "4732"`**: Se enfoca exclusivamente en el evento de "adición de un miembro a un grupo local con privilegios".
    
- **`winlog.event_data.GroupName : "Administrators"`**: Acota la alerta solo cuando el grupo afectado es el de Administradores.
    
- **`not user.name : ("SYSTEM" or "administrator")`**: **Exclusión de ruido.** Evita falsos positivos generados por procesos legítimos del sistema o la cuenta de administrador estándar, disparando la alerta solo ante comportamientos humanos o de atacantes no previstos.
    

### **B. Procedimiento en la Interfaz (Step-by-Step)**

1. **Acceso:** Cierre de sesión de usuario analista y login con cuenta administrativa (`elastic`).
    
2. **Navegación:** `Security` $\rightarrow$ `Rules` $\rightarrow$ `Detection rules (SIEM)`.
    
3. **Configuración de Lógica:**
    
    - **Create new rule** $\rightarrow$ **Custom query**.
        
    - Pegar la query KQL en el campo **Filter your data**.
        
4. **Metadatos de la Regla:**
    
    - **Título:** `T1136_creacion_de_usuario_con_privilegios_de_administrador`.
        
    - **Descripción:** Detecta la creación/adición de usuarios a grupos de administradores excluyendo cuentas de sistema estándar.
        
5. **Mapeo MITRE ATT&CK:**
    
    - **Táctica:** Persistence (TA0003).
        
    - **Técnica:** Create Account: Local Account (T1136.001).
        
6. **Severidad:** `Critical` (Dado que un administrador no autorizado tiene control total sobre el host).
    

---

## 5. Análisis y Conclusiones (Integración de Modelos)

Al activar esta regla, el SIEM asume la monitorización continua de la **Infraestructura** (Modelo Diamante). Se ha transformado un conocimiento táctico (TTP) de **MITRE ATT&CK** en una barrera tecnológica que detiene el avance del adversario en la **Cyber Kill Chain** antes de que pueda ejecutar acciones de exfiltración o destrucción de datos.

---

## 6. Recomendaciones y Mantenimiento

- **Aprobación Obligatoria:** No activar reglas en producción sin el permiso explícito del líder del SOC/CSIRT.
    
- **Tuning:** Si la regla genera muchos positivos legítimos (ej. por tareas de mantenimiento programadas), se deben añadir más exclusiones en la sección `not user.name`.
    
- **Mantenimiento:** Revisar trimestralmente si el nombre del grupo "Administrators" cambia por idioma (ej. "Administradores" en español) para ajustar la query.
    
