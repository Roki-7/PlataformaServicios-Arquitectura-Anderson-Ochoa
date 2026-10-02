# Análisis de Caso – Arquitectura de Software

## Proyecto

**Plataforma digital para la oferta, búsqueda y contratación de servicios técnicos, profesionales y de oficios en Ayacucho, 2026**

## 1. Arquitectura seleccionada

Para la plataforma se propone una **arquitectura de monolito modular con principios de arquitectura hexagonal**.

La solución se desarrollará inicialmente como una sola aplicación desplegable, pero estará organizada internamente en módulos con responsabilidades claramente definidas.

Esta decisión permite mantener una implementación inicial manejable, evitando la complejidad prematura de una arquitectura basada en microservicios, pero dejando preparada la solución para crecer y evolucionar en el futuro.

La arquitectura busca principalmente:

- Mantener separada la lógica del negocio de las tecnologías externas.
- Facilitar el mantenimiento del sistema.
- Reducir el acoplamiento entre módulos.
- Mejorar el rendimiento de las consultas frecuentes.
- Permitir el crecimiento progresivo de usuarios.
- Mantener disponibles las funcionalidades principales ante fallos parciales.
- Facilitar futuras integraciones.
- Permitir que una futura aplicación móvil utilice el mismo backend.

---

## 2. Objetivos arquitectónicos

Los principales objetivos de la arquitectura son:

1. Organizar la plataforma mediante módulos funcionales claramente definidos.
2. Mantener la lógica del negocio independiente de tecnologías como PostgreSQL, Redis o proveedores externos.
3. Permitir el crecimiento progresivo del sistema mediante escalamiento horizontal.
4. Mejorar el rendimiento mediante mecanismos de caché.
5. Evitar que una falla de servicios secundarios provoque la caída total de la plataforma.
6. Facilitar el mantenimiento, las pruebas y la incorporación de nuevas funcionalidades.

La solución se diseña considerando inicialmente hasta **20 000 usuarios registrados**.

La cantidad real de usuarios concurrentes que podrá soportar la plataforma deberá comprobarse posteriormente mediante pruebas de rendimiento y carga.

---

## 3. Drivers arquitectónicos

Los drivers arquitectónicos representan los factores que influyen directamente en las decisiones de diseño.

| Driver | Necesidad arquitectónica | Decisión relacionada |
|---|---|---|
| Escalabilidad | Permitir crecimiento progresivo de usuarios y solicitudes | Escalamiento horizontal |
| Disponibilidad | Evitar que una sola falla detenga toda la plataforma | Múltiples instancias y balanceo de carga |
| Rendimiento | Mantener búsquedas y consultas rápidas | Redis, índices y optimización de consultas |
| Resiliencia | Continuar operando ante fallos parciales | Desacoplamiento y procesamiento asíncrono |
| Seguridad | Proteger usuarios, datos y operaciones | Autenticación, autorización y HTTPS |
| Mantenibilidad | Facilitar cambios y nuevas funcionalidades | Monolito modular |
| Extensibilidad | Permitir nuevas integraciones y clientes | Puertos e interfaces |
| Integración | Conectar pagos, mapas y notificaciones | Adaptadores externos |

---

# 4. Vista general de la arquitectura

La solución se organiza en cinco zonas principales:

1. Usuarios del sistema.
2. Aplicación cliente.
3. API de entrada.
4. Núcleo del monolito modular.
5. Infraestructura y servicios externos.

