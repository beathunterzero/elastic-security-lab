# 🎯 elastic-security-lab

![Elasticsearch](https://img.shields.io/badge/Elasticsearch-8.17-blue)
![Kibana](https://img.shields.io/badge/Kibana-8.17-yellow)
![Filebeat](https://img.shields.io/badge/Filebeat-8.17-green)
![License](https://img.shields.io/badge/License-MIT-purple)

Elastic Security Lab es un laboratorio profesional diseñado para practicar Threat Hunting, ingestión de logs, análisis de datos y visualización usando Elasticsearch + Kibana + Filebeat.
Incluye arquitectura completa, datasets de entrenamiento y un pipeline de ingestión modular.

---

## 🚀 Requisitos

Antes de iniciar, asegúrate de tener instalado:

* Docker
* Docker Compose
* WSL2 (si estás en Windows)
* Git

---

## 📦 Instalación

Clona el repositorio:

```bash
git clone https://github.com/beathunterzero/elastic-security-lab.git
cd elastic-security-lab
```

---

## ⚙️ Configuración Inicial (IMPORTANTE)

Antes de levantar Kibana por primera vez, debes cambiar la contraseña del usuario `kibana_system` y colocarla en el `docker-compose.yml`.

Esto es obligatorio para que Kibana pueda autenticarse contra Elasticsearch.

---

## 🏗️ Levantar el laboratorio

Una vez configurado:

```bash
docker-compose up -d
```

Accede a Kibana:

```
http://localhost:5601
```

Inicia sesión con tu usuario personal (creado previamente en Kibana).

---

## 📂 Estructura del Proyecto

```
elastic-security-lab/
│
├── datasets/              # Logs de entrenamiento (AWS, Azure, Windows, Linux)
│
├── filebeat/
│   ├── filebeat.yml       # Configuración de ingestión
│   └── docker-compose.yml # Stack completo
│
├── docs/
│   └── architecture/      # Diagramas y documentación
│
├── .gitignore
├── docker-compose.yml
└── README.md
```

---

## 🛠️ Guía de Uso Rápido

Aquí tienes los escenarios más comunes para trabajar con el laboratorio.

### 1. Ingestar logs automáticamente

Coloca tus logs en la carpeta correspondiente:

```
datasets/windows/
datasets/linux/
datasets/aws/
datasets/azure/
```

Filebeat los detectará y enviará a Elasticsearch.

---

### 2. Crear Data Views en Kibana

Una vez que Filebeat haya enviado datos, crea un Data View:

* Stack Management
* Data Views
* Crear nuevo

Patrón sugerido:

```
filebeat-*
```

Esto te permitirá usar Discover, Dashboards y Lens.

(La guía detallada irá en docs/ más adelante.)

---

### 3. Crear tu usuario personal

Desde Kibana:

```
Stack Management → Security → Users → Create User
```

Roles recomendados para Threat Hunting:

* kibana_admin
* monitoring_user
* viewer

(La guía completa también irá en docs/.)

---

### 4. Reiniciar el stack

```bash
docker-compose down
docker-compose up -d
```

---

## 📖 Explicación de Componentes

| Componente    | Función                             |
| ------------- | ----------------------------------- |
| Elasticsearch | Almacenamiento y búsqueda de logs   |
| Kibana        | Visualización, dashboards, análisis |
| Filebeat      | Ingestión de logs desde datasets    |
| Datasets      | Logs públicos para entrenamiento    |

---

## 🧪 Datasets de Entrenamiento

Este laboratorio incluye datasets públicos para practicar:

* Windows Event Logs
* Linux auth logs
* AWS CloudTrail / GuardDuty
* Azure Activity / Sign-In
* Firewall logs

(No se incluyen logs privados ni sensibles.)

---

## 🛡️ Seguridad

Este proyecto está diseñado para uso educativo y de entrenamiento.
No contiene información privada ni datos reales de producción.

---

## 📜 Licencia

Este proyecto está bajo la licencia MIT.
Puedes usarlo, modificarlo y compartirlo libremente.

---

## 👤 Autor

Desarrollado por beathunterzero Entusiasta de la Caza de Amenazas