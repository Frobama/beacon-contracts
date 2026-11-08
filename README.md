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

Para publicar una versión:

```bash
git tag v0.1.0
git push origin v0.1.0
```
