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


# 04. Prompt de sistema

## Prompt principal

```text
Eres Service Ops OS, un agente de IA especializado en intake, priorización y control operativo de solicitudes de servicios técnicos para pequeñas y medianas empresas.

Tu función es ayudar a personal administrativo y coordinadores operativos a transformar correos, mensajes o notas de clientes en casos operativos claros, priorizados, trazables y listos para gestionar.

No eres un chatbot general. Debes analizar la solicitud y devolver una salida estructurada, breve, operativa y útil para un sistema interno.

Reglas críticas:
1. No inventes información.
2. No asumas ubicación.
3. No asumas pago confirmado.
4. Si pago = prepago y no hay comprobante o referencia, ready_for_next_step debe ser false.
5. Si falta información crítica, el caso debe quedar esperando respuesta.
6. Si hay ambigüedad, contradicción o baja confianza, marca revisión humana.
7. Responde únicamente con JSON válido.

Formato obligatorio:
{
  "tipo_solicitud": "",
  "resumen_operativo": "",
  "datos_identificados": {
    "cliente": "",
    "equipo": "",
    "ubicacion": "",
    "contacto": "",
    "fecha_requerida": "",
    "tipo_cliente": "",
    "condicion_pago": "",
    "evidencia_pago": ""
  },
  "informacion_faltante": "",
  "prioridad_sugerida": "alta | media | baja",
  "nivel_confianza": "alto | medio | bajo",
  "accion_recomendada": "SOLICITAR_INFORMACION | SOLICITAR_PAGO | GENERAR_ORDEN_SERVICIO | REQUIERE_REVISION_HUMANA",
  "estado_sugerido": "esperando_respuesta | listo_para_agendar | revision_humana",
  "ready_for_next_step": false,
  "human_review_required": false,
  "razon_decision": "",
  "mensaje_cliente": ""
}
```

## Validación posterior en Retool

Aunque la IA devuelva `ready_for_next_step`, Retool debe validar nuevamente con reglas de negocio.


# 05. SOP de operación

## Objetivo

Asegurar que Service Ops OS se use de manera clara, consistente y controlada.

## Flujo estándar

```text
1. Recibir solicitud.
2. Crear o seleccionar caso.
3. Ejecutar Analizar con IA.
4. Revisar salida.
5. Validar reglas.
6. Ejecutar acción permitida.
7. Enviar o registrar mensaje.
8. Procesar respuesta.
9. Actualizar estado.
10. Avanzar solo si el caso está listo.
11. Registrar evidencia.
```

## Protocolos clave

### Pago pendiente

Si requiere prepago: pedir comprobante o referencia, mantener `ready_for_next_step = false` y agendar deshabilitado.

### Información faltante

Pedir solo datos críticos para avanzar.

### SLA

Si pasan más de 48 horas sin seguimiento, mostrar alerta y elevar prioridad.

### Revisión humana

Escalar casos sensibles, contradictorios, ambiguos o fuera de alcance.


# 06. Onboarding de usuario

## Promesa

Service Ops OS te ayuda a convertir correos y mensajes de clientes en casos operativos claros, priorizados y trazables.

## Cuándo usarlo

- Mantenimiento.
- Calibración.
- Reparación.
- Soporte técnico.
- Coordinación de servicio.
- Seguimiento de cliente.
- Validación de pago.
- Riesgo SLA.

## Primer flujo

```text
1. Selecciona o crea un caso.
2. Revisa el contexto.
3. Ejecuta Analizar con IA.
4. Revisa la salida.
5. Ejecuta la acción permitida.
6. Registra o envía mensaje.
7. Procesa respuesta del cliente.
8. Avanza solo si ready_for_next_step = true.
```

## Regla clave

```text
El sistema no avanza por intención; avanza por evidencia.
```
arding_usuario.md…]()

# 07. FAQ y soporte básico

## Preguntas frecuentes

### ¿Qué es Service Ops OS?

Un sistema asistido por IA que transforma solicitudes de clientes en casos operativos claros, priorizados y trazables.

### ¿Toma decisiones automáticamente?

No completamente. La IA recomienda, las reglas validan y el operador supervisa.

### ¿Puedo agendar si el cliente dice que pagó?

No, si requiere prepago y no hay comprobante o referencia.

### ¿Reemplaza un sistema ISO?

No. Ayuda a fortalecer trazabilidad y control documental, pero no reemplaza un sistema formal de gestión de calidad.

## Soporte

Para reportar errores incluir: ID del caso, descripción, input, salida IA, estado esperado, estado real, captura y fecha.

## Escalamiento

Escalar a revisión humana si hay reclamo, riesgo SLA, conflicto comercial, baja confianza, salida inconsistente o caso sensible.
_faq_soporte.md…]()


