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

El MVP es exitoso si reduce tiempo, detecta faltantes, bloquea pagos sin evidencia, genera outputs válidos y conserva trazabilidad básica.

## Demo

### Demo 1: Solicitud incompleta

Mostrar que detecta faltantes, mantiene `esperando_respuesta` y bloquea agendar.

### Demo 2: Prepago sin comprobante

Mostrar que “ya pagué” no es suficiente y solicita comprobante o referencia.

### Demo 3: Caso listo

Mostrar que con ubicación, equipo, contacto y referencia, pasa a `listo_para_agendar`.

## Duración sugerida

4 a 7 minutos.


# 11.Proceso, decisiones de diseño y alineación con criterios de evaluación

## Objetivo de esta sección

En esta sección explico el proceso que seguí para construir Service Ops OS y las principales decisiones que tomé durante el desarrollo del proyecto.

Mi objetivo es mostrar que el proyecto no fue solamente una implementación técnica, sino un proceso completo de análisis de problema, diseño de producto, arquitectura, experiencia de usuario, uso de inteligencia artificial, automatización, validación y documentación.

## Cómo empezó el proyecto

El proyecto empezó a partir de un problema real en empresas de servicios técnicos: muchas solicitudes de clientes llegan por correo, mensajes, llamadas o formularios, pero no llegan de forma estructurada.

En ese contexto, el personal administrativo debe leer cada mensaje, interpretar qué pide el cliente, identificar si falta información, decidir prioridad, responder al cliente y preparar el caso para que la operación pueda avanzar.

Detecté que muchos errores no ocurren al final del proceso, sino al inicio. Si el intake está incompleto, si se pierde un correo, si no se pide una ubicación exacta o si se avanza sin comprobante de pago, todo el flujo posterior queda en riesgo.

Por eso decidí enfocar el proyecto en ese punto crítico: el intake operativo.

## Evolución de la idea inicial

La idea inicial fue crear un **AI Intake Agent**, es decir, un agente de IA capaz de transformar mensajes de clientes en solicitudes operativas claras.

Al principio, el enfoque era más simple:

```text
mensaje del cliente → análisis IA → solicitud estructurada
```

Pero durante el desarrollo entendí que, para que el agente tuviera más valor operativo, no bastaba con resumir o estructurar información. El sistema también debía ayudar a controlar el flujo.

Por eso evolucioné la idea hacia **Service Ops OS**, una solución más completa que combina:

- intake con IA;
- detección de faltantes;
- reglas de negocio;
- estados de workflow;
- validación de pago;
- procesamiento de respuestas;
- alertas SLA;
- trazabilidad orientada a procesos ISO.

La evolución fue:

```text
AI Intake Agent → Service Ops OS
```

Esto convirtió el proyecto de un agente conversacional en una propuesta de sistema operativo para servicios técnicos.


## Decisión 1: Enfocarme primero en el intake operativo

Una de las primeras decisiones fue no intentar automatizar toda la operación desde el inicio.

Pude haber planteado un sistema que asignara técnicos, confirmara citas, enviara correos automáticamente y gestionara todo el ciclo de servicio. Sin embargo, eso habría sido demasiado amplio para un MVP y habría aumentado el riesgo de errores.

Decidí empezar por el intake porque es el punto donde se originan muchos problemas:

- solicitudes incompletas;
- información ambigua;
- falta de ubicación;
- falta de contacto;
- falta de detalle técnico;
- mala priorización;
- seguimiento manual;
- pagos no confirmados.

Mi decisión fue resolver primero esta pregunta:

```text
¿Cómo convierto una solicitud desordenada en un caso operativo claro, priorizado y controlado?
```

Esta decisión hizo que el proyecto fuera más realista, más enfocado y más fácil de validar.


## Decisión 2: Usar Retool como base de la aplicación

Elegí Retool porque necesitaba una herramienta que me permitiera construir una aplicación operativa funcional sin desarrollar todo desde cero.

Retool me permitió combinar:

- tabla de casos;
- panel de detalle;
- botones de acción;
- conexión con base de datos;
- consultas SQL;
- lógica JavaScript;
- integración con IA;
- estados y validaciones.

Mi objetivo no era solo mostrar un prompt funcionando, sino demostrar cómo un agente de IA puede integrarse dentro de una herramienta operativa real.

Por eso tomé un enfoque **Retool-first**.

La arquitectura resultante fue:

