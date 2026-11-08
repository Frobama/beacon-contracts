# Subjects de NATS

Sin JetStream: usamos NATS core (pub/sub y request/reply). Los mensajes no se persisten; si un consumidor está caído, no recibe el evento. Notificaciones reconcilia al arrancar consultando los incidentes abiertos sin reconocer.

## Eventos (pub/sub)

| Subject | Mensaje | Publica | Escucha |
|---|---|---|---|
| `incident.opened` | `IncidentOpened` | Incidentes | Notificaciones, Orquestador |
| `incident.acknowledged` | `IncidentAcknowledged` | Incidentes | Notificaciones, Orquestador |
| `incident.resolved` | `IncidentResolved` | Incidentes | Notificaciones, Orquestador |
| `incident.severity_changed` | `IncidentSeverityChanged` | Incidentes | Orquestador |
| `escalation.triggered` | `EscalationTriggered` | Notificaciones | Incidentes, Orquestador |
| `notification.sent` | `NotificationSent` | Notificaciones | Incidentes, Orquestador |

El orquestador escucha los eventos para alimentar las *subscriptions* de GraphQL.
Incidentes escucha `escalation.triggered` y `notification.sent` para dejarlos en el timeline.

## Comandos y consultas (request/reply)

Solo el orquestador hace requests; solo Notificaciones responde.

| Subject | Request | Response |
|---|---|---|
| `notifications.oncall.get` | `GetOnCallRequest` | `GetOnCallResponse` |
| `notifications.policies.list` | `ListPoliciesRequest` | `ListPoliciesResponse` |
| `notifications.schedule.create` | `CreateScheduleRequest` | `CreateScheduleResponse` |

Si Notificaciones corre con varias réplicas, todas se suscriben con el *queue group* `notifications` para que solo una atienda cada request.

## Codificación

- El payload es el **JSON canónico de Protobuf** (no binario), para poder depurar con `nats sub`.
- Go: `protojson`. Java: `JsonFormat`. C#: `JsonFormatter` / `JsonParser`. TypeScript: `fromJSON` / `toJSON` (ts-proto).
- Content-Type informal: header `Content-Type: application/json`.

## Errores en request/reply

Si el responder falla, contesta con payload vacío y los headers:

- `Beacon-Error-Code`: `NOT_FOUND`, `INVALID_ARGUMENT`, `INTERNAL`, etc.
- `Beacon-Error-Message`: texto descriptivo.

El que hace la request debe revisar estos headers antes de decodificar el payload.

## Convenciones

- Subjects en minúsculas, palabras separadas por `.`, y snake_case dentro de cada token si hace falta.
- Eventos: `<entidad>.<verbo en pasado>`.
- Comandos y consultas: `<servicio>.<recurso>.<acción>`.
- Evolución: solo se agregan campos nuevos. Un cambio incompatible exige un paquete `v2`.

## Ruteo servicio → equipo

El evento `incident.opened` trae el campo `service`. Notificaciones mantiene su propia configuración que mapea servicio a equipo de guardia.
