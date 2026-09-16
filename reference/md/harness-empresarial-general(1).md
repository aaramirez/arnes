# Criterios de Diseño para un Harness de Agentes en Ambientes Empresariales

*Marco analítico para evaluar y construir runtimes de agentes de IA en contextos corporativos*

---

## Resumen ejecutivo

La mayoría de los harnesses de agentes de IA disponibles hoy (Claude Code, Claude Agent SDK, LangGraph, CrewAI, AutoGen, OpenAI Assistants, entre otros) fueron diseñados bajo un supuesto implícito: un usuario humano frente a un terminal o interfaz de chat, con un ciclo interactivo síncrono. Cuando se intenta llevar ese modelo a contextos empresariales —telecomunicaciones, banca, seguros, salud, retail, manufactura, sector público— los supuestos comienzan a chocar con requisitos no negociables: activación por evento, multi-tenencia, identidad delegada, auditoría regulatoria, gobernanza de acciones sensibles, y despliegue distribuido con niveles de servicio comprometidos.

Este documento propone un marco de catorce criterios de diseño para evaluar harnesses existentes y para orientar la construcción de una capa envolvente empresarial. Está pensado como instrumento de análisis para arquitectos, gerentes de plataforma, equipos de desarrollo y personal en formación que deban comprender qué separa un prototipo funcional de un sistema apto para producción corporativa.

## 1. Planteamiento del problema

Un *harness* es la capa de tiempo de ejecución que envuelve al modelo de lenguaje y hace posible que se comporte como agente: gestiona el bucle de razonamiento, la invocación de herramientas, la memoria de la conversación, el estado de la sesión, la aplicación de políticas y la comunicación con el usuario. Es, en la práctica, el sistema operativo del agente.

Los harnesses de mayor adopción actual están optimizados para casos de uso de desarrollo o asistencia interactiva. Cuando se examinan escenarios empresariales típicos —un agente que se activa por webhook cuando entra un reclamo, un agente que expone una API REST para que otros sistemas lo consuman, un agente que gestiona colecciones de tareas asíncronas o brinda soporte de primer nivel a miles de clientes concurrentes— aparecen brechas estructurales que no se resuelven agregando integraciones puntuales.

El objetivo de este marco no es descalificar los harnesses actuales, sino ofrecer un vocabulario preciso para diagnosticar sus limitaciones frente a un caso de uso empresarial concreto, y decidir qué capa envolvente construir.

## 2. El sesgo de origen de los harnesses actuales

La observación central es que los harnesses populares fueron construidos con un modelo mental "un humano, un terminal, un ciclo". Esto se traduce en decisiones arquitectónicas que se vuelven limitantes en producción: acoplamiento de la sesión al canal de entrada (stdin/stdout, socket de chat), estado en memoria del proceso, un único juego de credenciales, ausencia de multi-tenencia real, telemetría orientada a depuración y no a auditoría, y ausencia de contratos de versionado en las herramientas.

Estas decisiones son perfectamente razonables para su caso de uso original. El error es asumir que un harness diseñado para asistir a un desarrollador puede, sin transformación arquitectónica, sostener miles de sesiones concurrentes de clientes finales en un contexto regulado o de misión crítica.

## 3. Los catorce criterios de diseño

Cada criterio se presenta con una pregunta guía y una discusión breve. Recomendamos usarlos como una lista de verificación al evaluar un harness candidato o al diseñar una capa envolvente propia.

### 3.1. Modelo de activación

> **Pregunta guía.** ¿El ciclo de vida del agente está desacoplado del canal de entrada?

Los harnesses actuales asumen un patrón "usuario escribe, agente responde". Los contextos empresariales exigen múltiples modos de activación: webhook proveniente de un sistema central, mensaje en una cola de mensajería (Kafka, RabbitMQ, SQS, MQ), petición REST o gRPC de otro sistema, cron programado, cambio de estado en una base de datos, ticket entrante en un CRM o ITSM.

