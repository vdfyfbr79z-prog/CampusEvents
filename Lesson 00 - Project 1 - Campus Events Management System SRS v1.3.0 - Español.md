# Sistema de Gestión de Eventos del Campus - Especificación de Requisitos de Software (ERS)

## Tabla de Contenidos

- [Control del Documento](#control-del-documento)
  - [Historial de Revisiones](#historial-de-revisiones)
- [1. Introducción](#1-introduccion)
  - [1.1 Propósito](#11-proposito)
  - [1.2 Alcance](#12-alcance)
  - [1.3 Fuera de Alcance / Mejoras Futuras](#13-fuera-de-alcance-mejoras-futuras)
  - [1.4 Audiencia Objetivo](#14-audiencia-objetivo)
  - [1.5 Definiciones y Términos](#15-definiciones-y-terminos)
  - [1.6 Convenciones de Requisitos](#16-convenciones-de-requisitos)
- [2. Descripción General del Producto](#2-descripcion-general-del-producto)
  - [2.1 Perspectiva del Producto](#21-perspectiva-del-producto)
  - [2.2 Capacidades Principales del Producto](#22-capacidades-principales-del-producto)
  - [2.3 Clases de Usuarios y Actores](#23-clases-de-usuarios-y-actores)
  - [2.4 Entorno de Operación](#24-entorno-de-operacion)
  - [2.5 Suposiciones y Dependencias](#25-suposiciones-y-dependencias)
- [3. Restricciones de Diseño](#3-restricciones-de-diseno)
  - [3.1 Tecnología y Arquitectura Requeridas](#31-tecnologia-y-arquitectura-requeridas)
  - [3.2 Límites Arquitectónicos](#32-limites-arquitectonicos)
- [4. Modelo Conceptual del Dominio](#4-modelo-conceptual-del-dominio)
  - [4.1 Entidades Principales](#41-entidades-principales)
- [5. Reglas de Negocio](#5-reglas-de-negocio)
  - [5.1 Reglas de Autenticación y Autorización](#51-reglas-de-autenticacion-y-autorizacion)
  - [5.2 Reglas de Eventos](#52-reglas-de-eventos)
  - [5.3 Reglas de Registro](#53-reglas-de-registro)
  - [5.4 Reglas de Categorías](#54-reglas-de-categorias)
- [6. Casos de Uso](#6-casos-de-uso)
  - [6.1 UC-001 — Explorar, Buscar y Ver Eventos](#61-uc-001-explorar-buscar-y-ver-eventos)
  - [6.2 UC-002 — Crear una Cuenta e Iniciar Sesión](#62-uc-002-crear-una-cuenta-e-iniciar-sesion)
  - [6.3 UC-003 — Registrarse en un Evento](#63-uc-003-registrarse-en-un-evento)
  - [6.4 UC-004 — Revisar y Cancelar Sus Propios Registros](#64-uc-004-revisar-y-cancelar-sus-propios-registros)
  - [6.5 UC-005 — Crear y Administrar un Evento Propio](#65-uc-005-crear-y-administrar-un-evento-propio)
  - [6.6 UC-006 — Revisar Registros para un Evento Propio](#66-uc-006-revisar-registros-para-un-evento-propio)
  - [6.7 UC-007 — Administrar Categorías y Acceso de Organizadores](#67-uc-007-administrar-categorias-y-acceso-de-organizadores)
  - [6.8 UC-008 — Realizar Supervisión Administrativa de Eventos](#68-uc-008-realizar-supervision-administrativa-de-eventos)
- [7. Requisitos Funcionales](#7-requisitos-funcionales)
  - [7.1 Cuentas y Autenticación](#71-cuentas-y-autenticacion)
  - [7.2 Descubrimiento, Búsqueda y Navegación de Eventos](#72-descubrimiento-busqueda-y-navegacion-de-eventos)
  - [7.3 Gestión de Eventos](#73-gestion-de-eventos)
  - [7.4 Registro de Eventos](#74-registro-de-eventos)
  - [7.5 Administración](#75-administracion)
- [8. Requisitos de Datos](#8-requisitos-de-datos)
  - [8.1 Información Requerida del Evento](#81-informacion-requerida-del-evento)
  - [8.2 Información Requerida del Registro](#82-informacion-requerida-del-registro)
  - [8.3 Expectativas de Integridad de Datos](#83-expectativas-de-integridad-de-datos)
- [9. Requisitos de Interfaces Externas](#9-requisitos-de-interfaces-externas)
  - [9.1 Interfaz de Usuario](#91-interfaz-de-usuario)
  - [9.2 Interfaz del Navegador](#92-interfaz-del-navegador)
  - [9.3 Interfaz de Autenticación](#93-interfaz-de-autenticacion)
  - [9.4 Servicios Externos](#94-servicios-externos)
- [10. Requisitos No Funcionales](#10-requisitos-no-funcionales)
  - [10.1 Seguridad](#101-seguridad)
  - [10.2 Rendimiento](#102-rendimiento)
  - [10.3 Usabilidad](#103-usabilidad)
  - [10.4 Accesibilidad](#104-accesibilidad)
  - [10.5 Fiabilidad y Corrección de Datos](#105-fiabilidad-y-correccion-de-datos)
  - [10.6 Compatibilidad](#106-compatibilidad)
- [11. Manejo de Errores y Comportamiento de Excepciones](#11-manejo-de-errores-y-comportamiento-de-excepciones)
- [12. Criterios de Aceptación / Definición de Hecho](#12-criterios-de-aceptacion-definicion-de-hecho)
- [13. Matriz de Trazabilidad de Requisitos](#13-matriz-de-trazabilidad-de-requisitos)
- [14. Mapa Conceptual del Curso a los Requisitos](#14-mapa-conceptual-del-curso-a-los-requisitos)

---

<a id="control-del-documento"></a>
## Control del Documento

**Versión del documento:** 1.3

La versión 1.0 es la línea base del semestre. Los cambios posteriores aprobados en los requisitos deben asociarse con un identificador de Solicitud de Cambio, como `CR-001`. Los identificadores de requisitos existentes normalmente deben mantenerse estables. Los requisitos retirados se marcan como retirados; sus identificadores no se reasignan silenciosamente a comportamientos diferentes.

<br>

<a id="historial-de-revisiones"></a>
### Historial de Revisiones

| Versión | Fecha | Autor | Revisor/Aprobador | Solicitud de Cambio | Resumen de Cambios |
| --- | --- | --- | --- | --- | --- |
| 1.0 | 2026‑08‑30 | HBV | HBV | — | Línea base inicial del semestre |
| 1.1 | 2026-09-01 | HBV | HBV | CR-001 | Se añadieron marcadores navegables |
| 1.2 | 2026-09-02 | HBV | HBV | CR-002 | Se añadió la TOC navegable |
| 1.3 | 2026-09-26 | HBV | HBV | CR-003 | Se agregó paginación del catálogo público de eventos (`FR-SRH-009`); se actualizó UC-001, alcance, interfaces, rendimiento, accesibilidad, manejo de errores, criterios de aceptación, trazabilidad y mapa conceptual del curso. |

---

<br>

<a id="1-introduccion"></a>
# 1. Introducción

<a id="11-proposito"></a>
## 1.1 Propósito

Esta Especificación de Requisitos de Software define el comportamiento requerido del **Sistema de Gestión de Eventos del Campus (CEMS)** para el Proyecto #1 del curso de Desarrollo Full-Stack.

CEMS es una aplicación web renderizada en servidor que permite a los miembros de la comunidad universitaria descubrir eventos, registrarse en ellos y gestionar su participación. Los organizadores autorizados pueden crear y gestionar eventos, mientras que los administradores pueden gestionar categorías de eventos, acceso de organizadores y problemas excepcionales de eventos.

Este documento es la referencia autorizada de lo que debe hacer la aplicación. Se espera que los estudiantes lo usen como hoja de ruta mientras toman decisiones de diseño e implementación durante todo el proyecto.

<a id="12-alcance"></a>
## 1.2 Alcance

El sistema soportará estas capacidades principales:

- navegación pública de próximos eventos del campus;
- búsqueda de eventos, filtrado por categoría y paginación de resultados del catálogo público;
- registro de cuenta, inicio de sesión y cierre de sesión;
- registro de participantes en eventos disponibles;
- cancelación por parte del participante de un registro existente;
- creación y gestión de eventos propios por parte del organizador;
- visualización de registros para eventos propios por parte del organizador;
- gestión por parte del administrador de categorías de eventos y acceso de organizadores;
- supervisión administrativa de eventos;
- almacenamiento persistente de los datos de la aplicación en una base de datos relacional.

El proyecto se enfoca intencionalmente en una aplicación web tradicional renderizada en servidor y multípágina. Funcionalidades como chat en tiempo real, notificaciones en vivo, gestión de estado SPA, procesamiento de pagos e integraciones externas avanzadas quedan fuera del alcance requerido.

<a id="13-fuera-de-alcance-mejoras-futuras"></a>
## 1.3 Fuera de Alcance / Mejoras Futuras

Las siguientes capacidades están intencionalmente **no requeridas** para el Proyecto #1. Podrían ser apropiadas para un proyecto posterior o trabajos futuros del curso.

- arquitectura de aplicación de una sola página;
- React u otro framework SPA para la interfaz de usuario principal;
- una API REST pública separada para la aplicación principal;
- chat en tiempo real;
- notificaciones push en vivo;
- indicadores de escritura, presencia u otras funciones de colaboración en tiempo real;
- listas de espera para eventos;
- procesamiento de pagos o entradas pagadas;
- entradas con código QR o escaneo de registro;
- sincronización con calendarios;
- notificaciones por correo electrónico o SMS;
- carga de imágenes o documentos;
- mapas o servicios de geolocalización;
- recomendaciones complejas de eventos;
- paneles analíticos avanzados;
- aislamiento multi-campus o multi-tenant;
- integración con identidad universitaria externa;
- aplicaciones nativas para móviles;
- historial de auditoría o sistemas de event sourcing;
- procesamiento de trabajos en segundo plano;
- infraestructura avanzada de caché;
- microservicios.

Estas exclusiones son controles de alcance intencionales. No se espera que los estudiantes las implementen para cumplir con el Proyecto #1.

<a id="14-audiencia-objetivo"></a>
## 1.4 Audiencia Objetivo

Este documento está dirigido principalmente a:

- estudiantes que implementan el Proyecto #1;
- el instructor del curso evaluando el proyecto;
- asistentes de enseñanza o revisores que apoyan el curso.

El documento especifica el comportamiento esperado del sistema. No prescribe decisiones de implementación detalladas, salvo cuando una elección tecnológica o arquitectónica es una restricción explícita de diseño del curso.

<a id="15-definiciones-y-terminos"></a>
## 1.5 Definiciones y Términos

| Término | Significado |
|---|---|
| **Visitante** | Una persona que usa la aplicación sin autenticarse. |
| **Participante** | Un usuario autenticado que puede registrarse en eventos del campus. |
| **Organizador** | Un usuario autenticado con permiso para crear y gestionar eventos que le pertenecen. |
| **Administrador** | Un usuario autenticado con permisos administrativos de la aplicación. |
| **Evento** | Una actividad programada del campus que los usuarios pueden ver y, cuando se permite, inscribirse para asistir. |
| **Registro** | La asociación actual que muestra que un participante ha reservado un lugar en un evento. |
| **Categoría** | Una etiqueta usada para clasificar eventos, como Académico, Social, Deportes o Carrera. |
| **Evento Publicado** | Un evento que ha sido hecho visible para visitantes y participantes. |
| **Evento Borrador** | Un evento que existe pero aún no ha sido publicado públicamente. |
| **Evento Cancelado** | Un evento que fue publicado pero ya no está programado para realizarse. |

<a id="16-convenciones-de-requisitos"></a>
## 1.6 Convenciones de Requisitos

Los requisitos obligatorios usan la palabra **shall** (deberá).

Los identificadores de requisitos usan estos prefijos:

- [`FR-ACC`](#accounts-and-authentication) — requisitos de cuentas y autenticación;
- [`FR-SRH`](#event-discovery-search-and-browsing) — requisitos de descubrimiento, búsqueda y navegación de eventos;
- [`FR-EVT`](#event-management) — requisitos de gestión de eventos;
- [`FR-REG`](#event-registration) — requisitos de registro de eventos;
- [`FR-ADM`](#administration) — requisitos administrativos;
- [`BR-*`](#business-rules) — reglas de negocio;
- [`NFR-*`](#non-functional-requirements) — requisitos no funcionales.

---

<a id="2-descripcion-general-del-producto"></a>
# 2. Descripción General del Producto

<a id="21-perspectiva-del-producto"></a>
## 2.1 Perspectiva del Producto

CEMS es una pequeña aplicación de eventos del campus diseñada para demostrar cómo funciona un producto web completo a través del navegador, la aplicación del servidor, la lógica de negocio, el sistema de autenticación, la base de datos relacional, el flujo de pruebas y el entorno desplegado.

La aplicación no está pensada para representar un sistema completo de información universitaria. No se integra con registros oficiales de estudiantes, matrículas, pagos ni servicios de identidad del campus.

<a id="22-capacidades-principales-del-producto"></a>
## 2.2 Capacidades Principales del Producto

El sistema completado permitirá a los usuarios avanzar a través de varios flujos completos:

1. Un visitante explora, busca, filtra y navega por resultados públicos de eventos paginados antes de abrir los detalles de un evento.
2. Un visitante crea una cuenta e inicia sesión.
3. Un participante se registra en un evento disponible.
4. Un participante revisa o cancela un registro existente.
5. Un organizador crea un evento, lo edita y lo publica.
6. Un organizador revisa a los participantes registrados en un evento propio.
7. Un administrador gestiona categorías y concede o revoca el acceso de organizador.
8. Un administrador puede intervenir cuando un evento requiere supervisión a nivel de aplicación.

Estos flujos están destinados a ejercitar varias capas de la aplicación en lugar de operaciones CRUD aisladas.

<a id="23-clases-de-usuarios-y-actores"></a>
## 2.3 Clases de Usuarios y Actores

| Actor | Capacidades Generales |
|---|---|
| **Visitante** | Explorar, buscar, filtrar, navegar por resultados públicos de eventos paginados y ver eventos publicados; crear una cuenta; iniciar sesión. |
| **Participante** | Todas las capacidades del visitante, además de registrarse en eventos, cancelar sus propios registros y ver sus registros. |
| **Organizador** | Todas las capacidades del participante, además de crear, editar, publicar, cancelar y gestionar registros para eventos propios. |
| **Administrador** | Supervisión a nivel de aplicación; gestionar categorías; conceder o retirar acceso de organizador; revisar y cancelar cualquier evento. |

Un único usuario autenticado puede tener capacidades de participante y además tener el rol de Organizador o Administrador.

<a id="24-entorno-de-operacion"></a>
## 2.4 Entorno de Operación

La aplicación se ejecutará como una aplicación web accesible a través de un navegador web moderno de escritorio o móvil. Los datos de la aplicación se almacenarán en una base de datos relacional. El sistema final se desplegará en un entorno aprobado por el instructor.

<a id="25-suposiciones-y-dependencias"></a>
## 2.5 Suposiciones y Dependencias

- Los usuarios tienen acceso a un navegador web moderno y a una conexión de red.
- El instructor puede proporcionar datos iniciales de prueba, como cuentas de administrador y categorías de eventos.
- La aplicación no verifica si una cuenta registrada pertenece a un estudiante o empleado real de la universidad.
- Los organizadores de eventos son usuarios de confianza del campus que reciben acceso de organizador a través de un administrador.
- El sistema usa una sola zona horaria de la aplicación elegida para el entorno del curso.

---

<a id="3-restricciones-de-diseno"></a>
# 3. Restricciones de Diseño

Esta sección identifica las restricciones de proyecto impuestas. No son requisitos funcionales.

<a id="31-tecnologia-y-arquitectura-requeridas"></a>
## 3.1 Tecnología y Arquitectura Requeridas

La aplicación deberá implementarse utilizando la tecnología aprobada por el curso:

- ASP.NET Core;
- C#;
- ASP.NET Core MVC;
- Razor Views;
- una arquitectura de aplicación multípágina renderizada en servidor;
- Entity Framework Core;
- una base de datos relacional; preferiblemente SQL Server para mantener la compatibilidad de integración al máximo;
- ASP.NET Core Identity o un mecanismo equivalente de autenticación en ASP.NET aprobado por el instructor;
- HTML y CSS; sin framework CSS (por ejemplo, Bootstrap, Tailwind);
- JavaScript del lado del cliente solo cuando proporcione un beneficio claro de usabilidad; sin framework JS (por ejemplo, jQuery, React, Vue, Angular);
- Git y GitHub para control de versiones;
- pruebas automatizadas para comportamientos críticos seleccionados;
- despliegue en un entorno aprobado por el instructor.

El flujo conceptual de la aplicación es:

```text
Navegador
   ↓
ASP.NET Core MVC
   ↓
Lógica de la Aplicación / Negocio
   ↓
Entity Framework Core
   ↓
Base de Datos Relacional
```

<a id="32-limites-arquitectonicos"></a>
## 3.2 Límites Arquitectónicos

- La aplicación principal deberá seguir siendo una MPA renderizada en servidor.
- No se requiere un frontend SPA separado.
- No se requiere una API REST separada para la aplicación principal.
- Patrones arquitectónicos avanzados como CQRS, MediatR, microservicios o Clean Architecture no son requeridos.
- Los estudiantes pueden organizar el código de la aplicación usando una separación razonable de responsabilidades adecuada al tamaño del proyecto.

---

<a id="4-modelo-conceptual-del-dominio"></a>
# 4. Modelo Conceptual del Dominio

Los siguientes conceptos de dominio representan el modelo mínimo de datos requerido. Esta sección describe la información conceptualmente y no prescribe clases de C# ni definiciones de tablas de base de datos.

<a id="41-entidades-principales"></a>
## 4.1 Entidades Principales

| Entidad | Propósito | Relaciones Importantes |
|---|---|---|
| **Usuario** | Representa a un usuario autenticado de la aplicación y sus roles de acceso asignados. | Un Usuario puede organizar muchos Eventos. Un Usuario puede registrarse en muchos Eventos a través de Registros. |
| **Categoría** | Clasifica los eventos en grupos simples y descubiertos. | Una Categoría puede clasificar muchos Eventos. Cada Evento pertenece a una Categoría. |
| **Evento** | Almacena la actividad programada del campus y su estado de publicación. | Cada Evento pertenece a un Usuario Organizador y a una Categoría. Un Evento puede tener muchos Registros. |
| **Registro** | Representa la plaza reservada actual de un participante en un Evento. | Cada Registro pertenece a un Usuario y a un Evento. |

---

<a id="5-reglas-de-negocio"></a>
# 5. Reglas de Negocio

<a id="51-reglas-de-autenticacion-y-autorizacion"></a>
## 5.1 Reglas de Autenticación y Autorización

| ID | Regla |
|---|---|
| <a id="br-aut-001"></a>[**BR&#8209;AUT&#8209;001**](#br-aut-001) | La gestión de registros, las funciones de organizador y las funciones de administrador requieren un usuario autenticado. |
| <a id="br-aut-002"></a>[**BR&#8209;AUT&#8209;002**](#br-aut-002) | Un organizador solo puede administrar los eventos que le pertenecen, a menos que el usuario también tenga acceso de Administrador. |
| <a id="br-aut-003"></a>[**BR&#8209;AUT&#8209;003**](#br-aut-003) | Las funciones administrativas están disponibles solo para usuarios con acceso de Administrador. |
| <a id="br-aut-004"></a>[**BR&#8209;AUT&#8209;004**](#br-aut-004) | La autorización deberá aplicarse en el servidor incluso cuando la interfaz de usuario oculte o desactive una acción no disponible. |

<a id="52-reglas-de-eventos"></a>
## 5.2 Reglas de Eventos

| ID | Regla |
|---|---|
| <a id="br-evt-001"></a>[**BR&#8209;EVT&#8209;001**](#br-evt-001) | La fecha y hora de inicio de un evento deben ser anteriores a su fecha y hora de finalización. |
| <a id="br-evt-002"></a>[**BR&#8209;EVT&#8209;002**](#br-evt-002) | Un evento en borrador solo puede publicarse cuando su fecha y hora de inicio están en el futuro. |
| <a id="br-evt-003"></a>[**BR&#8209;EVT&#8209;003**](#br-evt-003) | La capacidad del evento deberá ser un número entero mayor que cero. |
| <a id="br-evt-004"></a>[**BR&#8209;EVT&#8209;004**](#br-evt-004) | Cada evento debe hacer referencia a una categoría de evento existente. |
| <a id="br-evt-005"></a>[**BR&#8209;EVT&#8209;005**](#br-evt-005) | Normalmente solo se muestran en los resultados públicos de navegación de eventos los eventos publicados y próximos. |
| <a id="br-evt-006"></a>[**BR&#8209;EVT&#8209;006**](#br-evt-006) | Un evento publicado solo puede cancelarse antes de que comience. |
| <a id="br-evt-007"></a>[**BR&#8209;EVT&#8209;007**](#br-evt-007) | Un evento cancelado no podrá volver a estado Publicado en la versión requerida del proyecto. |
| <a id="br-evt-008"></a>[**BR&#8209;EVT&#8209;008**](#br-evt-008) | Un evento en borrador solo puede eliminarse permanentemente cuando no tiene registros actuales. |
| <a id="br-evt-009"></a>[**BR&#8209;EVT&#8209;009**](#br-evt-009) | Un organizador no puede cambiar la capacidad de participantes de un evento publicado a un valor inferior al número de registros actuales. |
| <a id="br-evt-010"></a>[**BR&#8209;EVT&#8209;010**](#br-evt-010) | Un evento cancelado ya no puede ser editado ni publicado por su organizador en la versión requerida del proyecto. |

<a id="53-reglas-de-registro"></a>
## 5.3 Reglas de Registro

| ID | Regla |
|---|---|
| <a id="br-reg-001"></a>[**BR&#8209;REG&#8209;001**](#br-reg-001) | Solo los participantes autenticados pueden mantener registros de eventos. |
| <a id="br-reg-002"></a>[**BR&#8209;REG&#8209;002**](#br-reg-002) | Un participante puede tener como máximo un registro actual para un evento concreto. |
| <a id="br-reg-003"></a>[**BR&#8209;REG&#8209;003**](#br-reg-003) | El número de registros actuales no debe exceder la capacidad del evento. |
| <a id="br-reg-004"></a>[**BR&#8209;REG&#8209;004**](#br-reg-004) | El registro solo se permite para un evento publicado que no haya comenzado ni sido cancelado. |
| <a id="br-reg-005"></a>[**BR&#8209;REG&#8209;005**](#br-reg-005) | Un participante solo puede cancelar su propio registro. |
| <a id="br-reg-006"></a>[**BR&#8209;REG&#8209;006**](#br-reg-006) | Al cancelar un registro, se elimina ese participante del recuento actual de registros del evento. |

<a id="54-reglas-de-categorias"></a>
## 5.4 Reglas de Categorías

| ID | Regla |
|---|---|
| <a id="br-cat-001"></a>[**BR&#8209;CAT&#8209;001**](#br-cat-001) | Los nombres de las categorías deben ser únicos sin tener en cuenta el caso de las letras. |
| <a id="br-cat-002"></a>[**BR&#8209;CAT&#8209;002**](#br-cat-002) | Una categoría referenciada por uno o más eventos no se puede eliminar. |

---

<a id="6-casos-de-uso"></a>
# 6. Casos de Uso

Esta sección describe las principales interacciones de extremo a extremo soportadas por CEMS. Los casos de uso corresponden a los flujos resumidos en la Sección 2.2 y organizan el comportamiento ya definido por las reglas de negocio, los requisitos funcionales y el comportamiento de errores en otras partes de esta especificación. No introducen requisitos adicionales. Cuando un resumen de caso de uso y un requisito enlazado difieren en nivel de detalle, el requisito enlazado es el autoritativo.

Los casos de uso usan identificadores en la forma `UC-###`.

<a id="61-uc-001-explorar-buscar-y-ver-eventos"></a>
## 6.1 UC-001 — Explorar, Buscar y Ver Eventos

**Objetivo:** Permitir a un visitante o usuario autenticado descubrir eventos publicados y próximos mediante exploración paginada, búsqueda y filtrado, y ver los detalles del evento.

**Actor principal:** Visitante o usuario autenticado.

**Precondiciones:** Ninguna.

**Disparador:** El usuario quiere descubrir o inspeccionar un evento del campus.

**Flujo principal:**

1. El usuario abre la interfaz de exploración de eventos.
2. El sistema muestra la primera página acotada de eventos publicados y próximos en orden cronológico por fecha y hora de inicio, salvo que exista otro ordenamiento compatible.
3. El usuario puede buscar usando texto que coincida al menos con el título o la ubicación del evento.
4. El usuario puede filtrar eventos por categoría. Un envío nuevo de búsqueda o filtro devuelve a la primera página de sus resultados coincidentes.
5. Cuando los resultados coincidentes abarcan varias páginas, el usuario puede navegar a otra página disponible; el sistema conserva el texto de búsqueda activo, los criterios de categoría, el orden cronológico y el recuento total de resultados coincidentes.
6. El usuario selecciona un evento publicado de la página mostrada.
7. El sistema muestra el título del evento, la descripción, la categoría, la ubicación, la fecha y hora de inicio y fin, el nombre del organizador, la información de capacidad y la disponibilidad actual de registros.

**Flujos alternativos:**

- **A3.1 / A4.1.** Si una búsqueda o un filtro por categoría no produce eventos coincidentes, el sistema muestra un mensaje claro de “sin resultados” sin enlaces de paginación.
- **A5.1.** Si una solicitud...

**Flujos de excepción:**

- **E1.1.** Si la página de resultados solicitada está fuera de rango, el sistema muestra la última página disponible o la primera si es inválida.
- **E5.1.** Si el evento seleccionado no existe o no está publicado, el sistema muestra una respuesta apropiada de no encontrado.

**Postcondiciones:** El usuario ha visto información válida del evento o una respuesta clara cuando no hay resultados.

**Requisitos relacionados:** [`FR-SRH-001`](#fr-srh-001), [`FR-SRH-002`](#fr-srh-002), [`FR-SRH-003`](#fr-srh-003), [`FR-SRH-004`](#fr-srh-004), [`FR-SRH-005`](#fr-srh-005), [`FR-SRH-006`](#fr-srh-006), [`FR-SRH-007`](#fr-srh-007), [`FR-SRH-009`](#fr-srh-009).

**Reglas de negocio relacionadas:** [`BR-EVT-005`](#br-evt-005), [`BR-EVT-006`](#br-evt-006).

<a id="62-uc-002-crear-una-cuenta-e-iniciar-sesion"></a>
## 6.2 UC-002 — Crear una Cuenta e Iniciar Sesión

**Objetivo:** Permitir a un visitante crear una cuenta de participante y autenticarse, o cerrar su sesión actual.

**Actor principal:** Visitante o usuario autenticado.

**Precondiciones:** Ninguna para la creación de cuenta; el usuario debe estar autenticado para cerrar sesión.

**Disparador:** El usuario quiere crear una cuenta o iniciar o finalizar una sesión.

**Flujo principal:**

1. El visitante abre el formulario de registro.
2. El visitante proporciona la información requerida para la cuenta.
3. El sistema valida la información.
4. El sistema crea la cuenta de participante y la asocia con la identidad de inicio de sesión proporcionada.
5. El visitante puede iniciar sesión con credenciales válidas.
6. El usuario autenticado puede cerrar la sesión actual.

**Flujos alternativos:**

- **A3.1.** Si un usuario no autenticado intenta realizar una operación protegida, el sistema le dirige al flujo de autenticación antes de permitir la operación.

**Flujos de excepción:**

- **E2.1.** Si la identidad de inicio de sesión proporcionada ya pertenece a una cuenta existente, el sistema rechaza la creación.
- **E3.1.** Si las credenciales de inicio de sesión son inválidas, el sistema rechaza la autenticación y muestra un mensaje genérico sin revelar si existe una cuenta específica.

**Postcondiciones:** Una cuenta de participante puede haber sido creada, puede establecerse una sesión autenticada o puede cerrarse una sesión autenticada existente, según el camino completado.

**Requisitos relacionados:** [`FR-ACC-001`](#fr-acc-001), [`FR-ACC-002`](#fr-acc-002), [`FR-ACC-003`](#fr-acc-003), [`FR-ACC-004`](#fr-acc-004), [`FR-ACC-005`](#fr-acc-005), [`FR-ACC-006`](#fr-acc-006), [`FR-ACC-007`](#fr-acc-007).

**Reglas de negocio relacionadas:** [`BR-AUT-001`](#br-aut-001), [`BR-AUT-004`](#br-aut-004).

<a id="63-uc-003-registrarse-en-un-evento"></a>
## 6.3 UC-003 — Registrarse en un Evento

**Objetivo:** Permitir a un participante autenticado reservar un sitio en un evento elegible.

**Actor principal:** Participante.

**Precondiciones:** El participante está autenticado y el evento seleccionado existe.

**Disparador:** El participante solicita registrarse en un evento.

**Flujo principal:**

1. El participante ve un evento para el que la inscripción está disponible.
2. El participante solicita registrarse.
3. El sistema verifica que el evento está Publicado, no ha comenzado, no ha sido cancelado y tiene capacidad disponible.
4. El sistema verifica que el participante aún no tiene un registro actual para ese evento.
5. El sistema crea el registro.
6. El sistema proporciona una confirmación clara de que el participante está registrado.

**Flujos alternativos:**

- **A2.1.** Un usuario no autenticado que intente registrarse debe autenticarse antes de que la operación protegida pueda continuar.

**Flujos de excepción:**

- **E4.1.** Si el participante ya tiene un registro actual para el evento, el sistema rechaza el registro duplicado e informa al participante.
- **E3.1.** Si el evento alcanza la capacidad antes de que termine el registro, el sistema rechaza la inscripción e informa al participante que no quedan plazas.
- **E3.2.** Si el evento está en borrador, cancelado, pasado o ya comenzó, el sistema rechaza la inscripción y explica que la inscripción no está disponible.

**Postcondiciones:** Si se realiza con éxito, un registro actual asocia al participante con el evento y la capacidad restante del evento refleja ese registro. En caso de fallo, no se crea ningún registro.

**Requisitos relacionados:** [`FR-REG-001`](#fr-reg-001), [`FR-REG-002`](#fr-reg-002), [`FR-REG-003`](#fr-reg-003), [`FR-REG-004`](#fr-reg-004), [`FR-REG-005`](#fr-reg-005), [`FR-ACC-006`](#fr-acc-006), [`FR-ACC-007`](#fr-acc-007).

**Reglas de negocio relacionadas:** [`BR-REG-001`](#br-reg-001), [`BR-REG-002`](#br-reg-002), [`BR-REG-003`](#br-reg-003), [`BR-REG-004`](#br-reg-004).

<a id="64-uc-004-revisar-y-cancelar-sus-propios-registros"></a>
## 6.4 UC-004 — Revisar y Cancelar Sus Propios Registros

**Objetivo:** Permitir a un participante autenticado revisar sus registros actuales y cancelar un registro elegible.

**Actor principal:** Participante.

**Precondiciones:** El participante está autenticado.

**Disparador:** El participante quiere revisar o cancelar uno de sus registros actuales.

**Flujo principal:**

1. El participante abre la lista de registros del participante.
2. El sistema muestra los registros actuales del participante en eventos.
3. El participante selecciona uno de sus registros para cancelarlo antes de que el evento comience.
4. El sistema verifica que el registro pertenece al participante y es elegible para cancelarse.
5. El sistema cancela el registro.
6. El sistema elimina a ese participante del recuento actual de registros del evento, dejando la plaza disponible para otro participante elegible.
7. El sistema ofrece una retroalimentación visible de éxito o fallo para la operación de cancelación.

**Flujos alternativos:**

- **A1.1.** Un usuario no autenticado debe autenticarse antes de que pueda gestionar registros.

**Flujos de excepción:**

- **E4.1.** El sistema rechaza un intento de cancelar el registro de otro participante.
- **E4.2.** El sistema rechaza la cancelación cuando el evento ya ha comenzado.

**Postcondiciones:** Si la cancelación tiene éxito, el participante ya no tiene ese registro actual y la plaza liberada queda disponible para otro participante elegible. Ver los registros sin cancelarlos no cambia los datos de la aplicación.

**Requisitos relacionados:** [`FR-REG-006`](#fr-reg-006), [`FR-REG-007`](#fr-reg-007), [`FR-REG-008`](#fr-reg-008), [`FR-ACC-006`](#fr-acc-006), [`NFR-USA-004`](#nfr-usa-004).

**Reglas de negocio relacionadas:** [`BR-REG-005`](#br-reg-005), [`BR-REG-006`](#br-reg-006), [`BR-AUT-001`](#br-aut-001).

<a id="65-uc-005-crear-y-administrar-un-evento-propio"></a>
## 6.5 UC-005 — Crear y Administrar un Evento Propio

**Objetivo:** Permitir a un organizador crear y gestionar eventos que le pertenecen a través del ciclo de vida de eventos soportado.

**Actor principal:** Organizador.

**Precondiciones:** El organizador está autenticado y tiene acceso de Organizador.

**Disparador:** El organizador quiere crear un nuevo evento o gestionar un evento propio existente.

**Flujo principal:**

1. El organizador abre la lista de gestión de eventos que le pertenecen.
2. El organizador crea un nuevo evento y proporciona la información requerida del evento: título, descripción, categoría, ubicación, fecha y hora de inicio y fin, y capacidad de participantes.
3. El sistema valida la información del evento y las reglas de negocio aplicables.
4. El sistema guarda el evento recién creado como borrador.
5. El organizador puede editar un evento propio mientras siga siendo elegible para modificación.
6. El organizador solicita publicar un evento borrador elegible.
7. El sistema verifica que el evento es elegible para publicación, incluyendo que su fecha y hora de inicio están en el futuro.
8. El sistema cambia el evento a Publicado y lo hace disponible a través de la navegación y búsqueda pública normales mientras sigue siendo próximo.

**Flujos alternativos:**

- **A5.1.** Un organizador puede cancelar un evento publicado propio solo antes de que comience. El registro del evento se conserva y su estado pasa a Cancelado.
- **A5.2.** Un evento en borrador puede eliminarse permanentemente solo cuando no tiene registros actuales.

**Flujos de excepción:**

- **E3.1 / E5.1.** Si faltan datos del evento requeridos o violan reglas de negocio aplicables, el sistema rechaza la creación o modificación y proporciona retroalimentación de validación.
- **E5.2 / E6.1.** Un evento cancelado no puede volver a estado Publicado y ya no puede ser editado ni publicado por su organizador en la versión requerida del proyecto.
- **E5.3.** El sistema rechaza un intento de reducir la capacidad de un evento publicado por debajo de su número de registros actuales.
- **E1.1 / E5.4.** Un organizador no puede gestionar un evento propiedad de otro organizador a menos que el usuario también tenga acceso de Administrador.

**Postcondiciones:** Dependiendo de la acción completada, un evento propio se persiste como Borrador, actualizado, Publicado, Cancelado o, cuando corresponde, eliminado. No se aplican cambios inválidos o no autorizados.

**Requisitos relacionados:** [`FR-EVT-001`](#fr-evt-001), [`FR-EVT-002`](#fr-evt-002), [`FR-EVT-003`](#fr-evt-003), [`FR-EVT-004`](#fr-evt-004), [`FR-EVT-005`](#fr-evt-005), [`FR-EVT-006`](#fr-evt-006), [`FR-EVT-007`](#fr-evt-007), [`FR-EVT-008`](#fr-evt-008), [`FR-EVT-009`](#fr-evt-009), [`FR-EVT-010`](#fr-evt-010), [`FR-EVT-011`](#fr-evt-011), [`FR-ACC-006`](#fr-acc-006).

**Reglas de negocio relacionadas:** [`BR-AUT-001`](#br-aut-001), [`BR-AUT-002`](#br-aut-002), [`BR-AUT-004`](#br-aut-004), [`BR-EVT-001`](#br-evt-001), [`BR-EVT-002`](#br-evt-002), [`BR-EVT-003`](#br-evt-003), [`BR-EVT-004`](#br-evt-004), [`BR-EVT-006`](#br-evt-006), [`BR-EVT-007`](#br-evt-007), [`BR-EVT-008`](#br-evt-008), [`BR-EVT-009`](#br-evt-009), [`BR-EVT-010`](#br-evt-010).

<a id="66-uc-006-revisar-registros-para-un-evento-propio"></a>
## 6.6 UC-006 — Revisar Registros para un Evento Propio

**Objetivo:** Permitir a un organizador revisar los participantes actuales y el estado de capacidad de un evento que le pertenece.

**Actor principal:** Organizador.

**Precondiciones:** El organizador está autenticado, tiene acceso de Organizador y el evento seleccionado es de su propiedad, a menos que el usuario también tenga acceso de Administrador.

**Disparador:** El organizador quiere inspeccionar los registros de un evento propio.

**Flujo principal:**

1. El organizador selecciona un evento propio desde la interfaz de gestión del organizador.
2. El organizador abre la vista de registros para ese evento.
3. El sistema muestra los participantes actuales registrados para el evento.
4. El sistema muestra el recuento actual de registros y la capacidad restante.

**Flujos alternativos:**

- Ninguno.

**Flujos de excepción:**

- **E1.1.** Si un organizador intenta acceder a la gestión de registros de un evento propiedad de otro organizador, el sistema rechaza la operación no autorizada a menos que el usuario también tenga acceso de Administrador.

**Postcondiciones:** El organizador ha revisado la información actual de registros del evento; no se modifica ningún dato de registro con este caso de uso.

**Requisitos relacionados:** [`FR-REG-009`](#fr-reg-009), [`FR-REG-010`](#fr-reg-010), [`FR-ACC-006`](#fr-acc-006).

**Reglas de negocio relacionadas:** [`BR-AUT-001`](#br-aut-001), [`BR-AUT-002`](#br-aut-002), [`BR-AUT-004`](#br-aut-004).

<a id="67-uc-007-administrar-categorias-y-acceso-de-organizadores"></a>
## 6.7 UC-007 — Administrar Categorías y Acceso de Organizadores

**Objetivo:** Permitir a un administrador mantener categorías de eventos y conceder o retirar acceso de Organizador a usuarios existentes.

**Actor principal:** Administrador.

**Precondiciones:** El administrador está autenticado y tiene acceso de Administrador.

**Disparador:** El administrador quiere cambiar permisos de organizador o mantener categorías de eventos.

**Flujo principal:**

1. El administrador accede a una función administrativa.
2. El administrador puede conceder o quitar acceso de rol Organizador a un usuario existente.
3. El administrador puede crear una nueva categoría de evento.
4. El administrador puede editar el nombre de una categoría existente cuando el nombre resultante es válido y único sin tener en cuenta el caso de las letras.
5. El administrador puede eliminar una categoría cuando ningún evento hace referencia actualmente a esa categoría.
6. El sistema persiste cada cambio administrativo válido.

**Flujos alternativos:**

- Ninguno.

**Flujos de excepción:**

- **E1.1.** El sistema rechaza las funciones administrativas para un usuario sin acceso de Administrador.
- **E3.1 / E4.1.** El sistema rechaza un nombre de categoría que violaría la regla de unicidad del nombre de categoría.
- **E5.1.** Si uno o más eventos hacen referencia a una categoría, el sistema rechaza la eliminación y explica que la categoría está en uso.

**Postcondiciones:** Los cambios válidos de rol de organizador o de categoría se persisten; los cambios inválidos o no autorizados no se aplican.

**Requisitos relacionados:** [`FR-ADM-001`](#fr-adm-001), [`FR-ADM-002`](#fr-adm-002), [`FR-ADM-005`](#fr-adm-005), [`FR-ADM-006`](#fr-adm-006), [`FR-ADM-007`](#fr-adm-007).

**Reglas de negocio relacionadas:** [`BR-AUT-003`](#br-aut-003), [`BR-AUT-004`](#br-aut-004), [`BR-CAT-001`](#br-cat-001), [`BR-CAT-002`](#br-cat-002).

<a id="68-uc-008-realizar-supervision-administrativa-de-eventos"></a>
## 6.8 UC-008 — Realizar Supervisión Administrativa de Eventos

**Objetivo:** Permitir a un administrador revisar eventos en toda la aplicación y cancelar cualquier evento publicado elegible cuando sea necesaria una intervención administrativa.

**Actor principal:** Administrador.

**Precondiciones:** El administrador está autenticado y tiene acceso de Administrador.

**Disparador:** El administrador necesita visibilidad global de eventos o necesita cancelar un evento elegible independientemente del organizador.

**Flujo principal:**

1. El administrador abre la lista de supervisión de eventos.
2. El sistema muestra todos los eventos independientemente del organizador o del estado.
3. El administrador selecciona un evento publicado que aún no ha comenzado.
4. El administrador solicita la cancelación del evento.
5. El sistema verifica que el evento sigue siendo elegible para cancelación.
6. El sistema cancela el evento conservando el registro del evento y su estado Cancelado.

**Flujos alternativos:**

- Ninguno.

**Flujos de excepción:**

- **E5.1.** El sistema rechaza la cancelación del evento cuando este no es un evento publicado elegible o ya ha comenzado.
- **E1.1.** El sistema rechaza las operaciones de supervisión administrativa para un usuario sin acceso de Administrador.
- **E6.1.** Un evento cancelado no vuelve a estado Publicado en la versión requerida del proyecto.

**Postcondiciones:** En caso de cancelación exitosa, el evento permanece persistido con estado Cancelado y ya no está disponible para nuevos registros. Ver la lista de supervisión sin cancelar un evento no cambia los datos de la aplicación.

**Requisitos relacionados:** [`FR-ADM-001`](#fr-adm-001), [`FR-ADM-003`](#fr-adm-003), [`FR-ADM-004`](#fr-adm-004), [`FR-EVT-010`](#fr-evt-010).

**Reglas de negocio relacionadas:** [`BR-AUT-003`](#br-aut-003), [`BR-AUT-004`](#br-aut-004), [`BR-EVT-006`](#br-evt-006), [`BR-EVT-007`](#br-evt-007).

---

<a id="7-requisitos-funcionales"></a>
# 7. Requisitos Funcionales

<a id="71-cuentas-y-autenticacion"></a>
## 7.1 Cuentas y Autenticación

| ID | Título | Requisito |
|---|---|---|
| <a id="fr-acc-001"></a>[**FR&#8209;ACC&#8209;001**](#fr-acc-001) | Registro de cuenta | El sistema deberá permitir a un visitante crear una cuenta de participante usando la información requerida para la cuenta. |
| <a id="fr-acc-002"></a>[**FR&#8209;ACC&#8209;002**](#fr-acc-002) | Identidad única de la cuenta | El sistema rechazará la creación de una cuenta cuando la identidad de inicio de sesión proporcionada ya esté asociada a una cuenta existente. |
| <a id="fr-acc-003"></a>[**FR&#8209;ACC&#8209;003**](#fr-acc-003) | Inicio de sesión | El sistema deberá permitir a un usuario registrado autenticarse con credenciales válidas. |
| <a id="fr-acc-004"></a>[**FR&#8209;ACC&#8209;004**](#fr-acc-004) | Inicio de sesión inválido | El sistema rechazará los intentos de inicio de sesión inválidos sin revelar si existe una cuenta específica. |
| <a id="fr-acc-005"></a>[**FR&#8209;ACC&#8209;005**](#fr-acc-005) | Cierre de sesión | El sistema deberá permitir a un usuario autenticado finalizar la sesión autenticada actual. |
| <a id="fr-acc-006"></a>[**FR&#8209;ACC&#8209;006**](#fr-acc-006) | Operaciones protegidas | El sistema deberá requerir autenticación antes de que un usuario pueda registrarse en un evento, cancelar un registro o usar capacidades de organizador o administrador. |
| <a id="fr-acc-007"></a>[**FR&#8209;ACC&#8209;007**](#fr-acc-007) | Flujo de retorno al iniciar sesión | Cuando un visitante intenta acceder a una página que requiere autenticación, el sistema deberá proporcionar un flujo de inicio de sesión que permita al usuario volver a la página originalmente solicitada tras una autenticación correcta cuando sea práctico. |

<a id="72-descubrimiento-busqueda-y-navegacion-de-eventos"></a>
## 7.2 Descubrimiento, Búsqueda y Navegación de Eventos

| ID | Título | Requisito |
|---|---|---|
| <a id="fr-srh-001"></a>[**FR&#8209;SRH&#8209;001**](#fr-srh-001) | Explorar eventos próximos | El sistema deberá permitir a visitantes y usuarios autenticados ver una lista de eventos publicados y próximos. |
| <a id="fr-srh-002"></a>[**FR&#8209;SRH&#8209;002**](#fr-srh-002) | Ordenación de eventos | El sistema deberá presentar los eventos próximos en orden cronológico por fecha y hora de inicio del evento a menos que el usuario aplique otra búsqueda o filtro compatible. |
| <a id="fr-srh-003"></a>[**FR&#8209;SRH&#8209;003**](#fr-srh-003) | Detalles del evento | El sistema deberá permitir a los usuarios abrir una página de detalles para un evento publicado. |
| <a id="fr-srh-004"></a>[**FR&#8209;SRH&#8209;004**](#fr-srh-004) | Información del detalle del evento | La página de detalle del evento deberá mostrar el título, la descripción, la categoría, la ubicación, la fecha y hora de inicio y fin, el nombre del organizador, la información de capacidad y la disponibilidad actual de registros. |
| <a id="fr-srh-005"></a>[**FR&#8209;SRH&#8209;005**](#fr-srh-005) | Buscar eventos | El sistema deberá permitir a los usuarios buscar eventos publicados y próximos usando texto que coincida al menos con el título o la ubicación del evento. |
| <a id="fr-srh-006"></a>[**FR&#8209;SRH&#8209;006**](#fr-srh-006) | Filtrar por categoría | El sistema deberá permitir a los usuarios filtrar eventos publicados y próximos por categoría. |
| <a id="fr-srh-007"></a>[**FR&#8209;SRH&#8209;007**](#fr-srh-007) | Comentario de “sin resultados” | Cuando una búsqueda o filtro no produce eventos coincidentes, el sistema deberá mostrar un mensaje claro de “sin resultados” en lugar de una página vacía y sin explicación. |
| <a id="fr-srh-008"></a>[**FR&#8209;SRH&#8209;008**](#fr-srh-008) | Visibilidad de eventos cancelados | Un evento publicado previamente que ha sido cancelado deberá permanecer visible a través de un enlace directo y deberá identificarse claramente como cancelado, pero no aparecer como un evento disponible para nuevos registros. |
| <a id="fr-srh-009"></a>[**FR&#8209;SRH&#8209;009**](#fr-srh-009) | Exploración paginada de eventos | El sistema deberá dividir los resultados de búsqueda, filtrado por categoría y exploración pública de eventos publicados/próximos en páginas acotadas. Cuando corresponda, deberá proporcionar navegación Anterior y Siguiente, identificar la página actual y el recuento total de resultados coincidentes, y conservar el texto de búsqueda activo, la selección de categoría y el orden cronológico mientras se navega entre páginas. El envío de nueva búsqueda o criterios de categoría deberá mostrar la primera página de los resultados coincidentes. Las solicitudes de página inválidas o fuera de rango deberán seguir el comportamiento definido en la Sección 11. |

<a id="73-gestion-de-eventos"></a>
## 7.3 Gestión de Eventos

| ID | Título | Requisito |
|---|---|---|
| <a id="fr-evt-001"></a>[**FR&#8209;EVT&#8209;001**](#fr-evt-001) | Crear evento | El sistema deberá permitir a un organizador crear un nuevo evento propiedad de ese organizador. |
| <a id="fr-evt-002"></a>[**FR&#8209;EVT&#8209;002**](#fr-evt-002) | Creación en borrador | Un evento recién creado deberá guardarse inicialmente como evento en estado Borrador. |
| <a id="fr-evt-003"></a>[**FR&#8209;EVT&#8209;003**](#fr-evt-003) | Información del evento | El sistema deberá recopilar, como mínimo, un título, descripción, categoría, ubicación, fecha y hora de inicio y fin, y capacidad de participantes. |
| <a id="fr-evt-004"></a>[**FR&#8209;EVT&#8209;004**](#fr-evt-004) | Validación del evento | El sistema rechazará la creación o modificación del evento cuando falte información requerida o se incumplan las reglas de negocio aplicables. |
| <a id="fr-evt-005"></a>[**FR&#8209;EVT&#8209;005**](#fr-evt-005) | Lista de eventos del organizador | El sistema deberá permitir a un organizador ver una lista de gestión de eventos de su propiedad, incluyendo eventos en Borrador, Publicados y Cancelados. |
| <a id="fr-evt-006"></a>[**FR&#8209;EVT&#8209;006**](#fr-evt-006) | Editar evento propio | El sistema deberá permitir a un organizador editar un evento de su propiedad cuando el evento siga siendo elegible para modificación según las reglas de negocio. |
| <a id="fr-evt-007"></a>[**FR&#8209;EVT&#8209;007**](#fr-evt-007) | Publicar evento | El sistema deberá permitir a un organizador publicar un evento en borrador elegible. |
| <a id="fr-evt-008"></a>[**FR&#8209;EVT&#8209;008**](#fr-evt-008) | Disponibilidad pública tras la publicación | Después de publicar un evento, el sistema deberá hacerlo disponible a través de la exploración y descubrimiento normal de eventos mientras siga siendo próximo. |
| <a id="fr-evt-009"></a>[**FR&#8209;EVT&#8209;009**](#fr-evt-009) | Cancelar evento propio | El sistema deberá permitir a un organizador cancelar un evento publicado propio que aún no ha comenzado. |
| <a id="fr-evt-010"></a>[**FR&#8209;EVT&#8209;010**](#fr-evt-010) | Estado de evento cancelado | Cuando un evento se cancela, el sistema deberá conservar el registro del evento y mostrar su estado Cancelado a los usuarios que lo consulten. |
| <a id="fr-evt-011"></a>[**FR&#8209;EVT&#8209;011**](#fr-evt-011) | Eliminar evento borrador | El sistema deberá permitir a un organizador eliminar permanentemente un evento propio en borrador solo cuando no existan registros actuales para ese evento. |

<a id="74-registro-de-eventos"></a>
## 7.4 Registro de Eventos

| ID | Título | Requisito |
|---|---|---|
| <a id="fr-reg-001"></a>[**FR&#8209;REG&#8209;001**](#fr-reg-001) | Registrarse en un evento | El sistema deberá permitir a un participante autenticado registrarse en un evento elegible con capacidad disponible. |
| <a id="fr-reg-002"></a>[**FR&#8209;REG&#8209;002**](#fr-reg-002) | Confirmación del registro | Después de un registro exitoso, el sistema deberá proporcionar una confirmación clara de que el participante está registrado. |
| <a id="fr-reg-003"></a>[**FR&#8209;REG&#8209;003**](#fr-reg-003) | Evitar registro duplicado | El sistema deberá impedir que un participante mantenga más de un registro actual para el mismo evento. |
| <a id="fr-reg-004"></a>[**FR&#8209;REG&#8209;004**](#fr-reg-004) | Aplicar capacidad del evento | El sistema deberá impedir que se cree un registro cuando el evento haya alcanzado su capacidad de participantes. |
| <a id="fr-reg-005"></a>[**FR&#8209;REG&#8209;005**](#fr-reg-005) | Elegibilidad para registro | El sistema deberá impedir nuevos registros para eventos en borrador, cancelados, pasados o ya comenzados. |
| <a id="fr-reg-006"></a>[**FR&#8209;REG&#8209;006**](#fr-reg-006) | Ver mis registros | El sistema deberá permitir a un participante autenticado ver sus registros actuales en eventos. |
| <a id="fr-reg-007"></a>[**FR&#8209;REG&#8209;007**](#fr-reg-007) | Cancelar registro propio | El sistema deberá permitir a un participante cancelar su propio registro antes de que el evento comience. |
| <a id="fr-reg-008"></a>[**FR&#8209;REG&#8209;008**](#fr-reg-008) | Liberar capacidad | Cuando un participante cancela un registro, el sistema deberá poner esa plaza disponible para otro participante elegible. |
| <a id="fr-reg-009"></a>[**FR&#8209;REG&#8209;009**](#fr-reg-009) | Lista de registros del organizador | El sistema deberá permitir a un organizador ver los participantes actuales registrados en un evento de su propiedad. |
| <a id="fr-reg-010"></a>[**FR&#8209;REG&#8209;010**](#fr-reg-010) | Recuento de registros | El sistema deberá mostrar el número actual de registros y la capacidad restante de un evento a su organizador. |

<a id="75-administracion"></a>
## 7.5 Administración

| ID | Título | Requisito |
|---|---|---|
| <a id="fr-adm-001"></a>[**FR&#8209;ADM&#8209;001**](#fr-adm-001) | Acceso administrativo | El sistema deberá restringir las funciones administrativas a administradores autenticados. |
| <a id="fr-adm-002"></a>[**FR&#8209;ADM&#8209;002**](#fr-adm-002) | Gestionar acceso de organizador | El sistema deberá permitir a un administrador conceder o revocar el acceso de rol Organizador a un usuario existente. |
| <a id="fr-adm-003"></a>[**FR&#8209;ADM&#8209;003**](#fr-adm-003) | Ver lista de supervisión de eventos | El sistema deberá permitir a un administrador ver una lista de gestión de todos los eventos, independientemente del organizador o del estado. |
| <a id="fr-adm-004"></a>[**FR&#8209;ADM&#8209;004**](#fr-adm-004) | Cancelar cualquier evento elegible | El sistema deberá permitir a un administrador cancelar cualquier evento publicado que aún no haya comenzado. |
| <a id="fr-adm-005"></a>[**FR&#8209;ADM&#8209;005**](#fr-adm-005) | Crear categoría | El sistema deberá permitir a un administrador crear una categoría de evento. |
| <a id="fr-adm-006"></a>[**FR&#8209;ADM&#8209;006**](#fr-adm-006) | Editar categoría | El sistema deberá permitir a un administrador modificar el nombre de una categoría de evento existente cuando el nombre resultante sea válido y único. |
| <a id="fr-adm-007"></a>[**FR&#8209;ADM&#8209;007**](#fr-adm-007) | Eliminar categoría sin uso | El sistema deberá permitir a un administrador eliminar una categoría solo cuando ningún evento haga referencia actualmente a esa categoría. |

---

<a id="8-requisitos-de-datos"></a>
# 8. Requisitos de Datos

El sistema deberá persistir la información necesaria para soportar los requisitos funcionales.

<a id="81-informacion-requerida-del-evento"></a>
## 8.1 Información Requerida del Evento

Como mínimo, el concepto Evento debe soportar:

- identidad única;
- título;
- descripción;
- categoría;
- ubicación;
- fecha y hora de inicio;
- fecha y hora de fin;
- capacidad de participantes;
- estado: Borrador, Publicado o Cancelado;
- organizador propietario;
- información de creación suficiente para la gestión normal de la aplicación.

<a id="82-informacion-requerida-del-registro"></a>
## 8.2 Información Requerida del Registro

Como mínimo, el concepto Registro debe soportar:

- identidad única o equivalente relacional;
- participante;
- evento;
- fecha y hora de creación del registro.

El sistema requerido no necesita un modelo de historial de registros o listas de espera. La cancelación puede eliminar el registro actual en lugar de conservar un historial de auditoría.

<a id="83-expectativas-de-integridad-de-datos"></a>
## 8.3 Expectativas de Integridad de Datos

El modelo de datos deberá soportar estas expectativas de integridad:

- las identidades de inicio de sesión de las cuentas son únicas según el sistema de autenticación configurado;
- los nombres de las categorías son únicos según [`BR-CAT-001`](#br-cat-001);
- un participante no puede mantener registros actuales duplicados para un mismo evento;
- cada evento hace referencia a una categoría existente;
- cada evento hace referencia a un organizador propietario;
- cada registro hace referencia a un usuario y un evento existentes;
- la capacidad del evento es mayor que cero;
- las fechas almacenadas del evento siguen las reglas de tiempo del evento.

---

<a id="9-requisitos-de-interfaces-externas"></a>
# 9. Requisitos de Interfaces Externas

<a id="91-interfaz-de-usuario"></a>
## 9.1 Interfaz de Usuario

La aplicación deberá proporcionar páginas web renderizadas en servidor suficientes para completar todos los flujos requeridos.

La interfaz deberá incluir, como mínimo:

- navegación de la aplicación apropiada al estado de autenticación y rol del usuario actual;
- una página de exploración de eventos con búsqueda, filtrado por categoría, información de página actual/recuento de resultados y controles de paginación cuando proceda;
- una página de detalles del evento;
- interfaces de registro e inicio de sesión de cuentas;
- una lista de registros del participante;
- páginas de gestión de eventos del organizador;
- formularios de creación y edición del evento;
- una vista de registros del organizador;
- páginas de gestión de categorías del administrador;
- capacidad de gestión de usuarios/organizadores por parte del administrador;
- retroalimentación clara de éxito y error después de acciones importantes.

El diseño visual exacto y el diseño de la página son elecciones de implementación siempre que el comportamiento requerido siga siendo claro y usable.

<a id="92-interfaz-del-navegador"></a>
## 9.2 Interfaz del Navegador

El sistema funcionará mediante interacciones estándar HTTP/HTTPS con el navegador, usando HTML renderizado en servidor. JavaScript del lado del cliente puede mejorar la usabilidad, pero no deberá ser necesario para evitar la validación ni la autorización del servidor.

<a id="93-interfaz-de-autenticacion"></a>
## 9.3 Interfaz de Autenticación

La aplicación deberá proporcionar registro web, inicio de sesión, cierre de sesión y comportamiento de control de acceso a través del mecanismo de autenticación de ASP.NET aprobado por el curso.

<a id="94-servicios-externos"></a>
## 9.4 Servicios Externos

No se requiere ningún servicio externo de terceros para este proyecto.

---

<a id="10-requisitos-no-funcionales"></a>
# 10. Requisitos No Funcionales

<a id="101-seguridad"></a>
## 10.1 Seguridad

| ID | Requisito |
|---|---|
| <a id="nfr-sec-001"></a>[**NFR&#8209;SEC&#8209;001**](#nfr-sec-001) | El sistema deberá aplicar la autorización en el servidor para cada operación protegida. |
| <a id="nfr-sec-002"></a>[**NFR&#8209;SEC&#8209;002**](#nfr-sec-002) | El sistema deberá usar el framework de autenticación aprobado para el almacenamiento y verificación de contraseñas en lugar de guardar contraseñas en texto plano. |
| <a id="nfr-sec-003"></a>[**NFR&#8209;SEC&#8209;003**](#nfr-sec-003) | Las solicitudes de formularios que cambian el estado deberán usar la protección normal del framework contra falsificación de solicitudes entre sitios (CSRF) cuando corresponda. |
| <a id="nfr-sec-004"></a>[**NFR&#8209;SEC&#8209;004**](#nfr-sec-004) | La aplicación no deberá exponer claves secretas, credenciales de conexión ni contraseñas de producción en el control de versiones. |
| <a id="nfr-sec-005"></a>[**NFR&#8209;SEC&#8209;005**](#nfr-sec-005) | Las respuestas inesperadas visibles al usuario no deberán revelar trazas de pila, detalles de conexión a la base de datos ni otra información sensible del servidor. |

<a id="102-rendimiento"></a>
## 10.2 Rendimiento

| ID | Requisito |
|---|---|
| <a id="nfr-per-001"></a>[**NFR&#8209;PER&#8209;001**](#nfr-per-001) | Con un conjunto de datos representativo del aula de hasta 500 eventos y 2.000 registros actuales, las páginas normales de exploración y detalle de eventos deberán completarse en aproximadamente dos segundos bajo condiciones de prueba de un solo usuario en el entorno aprobado. |
| <a id="nfr-per-002"></a>[**NFR&#8209;PER&#8209;002**](#nfr-per-002) | La búsqueda pública de eventos, el filtrado por categoría y la paginación deberán realizarse mediante consultas de base de datos del lado del servidor antes de materializar filas de eventos. El servidor deberá recuperar solo las filas de eventos coincidentes de la página solicitada y no todo el conjunto de resultados coincidentes; el navegador no deberá cargar el conjunto completo de resultados en memoria del cliente. |

<a id="103-usabilidad"></a>
## 10.3 Usabilidad

| ID | Requisito |
|---|---|
| <a id="nfr-usa-001"></a>[**NFR&#8209;USA&#8209;001**](#nfr-usa-001) | La navegación deberá indicar claramente las principales acciones disponibles para el usuario actual. |
| <a id="nfr-usa-002"></a>[**NFR&#8209;USA&#8209;002**](#nfr-usa-002) | Los formularios deberán mostrar retroalimentación de validación comprensible y conservar los valores válidos ingresados por el usuario cuando una presentación falle en la validación. |
| <a id="nfr-usa-003"></a>[**NFR&#8209;USA&#8209;003**](#nfr-usa-003) | Las acciones destructivas, como cancelar un evento o eliminar un evento en borrador, deberán requerir una acción explícita del usuario que deje clara la naturaleza destructiva de la operación. |
| <a id="nfr-usa-004"></a>[**NFR&#8209;USA&#8209;004**](#nfr-usa-004) | Operaciones importantes, como la publicación de un evento, el registro en un evento, la cancelación de un registro y la cancelación del evento, deberán proporcionar retroalimentación visible de éxito o fallo. |

<a id="104-accesibilidad"></a>
## 10.4 Accesibilidad

| ID | Requisito |
|---|---|
| <a id="nfr-acc-001"></a>[**NFR&#8209;ACC&#8209;001**](#nfr-acc-001) | Los controles del formulario deberán tener etiquetas asociadas programáticamente o nombres accesibles equivalentes. |
| <a id="nfr-acc-002"></a>[**NFR&#8209;ACC&#8209;002**](#nfr-acc-002) | Los flujos principales de la aplicación, incluidas las pantallas de paginación pública de eventos, deberán ser operables usando teclado sin requerir interacción exclusiva con el ratón. |
| <a id="nfr-acc-003"></a>[**NFR&#8209;ACC&#8209;003**](#nfr-acc-003) | Las páginas deberán usar encabezados significativos del documento y HTML semántico apropiado para el contenido, incluida una navegación clara del estado de la página pública de eventos. |

<a id="105-fiabilidad-y-correccion-de-datos"></a>
## 10.5 Fiabilidad y Corrección de Datos

| ID | Requisito |
|---|---|
| <a id="nfr-rel-001"></a>[**NFR&#8209;REL&#8209;001**](#nfr-rel-001) | Las operaciones completadas con éxito de creación, actualización, registro y cancelación deberán reflejarse después de refrescar la página o comenzar una nueva sesión. |
| <a id="nfr-rel-002"></a>[**NFR&#8209;REL&#8209;002**](#nfr-rel-002) | Las fallas de validación o autorización no deberán aplicar parcialmente el cambio solicitado de datos. |
| <a id="nfr-rel-003"></a>[**NFR&#8209;REL&#8209;003**](#nfr-rel-003) | El estado guardado de la aplicación no deberá contener más registros actuales para un evento que la capacidad configurada del evento como resultado de flujos normales de la aplicación. |

<a id="106-compatibilidad"></a>
## 10.6 Compatibilidad

| ID | Requisito |
|---|---|
| <a id="nfr-com-001"></a>[**NFR&#8209;COM&#8209;001**](#nfr-com-001) | La aplicación deberá soportar las versiones estables actuales usadas en el curso de al menos dos navegadores modernos de escritorio importantes, como Chrome, Edge o Firefox. |
| <a id="nfr-com-002"></a>[**NFR&#8209;COM&#8209;002**](#nfr-com-002) | Las páginas principales seguirán siendo utilizables en anchos de pantalla comunes de portátiles y móviles sin desplazamiento horizontal para el contenido normal de la página. |

---

<a id="11-manejo-de-errores-y-comportamiento-de-excepciones"></a>
# 11. Manejo de Errores y Comportamiento de Excepciones

La aplicación deberá responder a problemas esperados de manera que mantenga informado al usuario y preserve datos válidos.

| Condición | Comportamiento Esperado del Sistema |
|---|---|
| Formulario de cuenta o evento inválido | Rechazar la presentación, volver a mostrar el formulario, identificar los problemas de validación y conservar los valores válidos ingresados cuando sea práctico. |
| Inicio de sesión inválido | Rechazar la autenticación y mostrar un mensaje genérico de fallo de autenticación. |
| Acceso no autenticado a una página protegida | Dirigir al usuario al flujo de autenticación y devolverle al destino solicitado cuando sea práctico. |
| Operación autenticada pero no autorizada | Rechazar la operación y mostrar una respuesta apropiada de acceso denegado sin cambiar datos protegidos. |
| El evento u otro recurso solicitado no existe | Mostrar una respuesta apropiada de “no encontrado” en lugar de una excepción no controlada genérica. |
| Número de página faltante, inválido o fuera de rango para resultados públicos de eventos | Usar la primera página de resultados cuando el número de página falte, sea incorrecto, cero o negativo. Para una solicitud más allá de la última página disponible, mostrar la última página. Si no hay eventos que coincidan con los criterios, mostrar el mensaje de “sin resultados” sin enlaces de paginación. Conservar los criterios de búsqueda y categoría aplicables. |
| Registro duplicado del evento | Rechazar el registro duplicado e informar al participante de que ya existe un registro actual. |
| El evento alcanza la capacidad antes de que termine el registro | Rechazar el registro e informar al participante de que no quedan plazas disponibles. |
| Intento de registrarse en un evento en borrador, cancelado o ya comenzado | Rechazar la inscripción y explicar que la inscripción no está disponible. |
| Intento de reducir la capacidad del evento por debajo de los registros actuales | Rechazar la actualización y explicar por qué la capacidad no puede reducirse a ese valor. |
| Intento de eliminar una categoría actualmente usada por eventos | Rechazar la eliminación y explicar que la categoría está en uso. |
| Fallo inesperado del servidor | Conservar los datos válidos ya confirmados, evitar exponer detalles técnicos sensibles al usuario y mostrar un mensaje general de fallo o una página de error. |

---

<a id="12-criterios-de-aceptacion-definicion-de-hecho"></a>
# 12. Criterios de Aceptación / Definición de Hecho

El Proyecto #1 se considera completo cuando la aplicación cumple el alcance requerido y demuestra una implementación coherente de extremo a extremo.

| ID | Criterio de Aceptación |
|---|---|
| <a id="ac-001"></a>[**AC&#8209;001**](#ac-001) | Los visitantes pueden explorar, buscar, filtrar, navegar por resultados paginados manteniendo los criterios activos de búsqueda/categoría y el orden cronológico, y ver eventos próximos publicados según [`FR-SRH-*`](#event-discovery-search-and-browsing). El comportamiento de primer/última página y sin resultados funciona según lo especificado. |
| <a id="ac-002"></a>[**AC&#8209;002**](#ac-002) | Los usuarios pueden registrarse, iniciar sesión, cerrar sesión y se les impide usar capacidades protegidas sin autenticación según [`FR-ACC-*`](#accounts-and-authentication). |
| <a id="ac-003"></a>[**AC&#8209;003**](#ac-003) | Los organizadores pueden crear, validar, editar, publicar, cancelar y, cuando corresponda, eliminar eventos propios según [`FR-EVT-*`](#event-management) y las reglas de negocio aplicables. |
| <a id="ac-004"></a>[**AC&#8209;004**](#ac-004) | Los participantes pueden registrarse en eventos elegibles, ver sus registros y cancelar sus propios registros según [`FR-REG-*`](#event-registration). |
| <a id="ac-005"></a>[**AC&#8209;005**](#ac-005) | Las reglas de capacidad, registro duplicado, estado del evento, propiedad y categoría se aplican en el servidor y los datos persistidos siguen siendo válidos. |
| <a id="ac-006"></a>[**AC&#8209;006**](#ac-006) | Los organizadores pueden ver los registros actuales de un evento propio sin obtener acceso de gestión a eventos que no les pertenecen. |
| <a id="ac-007"></a>[**AC&#8209;007**](#ac-007) | Los administradores pueden gestionar el acceso de organizador, las categorías de eventos y la cancelación de eventos a nivel de aplicación según [`FR-ADM-*`](#administration). |
| <a id="ac-008"></a>[**AC&#8209;008**](#ac-008) | Los datos de la aplicación persisten en la base de datos relacional y siguen disponibles tras la actualización de la página y el reinicio o redeploy normal de la aplicación compatible con el entorno del curso. |
| <a id="ac-009"></a>[**AC&#8209;009**](#ac-009) | Las fallas importantes de validación, autenticación, autorización y reglas de negocio producen retroalimentación controlada visible para el usuario en lugar de errores no manejados. |
| <a id="ac-010"></a>[**AC&#8209;010**](#ac-010) | Las pruebas automatizadas cubren un conjunto representativo de comportamientos críticos del negocio, incluidas al menos la elegibilidad/capacidad de registro y un flujo sensible a la autorización. |
| <a id="ac-011"></a>[**AC&#8209;011**](#ac-011) | La aplicación cumple las expectativas requeridas de seguridad, usabilidad, accesibilidad, fiabilidad y compatibilidad definidas en la Sección 10. |
| <a id="ac-012"></a>[**AC&#8209;012**](#ac-012) | La aplicación está desplegada en un entorno aprobado por el instructor y sus flujos requeridos de extremo a extremo se pueden demostrar allí. |
| <a id="ac-013"></a>[**AC&#8209;013**](#ac-013) | El repositorio incluye documentación breve de configuración que describe requisitos previos, configuración, pasos de configuración/migración de base de datos y cómo ejecutar la aplicación y las pruebas. |

---

<a id="13-matriz-de-trazabilidad-de-requisitos"></a>
# 13. Matriz de Trazabilidad de Requisitos

La matriz de trazabilidad puede completarse progresivamente a medida que se implementa el proyecto. Conecta un requisito con las partes y pruebas de la aplicación que lo materializan, permitiendo a los estudiantes (y a los instructores) verificar que los requisitos no se han perdido durante el desarrollo.

| ID de Requisito | Funcionalidad / Componente / Artefacto | Prueba(s) / Evidencia | Estado | Notas / Comentarios |
|---|---|---|---|---|
| [`FR-SRH-009`](#fr-srh-009) | Planeado: `EventsController.Index`, `EventSearchViewModel`, `Views/Events/Index.cshtml` y CSS de paginación | Escenarios de varias páginas y filtros combinados de la Lección 3; comprobaciones de límites / sin resultados; evidencia de regresión y prueba de humo de la Lección 9 | Planeado — Lección 3 | CR-003 (aprobado); aplica solo al catálogo público de eventos; no añade paginación a las listas de gestión de organizador o administrador. |
| &nbsp; | &nbsp; | &nbsp; | &nbsp; | &nbsp; |
| &nbsp; | &nbsp; | &nbsp; | &nbsp; | &nbsp; |

---

<a id="14-mapa-conceptual-del-curso-a-los-requisitos"></a>
# 14. Mapa Conceptual del Curso a los Requisitos

Esta tabla es una referencia didáctica, no un conjunto adicional de requisitos.

| Característica del Proyecto | Requisitos Representativos | Conceptos Full-Stack Encontrados |
|---|---|---|
| Explorar y ver eventos | [`FR-SRH-001`](#fr-srh-001)–[`FR-SRH-004`](#fr-srh-004) | flujo MVC, enrutamiento, consultas, modelos de vista, interfaz renderizada en servidor |
| Búsqueda y filtrado por categoría | [`FR-SRH-005`](#fr-srh-005)–[`FR-SRH-007`](#fr-srh-007) | parámetros de consulta, filtrado, consultas a base de datos, estados vacíos |
| Resultados públicos de eventos paginados | [`FR-SRH-009`](#fr-srh-009) | estado GET de página, recuentos totales coincidentes, orden cronológico determinista, consultas acotadas del lado de la base de datos, metadatos de página del modelo de vista, navegación Razor, accesibilidad con teclado |
| Flujos de cuenta e inicio de sesión | [`FR-ACC-*`](#accounts-and-authentication) | autenticación, sesiones/cookies, rutas protegidas, seguridad |
| Crear y editar evento | [`FR-EVT-001`](#fr-evt-001)–[`FR-EVT-006`](#fr-evt-006) | formularios, enlace de modelos, validación, EF Core, datos relacionales |
| Publicar y cancelar evento | [`FR-EVT-007`](#fr-evt-007)–[`FR-EVT-010`](#fr-evt-010) | reglas de negocio, transiciones de estado, autorización, manejo de errores |
| Registro de eventos | [`FR-REG-001`](#fr-reg-001)–[`FR-REG-008`](#fr-reg-008) | relaciones muchos a muchos, validación, lógica de negocio, reglas de capacidad |
| Vista de registros del organizador | [`FR-REG-009`](#fr-reg-009)–[`FR-REG-010`](#fr-reg-010) | autorización por propiedad, consultas relacionales, recuentos agregados |
| Administración de categorías | [`FR-ADM-005`](#fr-adm-005)–[`FR-ADM-007`](#fr-adm-007) | CRUD, integridad referencial, autorización administrativa |
| Despliegue y verificación final | [`AC-008`](#ac-008)–[`AC-013`](#ac-013) | entornos, configuración, migraciones, pruebas, flujo de producción |
