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

## Demo

### Demo 1: Solicitud incompleta

Mostrar que detecta faltantes, mantiene `esperando_respuesta` y bloquea agendar.

### Demo 2: Prepago sin comprobante

Mostrar que “ya pagué” no es suficiente y solicita comprobante o referencia.

### Demo 3: Caso listo

Mostrar que con ubicación, equipo, contacto y referencia, pasa a `listo_para_agendar`.

## Duración sugerida

4 a 7 minutos.

# Post-mortem del proyecto

## Service Ops OS

**Agente IA para intake, priorización, control operativo y trazabilidad ISO en servicios técnicos**

---

## Objetivo del post-mortem

Este post-mortem resume qué salió bien, qué salió mal y qué aprendizajes deja el desarrollo de **Service Ops OS**. El objetivo es documentar el proceso de forma honesta para mejorar futuros proyectos de IA, automatización y sistemas operativos internos.

---

## What went well — Éxitos y aspectos positivos

### Evolución de una idea simple a un sistema más completo

El proyecto empezó como un **AI Intake Agent**, enfocado en transformar mensajes de clientes en solicitudes operativas claras.

Durante el desarrollo, la idea evolucionó hacia **Service Ops OS**, una solución más completa que combina análisis con IA, reglas de negocio, estados de workflow, validación de pago, procesamiento de respuestas, seguimiento SLA, trazabilidad orientada a procesos ISO y documentación técnica/funcional.

Esto permitió que el proyecto no se quedara solo como un agente conversacional, sino como una propuesta de sistema operativo para servicios técnicos.

### Enfoque en un problema real

Uno de los mayores aciertos fue enfocarme en un problema real: el intake manual de solicitudes en empresas de servicios técnicos.

Muchas empresas reciben solicitudes por correo, mensajes, llamadas o formularios. Estas solicitudes pueden llegar incompletas, ambiguas o sin la evidencia necesaria para avanzar.

Este enfoque hizo que la solución fuera más clara, útil y fácil de explicar.

### Decisión correcta: empezar por intake

Una decisión positiva fue no intentar automatizar todo el ciclo de servicio desde el inicio.

En lugar de construir un sistema demasiado amplio, decidí enfocarme en el punto donde se originan muchos errores:

**solicitud del cliente → interpretación → detección de faltantes → acción recomendada**

Esto hizo que el proyecto fuera más realista, más manejable y más fácil de validar.

### Separación entre IA y reglas de negocio

Otro acierto importante fue separar la función de la IA de las reglas del sistema.

La lógica principal quedó así:

**IA = analiza y recomienda**  
**Reglas de negocio = validan y bloquean**  
**Base de datos = fuente de verdad**  
**Operador = supervisa y ejecuta**

Esto hizo que el sistema fuera más confiable, especialmente en casos donde la IA podría interpretar una intención, pero el negocio necesita evidencia.

Por ejemplo, si el cliente dice “ya pagué”, el sistema no debe avanzar si no existe comprobante, referencia, recibo, transaction ID o evidencia verificable.

### Validación de pago como diferenciador fuerte

El caso de **prepago sin comprobante** se convirtió en una de las mejores demostraciones del valor del sistema.

Este caso muestra que Service Ops OS no solo responde o resume, sino que protege el flujo operativo.

Principio central:

**Service Ops OS no avanza por intención; avanza por evidencia.**

Este principio ayudó a conectar IA, reglas de negocio, prevención de errores y trazabilidad.

### Incorporación de trazabilidad ISO-oriented

Agregar una capa de trazabilidad orientada a procesos ISO fortaleció mucho el proyecto.

Aunque Service Ops OS no reemplaza una certificación ISO ni un sistema formal de gestión de calidad, sí ayuda a mejorar prácticas importantes como registro de comunicaciones, cambios de estado, evidencia de pago, razones de bloqueo, historial de decisiones y seguimiento SLA.

Esto hizo que el proyecto se sintiera más cercano a una solución empresarial real.

### Documentación completa