La pregunta profunda no es "¿puedo llamarlo por API?" sino si el runtime del agente distingue conceptualmente entre el transporte de entrada y el bucle de razonamiento. Muchos harnesses acoplan sesión a stdin/stdout o a un canal websocket específico, y esto obliga a construir adaptadores fuera del runtime, con riesgo de pérdida de contexto o de estado.

### 3.2. Ciclo de vida y persistencia de la sesión

> **Pregunta guía.** ¿La sesión es efímera o de larga duración? ¿Puede hibernarse y reanudarse?

Un chat con un desarrollador dura minutos. Un caso de soporte técnico puede durar días. Un caso de reclamo o de gestión de siniestro puede durar semanas y atravesar múltiples handoffs. El harness debe permitir sesiones de larga duración con capacidad de checkpoint, hibernación en almacenamiento persistente, y reanudación tras caída del proceso o despliegue de una nueva versión.

Esto separa harnesses "conversacionales" de harnesses de "proceso de negocio". Los primeros tratan la sesión como un objeto en memoria; los segundos la tratan como una entidad persistida con ciclo de vida propio, versión, y estado formal.

### 3.3. Concurrencia y multi-tenencia

> **Pregunta guía.** ¿Hay aislamiento estricto entre sesiones de distintos tenants, con cuotas y sandboxing por-tenant?

Un harness tipo CLI asume un usuario, un contexto, un directorio de trabajo. En una organización con múltiples unidades de negocio, filiales o clientes corporativos, la operación exigirá miles de sesiones paralelas con aislamiento estricto. Un agente que sirve a un cliente no puede filtrar contexto a otro, ni siquiera por accidente en un embedding o en un caché compartido.

La pregunta operativa es si el harness ofrece ejecución por-tenant con cuotas de cómputo, límites de gasto en tokens, y sandboxing de código y herramientas, o si toca envolverlo en una capa propia que lo garantice.

### 3.4. Identidad, autorización y delegación

> **Pregunta guía.** Cuando el agente llama a un sistema interno, ¿con qué credenciales lo hace?

Este es uno de los criterios más críticos y menos resueltos en los harnesses públicos. Las opciones son: cuenta de servicio (el agente actúa siempre como sí mismo), delegación tipo OAuth *on-behalf-of* (el agente actúa por el usuario final autenticado), o esquemas mixtos.

La decisión afecta no solo la seguridad, sino la calidad de las respuestas: si el agente consulta un sistema con credenciales de servicio, obtiene todos los datos y debe filtrar en post-proceso, lo cual es frágil. Si consulta con credenciales delegadas, el propio sistema aplica seguridad a nivel de fila y responde solo lo que ese usuario podría ver. En sectores regulados (banca, seguros, salud, sector público) la segunda opción no es un lujo: es requisito regulatorio.

### 3.5. Auditoría y trazabilidad

> **Pregunta guía.** ¿Cada prompt, tool call y respuesta queda registrado de forma inmutable, con capacidad de replay?

Los harnesses open source suelen ofrecer logging estructurado, pero no auditoría forense en el sentido que exige un ente regulador. Se requiere almacenamiento inmutable (append-only), firma o hash encadenado para detectar manipulación, retención acorde a la normativa aplicable, y capacidad de reconstruir exactamente qué vio y qué decidió el agente en un momento dado.

Este es un buen indicador para separar un harness apto para prueba de concepto de uno apto para producción regulada.

### 3.6. Gobernanza y guardrails

> **Pregunta guía.** ¿Existen aprobaciones humanas, políticas por rol y límites de gasto como ciudadanos de primer orden?

Las acciones sensibles —mover dinero, cancelar un servicio, escalar prioridad, aplicar un ajuste tarifario, publicar comunicaciones masivas— deben pasar por aprobación humana explícita, no por una heurística del modelo. Las políticas por rol determinan qué herramientas puede invocar cada tipo de agente. Los límites de gasto en tokens por tenant y por sesión evitan sorpresas en la factura y contienen abusos.

