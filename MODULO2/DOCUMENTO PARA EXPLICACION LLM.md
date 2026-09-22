# Documento de Traspaso Técnico (Handoff) - Proyecto Agentes Autónomos n8n

**Estudiante:** Leonel Marinelli  
**Contexto Académico:** Coderhouse - Automatización con Inteligencia Artificial  
**Estado:** Transición de Módulo 1 (Finalizado y Aprobado) a Módulo 2 (En Progreso - Timestamp 2:09:00 Clase 2)  
**Fecha de corte:** Septiembre 2026  

---

## 1. Historial Consolidado: Módulo 1 (Checkpoint 1)

### 1.1. Origen y Fundamentos Conceptuales
* **Evolución inicial (Botpress a n8n):** Comprensión del manejo de variables dinámicas (`workflow.variable`), extracción desde bases de datos (`rows[0].values`), parseo y formateo estructurado (`JSON`), y sintaxis JavaScript básica para la manipulación de estados.
* **Modelo pedagógico adoptado:** Enfoque de *Maestro Técnico* estructurado en analogías funcionales (fábrica de ensamblaje, cajas de almacenamiento, estaciones de trabajo, timbres de entrada y contratos de datos), analizando código línea a línea y parámetros nodo a nodo.

### 1.2. Arquitectura del Checkpoint 1 (Workflow Monolítico de Triaje)
* **Objetivo:** Construir un motor agéntico autónomo para clasificación y registro de incidencias técnicas en un solo lienzo de trabajo.
* **Componentes implementados:**
  * **Trigger:** Chat Trigger nativo (`When chat message received`), mapeando el texto del usuario mediante la expresión dinámica `{{ $json.chatInput }}`.
  * **AI Agent (Supervisor Central):** Modo operativo *Tools Agent*.
  * **Modelo de Lenguaje (LLM):** Configurado con OpenAI Chat Model (`gpt-4o-mini`). *(Durante pruebas se utilizó Google Gemini/Groq por cuotas, pero el entregable final se ajustó estrictamente al nodo oficial de OpenAI según rúbrica)*.
  * **System Message (La Constitución):** Estructura modular estricta ([ROL Y ÁMBITO], [OBJETIVO OPERATIVO], [PROTOCOLO DE ESCALAMIENTO]) con prohibición explícita de giros de lenguaje inclusivo y acciones destructivas.
  * **Guardrails:** Límite estricto de iteraciones fijado en `maxIterations: 5` para mitigar bucles infinitos en el ciclo ReAct. Manejo de excepciones configurado en `On Error: Stop Workflow`.
  * **Herramienta (Tool):** Google Sheets conectado lateralmente en ranura de herramientas con operación de escritura (`Append row`), mapeo dinámico mediante fórmulas `{{ $fromAI(...) }}` y redacción de descripción semántica manual para evitar la *instrucción huérfana*.
  * **Observabilidad (Auditoría Humana):** Integración secuencial al nodo final de Slack enviando el reporte dinámico mediante `{{ $json.output }}`.
* **Entregable del Checkpoint 1:** Flujo exportado como `checkpoint1_leonel_marinelli.json`, documentación estructurada en `README.md` y repositorio público en GitHub.

---

## 2. Consigna y Objetivos del Módulo 2 (Checkpoint 2)

### 2.1. Objetivo de Negocio y Arquitectura
* **Transición:** Romper el antipatrón del *"Workflow Mono-Bloque"* y evolucionar hacia una **Arquitectura Agéntica Empresarial Distribuida** bajo el patrón **Manager-Worker** mediante sub-workflows independientes.
* **Componentes obligatorios:**
  1. **Workflow Manager (Orquestador Central):** Recibe la consulta, clasifica la intención semántica dentro de una taxonomía cerrada y delega la ejecución al especialista correspondiente.
  2. **Workers (Sub-workflows quirúrgicos):** Al menos dos (en nuestro diseño: tres especialistas) configurados en lienzos separados con punto de entrada obligatorio `Execute Workflow Trigger`.
  3. **Handoff de Datos (Contrato de Interfaz):** Limpieza previa de payloads (evitando el *Pasillo de la Muerte por Data Stuffing*) y activación obligatoria de `Wait for child to finish` en la llamada a cada flujo hijo.
  4. **Persistencia y Observabilidad:** Registro consolidado del worker invocado, parámetros enviados y respuesta estructurada retornada.
* **Formato de Entrega:** Archivo PDF formal (`preentrega_modulo2_marinelli_leonel.pdf`) con capturas de lienzos, configuraciones de paso de datos, corrida de punta a punta y justificación escrita del enrutamiento.

