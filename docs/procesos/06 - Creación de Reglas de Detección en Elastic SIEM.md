## 1. Introducción

Las **Reglas de Detección** en Elastic SIEM permiten transformar el resultado de un Threat Hunting manual en una **detección automática y continua**.

Este proceso representa la transición clave:

Hunting manual → Detección automatizada → Mejora del SOC

Sin este paso, el hunting no escala.

---

## 2. Objetivo del Proceso

Convertir una hipótesis validada en:

- Una alerta automática
    
- Un mecanismo de detección persistente
    
- Un caso de uso reutilizable
    

Esto permite:

- Reducir carga operativa
    
- Detectar amenazas en tiempo real
    
- Mejorar la madurez del SOC
    

---

## 3. Caso de Uso (Contexto del Lab)

ID: LAB-ELASTIC-REG-01  
Analista: Rhodyn Ildefonso  
Entorno: Elastic Security

Escenario:

Detección de **usuarios agregados al grupo Administrators**, lo cual indica:

- Escalamiento de privilegios
    
- Persistencia
    
- Compromiso potencial
    

---

## 4. Hipótesis Operativa

"Si un atacante compromete un host, intentará agregar cuentas al grupo Administrators para mantener control persistente."

---

## 5. Fuente de Datos

Origen:

- Logs de Windows (Filebeat)
    

Evento clave:

- Event ID 4732
    

Descripción:

"A member was added to a security-enabled local group"

---

## 6. Query de Detección (KQL)

log_type : "windows" and  
winlog.event_id : "4732" and  
winlog.event_data.GroupName : "Administrators" and  
not user.name : ("SYSTEM" or "administrator")

---

## 7. Explicación Técnica de la Query

### log_type : "windows"

- Filtra solo eventos de Windows
    
- Optimiza rendimiento
    

---

### winlog.event_id : "4732"

- Detecta adición de usuarios a grupos locales
    
- Evento crítico de seguridad
    

---

### winlog.event_data.GroupName : "Administrators"

- Limita la detección al grupo más crítico
    
- Reduce ruido
    

---

### not user.name : ("SYSTEM" or "administrator")

- Excluye cuentas legítimas del sistema
    
- Reduce falsos positivos
    

---

## 8. Procedimiento en Kibana

### Paso 1: Acceso

Ir a:

Security → Rules

---

### Paso 2: Crear Regla

Seleccionar:

Create new rule → Custom query

---

### Paso 3: Configurar Query

Pegar la query en:

Filter your data

---

### Paso 4: Configurar Metadatos

- Name: T1136_admin_group_modification
    
- Description: Detecta adición de usuarios a grupo Administrators
    
- Severity: Critical
    

---

### Paso 5: Configurar Frecuencia

Definir:

- Intervalo de ejecución (ej. cada 5 minutos)
    
- Lookback time (ej. 5-10 minutos)
    

---

### Paso 6: Mapeo MITRE ATT&CK

- Táctica: Persistence (TA0003)
    
- Técnica: Create Account (T1136.001)
    

---

### Paso 7: Guardar Regla

Guardar sin activar inicialmente.

---

## 9. Validación de la Regla

Antes de activar:

- Ejecutar la query manualmente en Discover
    
- Validar resultados reales
    
- Confirmar que no hay ruido excesivo
    

---

## 10. Activación Controlada

Buenas prácticas:

- Activar en modo prueba
    
- Monitorear alertas generadas
    
- Ajustar exclusiones si es necesario
    

---

## 11. Tuning de la Regla

Posibles mejoras:

- Incluir variaciones de idioma:
    
    - "Administrators"
        
    - "Administradores"
        
- Excluir cuentas de servicio específicas
    
- Agregar contexto:
    
    - hostname
        
    - proceso origen
        

---

## 12. Análisis Operativo

Esta regla permite:

- Detectar persistencia temprana
    
- Reducir dwell time
    
- Identificar abuso de privilegios
    

Impacto en Threat Hunting:

- Automatiza una hipótesis validada
    
- Libera tiempo para nuevas investigaciones
    

---

## 13. Relación con Modelos de Threat Hunting

### MITRE ATT&CK

- TA0003 → Persistence
    
- T1136 → Create Account
    

---

### Cyber Kill Chain

- Fase: Instalación / Persistencia
    

---

### Modelo Diamante

- Víctima → Host comprometido
    
- Capacidad → Manipulación de cuentas
    

---

## 14. Consideraciones de Laboratorio

- No hay correlación avanzada
    
- No hay SOAR integrado
    
- No hay alertas multi-evento
    

Esto es intencional para:

- Aprender lógica de detección
    
- Entender comportamiento base
    

---

## 15. Evolución Recomendada

A futuro:

- Correlación con logon events (4624)
    
- Integración con SOAR
    
- Alertas multi-condición
    
- Detección basada en comportamiento
    

---

## 16. Buenas Prácticas

- Nunca activar reglas sin validación
    
- Documentar cada regla creada
    
- Medir tasa de falsos positivos
    
- Revisar reglas periódicamente
    