Muchos harnesses tratan estos elementos como plugins opcionales o como responsabilidad de la aplicación que los envuelve. Para entornos empresariales, deben ser primitivas del runtime, con configuración declarativa y auditable.

### 3.7. Semántica de fallo

> **Pregunta guía.** ¿Cómo se comporta el agente cuando algo sale mal y no hay humano observando?

Los harnesses interactivos asumen que si el modelo se atasca, se equivoca o falla una herramienta, el humano frente a la pantalla lo notará y reintentará. Un agente detrás de un webhook o consumiendo una cola no tiene ese lujo. Necesita: idempotencia de tool calls (importante si el webhook se reintenta), reintentos con backoff exponencial, cola de mensajes muertos (dead letter queue), y escalamiento a humano cuando el agente detecta que no puede resolver.

La semántica de fallo bien definida es lo que convierte un demo impresionante en un sistema en el que se puede confiar.

### 3.8. Superficie de integración saliente

> **Pregunta guía.** ¿Puede el agente hablar con los sistemas reales de la empresa, incluyendo legacy?

La abstracción de "tool" o "function calling" está pensada para APIs modernas con esquemas bien definidos. La realidad empresarial incluye sistemas de generaciones anteriores, mainframes con protocolos propios, servicios SOAP con colas MQ, ERPs con esquemas cerrados y bases de datos accesibles solo por conexiones dedicadas. Adaptar el agente a estos sistemas es trabajo de integración que ningún harness resuelve por sí solo.

La pregunta útil es qué tan bien el harness expone puntos de extensión limpios (por ejemplo, a través del Model Context Protocol) para conectar estos sistemas sin acoplar la lógica de integración al prompt del agente.

### 3.9. Estado y memoria a largo plazo

> **Pregunta guía.** ¿Dónde vive la memoria persistente del agente, y bajo qué régimen de datos?

Un agente empresarial requiere memoria por cliente, por caso, por entidad. Esa memoria se sincroniza con CRM, ITSM, sistemas transaccionales, y con almacenes vectoriales para recuperación semántica. Cada uno de estos almacenes tiene requisitos de niveles de servicio, residencia de datos y cifrado.

La residencia de datos importa especialmente en organizaciones con operación transnacional, donde cada país tiene marcos regulatorios distintos sobre datos personales y sensibles. Un harness que no separa claramente el lugar donde vive la memoria del lugar donde corre el runtime deja al integrador con problemas serios de cumplimiento.

### 3.10. Topología de despliegue

> **Pregunta guía.** ¿El harness fue diseñado para correr como servicio distribuido, o como proceso único?

Un harness CLI no se pensó para ejecutar cientos de réplicas en Kubernetes atendiendo tráfico real. Las preguntas a responder son: ¿el estado por sesión permite balanceo entre réplicas? ¿la escala horizontal es lineal? ¿el harness distingue entre workloads síncronos (chat en vivo) y batch (procesamiento de casos acumulados)? ¿soporta despliegue on-premise cuando la regulación lo exige, o solo cloud gestionado?

La respuesta suele determinar si el harness puede reutilizarse o si hay que envolverlo en una capa de orquestación propia.

### 3.11. Contrato de herramientas y versionado

> **Pregunta guía.** ¿Cómo evoluciona el catálogo de tools sin romper agentes desplegados?

En producción, un tool que cambia su firma rompe agentes que lo usaban. Se requiere versionado explícito, compatibilidad hacia atrás, feature flags para activar herramientas nuevas por tenant, y un ciclo de deprecación ordenado.

Ningún harness público resuelve esto bien hoy. Es un espacio donde la capa envolvente empresarial debe aportar disciplina propia, tomando prestadas prácticas de gestión de APIs.

### 3.12. Evaluación y regresión

> **Pregunta guía.** ¿Cómo verificamos que un cambio de prompt o de modelo no degrada respuestas críticas?