Otro punto positivo fue la calidad y profundidad de la documentación.

La entrega incluye README principal, arquitectura, reglas de negocio, prompt de sistema, SOP de operación, onboarding, FAQ, checklist QA, casos de prueba, KPIs, roadmap, guía para probar la versión publicada, explicación del proceso y decisiones, y demo script.

Esto convierte el proyecto en una entrega más clara para evaluación y también en una pieza útil para portafolio.

---

## What went wrong — Retos, fallos y dificultades

### El alcance creció mucho

Uno de los principales retos fue que el alcance del proyecto creció bastante.

La idea inicial era construir un agente de intake. Luego se sumaron Retool, base de datos, reglas de negocio, SLA, validación de pagos, trazabilidad ISO, documentación GitHub, demo, guía para evaluador y presentación.

Aunque esto fortaleció el proyecto, también aumentó la complejidad y el tiempo necesario para organizarlo.

### Mantener coherencia entre IA, estado y botones fue complejo

Uno de los desafíos técnicos fue mantener coherencia entre salida de IA, estado del caso, `ready_for_next_step`, `waiting_for`, acción recomendada y botón de agendamiento.

Por ejemplo:

**Si falta comprobante de pago:**  
**estado = esperando_respuesta**  
**ready_for_next_step = false**  
**accion_recomendada = SOLICITAR_PAGO**  
**Agendar servicio = deshabilitado**

Coordinar todas estas piezas fue más complejo que simplemente generar una respuesta con IA.

### La IA podía asumir información

Otro reto fue evitar que la IA interpretara frases ambiguas como datos confirmados.

Ejemplos problemáticos:

- “la planta de siempre”
- “el equipo de siempre”
- “ya pagué”
- “ustedes ya tienen eso”

En un flujo operativo real, esas frases no son suficientes. El sistema debe pedir confirmación o evidencia.

Esto obligó a reforzar reglas de no invención y validación.

### Diferencia entre versión publicada y app interna

Otro desafío fue que la versión publicada del agente no necesariamente muestra toda la integración interna de Retool.

El profesor puede probar la lógica del agente, el prompt, la detección de faltantes, la validación conceptual de pago, la priorización y la revisión humana.

Pero no necesariamente puede interactuar directamente con la base de datos interna, logs reales, botones de Retool, queries internas o automatizaciones completas.

Por eso fue necesario documentar claramente la diferencia entre:

**versión publicada = prueba del agente**  
**app interna documentada = evidencia técnica de integración**

### La documentación tomó más trabajo del esperado

La documentación fue más grande de lo previsto.

No bastaba con explicar qué hace el sistema. También fue necesario documentar por qué existe, cómo funciona, qué decisiones se tomaron, cómo se prueba, qué casos valida, qué criterios de evaluación cubre, cómo puede evolucionar y cómo debe revisarlo el profesor.

Esto tomó tiempo, pero mejoró mucho la calidad final del proyecto.

---

## Mistakes — Errores o decisiones que mejoraría

### Definir el alcance final más temprano

El proyecto habría sido más eficiente si desde el inicio hubiera definido claramente la diferencia entre MVP público, app interna, documentación, demo y roadmap futuro.

Esto habría evitado reorganizar partes del proyecto más adelante.

### Preparar screenshots durante el desarrollo

Otra mejora sería capturar screenshots desde el momento en que cada flujo funciona.

Por ejemplo:

- solicitud incompleta;
- prepago sin comprobante;
- caso listo para agendar;
- communications log;
- state log;
- SLA alert.

Esperar hasta el final hace más difícil reconstruir la evidencia visual.

### Separar antes la lógica pública de la lógica interna

La versión publicada y la app interna tienen propósitos diferentes.

Para futuros proyectos, separaría desde el inicio lo que el evaluador puede probar directamente, lo que se muestra como evidencia técnica y lo que queda como roadmap.

### Documentar decisiones técnicas mientras se construyen

Muchas decisiones importantes se entendieron mejor después de resolver problemas.