```mermaid
flowchart TB

    subgraph ACTORES["USUARIOS DEL SISTEMA"]
        direction LR
        CLIENTE["Cliente"]
        PRESTADOR["Prestador"]
        ADMIN["Administrador"]
    end

    WEB["Aplicación Web<br/>React"]

    API["API REST<br/>NestJS"]

    subgraph SISTEMA["MONOLITO MODULAR"]
        direction TB

        ENTRADA["Adaptadores de entrada<br/>Controladores REST"]

        APLICACION["Aplicación<br/>Casos de uso"]

        DOMINIO["Dominio<br/>Reglas de negocio y entidades"]

        PUERTOS["Puertos e interfaces<br/>Arquitectura hexagonal"]

        ENTRADA --> APLICACION
        APLICACION --> DOMINIO
        DOMINIO --> PUERTOS
    end

    subgraph INFRA["INFRAESTRUCTURA"]
        direction LR
        REDIS[("Redis<br/>Caché")]
        POSTGRES[("PostgreSQL<br/>Base de datos")]
    end

    subgraph EXTERNOS["SERVICIOS EXTERNOS"]
        direction LR
        PAGO["Pasarela de pago"]
        MAPAS["Servicio de mapas"]
        NOTIFICACION["Correo y notificaciones"]
    end

    CLIENTE --> WEB
    PRESTADOR --> WEB
    ADMIN --> WEB

    WEB --> API
    API --> ENTRADA

    PUERTOS --> REDIS
    PUERTOS --> POSTGRES

    PUERTOS --> PAGO
    PUERTOS --> MAPAS
    PUERTOS --> NOTIFICACION


    classDef actor fill:#E3F2FD,stroke:#1565C0,color:#0D47A1,stroke-width:2px;
    classDef frontend fill:#E0F7FA,stroke:#00838F,color:#004D40,stroke-width:2px;
    classDef api fill:#EDE7F6,stroke:#5E35B1,color:#311B92,stroke-width:2px;
    classDef application fill:#FFF3E0,stroke:#FB8C00,color:#E65100,stroke-width:2px;
    classDef domain fill:#FFF8E1,stroke:#F9A825,color:#5D4037,stroke-width:2px;
    classDef ports fill:#F3E5F5,stroke:#8E24AA,color:#4A148C,stroke-width:2px;
    classDef infrastructure fill:#E8F5E9,stroke:#2E7D32,color:#1B5E20,stroke-width:2px;
    classDef external fill:#FCE4EC,stroke:#C2185B,color:#880E4F,stroke-width:2px;

    class CLIENTE,PRESTADOR,ADMIN actor;
    class WEB frontend;
    class API api;
    class ENTRADA,APLICACION application;
    class DOMINIO domain;
    class PUERTOS ports;
    class REDIS,POSTGRES infrastructure;
    class PAGO,MAPAS,NOTIFICACION external;

    style ACTORES fill:#F8FBFF,stroke:#90CAF9,stroke-width:2px
    style SISTEMA fill:#FFFDF8,stroke:#546E7A,stroke-width:2px
    style INFRA fill:#F5FBF6,stroke:#81C784,stroke-width:2px
    style EXTERNOS fill:#FFF8FA,stroke:#F48FB1,stroke-width:2px
```

### Interpretación

Los usuarios acceden inicialmente mediante una aplicación web desarrollada en React.

La aplicación web se comunica con el backend mediante una API REST.

Dentro del monolito modular se encuentran los casos de uso y las reglas principales del negocio.

La lógica del sistema no deberá depender directamente de PostgreSQL, Redis, la pasarela de pago, los mapas o los servicios de notificación.

Estas tecnologías serán utilizadas mediante puertos, interfaces y adaptadores.

---

# 5. Organización interna del monolito modular

La aplicación estará dividida internamente en módulos funcionales.

Cada módulo tendrá responsabilidades específicas y deberá evitar depender innecesariamente de la implementación interna de otros módulos.

