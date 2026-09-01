# Stack Tecnológico — Plataforma Web de Analítica Académica, Convivencia Escolar y Comunicación Automatizada

> Documento de soporte técnico del Proyecto APT. Describe cada tecnología elegida, su rol dentro de la arquitectura y la justificación de su selección, en coherencia con la factibilidad declarada en el documento 1.5 (herramientas de código abierto y tiers gratuitos, sin inversiones en licencias ni hardware).

---

## 1. Resumen del stack

| Capa | Tecnología | Rol en el proyecto |
|---|---|---|
| Plataforma web (backend) | **Laravel 12 (PHP 8.3)** | Sistema principal: auth, roles (Spatie), módulos académicos, scheduler y orquestación |
| Puente de integración | **Inertia.js** | Conector monolito moderno: pasa datos de controladores Laravel a React como props sin API REST manual |
| Frontend interactivo | **React + Tailwind CSS** | Interfaz SPA reactiva: componentes modulares por rol, interacción fluida y moderna |
| Visualización de datos | **Recharts / Chart.js (`react-chartjs-2`)** | Dashboards interactivos integrados en componentes React por perfil |
| Base de datos | **PostgreSQL 16** | Modelo relacional: alumnos, notas, asistencia, alertas y campos JSONB |
| Microservicio de minería de datos | **FastAPI (Python)** + **scikit-learn** | Modelo predictivo de riesgo de repitencia/deserción escolar |
| Generación de datos sintéticos | **Faker** (PHP/Python) + **pandas** | Miles de registros ficticios para entrenar y probar el sistema |
| Contenerización / orquestación | **Docker + Docker Compose** | Entorno reproducible: servicios backend, frontend (Vite) y microservicio ML |
| Comunicación automatizada | **WhatsApp Click to Chat** (wa.me) | Enlaces dinámicos con mensaje pre-escrito para contactar apoderados (incidentes y alertas de riesgo) |
| Asistente de IA | **API de IA con tier gratuito** (OpenAI/Gemini) | Recomendaciones de estudio personalizadas para apoderados |
| Permisos y roles | **Spatie Laravel Permission** | 4 perfiles: Apoderado, Profesor de Asignatura, Profesor Jefe, Administrador |
| Control de versiones y gestión ágil | **Git + GitHub + GitHub Projects** | Repositorio, ramas `feature/*`, Product Backlog y tablero Kanban (Scrum) |
| Hosting y despliegue en producción | **Hostinger VPS KVM 2** | Servidor privado en la nube para el stack Docker en producción |
| Testing automatizado | **Pest** (sobre PHPUnit) | Pruebas unitarias y de integración de los módulos críticos |
| Integración continua | **GitHub Actions** | Lint + tests automáticos en cada push a `dev` |

---

## 2. Arquitectura general

```
┌──────────────────────────────────────────────────────────────┐
│                    docker-compose.yml                        │
│                                                              │
│  ┌─────────────────────────────────┐        ┌─────────────┐  │
│  │           laravel-app           │  HTTP  │ fastapi-ml  │  │
│  │         PHP 8.3 + Nginx         │ ─────► │   (:8001)   │  │
│  │                                 │  JSON  │ Python +    │  │
│  │  ┌───────────────────────────┐  │ ◄───── │ scikit-learn│  │
│  │  │  Frontend React + Inertia │  │        └─────────────┘  │
│  │  │  (Tailwind + Recharts)    │  │               ▲         │
│  │  └───────────────────────────┘  │        ┌──────┴──────┐  │
│  │  - Auth & Roles (Spatie)        │        │ Generación  │  │
│  │  - Eloquent ORM / Scheduler     │        │ sintética   │  │
│  │  - HTTP Client (Guzzle)         │        │ (Faker/pd)  │  │
│  └────────────────┬────────────────┘        └─────────────┘  │
│                   │ Eloquent                                 │
│                   ▼                                          │
│  ┌─────────────────────────────────┐                         │
│  │           postgres:16           │                         │
│  └─────────────────────────────────┘                         │
└──────────────────────────────────────────────────────────────┘
           │
           │ Enlaces dinámicos wa.me +
           │ HTTP saliente (Guzzle, IA)
           ▼
   ┌─────────────────────┐    ┌─────────────────────┐
   │  WhatsApp wa.me     │    │  API de IA          │
   │  (Click to Chat, $0)│    │  (OpenAI/Gemini)    │
   └─────────────────────┘    └─────────────────────┘
```

