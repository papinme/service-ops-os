
Sistema operativo de servicios asistido por IA para PYMEs de servicios técnicos. Convierte solicitudes desordenadas de clientes en casos claros, priorizados y trazables mediante Retool, IA, reglas de negocio, control de flujo, validación de pago, SLA y trazabilidad orientada a ISO.

# 01. Problema y oportunidad

## Contexto

Muchas PYMEs de servicios técnicos gestionan solicitudes a través de correos, mensajes, llamadas o formularios. Estas solicitudes suelen llegar incompletas, ambiguas o desordenadas.

## Problema principal

El intake operativo se realiza de forma manual, poco estructurada y con baja trazabilidad. Esto provoca errores administrativos, pérdida de información, mala priorización, retrasos, avance de casos sin datos mínimos y poca evidencia para revisión interna o auditoría.

## Causas

- Entrada de solicitudes por múltiples canales.
- Mensajes incompletos o ambiguos.
- Ausencia de formato estándar.
- Dependencia de revisión manual.
- Falta de reglas para bloquear casos incompletos.
- Poca trazabilidad de comunicaciones y estados.

## Consecuencias

- Pérdida de información crítica.
- Retrasos en atención.
- Agendamientos sin datos suficientes.
- Avance de casos sin comprobante cuando aplica prepago.
- Riesgo SLA.
- Falta de evidencia para auditoría.

## Oportunidad

Crear una capa de inteligencia operativa que transforme solicitudes desordenadas en casos claros, priorizados y trazables.

```text
Service Ops OS ayuda a PYMEs de servicios técnicos a transformar solicitudes no estructuradas de clientes en casos operativos claros, priorizados y trazables.
```


# 02. Arquitectura

## Objetivo

Conectar interfaz operativa, inteligencia artificial, reglas de negocio, base de datos y trazabilidad en un flujo funcional.

## Vista general

```text
Cliente → Canales de entrada → Retool App → AI Analysis Layer → Business Rules Layer → Workflow Control → Base de datos → Logs → Acción operativa
```

## Capas

1. Intake Layer.
2. AI Analysis Layer.
3. Business Rules Layer.
4. Workflow Control Layer.
5. ISO Traceability Layer.

## Retool

Componentes: tabla de casos, panel de detalle, acción recomendada, mensaje al cliente, Analizar con IA, Ejecutar, Enviar, Procesar respuesta, Agendar servicio y alertas SLA.

## Base de datos

Tablas: `cases`, `communications`, `state_log`, `evidence_log`.

## Principio

```text
Retool = interfaz y control operativo
IA = análisis y estructuración
SQL/JS = reglas y automatización
Base de datos = fuente de verdad
Logs = trazabilidad y evidencia
Operador = supervisión humana
```

# 03. Reglas de negocio

## Principio central

```text
La IA recomienda, pero las reglas de negocio validan y controlan.
```

## Estados

- `esperando_respuesta`
- `listo_para_agendar`
- `agendado`
- `revision_humana`

## Acciones

- `SOLICITAR_INFORMACION`
- `SOLICITAR_PAGO`
- `GENERAR_ORDEN_SERVICIO`
- `REQUIERE_REVISION_HUMANA`

## Reglas críticas

1. Si falta información crítica, el caso no avanza.
2. Si `pago = prepago` y no hay comprobante o referencia, el caso no avanza.
3. Si `ready_for_next_step = false`, agendar debe estar deshabilitado.
4. Respuestas ambiguas no completan el caso.
5. Casos con más de 48 horas sin seguimiento generan alerta SLA o prioridad alta.
6. Casos sensibles o contradictorios requieren revisión humana.
7. Toda comunicación relevante debe registrarse.
8. Todo cambio de estado debe registrarse.
9. No inventar datos críticos.

## Matriz rápida

| Condición | Acción | Estado | Ready |
|---|---|---|---|
| Falta ubicación | SOLICITAR_INFORMACION | esperando_respuesta | false |
| Prepago sin comprobante | SOLICITAR_PAGO | esperando_respuesta | false |
| Información completa + pago resuelto | GENERAR_ORDEN_SERVICIO | listo_para_agendar | true |
| Caso sensible | REQUIERE_REVISION_HUMANA | revision_humana | false |