Un harness de tiempo de ejecución no basta: hace falta un harness de evaluación separado, con datasets propios del negocio, corriendo en integración continua. Se necesita medir calidad de respuesta, tasa de escalamiento, adherencia a políticas, y comportamiento en casos adversariales. Sin este anillo de pruebas, cualquier actualización del modelo base o del prompt del sistema es un salto al vacío.

Este es un producto aparte del runtime, y su ausencia es una de las razones más comunes de fracaso al pasar de prueba de concepto a producción.

### 3.13. Costeo y observabilidad de negocio

> **Pregunta guía.** ¿Sabemos cuánto cuesta cada tipo de caso y cuál es el retorno del agente?

La telemetría técnica que traen los harnesses (latencia, errores, tokens) es útil pero insuficiente. La operación necesita observabilidad de negocio: costo por tenant, por vertical, por tipo de caso; costo por resolución exitosa vs. costo por escalamiento humano; comparación contra el costo del canal alterno (agente humano); tasa de resolución en primer contacto; satisfacción del cliente final.

Sin estos indicadores, la conversación con las áreas de negocio y con la dirección se queda en el terreno de la impresión. Con ellos, el agente deja de ser un experimento y pasa a ser un canal medible.

### 3.14. Patrones de interacción con humanos

> **Pregunta guía.** ¿Qué formas de colaboración humano-agente soporta el harness?

El chat síncrono es solo uno de los patrones posibles. Otros, igualmente valiosos en contextos empresariales, incluyen: tablero de tareas asíncrono, donde el agente propone y el humano aprueba en lote; interrupción por umbral, donde el agente pausa y pide validación al detectar una anomalía; colaboración multi-agente supervisada; handoff limpio a un agente humano con transferencia completa de contexto.

Un harness que solo soporta chat síncrono limita al integrador a un solo formato de servicio. Uno que soporta varios permite modelar procesos reales del negocio.

## 4. Método de evaluación y arquitectura recomendada

Sugerimos que este marco se use como lista de verificación aplicada a un caso de uso concreto. Elija primero un caso realista y acotado —por ejemplo, un agente de soporte de primer nivel, un agente de triage de reclamos, un agente de generación de propuestas comerciales o un agente de conciliación contable— y confronte contra este marco dos o tres harnesses candidatos. Verá rápidamente que ningún harness público cubre todos los criterios.

De ese ejercicio suele emerger la misma conclusión: la respuesta empresarial no es "el harness definitivo", sino un **harness envuelto**. Un runtime de un proveedor (por ejemplo, Claude Agent SDK, OpenAI Assistants, LangGraph) sirve como núcleo de razonamiento; alrededor de él se construye una capa propia que resuelve los criterios que el núcleo no cubre: activación, multi-tenencia, identidad delegada, auditoría, gobernanza, evaluación y despliegue.

Este patrón es lo que están construyendo hoy los adoptantes serios de IA generativa en banca, telecomunicaciones, seguros, salud y sector público. Los proyectos que lo saltan y buscan un harness "todo en uno" listo para producción típicamente descubren en piloto que la brecha entre el demo y el sistema real es exactamente la lista que este documento describe.

## 5. Ejercicio práctico: cómo aplicar este marco

Para quien esté aprendiendo el tema —por ejemplo un pasante o un desarrollador que se aproxima por primera vez a llevar agentes a producción— sugerimos el siguiente ejercicio de tres pasos:

**Paso uno: escoger un caso realista y acotado.** No hay que resolver toda la organización; basta con un proceso concreto: "un agente que responda consultas de saldo", "un agente que triage reclamos entrantes", "un agente que genere borradores de contratos". La ganancia analítica del marco crece con la especificidad del caso.

**Paso dos: llenar la matriz de criterios.** Para cada uno de los catorce criterios, escribir en una tabla: ¿qué requisito impone el caso? ¿qué ofrece el harness candidato? ¿cuál es la brecha? Este ejercicio es más valioso que el resultado final: obliga a hacer explícitos supuestos que normalmente quedan implícitos.

