# NAP — Documentación técnica

Repositorio de **documentación técnica** de la plataforma de gestión y creación de contratos **«NAP»**, de la empresa **Nothing Sense** (en formación). No contiene código fuente: reúne los documentos de ingeniería que sirven de base para las fases de diseño e implementación del producto.

## ¿Qué es NAP?

NAP es una plataforma web que integra, en un único entorno, la conexión ágil entre personas con necesidades puntuales y profesionales que pueden resolverlas, junto con la administración estructurada de contratos formales y corporativos. Cubre el registro de usuarios, el emparejamiento de oferta y demanda, la generación de contratos a partir de plantillas (informales y formales), la firma electrónica, los pagos con custodia de fondos (*escrow*), el seguimiento del ciclo de vida contractual y la gestión de valoraciones, notificaciones y disputas.

## Contenido del repositorio

| Documento | Descripción |
| --- | --- |
| [`ERS_NAP_IEEE830.md`](ERS_NAP_IEEE830.md) | **Especificación de Requisitos de Software (ERS)**, elaborada conforme al estándar IEEE 830-1998. Define los requisitos funcionales y no funcionales, reglas de negocio, criterios de aceptación, casos de uso, modelo conceptual de datos, diagramas de estados y matrices de trazabilidad. |
| [`ADR_NAP.md`](ADR_NAP.md) | **Registro de Decisiones de Arquitectura (ADR)**, en formato MADR y alineado con ISO/IEC/IEEE 42010:2022. Es la *fuente de verdad* del *stack* tecnológico; cada decisión enlaza con los requisitos de la ERS que la motivan. |
| [`.markdownlint.json`](.markdownlint.json) | Configuración de *linting* de Markdown para mantener el formato de los documentos. |

## Stack tecnológico decidido

Resumen de las decisiones vigentes (ver [`ADR_NAP.md`](ADR_NAP.md) para el detalle y la justificación de cada una):

| Capa / Aspecto | Tecnología / Herramienta | ADR |
| --- | --- | --- |
| Estilo arquitectónico | Monolito modular (por *features*) | ADR-0001 |
| Backend | Spring Boot (Java) | ADR-0002 |
| Base de datos | PostgreSQL | ADR-0003 |
| Frontend | Angular | ADR-0004 |
| Estilos de interfaz | Tailwind CSS | ADR-0005 |
| Hosting de base de datos | Supabase (PostgreSQL gestionado) | ADR-0006, ADR-0007 |
| Hosting del backend | Render | ADR-0006 |
| Hosting del frontend | Vercel | ADR-0006 |
| Autenticación y autorización | Spring Security | ADR-0007 |
| Almacenamiento de archivos | Cloudflare R2 (compatible con S3) | ADR-0007 |

## Cómo leer la documentación

Los documentos están en **Markdown** y pueden leerse directamente en GitHub o en cualquier editor. Cada uno incluye su propia tabla de contenido e índice interno.

- Para entender **qué** debe hacer el sistema y bajo qué restricciones, empieza por la **ERS**.
- Para entender **con qué** tecnologías se construye y **por qué**, consulta el **ADR**.

### Linting de Markdown (opcional)

El repositorio incluye una configuración de [markdownlint](https://github.com/DavidAnson/markdownlint). Para validar el formato:

```bash
npx markdownlint-cli2 "**/*.md"
```

## Estado y convenciones

- El **ADR es la fuente de verdad** del *stack*. Cualquier cambio futuro se registra como una nueva ADR; las decisiones anteriores no se borran, sino que se marcan como *Obsoleta* o *Reemplazada*.
- La siguiente fase del proyecto es el **diseño**, que se abordará por requisito de forma incremental, incluyendo los diagramas UML correspondientes.

## Metadatos

| Campo | Detalle |
| --- | --- |
| **Empresa** | Nothing Sense (en formación) |
| **Autor / CTO** | Steven Ricardo Quiñones |
| **Revisor / CEO** | Fredinson Solano |
| **Ciudad** | Montería, Córdoba, Colombia |
| **Carácter** | Uso interno |