```text
Retool = interfaz y control operativo
IA = análisis y estructuración
SQL/JS = reglas y automatización
Base de datos = fuente de verdad
Logs = trazabilidad
Operador = supervisión humana
```

## Decisión 3: Separar la IA de las reglas de negocio

Una decisión técnica clave fue separar lo que hace la IA de lo que hacen las reglas del sistema.

La IA puede analizar un mensaje y recomendar una acción, pero no debe tener autoridad final para mover un caso si falta información crítica o evidencia obligatoria.

Por eso definí esta lógica:

```text
IA analiza y recomienda.
Reglas de negocio validan y bloquean.
La base de datos conserva la verdad del caso.
El operador supervisa y ejecuta.
```

Esta separación fue muy importante porque reduce el riesgo de depender ciegamente del modelo.

Por ejemplo, si el cliente escribe:

```text
Ya pagué.
```

la IA puede entender que el cliente afirma haber pagado. Pero la regla de negocio debe validar si existe:

```text
comprobante
número de referencia
recibo
transaction ID
evidencia verificable
```

Si no existe evidencia, el caso no debe avanzar.

Esta decisión dio origen a uno de los principios principales del proyecto:

```text
Service Ops OS no avanza por intención; avanza por evidencia.
```


## Decisión 4: Usar `ready_for_next_step` como señal central

Durante el diseño del flujo, necesitaba una forma simple de indicar si un caso podía avanzar o no.

Por eso usé el campo:

```text
ready_for_next_step
```

Este campo resume el estado operativo del caso:

```text
ready_for_next_step = false → el caso no puede avanzar
ready_for_next_step = true → el caso puede avanzar al siguiente paso
```

Esta decisión ayudó a simplificar la experiencia del operador. En lugar de revisar muchos campos manualmente, el sistema puede mostrar claramente si el caso está bloqueado o listo.

También permite controlar botones como:

```text
Agendar servicio
```

Si el caso no está listo, el botón debe permanecer deshabilitado.

Esto conecta la lógica técnica con la experiencia de usuario.

---

## Decisión 5: Mantener una interfaz simple y en español

Decidí mantener la interfaz en español porque el usuario final del sistema es personal administrativo o coordinador operativo que trabajaría en ese idioma.

Usé etiquetas claras como:

```text
Analizar con IA
Ejecutar
Enviar al cliente
Procesar respuesta cliente
Agendar servicio
Acción Recomendada
Mensaje al cliente
Esperando respuesta
Listo para agendar
```

La intención fue evitar una interfaz demasiado técnica. El operador no debería tener que entender el prompt, el JSON o la lógica interna para usar el sistema.

La interfaz debía responder preguntas simples:

```text
¿Qué está pasando con este caso?
¿Qué falta?
¿Qué debo hacer ahora?
¿Puedo avanzar o no?
```

---

## Decisión 6: Usar casos de prueba basados en riesgos reales

Para validar el sistema, decidí usar casos que representaran riesgos reales de operación, no solamente ejemplos simples.

Los casos principales fueron:

```text
1. Solicitud incompleta
2. Cliente prepago sin comprobante
3. Caso completo listo para agendar
4. Respuesta parcial del cliente
5. Riesgo SLA
6. Reclamo sensible
7. Ubicación ambigua
```

Elegí estos casos porque prueban si el sistema realmente ayuda a prevenir errores.

El caso más importante es el de prepago sin comprobante, porque demuestra que el sistema no debe permitir avanzar solo porque el cliente dice que pagó.


## Decisión 7: Incluir trazabilidad ISO-oriented

Durante el desarrollo identifiqué que empresas de servicios técnicos no solo necesitan velocidad. También necesitan evidencia, consistencia y control.

Por eso incorporé el enfoque de trazabilidad orientada a procesos ISO.

Service Ops OS no reemplaza un sistema de gestión de calidad ni una certificación ISO. Sin embargo, ayuda a fortalecer prácticas importantes como:

- registrar comunicaciones;
- registrar cambios de estado;
- conservar evidencia de pago;
- documentar por qué un caso fue bloqueado;
- mantener historial de acciones recomendadas;
- identificar casos en riesgo SLA.

Esta decisión hizo que el proyecto fuera más sólido como solución para empresas reales, no solo como ejercicio académico.


## 12. Principales desafíos técnicos

### 12.1 Mantener coherencia entre IA, reglas y estado

Uno de los desafíos fue asegurar que la salida de la IA coincidiera con el estado final del caso.