---

## 3. Estado Actual de la Implementación (Módulo 2)

### 3.1. Taxonomía de Negocio y Especialistas
* **Taxonomía cerrada:**
  * `TECH_SUPPORT` (o `TECH`): Fallas de servicio, bugs, caídas de plataforma.
  * `BILLING`: Consultas de facturación, comprobantes de pago, cuentas a cobrar.
  * `SALES`: Solicitudes comerciales, demos, captación de leads corporativos.
  * `UNKNOWN` / `HITL`: Casos ambiguos o de alto riesgo derivados a intervención humana (*Human-in-the-Loop*).

### 3.2. Sub-workflows de Especialistas (Workers) - Estado: Terminados
Cada Worker vive en su propio lienzo y cuenta con:
1. **Trigger de Entrada:** Nodo `Execute Workflow Trigger` (`Entrada desde Manager`).
2. **Cerebro Especialista:** AI Agent con prompt de dominio específico acotado a su función única.
3. **Contrato de Salida Estandarizado (`Code in JavaScript`):** Implementado y validado en los 3 Workers:
   * **Worker 1:** `1-Facturación` (etiqueta `BILLING`).
   * **Worker 2:** `2- Sales` (etiqueta `SALES`).
   * **Worker 3:** `3- Tech` (etiqueta `TECH`).
   * **Estructura del Contrato de Retorno:**
     ```json
     {
       "status": "success",
       "worker": "BILLING | SALES | TECH",
       "respuesta": "Texto procesado por el especialista...",
       "requires_human": false
     }
     ```
   * **Control de Resiliencia:** Bloque `try / catch` que remueve etiquetas Markdown (```` ```json ````) y construye un objeto de contingencia (`status: 'error'`, `requires_human: true`) ante alucinaciones o fallos de parseo del LLM.

### 3.3. Workflow Manager (Orquestador) - Estado: En Construcción
* **Lienzo principal actual:**
  1. `Telegram Trigger`: Timbre de entrada que captura mensajes reales del cliente.
  2. `Edit Fields`: Nodo de preparación de campos.
  3. `Router IA - Groq`: Clasificador semántico que categoriza la consulta y evalúa métricas operativas (`intent`, `confidence`, `risk`).
  4. `Code in JavaScript`: Validador y parser de respuesta del enrutador. Corregido para extraer dinámicamente el mensaje original mediante:
     ```javascript
     const original = $('Telegram Trigger').first().json.message?.text ?? $('Edit Fields').first().json.chatInput ?? '';
     ```
  5. `If (Filtro de Seguridad / Guardrail)`:
     * Condición: `risk == 'HIGH'` OR `confidence == 0.5`.
     * **Rama `true`:** Conectada al nodo `Edit Fields1` para manejo de contingencia humana (`status: 'need_human'`, `worker: 'HITL'`, `requires_human: true`). Validado estructuralmente.
     * **Rama `false`:** Vía segura por donde fluyen las consultas normales (`risk: LOW`, `confidence: 0.9`). Validada en ejecución de prueba con input `"hola"`.

---

## 4. Punto de Reanudación y Próximos Pasos Técnicos

* **Referencia de clase:** Video de Clase 2, minuto **2:09:00**.
* **Acciones pendientes inmediatas para la reanudación:**
  1. **Enlazar el Router a la salida `false` del nodo `If`:** Insertar un nodo **Switch** (o evaluar reglas deterministas) condicionado sobre `{{ $json.intent }}` con salidas para:
     * Regla 0: `TECH_SUPPORT`
     * Regla 1: `BILLING`
     * Regla 2: `SALES`
  2. **Colocar filtros anti Data Stuffing:** Insertar un nodo **Edit Fields** (Set) en cada rama antes de la invocación para pasar estrictamente los datos mínimos requeridos (`cliente_nombre`, `cliente_contacto`, `consulta`, `categoria`).
  3. **Conectar los nodos `Execute Sub-workflow`:**
     * Enlazar los sub-workflows correspondientes (`3- Tech`, `1-Facturación`, `2- Sales`).
     * Asegurar la activación estricta del parámetro **`Wait for Sub-workflow to Finish`** (`Wait for child to finish: ON`).
  4. **Consolidar Observabilidad / Salida:** Reintegrar el resultado devuelto por los Workers hacia el nodo de notificación y auditoría final (Slack / Telegram / Sheets).
  5. **Testing de Regresión y Armado de PDF:** Ejecutar corrida de punta a punta con caso real, capturar los 5 puntos exigidos en la rúbrica y compilar el archivo `preentrega_modulo2_marinelli_leonel.pdf`.