**Patrón arquitectónico:** Monolito moderno para la aplicación web (Laravel + Inertia.js + React) desacoplado mediante microservicios HTTP/JSON para la minería de datos (FastAPI + scikit-learn). 
- Laravel nunca ejecuta código pesado de ML en PHP; delega la predicción al microservicio en Python.
- El frontend en React no requiere una API REST independiente con autenticación por tokens; Inertia.js permite entregar datos directamente a los componentes React desde los controladores de Laravel manteniendo sesiones seguras.

---

## 3. Justificación de cada tecnología

### 3.1 Laravel 12 (PHP) + Inertia.js + React — Plataforma web y Frontend Reactivo

**Por qué:**
- **Laravel 12 en Backend:** Framework robusto, maduro y de código abierto con soporte oficial activo. Resuelve de forma nativa autenticación (Laravel Breeze), autorización por roles (Spatie Permission), ORM (Eloquent) y programación de alertas nocturnas (Scheduler).
- **Inertia.js como puente ("El monolito moderno"):** Permite construir una aplicación Single Page Application (SPA) en React sin la complejidad añadida de crear APIs REST tradicionales, enrutadores de cliente o gestión de tokens JWT/CORS. Los controladores de Laravel devuelven componentes React con sus propiedades (`props`) de manera fluida y transparente.
- **React en Frontend:** Estándar líder de la industria en desarrollo de interfaces web. Permite crear componentes altamente interactivos, reutilizables y modulares para los 4 perfiles del sistema (dashboards de analítica, formularios de convivencia escolar, modales de confirmación y tablas de notas dinámicas).
- **Tailwind CSS:** Framework de estilos utilitarios que facilita un diseño responsive, limpio y profesional adaptado a paneles administrativos.
- **Valor curricular y profesional:** Desarrollar en React eleva el nivel técnico del proyecto Capstone y se alinea con la tecnología más solicitada en el mercado laboral chileno e internacional.

**Alternativas descartadas:**
- *Blade puro:* Descartado para aprovechar la reactividad, modularidad de componentes y dinamismo de React.
- *React SPA desacoplado con API REST tradicional:* Descartado por requerir el doble de trabajo (configurar CORS, endpoints JSON dobles y tokens Sanctum) innecesario para un equipo de 3 personas en un proyecto semestral.
- *Laravel Livewire:* Se evaluó, pero se prefirió React para aprovechar el ecosistema abierto de librerías de UI y potenciar el portafolio profesional del equipo.

### 3.2 PostgreSQL 16 — Base de datos

**Por qué:**
- 100% **gratis y de código abierto**, sin límites de uso (licencia PostgreSQL liberal).
- Superior a MySQL en el **perfil analítico** del proyecto: consultas agregadas complejas para los dashboards, funciones de ventana y tipos de datos avanzados (**JSONB** útil para guardar respuestas estructuradas del asistente de IA o configuraciones).
- Familiaridad conceptual previa del equipo con PostgreSQL.
- Con Eloquent ORM la integración es nativa y robusta.
- Mantener la BD local en contenedor Docker refuerza el argumento del informe sobre **privacidad de datos de menores**: nada sale de los servidores del proyecto.

**Alternativas descartadas:** MySQL (más limitado analíticamente); Supabase en nube (añade dependencia de internet, latencia y riesgo de pausa por inactividad).