```mermaid
flowchart TB

    subgraph IDENTIDAD["IDENTIDAD Y ACCESO"]
        direction LR

        USUARIOS["Usuarios y<br/>Autenticación"]
        PERFILES["Perfiles"]

        USUARIOS --> PERFILES
    end

    subgraph OFERTA["OFERTA Y DESCUBRIMIENTO"]
        direction LR

        CATALOGO["Catálogo de<br/>Servicios"]
        BUSQUEDA["Búsqueda y<br/>Ubicación"]

        CATALOGO --> BUSQUEDA
    end

    subgraph NEGOCIO["PROCESO PRINCIPAL DE CONTRATACIÓN"]
        direction LR

        SOLICITUDES["Solicitudes de<br/>Trabajo"]
        PROPUESTAS["Propuestas y<br/>Cotizaciones"]
        CONTRATACION["Contrataciones"]

        SOLICITUDES --> PROPUESTAS
        PROPUESTAS --> CONTRATACION
    end

    subgraph APOYO["SERVICIOS DE APOYO"]
        direction LR

        PAGOS["Pagos y<br/>Comisiones"]
        REPUTACION["Reputación y<br/>Calificaciones"]
        NOTIFICACIONES["Notificaciones"]
        ADMINISTRACION["Administración y<br/>Reportes"]
    end

    PERFILES --> CATALOGO
    BUSQUEDA --> SOLICITUDES

    CONTRATACION --> PAGOS
    CONTRATACION --> REPUTACION
    CONTRATACION --> NOTIFICACIONES
    ADMINISTRACION -. supervisa .-> CONTRATACION


    classDef identity fill:#E3F2FD,stroke:#1976D2,color:#0D47A1,stroke-width:2px;
    classDef discovery fill:#E0F2F1,stroke:#00897B,color:#004D40,stroke-width:2px;
    classDef business fill:#FFF3E0,stroke:#F57C00,color:#E65100,stroke-width:2px;
    classDef support fill:#F3E5F5,stroke:#7B1FA2,color:#4A148C,stroke-width:2px;

    class USUARIOS,PERFILES identity;
    class CATALOGO,BUSQUEDA discovery;
    class SOLICITUDES,PROPUESTAS,CONTRATACION business;
    class PAGOS,REPUTACION,NOTIFICACIONES,ADMINISTRACION support;

    style IDENTIDAD fill:#F7FBFF,stroke:#90CAF9,stroke-width:2px
    style OFERTA fill:#F5FCFB,stroke:#80CBC4,stroke-width:2px
    style NEGOCIO fill:#FFFBF6,stroke:#FFB74D,stroke-width:2px
    style APOYO fill:#FCF8FD,stroke:#CE93D8,stroke-width:2px
```

## 5.1 Usuarios y autenticación

Este módulo será responsable de:

- Registro de usuarios.
- Inicio y cierre de sesión.
- Recuperación de acceso.
- Gestión de roles.
- Gestión de permisos.

## 5.2 Perfiles

Gestionará información relacionada con:

- Clientes.
- Prestadores.
- Experiencia.
- Especialidades.
- Ubicación.
- Disponibilidad.
- Información profesional.

## 5.3 Catálogo de servicios

Será responsable de:

- Categorías.
- Subcategorías.
- Servicios publicados.
- Administración del catálogo.

## 5.4 Búsqueda y ubicación

Permitirá:

- Buscar prestadores.
- Buscar servicios.
- Aplicar filtros.
- Consultar ubicación.
- Utilizar información almacenada temporalmente en caché.

## 5.5 Solicitudes de trabajo

Permitirá que los clientes publiquen trabajos indicando información como:

- Descripción.
- Categoría.
- Ubicación.
- Fecha.
- Requerimientos adicionales.

## 5.6 Propuestas y cotizaciones

Permitirá a los prestadores enviar propuestas asociadas a las solicitudes publicadas.

Las propuestas podrán incluir:

- Precio.
- Descripción.
- Condiciones.
- Tiempo estimado.
- Disponibilidad.

## 5.7 Contrataciones

Este módulo controlará el proceso principal después de aceptar una propuesta.

Permitirá gestionar estados como:

- Contratado.
- En ejecución.
- Finalizado.
- Cancelado.

## 5.8 Pagos y comisiones

Gestionará:

- Registro de pagos.
- Estado del pago.
- Comisión de la plataforma.
- Integración con pasarelas externas.

## 5.9 Reputación y calificaciones

Permitirá registrar calificaciones y comentarios después de finalizar una contratación.

## 5.10 Notificaciones

Gestionará avisos relacionados con:

- Nuevas propuestas.
- Aceptación de propuestas.
- Cambios de estado.
- Finalización de trabajos.
- Eventos importantes.

## 5.11 Administración y reportes

Permitirá al administrador gestionar:

- Usuarios.
- Categorías.
- Publicaciones.
- Incidencias.
- Contenido reportado.
- Reportes administrativos.

---

# 6. Aplicación de arquitectura hexagonal

La arquitectura hexagonal tiene como objetivo mantener la lógica principal del sistema independiente de tecnologías externas.

Dentro del núcleo se encuentran las reglas del negocio y los casos de uso.

Las tecnologías externas se conectarán mediante adaptadores.

