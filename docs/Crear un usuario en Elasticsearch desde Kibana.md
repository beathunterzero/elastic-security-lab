# Iniciar sesión en Kibana

Una vez que el laboratorio está levantado:

```code
http://localhost:5601
```

Inicia sesión con las credenciales por defecto:
- **Usuario:** `elastic`
- **Contraseña:** `changeme`

<img width="677" height="636" alt="image" src="https://github.com/user-attachments/assets/d867d909-d5c5-43f0-ae1b-3b6d3611f640" />


> Este usuario tiene permisos completos y solo debe usarse para tareas administrativas.

# Ir al panel de administración de usuarios

En Kibana:
1. Abre el menú lateral izquierdo
2. Entra a **Stack Management**
3. Selecciona **Security**
4. Haz clic en **Users**

<img width="1597" height="839" alt="image" src="https://github.com/user-attachments/assets/07ba3c61-1ced-4244-9d1e-63bafb819fe3" />


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

<img width="1140" height="942" alt="image" src="https://github.com/user-attachments/assets/8e7a2c9d-0ae8-44ce-a62a-c646dd49adcc" />


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

<img width="1105" height="313" alt="image" src="https://github.com/user-attachments/assets/66219448-0208-4687-9d4c-3b79873b7097" />


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

<img width="627" height="700" alt="image" src="https://github.com/user-attachments/assets/48db24f2-d140-4fe9-9882-2a10f79af20d" />


Si todo está correcto, podrás:

- acceder a Discover
- visualizar dashboards
- crear Data Views
- analizar logs enviados por Filebeat

<img width="1902" height="1128" alt="image" src="https://github.com/user-attachments/assets/ad5981cf-e935-437b-99d4-ec347e161d59" />


# Buenas prácticas

- No uses `elastic` para tareas diarias
- No uses `kibana_system` para iniciar sesión (es un usuario interno)
- Crea un usuario por analista
- Usa contraseñas fuertes
- Documenta roles y permisos asignados
