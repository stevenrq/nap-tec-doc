# Registro de Decisiones de Arquitectura (ADR)

## Plataforma de gestión y creación de contratos «NAP»

| Campo | Detalle |
| --- | --- |
| **Documento** | Registro de Decisiones de Arquitectura (Architecture Decision Records) |
| **Versión** | 1.2 |
| **Fecha** | 20 de junio de 2026 |
| **Empresa** | Nothing Sense (En formación) |
| **Autor / CTO** | Steven Ricardo Quiñones |
| **Revisor** | Fredinson Solano (CEO) |
| **Ciudad** | Montería, Córdoba, Colombia |
| **Documento base** | Especificación de Requisitos de Software (ERS) v1.1, IEEE 830-1998 |
| **Marco de referencia** | Formato MADR, alineado con ISO/IEC/IEEE 42010:2022 (descripción de arquitectura) |
| **Propósito** | Fuente de verdad del *stack* tecnológico y de herramientas del proyecto NAP |
| **Carácter** | Uso interno |

---

## ¿Qué es este documento?

Un **ADR (Architecture Decision Record)** es un registro breve de una decisión de arquitectura significativa: su contexto, las opciones evaluadas, la decisión tomada y sus consecuencias. El objetivo es dejar constancia del **porqué** detrás de cada elección técnica, de modo que el equipo (presente y futuro) entienda las razones y los compromisos asumidos sin tener que reconstruirlos de memoria.

Este documento es la **fuente de verdad** de las tecnologías y herramientas decididas para el proyecto NAP. Se apoya directamente en la ERS v1.1 y cada decisión enlaza con los requisitos funcionales (RF) y no funcionales (RNF) que la motivan, extendiendo la trazabilidad establecida en el capítulo 5 de la ERS. Cerrar estas decisiones es el último paso antes de **pasar a la fase de diseño**.

Se utiliza el formato **MADR** (Markdown Architectural Decision Records), alineado con el marco de **ISO/IEC/IEEE 42010:2022**, que recomienda registrar las decisiones de arquitectura y su justificación (*architecture rationale*), incluyendo las alternativas no elegidas.

### Resumen del stack tecnológico

Vista rápida de las decisiones vigentes. El detalle y la justificación de cada una están en su ADR correspondiente.

| Capa / Aspecto | Tecnología / Herramienta | ADR |
| --- | --- | --- |
| Estilo arquitectónico | Monolito modular (por features) | ADR-0001 |
| Backend | Spring Boot (Java) | ADR-0002 |
| Base de datos | PostgreSQL | ADR-0003 |
| Frontend | Angular | ADR-0004 |
| Estilos de interfaz | Tailwind CSS | ADR-0005 |
| Hosting de base de datos | Supabase (PostgreSQL gestionado) | ADR-0006, ADR-0007 |
| Hosting del backend | Render | ADR-0006 |
| Hosting del frontend | Vercel | ADR-0006 |
| Autenticación y autorización | Spring Security | ADR-0007 |
| Almacenamiento de archivos | Cloudflare R2 (compatible con S3) | ADR-0007 |

### Leyenda de estados

| Estado | Significado |
| --- | --- |
| **Propuesta** | En discusión, aún no adoptada. |
| **Aceptada** | Decisión vigente y en aplicación. |
| **Rechazada** | Evaluada y descartada. |
| **Obsoleta** | Ya no aplica. |
| **Reemplazada** | Sustituida por otra ADR (se indica cuál). |

### Índice de decisiones