```mermaid
flowchart LR

    ENTRADA["Adaptador de entrada<br/>API REST"]

    CASO["Caso de uso<br/>Procesar contratación"]

    DOMINIO["Dominio<br/>Reglas del negocio"]

    PUERTO["Puerto<br/>Procesar pago"]

    ADAPTADOR["Adaptador de pago"]

    PROVEEDOR["Proveedor externo<br/>Pasarela de pago"]

    ENTRADA --> CASO
    CASO --> DOMINIO
    DOMINIO --> PUERTO
    PUERTO --> ADAPTADOR
    ADAPTADOR --> PROVEEDOR


    classDef entrada fill:#E3F2FD,stroke:#1976D2,color:#0D47A1,stroke-width:2px;
    classDef core fill:#FFF3E0,stroke:#FB8C00,color:#E65100,stroke-width:2px;
    classDef port fill:#F3E5F5,stroke:#8E24AA,color:#4A148C,stroke-width:2px;
    classDef adapter fill:#E8F5E9,stroke:#43A047,color:#1B5E20,stroke-width:2px;
    classDef external fill:#FCE4EC,stroke:#D81B60,color:#880E4F,stroke-width:2px;

    class ENTRADA entrada;
    class CASO,DOMINIO core;
    class PUERTO port;
    class ADAPTADOR adapter;
    class PROVEEDOR external;
```

Por ejemplo, la lógica central del sistema no debería depender directamente de una empresa específica de pagos.

El dominio solicitará una operación mediante un puerto como:

```text
ProcesarPago()
```

Un adaptador será responsable de comunicarse con el proveedor externo correspondiente.

Si posteriormente se cambia de proveedor, la lógica principal de contratación podrá mantenerse sin modificaciones significativas.

---

# 7. Vista de despliegue y escalabilidad

La primera versión del sistema podrá utilizar una sola instancia del backend.

Cuando aumente la demanda, podrán agregarse nuevas instancias del mismo monolito modular.

Un balanceador de carga distribuirá las solicitudes entre las instancias disponibles.

```mermaid
flowchart TB

    USERS["Usuarios"]

    INTERNET["Internet / HTTPS"]

    LB["Balanceador de carga"]

    subgraph BACKENDS["INSTANCIAS DE LA APLICACIÓN"]
        direction LR

        B1["Instancia 1<br/>Monolito modular"]
        B2["Instancia 2<br/>Monolito modular"]
        B3["Instancia 3<br/>Monolito modular"]
    end

    subgraph DATA["DATOS COMPARTIDOS"]
        direction LR

        REDIS[("Redis<br/>Caché")]
        DB[("PostgreSQL<br/>Base de datos")]
    end

    COLA["Cola de tareas<br/>Procesamiento asíncrono"]

    EXTERNOS["Servicios externos"]

    USERS --> INTERNET
    INTERNET --> LB

    LB --> B1
    LB --> B2
    LB --> B3

    B1 --> REDIS
    B2 --> REDIS
    B3 --> REDIS

    B1 --> DB
    B2 --> DB
    B3 --> DB

    B1 --> COLA
    B2 --> COLA
    B3 --> COLA

    COLA --> EXTERNOS


    classDef users fill:#E3F2FD,stroke:#1976D2,color:#0D47A1,stroke-width:2px;
    classDef network fill:#ECEFF1,stroke:#546E7A,color:#263238,stroke-width:2px;
    classDef balancer fill:#EDE7F6,stroke:#5E35B1,color:#311B92,stroke-width:2px;
    classDef backend fill:#FFF3E0,stroke:#FB8C00,color:#E65100,stroke-width:2px;
    classDef data fill:#E8F5E9,stroke:#388E3C,color:#1B5E20,stroke-width:2px;
    classDef async fill:#FFFDE7,stroke:#F9A825,color:#5D4037,stroke-width:2px;
    classDef external fill:#FCE4EC,stroke:#C2185B,color:#880E4F,stroke-width:2px;

    class USERS users;
    class INTERNET network;
    class LB balancer;
    class B1,B2,B3 backend;
    class REDIS,DB data;
    class COLA async;
    class EXTERNOS external;

    style BACKENDS fill:#FFF9F2,stroke:#FFB74D,stroke-width:2px
    style DATA fill:#F6FBF6,stroke:#81C784,stroke-width:2px
```

## Escalamiento horizontal

El crecimiento de la plataforma se realizará principalmente mediante **escalamiento horizontal**.

Esto significa que, en lugar de depender únicamente de un servidor más potente, podrán agregarse nuevas instancias de la aplicación cuando la demanda aumente.