### 3.3 FastAPI (Python) + scikit-learn — Microservicio de minería de datos

**Por qué:**
- **scikit-learn es explícitamente mencionado en el documento 1.5** (Parte I) como herramienta del proyecto; es la librería de ML de referencia en la academia.
- Python es el lenguaje dominante en ciencia de datos (pandas, scikit-learn, matplotlib).
- **FastAPI** permite exponer el modelo entrenado (`.joblib`) como un microservicio REST de alta velocidad con documentación automática OpenAPI/Swagger.
- **Arquitectura desacoplada Laravel ↔ FastAPI:** Si el modelo se re-entrena con nuevos datos, se actualiza el archivo `.joblib` en el microservicio sin modificar el código de la plataforma Laravel/React.
- El entrenamiento se realiza offline (metodología CRISP-DM) y solo el modelo final se pone en producción para minimizar el consumo de recursos.

### 3.4 Recharts / Chart.js (`react-chartjs-2`) — Dashboards y Visualización de datos

**Por qué:**
- Diseñadas para integrarse de forma natural y declarativa dentro del ciclo de vida de componentes React.
- Permite construir gráficos dinámicos, adaptables y estilizados con Tailwind CSS: líneas de tendencia de rendimiento escolar, barras comparativas de asistencia por curso y gráficos de dona para niveles de riesgo de deserción.
- Gratuito, liviano y con excelente experiencia de usuario.

### 3.5 Docker + Docker Compose — Contenerización

**Por qué:**
- Un solo archivo `docker-compose.yml` levanta el stack completo (Laravel + Node/Vite + PostgreSQL + FastAPI) con `docker compose up`: entorno idéntico en las máquinas de los 3 integrantes y en las demostraciones.
- Aísla los servicios en una red interna privada.
- Práctica profesional estándar de la industria y evidencia de ingeniería de software para el portafolio.

### 3.6 WhatsApp Click to Chat (wa.me) — Comunicación automatizada

**Por qué:**
- Función **pública y gratuita** de WhatsApp: el sistema genera dinámicamente la URL `https://wa.me/{número}?text={mensaje}` usando el teléfono del apoderado (extraído de PostgreSQL) y un mensaje pre-escrito con el contexto del incidente o de la alerta de riesgo.
- El profesor ve un botón interactivo **"Citar Apoderado"** en React: al hacer clic, abre WhatsApp Web o la app móvil con el chat y mensaje listo para enviar.
- **Costo $0 absoluto y sin requisitos de Meta:** no aplica cobros por mensaje, no exige verificación de negocio ni número dedicado.
- Registra trazabilidad en PostgreSQL (auditoría de qué docente inició el contacto y cuándo).

### 3.7 API de IA (tier gratuito) — Asistente para apoderados

**Por qué:**
- Integración vía llamada HTTP desde Laravel (OpenAI o Google Gemini en tier gratuito).
- Genera recomendaciones de estudio personalizadas según notas y asistencia del pupilo.
- **Privacidad:** se envían solo datos académicos agregados o identificadores sintéticos, nunca RUT ni datos personales sensibles de menores.

### 3.8 Git + GitHub + GitHub Projects — Gestión ágil y versionamiento

**Por qué:**
- Control de versiones colaborativo con ramas `feature/*` a `dev`.
- Tablero Kanban en GitHub Projects para gestión de Sprints bajo metodología Scrum.
- Historial de commits como evidencia del proceso formativo.

### 3.9 Spatie Laravel Permission — Roles y permisos

**Por qué:**
- Paquete estándar para administración de roles y permisos.
- Implementa los 4 perfiles del sistema (*Apoderado, Profesor de Asignatura, Profesor Jefe, Administrador*) conectados con la competencia de seguridad y administración de accesos del perfil de egreso.

### 3.10 Hostinger VPS KVM 2 — Hosting y despliegue en producción

