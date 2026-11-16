# Plataforma de Banca Digital: arquitectura de solución

Propuesta de arquitectura para la plataforma de banca digital (web y móvil) de Banco Ecuador: consulta de movimientos, transferencias propias e interbancarias y onboarding digital con reconocimiento facial.

## Contenido

| Entregable | Archivo |
|------------|---------|
| Documento completo en PDF (decisiones, consideraciones técnicas y anexo de diagramas) | [Arquitectura-Banca-Digital.pdf](Arquitectura-Banca-Digital.pdf) |
| Decisiones de diseño y justificaciones | [docs/decisiones-de-diseno.md](docs/decisiones-de-diseno.md) |
| Consideraciones técnicas (normativa, alta disponibilidad, DR, seguridad, escalabilidad, cloud) | [docs/consideraciones-tecnicas.md](docs/consideraciones-tecnicas.md) |
| C4 nivel 1: contexto | [diagrams/img/c4-1-contexto.png](diagrams/img/c4-1-contexto.png) |
| C4 nivel 2: contenedores | [diagrams/img/c4-2-contenedores.png](diagrams/img/c4-2-contenedores.png) |
| C4 nivel 3: componentes, API Gateway y BFF | [diagrams/img/c4-3-componentes-gateway-bff.png](diagrams/img/c4-3-componentes-gateway-bff.png) |
| C4 nivel 3: componentes, Servicio de Transferencias | [diagrams/img/c4-3-componentes-transferencias.png](diagrams/img/c4-3-componentes-transferencias.png) |
| C4 nivel 3: componentes, Servicio de Clientes y caché | [diagrams/img/c4-3-componentes-clientes-cache.png](diagrams/img/c4-3-componentes-clientes-cache.png) |
| C4 nivel 3: componentes, Onboarding | [diagrams/img/c4-3-componentes-onboarding.png](diagrams/img/c4-3-componentes-onboarding.png) |
| C4 nivel 3: componentes, Notificaciones y Auditoría | [diagrams/img/c4-3-componentes-notificaciones-auditoria.png](diagrams/img/c4-3-componentes-notificaciones-auditoria.png) |
| Infraestructura cloud (AWS) | [diagrams/img/infra-aws.png](diagrams/img/infra-aws.png) |
| Flujo de autenticación (OAuth 2.0, biometría, step-up) | [diagrams/img/flujo-login.png](diagrams/img/flujo-login.png) |
| Flujo de transferencia interbancaria | [diagrams/img/flujo-transferencia.png](diagrams/img/flujo-transferencia.png) |
| Flujo de onboarding | [diagrams/img/flujo-onboarding.png](diagrams/img/flujo-onboarding.png) |

Cada diagrama está en PNG y en SVG (vectorial, recomendado para ampliar) en [`diagrams/img`](diagrams/img).

## Resumen de la solución

- **Estilo**: microservicios orientados a eventos sobre Kubernetes (AWS EKS), con el Core Bancario y la base principal como sistema de registro en el datacenter del banco, detrás de una capa anticorrupción.
- **Entrada**: Kong como API Gateway y un BFF que orquesta las llamadas a Clientes, Movimientos y Transferencias. Se agregan Onboarding, Notificaciones, Auditoría y Riesgo.
- **Identidad**: Keycloak con OAuth 2.0 Authorization Code + PKCE (PAR y DPoP), autenticación reforzada para operaciones sensibles, biometría local ligada a la clave del dispositivo y KYC con proveedor especializado.
- **Datos**: base por servicio, modelo de lectura alimentado por CDC, caché Redis con precarga para clientes frecuentes y auditoría inmutable (PostgreSQL con cadena de hashes y archivo S3 Object Lock).
- **Transferencias**: patrón Strategy por tipo, saga orquestada con idempotencia y conciliación.
- **Notificaciones**: push y SMS como canales obligatorios, email e in-app complementarios, disparados por los movimientos del Core.
- **Frontend**: React + React Native (recomendado) o Angular + Flutter.
- **Resiliencia**: tres zonas de disponibilidad, degradación controlada, DR en segunda región con warm standby.

## Herramienta de modelado

Los diagramas están escritos como texto con [Mermaid](https://mermaid.js.org) y se versionan junto al código en [`diagrams/src`](diagrams/src). Siguen la notación C4 (colores: persona, sistema o contenedor, componente, base de datos, sistema externo).

Para regenerar las imágenes:

```bash
npm install -g @mermaid-js/mermaid-cli
mmdc -i diagrams/src/c4-2-contenedores.mmd -o diagrams/img/c4-2-contenedores.png -c diagrams/mermaid-config.json -s 2 -b white
```

## Supuestos y alcance

Los supuestos de partida y los riesgos abiertos están en la sección 1 y la sección 10 de [decisiones-de-diseno.md](docs/decisiones-de-diseno.md). Las referencias normativas deben ser confirmadas por Legal y Cumplimiento antes de su adopción formal.
