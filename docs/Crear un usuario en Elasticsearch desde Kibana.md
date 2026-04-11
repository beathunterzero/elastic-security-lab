# Iniciar sesión en Kibana

Una vez que el laboratorio está levantado:

```code
http://localhost:5601
```

Inicia sesión con las credenciales por defecto:
- **Usuario:** `elastic`
- **Contraseña:** `changeme`

> Este usuario tiene permisos completos y solo debe usarse para tareas administrativas.

# Ir al panel de administración de usuarios

En Kibana:
1. Abre el menú lateral izquierdo
2. Entra a **Stack Management**
3. Selecciona **Security**
4. Haz clic en **Users**

Aquí verás la lista de usuarios del sistema.

# Crear un nuevo usuario

Haz clic en:

```code
Create user
```

Se abrirá un formulario con los siguientes campos:

### Completar los datos del usuario

- **Username:**

```code
hunter
```
  
- **Password:** Una contraseña segura, por ejemplo:

```code
MiClaveSegura123
```

- **Full name:**

```code
Threat Hunter
```

- **Email:** Opcional

# Asignar roles para Threat Hunting

En la sección **Roles**, selecciona:
- `kibana_admin`
- `monitoring_user`
- `viewer`

Estos roles permiten:

|Rol|Permiso|
|---|---|
|**kibana_admin**|Administrar Kibana sin privilegios críticos|
|**monitoring_user**|Ver métricas del stack|
|**viewer**|Ver dashboards, Discover, Lens|

> Estos roles son suficientes para un analista de Threat Hunting sin exponer permisos peligrosos como `superuser`.

# Guardar el usuario

Haz clic en:

```code
Create user
```

El usuario quedará registrado inmediatamente.

# Probar el acceso

Cierra sesión en Kibana y vuelve a entrar con:

- **Usuario:** `hunter`
- **Contraseña:** `MiClaveSegura123`

Si todo está correcto, podrás:

- acceder a Discover
- visualizar dashboards
- crear Data Views
- analizar logs enviados por Filebeat

# Buenas prácticas

- No uses `elastic` para tareas diarias
- No uses `kibana_system` para iniciar sesión (es un usuario interno)
- Crea un usuario por analista
- Usa contraseñas fuertes
- Documenta roles y permisos asignados