Ejemplo:

```text
1 instancia
      ↓
aumenta la demanda
      ↓
2 instancias
      ↓
aumenta nuevamente
      ↓
3 o más instancias
```

El balanceador será responsable de distribuir las solicitudes entre las instancias disponibles.

---

# 8. Alta disponibilidad y resiliencia

La arquitectura busca reducir el riesgo de que una falla aislada provoque la caída completa de la plataforma.

Cuando exista más de una instancia, el balanceador podrá evitar enviar nuevas solicitudes hacia una instancia que no se encuentre disponible.

```mermaid
flowchart TB

    U["Usuarios"]

    LB["Balanceador de carga"]

    B1["Instancia 1<br/>Disponible"]

    B2["Instancia 2<br/>No disponible"]

    B3["Instancia 3<br/>Disponible"]

    SISTEMA["Servicio continúa disponible"]

    U --> LB

    LB --> B1
    LB -. "Health check fallido" .-> B2
    LB --> B3

    B1 --> SISTEMA
    B3 --> SISTEMA


    classDef user fill:#E3F2FD,stroke:#1976D2,color:#0D47A1,stroke-width:2px;
    classDef balancer fill:#EDE7F6,stroke:#5E35B1,color:#311B92,stroke-width:2px;
    classDef healthy fill:#E8F5E9,stroke:#2E7D32,color:#1B5E20,stroke-width:2px;
    classDef failed fill:#FFEBEE,stroke:#C62828,color:#B71C1C,stroke-width:2px;
    classDef result fill:#E0F2F1,stroke:#00796B,color:#004D40,stroke-width:2px;

    class U user;
    class LB balancer;
    class B1,B3 healthy;
    class B2 failed;
    class SISTEMA result;
```

La alta disponibilidad no significa que un sistema nunca pueda fallar.

El objetivo es reducir los puntos únicos de falla y permitir que las funciones principales continúen operando ante determinados fallos parciales.

También se aplicará resiliencia frente a servicios secundarios.

Por ejemplo, si temporalmente falla el servicio de correo, la búsqueda de prestadores no debería dejar de funcionar.

---

# 9. Estrategia de caché

Se propone utilizar **Redis** para almacenar temporalmente información consultada frecuentemente.

El flujo será el siguiente:

```mermaid
flowchart LR

    USER["Usuario"]

    APP["Aplicación"]

    CACHE{"¿Existe en<br/>Redis?"}

    REDIS[("Redis<br/>Caché")]

    DB[("PostgreSQL")]

    RESULT["Respuesta"]

    USER --> APP
    APP --> CACHE

    CACHE -->|"Sí"| REDIS
    REDIS --> RESULT

    CACHE -->|"No"| DB
    DB --> REDIS
    DB --> RESULT

    RESULT --> USER


    classDef user fill:#E3F2FD,stroke:#1976D2,color:#0D47A1,stroke-width:2px;
    classDef app fill:#FFF3E0,stroke:#FB8C00,color:#E65100,stroke-width:2px;
    classDef decision fill:#FFFDE7,stroke:#F9A825,color:#5D4037,stroke-width:2px;
    classDef cache fill:#E0F2F1,stroke:#00897B,color:#004D40,stroke-width:2px;
    classDef database fill:#E8F5E9,stroke:#2E7D32,color:#1B5E20,stroke-width:2px;
    classDef result fill:#F3E5F5,stroke:#8E24AA,color:#4A148C,stroke-width:2px;

    class USER user;
    class APP app;
    class CACHE decision;
    class REDIS cache;
    class DB database;
    class RESULT result;
```

## Datos candidatos para caché

Entre los datos que podrían almacenarse temporalmente se encuentran:

- Categorías.
- Subcategorías.
- Configuraciones generales.
- Información pública consultada frecuentemente.
- Determinados resultados reutilizables de búsqueda.

No toda la información deberá almacenarse en caché.

La información crítica o altamente cambiante deberá consultarse directamente desde la fuente correspondiente cuando sea necesario.

---

# 10. Procesamiento asíncrono

Algunas tareas no necesitan ejecutarse antes de responder al usuario.

Por ejemplo:

- Envío de correos.
- Algunas notificaciones.
- Generación de reportes.
- Procesos secundarios posteriores a una contratación.