| ID | Decisión | Estado |
| --- | --- | --- |
| [ADR-0001](#adr-0001--estilo-arquitectónico-monolito-modular) | Estilo arquitectónico: monolito modular | Aceptada |
| [ADR-0002](#adr-0002--plataforma-de-backend-spring-boot-java) | Plataforma de backend: Spring Boot (Java) | Aceptada |
| [ADR-0003](#adr-0003--motor-de-base-de-datos-postgresql) | Motor de base de datos: PostgreSQL | Aceptada |
| [ADR-0004](#adr-0004--framework-de-frontend-angular) | Framework de frontend: Angular | Aceptada |
| [ADR-0005](#adr-0005--estrategia-de-estilos-tailwind-css) | Estrategia de estilos: Tailwind CSS | Aceptada |
| [ADR-0006](#adr-0006--infraestructura-de-despliegue-supabase--render--vercel) | Infraestructura de despliegue: Supabase + Render + Vercel | Aceptada |
| [ADR-0007](#adr-0007--alcance-de-supabase-autenticación-y-almacenamiento-de-archivos) | Alcance de Supabase: autenticación (Spring Security) y archivos (Cloudflare R2) | Aceptada |

---

## ADR-0001 — Estilo arquitectónico: monolito modular

**Estado:** Aceptada
**Fecha:** 2026-06-20
**Decisor:** Steven R. Quiñones (CTO)

### Contexto

La plataforma NAP agrupa su funcionalidad en módulos con límites naturales bien definidos (usuarios y autenticación, perfiles, marketplace/emparejamiento, contratos informales, contratos formales, firma electrónica, pagos y custodia, reputación, notificaciones, disputas, reportes y administración), tal como se organizan los RF en la sección 3.2 de la ERS.

El equipo es pequeño y la empresa está en formación, con una estrategia de lanzamiento gradual por fases (ERS 2.7). Se necesita una arquitectura que favorezca la modularidad (RNF-015), permita el escalado horizontal a futuro (RNF-019) y mantenga la portabilidad (RNF-016), pero sin la carga operativa que implican los microservicios (orquestación, observabilidad distribuida, consistencia entre servicios) en una etapa temprana.

### Decisión

Se adopta un **monolito modular**: una única aplicación desplegable, organizada internamente por módulos/features (*package-by-feature*), con fronteras explícitas entre módulos. La comunicación entre módulos se realiza a través de servicios de aplicación e interfaces, evitando el acceso directo a las tablas de otros módulos.

### Opciones consideradas

| Opción | A favor | En contra |
| --- | --- | --- |
| **Monolito modular** (elegida) | Despliegue único y simple; transacciones ACID que cruzan módulos (clave para firma + pago + escrow); modularidad sin sobrecarga operativa. | Riesgo de acoplamiento si no se disciplinan los límites internos. |
| Microservicios | Despliegue y escalado independiente por servicio. | Sobrecarga operativa, complejidad de transacciones distribuidas y necesidad de DevOps que el equipo aún no tiene. Prematuro para el MVP. |
| Monolito tradicional en capas (sin módulos) | Máxima simplicidad inicial. | Tiende al "código espagueti"; dificulta mantenibilidad (RNF-015) y una futura extracción de servicios. |

### Consecuencias

#### Positivas

- Cumple RNF-015 (arquitectura modular y documentada) manteniendo simplicidad operativa.
- Las operaciones críticas que abarcan varios módulos (firma → pago → custodia) pueden resolverse con una única transacción ACID, lo que favorece RNF-003 y RNF-009.
- Permite el escalado horizontal replicando la instancia detrás de un balanceador, siempre que la aplicación se mantenga *stateless* (compatible con RNF-019 y con el despliegue en Render).

#### Negativas / riesgos

- Riesgo de erosión de los límites entre módulos con el tiempo.
- **Mitigación:** cada módulo en su propio paquete con API interna explícita; revisiones de código que vigilen las dependencias entre módulos; posibilidad de extraer un módulo a servicio independiente más adelante (el candidato natural es **Pagos/Custodia** por su criticidad y posible escalado diferenciado).

**Requisitos relacionados:** RNF-015, RNF-016, RNF-019, RNF-003, RNF-009.

---

## ADR-0002 — Plataforma de backend: Spring Boot (Java)

**Estado:** Aceptada
**Fecha:** 2026-06-20
**Decisor:** Steven R. Quiñones (CTO)

### Contexto

El equipo de backend, liderado por el CTO, domina **Java**. El backend de NAP concentra la lógica más sensible del sistema: control de acceso por roles (RNF-013), autenticación multifactor en operaciones críticas (RN-010), firma electrónica y pagos con garantía de consistencia (RNF-003, RNF-009) e idempotencia en pagos (RNF-025), además de múltiples integraciones externas (pasarela de pago, firma, correo, facturación DIAN — ERS 3.1.3 y Apéndice A).

### Decisión

Se adopta **Spring Boot** (Java) como plataforma del backend, con un backend organizado por módulos/features alineado con ADR-0001.

### Opciones consideradas

| Opción | A favor | En contra |
| --- | --- | --- |
| **Spring Boot / Java** (elegida) | Dominio del equipo; ecosistema maduro; Spring Security para RBAC y MFA; transaccionalidad declarativa robusta; Spring Data JPA; amplio soporte de integraciones. | Mayor consumo de memoria y arranque más lento (sensible en planes gratuitos); más verboso. |
| NestJS (Node.js / TypeScript) | Mismo lenguaje (TS) que el frontend; arranque ligero. | El equipo no lo domina; ecosistema transaccional/financiero menos maduro que Spring. |
| Django (Python) | Productividad alta; admin integrado. | Fuera del dominio del equipo; menor afinidad con el modelo transaccional fuerte requerido. |

### Consecuencias

#### Positivas

- Aprovecha la experiencia del equipo, reduciendo riesgo y tiempo de desarrollo.
- **Spring Security** cubre directamente RBAC (RNF-013) y habilita el flujo MFA (RN-010, RNF-013).
- La transaccionalidad declarativa (`@Transactional`) facilita garantizar consistencia en firma y pago (RNF-003, RNF-009) e implementar idempotencia (RNF-025).

#### Negativas / riesgos

- El arranque más lento y el consumo de recursos pueden provocar *cold starts* en planes gratuitos de hosting (ver ADR-0006), afectando RNF-001 (tiempo de respuesta) y RNF-010 (disponibilidad).
- **Mitigación:** evaluar plan pago de Render para producción; mantener la instancia activa; optimizar la configuración de arranque.

**Requisitos relacionados:** RNF-013, RNF-003, RNF-009, RNF-025, RN-010, RF-007.

---

## ADR-0003 — Motor de base de datos: PostgreSQL

**Estado:** Aceptada
**Fecha:** 2026-06-20
**Decisor:** Steven R. Quiñones (CTO)

### Contexto

NAP gestiona datos contractuales y financieros que exigen integridad y consistencia ante fallos (RNF-009), retención y archivo legal de los contratos firmados (RNF-022), versionado inmutable de contratos (RN-012, RN-013) y auditoría de acciones relevantes (RF-059). Además, los importes deben manejarse con precisión (RN-014) y las marcas de tiempo registrarse en UTC (RNF-029).

### Decisión

Se adopta **PostgreSQL** como motor de base de datos relacional.

### Opciones consideradas

| Opción | A favor | En contra |
| --- | --- | --- |
| **PostgreSQL** (elegida) | ACID sólido e integridad referencial; tipos avanzados (`JSONB` para versiones/cláusulas, `numeric` para importes, `timestamptz` para UTC); maduro y open source; excelente soporte en proveedores gestionados (Supabase). | — |
| MySQL / MariaDB | Amplia adopción. | Tipos avanzados y funcionalidades transaccionales menos ricos que PostgreSQL. |
| NoSQL documental (MongoDB) | Flexibilidad de esquema. | El dominio es fuertemente relacional y transaccional; la integridad referencial y las transacciones multi-entidad serían más difíciles de garantizar. |

### Consecuencias

#### Positivas

- Las garantías ACID sostienen RNF-009 y las operaciones de firma/pago (RNF-003, RNF-025).
- `numeric` permite manejar importes sin errores de redondeo (RN-014); `timestamptz` apoya el manejo de UTC (RNF-029); `JSONB` es útil para almacenar versiones y cláusulas de contrato (RN-012).
- Integración nativa y gestionada vía Supabase (ver ADR-0006).

#### Negativas / riesgos

- Requiere diseño cuidadoso de índices para cumplir los tiempos de respuesta (RNF-001) a medida que crece el volumen (RNF-024).

**Requisitos relacionados:** RNF-009, RNF-022, RNF-024, RNF-029, RN-012, RN-013, RN-014, RF-059.

---

## ADR-0004 — Framework de frontend: Angular

**Estado:** Aceptada
**Fecha:** 2026-06-20
**Decisor:** Steven R. Quiñones (CTO)

### Contexto

El equipo de frontend, liderado por el CTO, trabaja con **Angular**. El frontend de NAP es una aplicación web responsiva (RNF-005) y relativamente extensa: el inventario inicial contempla 14 pantallas (IU-01 a IU-14, ERS 3.1.1), con formularios complejos (publicación de necesidades, edición y negociación de contratos, firma, pagos). La interfaz debe estar en español (RNF-017), buscar accesibilidad WCAG 2.1 AA (RNF-018) y estar preparada para internacionalización futura (RNF-023).

### Decisión

Se adopta **Angular** como framework de frontend, con TypeScript.

### Opciones consideradas

| Opción | A favor | En contra |
| --- | --- | --- |
| **Angular** (elegida) | Dominio del equipo; framework completo (enrutamiento, formularios, HTTP, i18n integrado); TypeScript de fábrica; estructura opinada que favorece la consistencia en equipo. | Curva de aprendizaje más empinada; *bundles* potencialmente grandes. |
| React | Gran ecosistema y flexibilidad. | Requiere ensamblar librerías (router, forms, estado); menos opinado. |
| Vue | Curva de aprendizaje suave. | Menor afinidad con el equipo; ecosistema corporativo más pequeño. |

### Consecuencias

#### Positivas

- Aprovecha la experiencia del equipo.
- Los **formularios reactivos** de Angular encajan bien con los formularios complejos del producto (IU-05, IU-09, IU-11).
- El soporte de **i18n** integrado facilita RNF-017 (español) y prepara el terreno para RNF-023 (multi-idioma).

#### Negativas / riesgos

- Tamaño de *bundle* y tiempo de carga inicial pueden tensionar RNF-004 (carga < 2 s).
- **Mitigación:** *lazy loading* por módulos de ruta, *build* de producción optimizado y aprovechamiento del despliegue en Vercel (CDN).

**Requisitos relacionados:** RNF-005, RNF-017, RNF-018, RNF-023, RNF-004, IU-01 a IU-14.

---

## ADR-0005 — Estrategia de estilos: Tailwind CSS

**Estado:** Aceptada
**Fecha:** 2026-06-20
**Decisor:** Steven R. Quiñones (CTO)

### Contexto

Se requiere una interfaz consistente y responsiva (RNF-005), construida con agilidad y con margen para definir una identidad visual propia del producto, sin atarse a la estética de una librería de componentes predefinida.

### Decisión

Se adopta **Tailwind CSS** como estrategia de estilos, integrado con Angular.

### Opciones consideradas

| Opción | A favor | En contra |
| --- | --- | --- |
| **Tailwind CSS** (elegida) | Desarrollo rápido; consistencia mediante tokens utilitarios; responsive sencillo; CSS final reducido (*purge*); control total del diseño. | HTML más verboso; requiere disciplina y componentización; **no aporta componentes accesibles listos para usar**. |
| Angular Material | Componentes accesibles y probados; integración nativa con Angular. | Estética reconocible y más difícil de personalizar a fondo. |
| Bootstrap | Rápido de arrancar. | Estética genérica; personalización más rígida. |
| CSS/SCSS plano | Máximo control. | Lento; mayor riesgo de inconsistencias. |

### Consecuencias

#### Positivas

- Acelera el desarrollo de las 14 pantallas con consistencia visual.
- Tamaño de salida pequeño, lo que apoya RNF-004 (tiempo de carga).

#### Negativas / riesgos

- Tailwind **no garantiza accesibilidad por sí mismo**, y NAP apunta a WCAG 2.1 AA (RNF-018).
- **Mitigación:** combinar Tailwind con primitivos accesibles (por ejemplo, **Angular CDK** o una librería de componentes *headless*) para diálogos, menús, formularios y foco; definir un *design system* interno con componentes reutilizables y accesibles.

**Requisitos relacionados:** RNF-005, RNF-004, RNF-018, RNF-017.

---

## ADR-0006 — Infraestructura de despliegue: Supabase + Render + Vercel

**Estado:** Aceptada
**Fecha:** 2026-06-20
**Decisor:** Steven R. Quiñones (CTO)

### Contexto

El equipo es pequeño y no cuenta con un rol de DevOps dedicado; la empresa está en formación, con presupuesto inicial limitado. Hay que desplegar tres piezas: la base de datos PostgreSQL (ADR-0003), el backend Spring Boot (ADR-0002) y el frontend Angular (ADR-0004). Se busca simplicidad operativa, costo inicial bajo, integración con flujos de despliegue por *git* y portabilidad razonable (RNF-016).

### Decisión

Se adopta una estrategia **PaaS por capa**:

- **Supabase** como PostgreSQL gestionado.
- **Render** como host del backend Spring Boot (servicio web / contenedor).
- **Vercel** como host del frontend Angular.

### Opciones consideradas

| Opción | A favor | En contra |
| --- | --- | --- |
| **Supabase + Render + Vercel** (elegida) | Cada capa en una plataforma optimizada; despliegue por *git push*; costo inicial bajo; tecnologías estándar (Postgres, contenedores, *build* estático) que limitan el *lock-in*. | Posible latencia entre proveedores; *cold starts* en planes gratuitos; coordinación de tres paneles distintos. |
| Todo en un proveedor cloud (AWS/GCP/Azure) | Integración y red internas; control fino. | Mayor complejidad operativa y necesidad de DevOps; sobredimensionado para el MVP. |
| Railway / Fly.io (backend + DB) | Despliegue simple; backend y DB juntos. | Menor afinidad con las herramientas elegidas por el equipo. |

### Consecuencias

#### Positivas

- Despliegue ágil y de bajo costo para el MVP, acorde a la etapa de la empresa.
- Tecnologías estándar en las tres capas, lo que mantiene la portabilidad (RNF-016) siempre que se eviten dependencias fuertes de funcionalidades propietarias.

#### Negativas / riesgos

- ***Cold starts*** en el plan gratuito de Render afectarían RNF-001 (tiempo de respuesta) y RNF-010 (disponibilidad ≥ 99,5 %). **Mitigación:** plan pago / instancia siempre activa para producción.
- **Latencia entre regiones:** si Supabase, Render y Vercel quedan en regiones distintas, se degradan RNF-001 y RNF-004. **Mitigación:** alinear todas las capas a una región cercana a Colombia (preferiblemente *us-east*) y verificar los límites vigentes de cada plan.
- **Alcance de Supabase (resuelto en ADR-0007):** Supabase no es solo PostgreSQL; incluye **Auth**, **Storage** y **Realtime**. Se decidió usar Supabase **únicamente como PostgreSQL gestionado**; la autenticación se resuelve con Spring Security y el almacenamiento de archivos con Cloudflare R2. Ver ADR-0007.

**Requisitos relacionados:** RNF-016, RNF-001, RNF-004, RNF-010, RNF-024.

---

## ADR-0007 — Alcance de Supabase, autenticación y almacenamiento de archivos

**Estado:** Aceptada
**Fecha:** 2026-06-20
**Decisor:** Steven R. Quiñones (CTO)

### Contexto

ADR-0006 adoptó Supabase como PostgreSQL gestionado, pero Supabase es una plataforma BaaS completa que también ofrece **Auth**, **Storage** y **Realtime**, por lo que quedó pendiente definir hasta dónde apoyarse en ella.

El proyecto arranca desde cero con la restricción de operar **íntegramente en capas gratuitas** para evitar costos iniciales. Al mismo tiempo, NAP tiene un RBAC complejo con cinco roles y permisos diferenciados (RNF-013), MFA en operaciones sensibles (RN-010) y auditoría de acciones relevantes (RF-059). Además maneja documentos —contratos firmados, documentos de KYC (RF-003) y evidencias de disputa (RF-051)— que crecen de forma sostenida y deben conservarse por retención legal (RNF-022). El backend ya es Spring Boot (ADR-0002).

En capa gratuita, Supabase ofrece aproximadamente 1 GB de almacenamiento de archivos, 500 MB de base de datos, 5 GB de egreso y 50.000 usuarios activos mensuales para Auth; Cloudflare R2 ofrece 10 GB de almacenamiento, egreso gratuito de forma permanente y compatibilidad con la API de S3. *(Cifras de referencia a junio de 2026; conviene verificar los valores vigentes.)*

### Decisión

Usar Supabase **exclusivamente como PostgreSQL gestionado**. En consecuencia:

- **Autenticación y autorización:** se implementan en el backend con **Spring Security** (no se usa Supabase Auth).
- **Almacenamiento de archivos:** se usa **Cloudflare R2**, almacenamiento de objetos compatible con S3 (no se usa Supabase Storage).
- En PostgreSQL se guardan únicamente los **metadatos y el hash** de cada documento; los archivos residen en R2 y se acceden a través del backend mediante URLs prefirmadas.

### Opciones consideradas

#### Autenticación

| Opción | A favor | En contra |
| --- | --- | --- |
| **Spring Security** (elegida) | Gratis y sin techo de uso (es una librería, no un servicio medido); idiomática para el equipo Java; control total del RBAC complejo y del MFA; auditoría centralizada; sin lock-in. | Hay que implementar registro, recuperación de contraseña, verificación de correo y MFA. |
| Supabase Auth | Registro, inicio de sesión y OAuth listos para usar; 50.000 MAU gratis. | Igual exige construir la autorización en Spring → identidad partida en dos sistemas; el contador de MAU sube rápido con clientes B2B corporativos; mayor acoplamiento al proveedor. |

#### Almacenamiento de archivos

| Opción | A favor | En contra |
| --- | --- | --- |
| **Cloudflare R2** (elegida) | 10 GB gratis permanentes; cero cargos por egreso; compatible con S3 (cliente estándar desde Spring); el tráfico de archivos no compite con la cuota de egreso de Supabase; sin lock-in. | Un proveedor y unas credenciales adicionales que gestionar. |
| Supabase Storage | Un solo proveedor y panel; integración con políticas de acceso (RLS). | Solo 1 GB gratis; el egreso comparte cuota con la base de datos; su integración con RLS no aporta valor aquí porque no se usa Supabase Auth. |

### Consecuencias

#### Positivas

- Operación íntegramente gratuita y sin techos que fuercen un pago anticipado.
- Autenticación, RBAC (RNF-013) y MFA (RN-010) bajo control total del backend, con auditoría centralizada (RF-059) y sin lock-in (RNF-016).
- Capacidad de almacenamiento 10× mayor y egreso gratuito, adecuado para documentos que crecen y se conservan por ley (RNF-022).
- Mantener solo metadatos y hash en PostgreSQL respeta el límite de 500 MB de la base de datos gratuita y refuerza la integridad documental (RN-013; la entidad FIRMA conserva el hash y el sello de tiempo).

#### Negativas / riesgos

- Mayor trabajo inicial de autenticación (registro, recuperación, verificación, MFA). **Mitigación:** aprovechar Spring Security y el dominio del equipo en Java.
- Un servicio externo adicional (R2) que integrar y cuyas credenciales hay que custodiar. **Mitigación:** cliente S3 estándar y gestión de secretos por variables de entorno.
- El proyecto gratuito de Supabase se pausa tras una semana de inactividad. **Mitigación:** un *ping* programado (cron) que mantenga el proyecto activo; se relaciona con los *cold starts* de Render señalados en ADR-0006.

**Requisitos relacionados:** RNF-013, RNF-016, RNF-022, RNF-024, RN-010, RN-013, RF-003, RF-051, RF-059.

---

## Apéndice — Convención y siguiente fase

- **Alcance de este documento.** Reúne el conjunto completo de decisiones sobre tecnologías y herramientas del proyecto NAP y se considera su **fuente de verdad**. Cualquier cambio futuro de *stack* (por ejemplo, sustituir una herramienta o un proveedor) se registra aquí como una nueva ADR; la decisión anterior no se borra, sino que se marca como *Obsoleta* o *Reemplazada*.
- **Convención de archivos.** El documento reúne todas las ADR en un único archivo, como se solicitó. Si el repositorio crece, la práctica habitual es mantener **un archivo por ADR** dentro de `docs/adr/` (por ejemplo, `0001-monolito-modular.md`), conservando la numeración y el estado.
- **Siguiente fase — Diseño.** Con el *stack* cerrado, el proyecto pasa a la fase de diseño, donde el modelado se abordará **por requisito** (de forma incremental, no todo a la vez), incluyendo los diagramas **UML** que correspondan a cada uno (casos de uso, clases, secuencia, actividad y estados), apoyados en los modelos de análisis ya existentes en el capítulo 4 de la ERS.