**Paso tres: decidir la capa envolvente.** Con la matriz completa, se ve con claridad qué elementos del harness se aprovechan, qué elementos hay que construir alrededor, y qué elementos exigen adaptadores puntuales. Ese es el boceto de arquitectura empresarial.

Repetir este ejercicio con dos o tres casos distintos revela patrones comunes: casi siempre la activación, la auditoría, la gobernanza y la evaluación son las capas que hay que construir por fuera del harness, independientemente del proveedor elegido.

## 6. Conclusión

La conclusión operativa es que hoy no existe un harness listo para casos de uso empresariales complejos, y probablemente pasarán varios ciclos antes de que exista. En el intervalo, la ventaja competitiva pertenece a las organizaciones que sepan combinar un buen núcleo de runtime con una capa envolvente propia bien diseñada.

Este documento es un primer paso: fijar el vocabulario y los criterios. El siguiente paso natural es escoger un caso de uso concreto, aplicar este marco, y bocetar la arquitectura envolvente correspondiente. Los criterios enumerados aquí no son exhaustivos ni definitivos; son un punto de partida útil para conversaciones informadas entre equipos de tecnología, de negocio y de cumplimiento.

## 7. Extensión del marco — criterios adicionales para una plataforma agentic empresarial

Los catorce criterios originales siguen vigentes. Para cubrir el runtime empresarial completo se agregan los siguientes criterios:

### 7.1 Comunicación y federación entre agentes
Distinguir internal delegation de external federation. Exigir contratos explícitos de Task, Message, Result, Artifact, Progress, Failure y Cancellation. Considerar A2A como opción de interoperabilidad externa sin acoplar el core al estándar.

### 7.2 Independencia entre semántica, protocolo y transporte
La semántica de colaboración no debe depender de HTTP, webhook, WebSocket, SSE, gRPC, queue, Kafka o Service Bus. Protocol adapters y transport adapters deben evolucionar independientemente.

### 7.3 Admission control y routing
Una activación no equivale a una ejecución autorizada. Deben existir admission, deduplication, rate limits, capacity, budget, policy y routing hacia Agent, Workflow, Job o Human Task.

### 7.4 Execution Fabric
Definir workers interactivos/asíncronos, horizontal scaling, regional/on-prem placement, workload classes, autoscaling y session/state independence.

### 7.5 Data Governance
Clasificación, residencia, retención, cifrado, lineage, deletion, legal hold y restricciones de uso de modelos externos deben ser políticas ejecutables.

### 7.6 Capability lifecycle
Tools/capabilities requieren versioning, compatibility, feature flags, canary rollout, deprecation y retirement.

### 7.7 Messaging reliability
Definir acknowledgement, retry, timeout, ordering, deduplication, idempotency, backpressure, poison-message handling y DLQ/reprocessing.

### 7.8 Immutable audit
Separar telemetry, execution reconstruction y regulatory evidence. El audit ledger debe ser append-only/tamper-evident cuando el riesgo lo requiera.

### 7.9 Agent operations
La operación necesita RunRegistry, health, backlog, pending approvals, stuck runs, DLQ, cost anomalies, SLOs, capability health y kill switches.

### 7.10 Business outcomes
Correlacionar runs con resolución, escalamiento, revenue/value protected, SLA, customer outcome, time-to-resolution y costo total.

## 8. Arquitectura empresarial consolidada

```text
Ingress/Activation
        ↓
Admission/Identity/Policy
        ↓
Routing
        ↓
Workflow Runtime ── Agent Runtime ── Job Runtime
                         │
               Agent Interoperability
                Internal / A2A adapters
                         │
                    Tool Runtime
                         │
 Capability Registry/Lifecycle + Credential Broker
                         │
                Enterprise Systems
```

Cross-cutting planes: Data & Context, Control, Reliability, Observability & Governance, and Execution Fabric.