**Por qué:**
- Servidor privado Linux con acceso root completo, capaz de ejecutar Docker, PostgreSQL, Python/FastAPI y Laravel en producción.
- Plan KVM 2 (2 vCPU, 8 GB RAM, 100 GB NVMe): recursos suficientes para stack multi-contenedor.
- Proxy inverso **Caddy** con SSL/TLS automático y gratuito (Let's Encrypt) → `https://plataforma.midominio.cloud`.
- Demuestra la competencia de administración de infraestructura, hardening de firewall (UFW) y continuidad operacional.

### 3.11 Pest — Testing automatizado

**Por qué:**
- Framework de testing moderno sobre PHPUnit con sintaxis declarativa y legible.
- Cobertura enfocada en rutas críticas: resolución de alertas, permisos por rol, escalamiento de incidentes y deduplicación.

### 3.12 GitHub Actions — Integración continua (CI)

**Por qué:**
- Verificación automática de estilo de código (`laravel/pint`) y ejecución de pruebas (`pest`) en cada push o pull request a `dev`.

---

## 4. Cumplimiento de la factibilidad declarada (documento 1.5, Parte I)

| Factor de factibilidad declarado | Cómo lo cumple el stack |
|---|---|
| Sin inversiones en licencias | 100% tecnologías de código abierto (Laravel, React, Inertia.js, Tailwind CSS, PostgreSQL, FastAPI, scikit-learn, Docker, Pest) o tiers gratuitos (API de IA, GitHub Actions, wa.me) |
| Sin inversiones en hardware | Desarrollo local con Docker; única inversión acotada en hosting de producción (Hostinger VPS KVM 2, ~$13.000 CLP por integrante al año) con dominio .cloud incluido |
| Ajuste a los 5 meses del semestre | Inertia.js agiliza el uso de React sin crear APIs REST dobles; Laravel resuelve auth y roles de fábrica |
| Datos sintéticos por normativa de menores | Faker + pandas generan los datos ficticios; BD local en contenedores bajo control del equipo |
| Metodología Scrum | GitHub Projects como tablero de Sprints y ramas `feature/*` a `dev` |
| Minería de datos con scikit-learn | Microservicio FastAPI con modelo entrenado offline bajo CRISP-DM |

---

## 5. Riesgos técnicos y mitigación

| Riesgo | Impacto | Mitigación |
|---|---|---|
| Curva de aprendizaje inicial con React + Inertia | Retraso en las primeras vistas | Uso del starter kit oficial Laravel Breeze (React + Inertia) que provee autenticación y estructura base lista; componentes modulares |
| Primera experiencia con Docker del equipo | Entorno base demora en quedar operativo | Semana 1–2 dedicada exclusivamente a configurar y verificar el `docker-compose.yml` entre los 3 integrantes |
| Datos sintéticos poco realistas → métricas pobres del modelo | Predicciones poco creíbles | Iteración conjunta generador ↔ entrenamiento; validación con estadística descriptiva previa |
| Tier gratuito de IA con límites de cuota | Asistente deja de responder | Control de caché de respuestas en BD y fallback con recomendaciones predeterminadas |
| Dependencia entre módulos (dashboards en React necesitan modelo ML) | Cuellos de botella | Desarrollo desacoplado: dashboards consumen datos académicos directos mientras se entrena el microservicio |
| Caída o indisponibilidad del VPS cerca de la entrega | Demo final en riesgo | Plan B documentado: demo local con `docker compose up` + video de respaldo grabado con anticipación |
| Escribir pruebas consume tiempo de los sprints | Velocidad de desarrollo reducida | Cobertura acotada a las rutas críticas del backend (alertas, roles, permisos) a cargo del rol QA |

---

*Documento elaborado como anexo técnico del Proyecto APT — Equipo: José Valenzuela, Felipe Verdugo, Felipe Catalán (Ing. Informática, sede Melipilla).*