Estas operaciones podrán utilizar procesamiento asíncrono.

```mermaid
flowchart LR

    APP["Aplicación"]

    EVENT["Evento<br/>Contratación creada"]

    QUEUE["Cola de tareas"]

    WORKER["Procesador"]

    EMAIL["Correo"]
    NOTIF["Notificación"]

    APP --> EVENT
    EVENT --> QUEUE
    QUEUE --> WORKER

    WORKER --> EMAIL
    WORKER --> NOTIF


    classDef app fill:#FFF3E0,stroke:#FB8C00,color:#E65100,stroke-width:2px;
    classDef event fill:#EDE7F6,stroke:#7E57C2,color:#311B92,stroke-width:2px;
    classDef queue fill:#FFFDE7,stroke:#F9A825,color:#5D4037,stroke-width:2px;
    classDef worker fill:#E0F2F1,stroke:#00897B,color:#004D40,stroke-width:2px;
    classDef external fill:#FCE4EC,stroke:#D81B60,color:#880E4F,stroke-width:2px;

    class APP app;
    class EVENT event;
    class QUEUE queue;
    class WORKER worker;
    class EMAIL,NOTIF external;
```

Esto permitirá evitar que operaciones secundarias aumenten innecesariamente el tiempo de respuesta de las operaciones principales.

---

# 11. Rendimiento

Como objetivo inicial, las búsquedas y consultas frecuentes deberán responder en un tiempo aproximado no mayor a **3 segundos en condiciones normales de operación**.

Las principales estrategias consideradas son:

- Redis como caché.
- Índices en PostgreSQL.
- Paginación de resultados.
- Optimización de consultas.
- Procesamiento asíncrono.
- Escalamiento horizontal.
- Monitoreo del rendimiento.

El rendimiento real deberá comprobarse posteriormente mediante pruebas de carga.

---

# 12. Seguridad

La arquitectura deberá proteger la información y las operaciones de los usuarios.

Se consideran las siguientes medidas:

- Autenticación mediante tokens.
- Autorización basada en roles y permisos.
- Almacenamiento seguro de contraseñas mediante hash.
- HTTPS para comunicaciones.
- Validación de datos de entrada.
- Rate limiting.
- Protección de endpoints.
- Registro de operaciones críticas.
- Control de acceso administrativo.
- Manejo seguro de secretos y credenciales.

---

# 13. Observabilidad y monitoreo

La plataforma deberá registrar información suficiente para identificar problemas de operación.

Se consideran:

- Registro de errores.
- Registro de eventos relevantes.
- Monitoreo del estado de las instancias.
- Health checks.
- Medición de tiempos de respuesta.
- Monitoreo del uso de recursos.
- Alertas ante fallos críticos.

La observabilidad permitirá detectar problemas antes de que afecten significativamente a los usuarios.

---

# 14. Persistencia de datos

Se propone **PostgreSQL** como base de datos principal.

Almacenará información relacionada con:

- Usuarios.
- Perfiles.
- Categorías.
- Servicios.
- Solicitudes.
- Propuestas.
- Contrataciones.
- Pagos.
- Comisiones.
- Calificaciones.
- Incidencias.
- Historial.

También deberán considerarse mecanismos de respaldo y recuperación de la información.

---

# 15. Servicios externos

La plataforma podrá integrarse con diferentes proveedores externos.

```mermaid
flowchart TB

    SISTEMA["Monolito modular"]

    PUERTOS["Puertos de salida"]

    subgraph SERVICIOS["SERVICIOS EXTERNOS"]
        direction LR

        PAGO["Pasarela de pago"]
        MAPAS["Mapas y ubicación"]
        CORREO["Correo electrónico"]
        NOTIF["Notificaciones"]
    end

    SISTEMA --> PUERTOS

    PUERTOS --> PAGO
    PUERTOS --> MAPAS
    PUERTOS --> CORREO
    PUERTOS --> NOTIF


    classDef core fill:#FFF3E0,stroke:#FB8C00,color:#E65100,stroke-width:2px;
    classDef ports fill:#F3E5F5,stroke:#8E24AA,color:#4A148C,stroke-width:2px;
    classDef external fill:#E3F2FD,stroke:#1976D2,color:#0D47A1,stroke-width:2px;

    class SISTEMA core;
    class PUERTOS ports;
    class PAGO,MAPAS,CORREO,NOTIF external;

    style SERVICIOS fill:#F8FBFF,stroke:#90CAF9,stroke-width:2px
```

