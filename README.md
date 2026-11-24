# beacon-contracts

Contratos compartidos de **Beacon**: definiciones Protobuf para gRPC y para los mensajes que viajan por NATS. Es la única fuente de verdad; ningún servicio define su propia copia.

## Contenido

| Archivo | Qué define |
|---|---|
| `proto/beacon/common/v1` | Enums compartidos (severidad, estados) |
| `proto/beacon/alerts/v1` | gRPC `AlertService` |
| `proto/beacon/incidents/v1` | gRPC `IncidentService` |
| `proto/beacon/notifications/v1` | Mensajes request/reply por NATS |
| `proto/beacon/events/v1` | Eventos pub/sub por NATS |
| `docs/nats-subjects.md` | Subjects, codificación y convenciones |

## Cómo se consumen

Cada repo fija una versión (tag) y genera sus stubs con Buf:

```bash
make proto   # dentro de cada servicio
```

| Repo | Lenguaje | Genera |
|---|---|---|
| `beacon-backend` | TypeScript (ts-proto + NestJS) | gRPC clientes + mensajes NATS |
| `beacon-alerts` | Go | gRPC servidor + cliente de Incidentes |
| `beacon-incidents` | Java | gRPC servidor + eventos NATS |
| `beacon-notifications` | C# | Eventos y mensajes NATS |

## Desarrollo

```bash
buf format -w   # formatear
buf lint        # validar estilo
buf generate --template buf.gen.verify.yaml   # probar generación
```

## Versionado

Semantic Versioning por tags. En `v1` solo se agregan campos o mensajes; nunca se renombra ni se borra nada. Cambios incompatibles requieren un paquete `v2`.

Los tags son la fuente de verdad y **un consumidor nunca apunta a un tag que no exista**: primero se publica el tag aquí, después se sube `CONTRACTS_TAG` en un PR del repo consumidor. Si el consumidor necesita un campo nuevo, el bump y el código que lo usa van en el mismo PR.

| Repo | `CONTRACTS_TAG` | Por qué |
|---|---|---|
| `beacon-incidents` | `v0.2.0` → `v0.3.0` | publica `incident.alert_attached`; el bump va en su PR de Fase 3 |
| `beacon-backend` | `v0.2.0` → `v0.3.0` | mapea ese evento a las subscriptions de GraphQL; el bump va en su PR de Fase 3 |
| `beacon-alerts` | `v0.2.0` | solo habla gRPC con Incidentes; no usa los eventos nuevos |
| `beacon-notifications` | `v0.2.0` | usa `Team`, `ListTeams*` y `ListSchedules*` desde `v0.2.0` |

Al publicar `v0.3.0`, solo `beacon-incidents` y `beacon-backend` necesitan el bump.
`beacon-alerts` y `beacon-notifications` pueden quedarse en `v0.2.0` porque no
consumen `incident.alert_attached`.

Para publicar una versión:

```bash
git tag v0.3.0
git push origin v0.3.0
```