Por ejemplo:

```text
Si la IA detecta información faltante,
el estado debe ser esperando_respuesta
y ready_for_next_step debe ser false.
```

### 12.2 Evitar que el sistema avanzara con pago no confirmado

Otro desafío fue controlar el caso donde el cliente dice que pagó, pero no entrega evidencia.

Este caso era importante porque representa un error administrativo real.

### 12.3 Procesar respuestas del cliente

El sistema debía poder procesar una respuesta del cliente y decidir si los faltantes se resolvieron o si todavía faltaba información.

### 12.4 Manejar JSON de la IA

La salida del modelo debía ser estructurada para que Retool pudiera leerla y actualizar campos del caso.

### 12.5 Controlar botones y visibilidad

El botón de agendamiento debía reflejar correctamente el estado del caso. Esto fue importante para evitar que el operador saltara pasos.

---

## 13. Principales desafíos de UX

### 13.1 Evitar saturar al operador

El sistema maneja mucha lógica interna, pero el operador necesita una vista simple.

Por eso organicé la interfaz alrededor de:

```text
Contexto
Acción Recomendada
Mensaje al cliente
Estado
Faltantes
```

### 13.2 Hacer visible el bloqueo

Si un caso no puede avanzar, el usuario debe entender por qué.

Por eso el sistema muestra campos como:

```text
waiting_for
ready_for_next_step
estado
accion_recomendada
```

### 13.3 Usar lenguaje operativo

Decidí usar lenguaje directo y orientado a acción, por ejemplo:

```text
Solicitar información
Solicitar pago
Generar orden de servicio
Listo para agendar
```

---

## 14. Principales desafíos con IA

### 14.1 Evitar invenciones

El agente debía evitar inventar datos como ubicación, contacto, equipo o pago.

### 14.2 Pedir solo información necesaria

El agente no debía pedir demasiados datos si solo faltaban elementos críticos.

### 14.3 Mantener formato estructurado

Para integrarse con Retool, la respuesta debía estar en formato estructurado, idealmente JSON.

### 14.4 Detectar casos sensibles

El agente debía reconocer situaciones donde no debía responder como si fuera un caso estándar, por ejemplo reclamos o amenazas de escalamiento.

---

## 15. Aprendizajes principales

Los principales aprendizajes del proyecto fueron:

1. Un agente de IA es más útil cuando está conectado a un flujo operativo real.
2. La IA necesita reglas de negocio para ser confiable en procesos empresariales.
3. El control de estados es tan importante como la respuesta generada.
4. La UX debe hacer visible qué falta y qué acción corresponde.
5. La trazabilidad aumenta el valor del sistema en empresas con procesos de calidad.
6. La documentación es parte del producto, no solo un requisito académico.
7. La autonomía debe crecer gradualmente y siempre con evidencia.

---

## 16. Alineación con criterios de evaluación

| Criterio de evaluación | Cómo lo cubre mi proyecto | Evidencia |
|---|---|---|
| Completitud técnica | Integro interfaz, IA, base de datos, reglas, estados y documentación técnica | README, arquitectura, exports, screenshots |
| Innovación y utilidad | Resuelvo un problema real de intake, pago, SLA y trazabilidad en servicios técnicos | propuesta de valor, casos de prueba, demo |
| Calidad de interfaz de usuario | Diseño una interfaz con tabla de casos, panel operativo, mensajes y botones controlados | screenshots del dashboard y panel |
| Profundidad en uso de IA | Uso prompt estructurado, salida JSON, detección de faltantes, prioridad, mensajes y revisión humana | prompt de sistema, pruebas, documentación IA |
| Eficacia de automatizaciones | Automatizo análisis, actualización de estado, generación de mensaje, procesamiento de respuesta y bloqueo de agendamiento | automation_logic, demo, screenshots |
| Calidad de documentación | Incluyo README, docs, SOP, FAQ, QA, KPIs, roadmap y guía de prueba | carpeta docs |
| Calidad de diapositivas/presentación | Preparo estructura de presentación, guion de demo y narrativa de valor | presentación, demo script, screenshots |

---

## 17. Cómo explicaría mi proceso en la presentación

En la presentación, resumiría mi proceso así:

```text
Primero identifiqué un problema real: el intake manual de solicitudes en empresas de servicios técnicos.

Luego diseñé un agente de IA para estructurar esas solicitudes.

Después evolucioné la idea hacia una aplicación en Retool, porque quería demostrar integración real con interfaz, base de datos, reglas y workflow.

Durante el desarrollo separé la IA de las reglas de negocio para reducir riesgos.

Finalmente agregué validación de pago, control de faltantes, SLA, trazabilidad ISO-oriented, documentación y casos de prueba.
```

---

## 18. Diapositiva sugerida: Proceso y decisiones de diseño

Título:

```text
Proceso y decisiones de diseño
```

Contenido sugerido:

```text
1. Identifiqué el problema: intake manual y errores administrativos.
2. Definí el usuario: operador administrativo/coordinador.
3. Diseñé un agente inicial de intake.
4. Evolucioné hacia Service Ops OS en Retool.
5. Separé IA de reglas de negocio.
6. Incorporé pago, faltantes, SLA y trazabilidad.
7. Documenté el sistema para GitHub, demo y futura validación.
```

---

## 19. Diapositiva sugerida: Desafíos y aprendizajes

Título:

```text
Desafíos y aprendizajes
```

| Desafío | Decisión tomada |
|---|---|
| La IA podía asumir información | Definí reglas de no invención |
| El cliente podía decir “ya pagué” sin evidencia | Exigí comprobante o referencia |
| El operador podía avanzar por error | Usé `ready_for_next_step` para controlar acciones |
| El flujo podía perder trazabilidad | Incorporé communications, state_log y evidence_log |
| La interfaz podía ser compleja | Organicé la UI en contexto, acción, mensaje y estado |

Frase de cierre:

```text
El aprendizaje principal fue que la IA debe trabajar dentro de un sistema con reglas, evidencia y supervisión humana.
```

---

## 20. Diapositiva sugerida: Alineación con evaluación

Título:

```text
Alineación con criterios de evaluación
```

Contenido sugerido:

```text
Completitud técnica → Retool + IA + base de datos + workflow
Innovación y utilidad → intake + pago + SLA + trazabilidad ISO
UI → dashboard + panel operativo + botones controlados
IA → prompt estructurado + JSON + lógica de decisión
Automatización → análisis, ejecución, envío y procesamiento
Documentación → README + docs + QA + roadmap
Presentación → demo guiada con casos reales simulados
```

---

## 21. Texto para incluir en README

```markdown
## Proceso y decisiones de diseño

El desarrollo de Service Ops OS partió de un problema operativo real: muchas PYMEs de servicios técnicos gestionan solicitudes de clientes de forma manual, lo que genera errores, pérdida de información y poca trazabilidad.

La primera decisión fue enfocarme en el intake operativo, porque es el punto donde se originan muchos errores posteriores. En lugar de automatizar toda la operación desde el inicio, el proyecto se centró en transformar mensajes desordenados en casos claros, priorizados y listos para gestionar.

Durante el desarrollo, el proyecto evolucionó desde un AI Intake Agent hacia Service Ops OS, una aplicación operativa en Retool con integración de IA, base de datos, reglas de negocio, estados de workflow y trazabilidad.

Una decisión técnica clave fue separar la recomendación de IA del control operativo. La IA analiza y sugiere, pero las reglas de negocio validan si el caso puede avanzar. Esto permite bloquear casos incompletos, exigir comprobante cuando aplica prepago y evitar que el sistema avance solo por intención del cliente.

Desde el punto de vista UX, prioricé una interfaz simple para el operador administrativo: tabla de casos, panel de contexto, acción recomendada, mensaje al cliente y botones de flujo. El campo `ready_for_next_step` se convirtió en una señal central para habilitar o bloquear acciones como agendamiento.

Uno de los principales desafíos fue diseñar un sistema que usara IA sin depender ciegamente de ella. Por eso incorporé reglas de no invención, revisión humana, validación de pago, control de SLA y registros de trazabilidad.

El resultado es un sistema que no solo responde mensajes, sino que estructura, valida, bloquea, registra y prepara casos para avanzar con mayor control operativo.
```

---

## 22. Conclusión de esta sección

Mi proceso de construcción demuestra una evolución desde una idea de agente de intake hasta una solución operativa más completa.

El proyecto integra decisiones de producto, decisiones técnicas, decisiones de UX y decisiones de control operativo.

La idea central que guió el desarrollo fue:

```text
La IA aporta valor cuando está conectada a reglas, datos, flujo de trabajo y trazabilidad.
```

Por eso, Service Ops OS debe evaluarse como un sistema funcional de apoyo operativo, no solo como un prompt o chatbot.







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