Las integraciones deberán utilizar interfaces para evitar que la lógica principal dependa directamente de un proveedor específico.

---

# 16. Tecnologías propuestas

| Componente | Tecnología propuesta |
|---|---|
| Frontend | React |
| Backend | Node.js + NestJS |
| API | REST |
| Base de datos | PostgreSQL |
| Caché | Redis |
| Autenticación | JWT |
| Documentación API | Swagger / OpenAPI |
| Contenedores | Docker |
| Control de versiones | Git y GitHub |

Estas tecnologías representan una propuesta inicial.

La arquitectura deberá permitir sustituir determinados componentes tecnológicos sin modificar innecesariamente la lógica principal del sistema.

---

# 17. Decisiones arquitectónicas principales

## Monolito modular en lugar de microservicios

Se selecciona un monolito modular porque la primera versión del proyecto no necesita asumir desde el inicio la complejidad operativa de una arquitectura de microservicios.

La modularidad permitirá mantener separadas las responsabilidades y facilitar una posible evolución futura.

## Principios de arquitectura hexagonal

Se utilizan para mantener la lógica principal independiente de bases de datos, proveedores y frameworks externos.

## PostgreSQL

Se propone como base de datos principal por la naturaleza estructurada y relacional de gran parte de la información manejada por la plataforma.

## Redis

Se utiliza como mecanismo de caché para reducir consultas repetitivas y mejorar el tiempo de respuesta.

## API REST

Permitirá separar el frontend del backend y facilitar que posteriormente otros clientes, como una aplicación móvil, puedan utilizar los mismos servicios.

## Escalamiento horizontal

Permitirá incrementar la capacidad agregando nuevas instancias de la aplicación cuando la demanda aumente.

---

# 18. Riesgos arquitectónicos

| Riesgo | Impacto | Estrategia |
|---|---|---|
| Crecimiento inesperado de usuarios | Saturación del backend | Escalamiento horizontal |
| Exceso de consultas a la base de datos | Degradación del rendimiento | Redis e índices |
| Falla de una instancia | Pérdida parcial del servicio | Balanceador y múltiples instancias |
| Falla de notificaciones | Pérdida temporal de avisos | Procesamiento asíncrono |
| Falla de proveedor de pagos | Imposibilidad temporal de procesar pagos | Adaptadores desacoplados |
| Consultas de búsqueda costosas | Respuesta lenta | Caché, filtros, índices y paginación |
| Acoplamiento entre módulos | Mantenimiento complejo | Monolito modular |
| Dependencia tecnológica | Dificultad para cambiar proveedores | Arquitectura hexagonal |

---

# 19. Evolución futura

La arquitectura deberá permitir que el sistema evolucione progresivamente.

Entre las posibles evoluciones se consideran:

- Aplicación móvil.
- Nuevas zonas geográficas.
- Nuevas categorías de servicios.
- Recomendaciones personalizadas.
- Verificación avanzada de identidad.
- Nuevos proveedores de pago.
- Nuevos mecanismos de notificación.
- Extracción de módulos específicos si el crecimiento futuro lo justifica.

Una posible migración hacia microservicios no se realizará de forma anticipada.

Solo se considerará si existen necesidades técnicas u operativas reales que justifiquen aumentar la complejidad del sistema.

---

# 20. Conclusión

La arquitectura propuesta utiliza un **monolito modular con principios de arquitectura hexagonal**.

Esta combinación permite mantener una solución inicialmente manejable y al mismo tiempo proporcionar una estructura preparada para crecer.

La división interna en módulos facilita la mantenibilidad y reduce el acoplamiento.

La arquitectura hexagonal permite mantener la lógica principal separada de tecnologías externas.

Redis permitirá mejorar el rendimiento de información consultada frecuentemente, mientras que PostgreSQL actuará como almacenamiento principal.

El balanceo de carga y el escalamiento horizontal permitirán aumentar la capacidad cuando la demanda lo requiera.

Finalmente, la arquitectura busca que la plataforma pueda evolucionar de forma progresiva sin requerir un rediseño completo de su lógica principal.