En futuros proyectos, documentaría antes por qué se crea cada campo, por qué se bloquea cada acción, qué error intenta prevenir cada regla y qué parte pertenece a IA versus qué parte pertenece a reglas.

---

## Key lessons learned — Aprendizajes clave

### La IA necesita estar conectada a un flujo real

Un agente de IA puede generar buenas respuestas, pero su valor aumenta cuando se conecta a una interfaz, una base de datos, reglas, estados, automatizaciones y trazabilidad.

El aprendizaje principal fue:

**La IA aporta más valor cuando opera dentro de un sistema.**

### No conviene automatizar todo desde el inicio

Intentar automatizar todo el ciclo de servicio habría hecho el proyecto demasiado amplio.

Fue mejor empezar por un punto crítico: **intake operativo**.

Resolver bien ese punto permitió construir una base más sólida para futuras automatizaciones.

### Las reglas de negocio son esenciales

En operaciones reales, la IA no debe tener control total.

Las reglas de negocio permiten bloquear avances incompletos, exigir evidencia, mantener coherencia, reducir errores y proteger el flujo operativo.

Esto es especialmente importante en casos de pago, SLA o procesos de calidad.

### La UX debe simplificar decisiones complejas

El operador no necesita ver toda la complejidad interna del sistema.

Necesita saber qué está pasando, qué falta, qué acción corresponde y si puede avanzar o no.

Campos como `accion_recomendada`, `waiting_for`, `estado`, `ready_for_next_step` y `mensaje_cliente` ayudan a simplificar la experiencia.

### La trazabilidad aumenta el valor del sistema

La trazabilidad no es solo documentación extra. Es parte del valor operativo.

Registrar comunicaciones, estados y evidencia permite responder preguntas como:

- ¿Qué se pidió?
- ¿Cuándo se pidió?
- ¿Por qué no avanzó el caso?
- ¿Existe comprobante?
- ¿Cuándo quedó listo?

Esto hace que el sistema sea más útil para empresas reales.

### Los casos de prueba deben representar riesgos reales

Los mejores casos de prueba no son los más simples, sino los que demuestran que el sistema previene errores.

Los casos más útiles fueron prepago sin comprobante, solicitud incompleta, ubicación ambigua, reclamo sensible y riesgo SLA.

Estos casos muestran si el sistema realmente aporta control.

### La documentación es parte del producto

El README, SOP, FAQ, QA, casos de prueba y roadmap no son solo anexos.

Son parte de la entrega porque explican cómo usar el sistema, cómo probarlo, cómo evaluarlo, cómo mantenerlo y cómo mejorarlo.

Para futuros proyectos, documentaría desde el primer día.

---

## Qué haría diferente en un próximo proyecto

En un próximo proyecto, haría lo siguiente:

1. Definir el alcance exacto del MVP desde el inicio.
2. Separar claramente versión pública, app interna y roadmap.
3. Preparar screenshots durante cada etapa funcional.
4. Crear casos de prueba antes de construir toda la lógica.
5. Documentar cada decisión técnica en el momento en que se toma.
6. Mantener una lista de riesgos del sistema desde el inicio.
7. Diseñar primero la experiencia del evaluador o tester.
8. Crear una demo mínima tan pronto el flujo principal funcione.

---

## Conclusión

El desarrollo de Service Ops OS dejó un aprendizaje claro: un agente de IA es más valioso cuando está integrado con reglas, datos, flujo de trabajo y trazabilidad.

El proyecto no solo demostró que la IA puede analizar solicitudes de clientes. También mostró que puede formar parte de un sistema operativo más controlado, donde las decisiones se validan con reglas y evidencia.

La mayor lección del proyecto fue:

**No basta con que la IA responda bien. El sistema debe ayudar a actuar mejor.**

Service Ops OS logró avanzar en esa dirección al combinar IA, Retool, reglas de negocio, workflow, validación de pago, SLA, trazabilidad, documentación y casos de prueba.

Este post-mortem servirá como referencia para mejorar futuros proyectos de IA aplicada a operaciones reales.

