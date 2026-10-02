# Análisis de Caso – Arquitectura de Software

## Proyecto

**Plataforma digital para la oferta, búsqueda y contratación de servicios técnicos, profesionales y de oficios en Ayacucho, 2026**

## Estudiante

**Anderson Roki Ochoa Medrano**

## Descripción de la arquitectura

Para el proyecto se propone una **arquitectura de monolito modular con principios de arquitectura hexagonal**.

La plataforma se implementará inicialmente como una sola aplicación desplegable, pero estará organizada internamente en módulos independientes y con responsabilidades claramente definidas.

Esta organización permitirá mantener una estructura sencilla durante las primeras etapas del desarrollo, evitando la complejidad prematura de una arquitectura de microservicios, pero dejando preparado el sistema para crecer de manera progresiva.

Los principios de arquitectura hexagonal permitirán separar la lógica principal del sistema de elementos externos como la base de datos, la caché, los servicios de pago, los sistemas de notificaciones y los servicios de ubicación.

## Objetivo arquitectónico

El objetivo de la arquitectura es proporcionar una estructura que facilite el crecimiento, mantenimiento y evolución de la plataforma, permitiendo incorporar nuevas funcionalidades sin afectar innecesariamente los componentes existentes.

Asimismo, la arquitectura busca reducir el riesgo de que una falla parcial provoque la indisponibilidad total del sistema.

## Organización modular

La solución estará organizada principalmente en los siguientes módulos:

- Usuarios y autenticación.
- Perfiles de clientes y prestadores.
- Catálogo de servicios.
- Solicitudes de trabajo.
- Propuestas y cotizaciones.
- Contrataciones.
- Reputación y calificaciones.
- Pagos y comisiones.
- Notificaciones.
- Administración y reportes.
- Búsqueda y ubicación.

Cada módulo tendrá responsabilidades específicas, evitando concentrar toda la lógica de la aplicación en un único componente.

## Escalabilidad y disponibilidad

La arquitectura se plantea considerando el crecimiento progresivo de la plataforma.

Como referencia inicial, el sistema estará diseñado para gestionar hasta **20 000 usuarios registrados**, aunque la cantidad real de usuarios concurrentes deberá comprobarse posteriormente mediante pruebas de carga.

Cuando la demanda aumente, la aplicación podrá desplegarse en varias instancias y utilizar un balanceador de carga para distribuir las solicitudes.

También se contempla el uso de caché para reducir consultas repetitivas y mejorar el rendimiento de las operaciones de lectura frecuente.

## Componentes principales

La arquitectura estará conformada por una aplicación web, una API para la comunicación con el backend, los módulos del dominio, una base de datos principal, un mecanismo de caché y diferentes servicios externos.

Se propone utilizar **PostgreSQL** para la persistencia de información y **Redis** para el almacenamiento temporal de datos consultados frecuentemente.

Las integraciones externas, como pagos, mapas y notificaciones, se realizarán mediante interfaces desacopladas para reducir la dependencia de proveedores específicos.

## Atributos de calidad

La propuesta arquitectónica considera principalmente los siguientes atributos:

**Escalabilidad**, para permitir el crecimiento del sistema.

**Disponibilidad y resiliencia**, para mantener operativas las funciones principales ante fallos parciales.

**Rendimiento**, para reducir los tiempos de respuesta en búsquedas y consultas frecuentes.

**Seguridad**, para proteger cuentas, información personal y operaciones.

**Mantenibilidad**, para facilitar cambios y nuevas funcionalidades.

**Extensibilidad**, para permitir futuras integraciones y una posible aplicación móvil.

## Tecnologías propuestas

| Componente | Tecnología propuesta |
|---|---|
| Frontend | React |
| Backend | Node.js + NestJS |
| API | REST |
| Base de datos | PostgreSQL |
| Caché | Redis |
| Autenticación | JWT |
| Documentación de API | Swagger / OpenAPI |
| Contenedores | Docker |
| Control de versiones | Git y GitHub |

Las tecnologías indicadas constituyen una propuesta inicial y podrán ajustarse durante el desarrollo sin modificar los principios generales de la arquitectura.

## Análisis de Caso – Arquitectura

La propuesta arquitectónica del sistema se basa en un **monolito modular con principios de arquitectura hexagonal**, considerando escalabilidad, rendimiento, disponibilidad, resiliencia, seguridad y mantenibilidad.

El análisis completo de la arquitectura, sus componentes, decisiones de diseño y diagramas se encuentra en:

[Ver análisis de caso – arquitectura](arquitectura/arquitectura-inicial.md)

## Curso

**Arquitectura de Software (IS-488)**

## Docente

**Ing. Lizbeth Jaico Quispe**

## Semestre

**2026-II**

## Ubicación

**Ayacucho, Perú**