# 08. Checklist QA

## Output IA

| Criterio | Puntaje |
|---|---|
| Claridad | 0/1 |
| Estructura | 0/1 |
| Acción útil | 0/1 |
| Coherencia con rol | 0/1 |
| Priorización razonable | 0/1 |
| Detección de faltantes | 0/1 |
| Límites claros | 0/1 |
| No invención | 0/1 |

Umbral: 7/8 o 8/8 = válido.

## Reglas

- Prepago sin comprobante bloqueado.
- Casos incompletos bloqueados.
- Agendar deshabilitado si ready=false.
- SLA visible si >48h.
- Casos sensibles escalados.
- Estados coherentes.

## Fallos críticos

- Agendar prepago sin comprobante.
- Marcar listo un caso incompleto.
- Inventar ubicación, pago, equipo o contacto.
- No registrar comunicaciones importantes.
_checklist_qa.md…]()

[10_kpis_validacion.md](https://github.com/user-attachments/files/27959442/10_kpis_validacion.md)
# 09. Casos de prueba

| Caso | Escenario | Acción esperada | Estado | Ready |
|---|---|---|---|---|
| 1 | Solicitud clara | GENERAR_ORDEN_SERVICIO | listo_para_agendar | true |
| 2 | Solicitud incompleta | SOLICITAR_INFORMACION | esperando_respuesta | false |
| 3 | Prepago sin comprobante | SOLICITAR_PAGO | esperando_respuesta | false |
| 4 | Prepago con referencia | GENERAR_ORDEN_SERVICIO | listo_para_agendar | true |
| 5 | Respuesta parcial | SOLICITAR_INFORMACION | esperando_respuesta | false |
| 6 | Respuesta completa | GENERAR_ORDEN_SERVICIO | listo_para_agendar | true |
| 7 | Riesgo SLA | seguimiento / alerta | esperando_respuesta | según caso |
| 8 | Reclamo sensible | REQUIERE_REVISION_HUMANA | revision_humana | false |

## Demo prioritaria

1. Solicitud incompleta.
2. Prepago sin comprobante.
3. Caso listo para agendar.
4. Riesgo SLA opcional.

# 10. KPIs de validación

| KPI | Meta |
|---|---|
| Tiempo medio de procesamiento | Mejora ≥ 30% |
| Primer resultado útil | ≥ 80% |
| Precisión operativa percibida | ≥ 75% |
| Detección correcta de faltantes | ≥ 80% |
| Claridad y confianza | ≥ 4/5 |
| Tasa de repetición de uso | ≥ 60% |
| Bloqueo correcto de prepago | ≥ 90%, ideal 100% |
| Habilitación correcta de agendamiento | ≥ 95% |
| Registro de comunicaciones | ≥ 90% |
| Detección SLA | ≥ 80% |

## Criterio de MVP exitoso

El MVP es exitoso si reduce tiempo, detecta faltantes, bloquea pagos sin evidencia, genera outputs válidos y conserva trazabilidad básica.[12_presentacion_demo.md](https://github.com/user-attachments/files/27959461/12_presentacion_demo.md)



# 11. Roadmap

```text
Fase 0: Concepto y oportunidad
  ↓
Fase 1: AI Intake Agent MVP
  ↓
Fase 2: Service Ops OS MVP en Retool
  ↓
Fase 3: v1.1 operable — SOP + QA + onboarding
  ↓
Fase 4: Piloto controlado
  ↓
Fase 5: Integración con correo
  ↓
Fase 6: Dashboard SLA
  ↓
Fase 7: Evidence Log + trazabilidad ISO
  ↓
Fase 8: Calendario y agendamiento asistido
  ↓
Fase 9: CRM / ERP / órdenes de servicio
  ↓
Fase 10: Mayor autonomía con supervisión
```

Principio final:

```text
Más autonomía solo después de más evidencia, más control y más confianza.
```
# 12. Presentación y demo final

## Estructura de 12 slides

1. Service Ops OS.
2. Problema.
3. Usuario y cliente ideal.
4. Solución propuesta.
5. Arquitectura.
6. Flujo operativo.
7. Reglas de negocio.
8. Enfoque ISO.
9. Demo funcional.
10. Validación y KPIs.
11. Roadmap.
12. Cierre.

## Demo

### Demo 1: Solicitud incompleta

Mostrar que detecta faltantes, mantiene `esperando_respuesta` y bloquea agendar.

### Demo 2: Prepago sin comprobante

Mostrar que “ya pagué” no es suficiente y solicita comprobante o referencia.

### Demo 3: Caso listo

Mostrar que con ubicación, equipo, contacto y referencia, pasa a `listo_para_agendar`.

## Duración sugerida

4 a 7 minutos.

