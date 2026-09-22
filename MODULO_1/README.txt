# Agente Automático de Triaje para Soporte Técnico

**Proyecto Checkpoint 1 · Comisión [#103270] · Leonel Marinelli**

Sistema de automatización diseñado en n8n que recibe consultas de soporte técnico en lenguaje natural, utiliza inteligencia artificial para clasificar la prioridad y extraer datos clave, y registra el ticket estructurado en una base de datos, notificando al equipo por Slack.

---

## El problema que resuelve

Los equipos de soporte B2B reciben decenas de mensajes desestructurados diarios ("se cayó el sistema", "tengo un problema", etc.). Leer, interpretar, pedir datos faltantes y clasificar la urgencia de cada mensaje consume horas productivas y retrasa la atención de incidentes críticos. 

Este sistema automatiza la primera capa de atención (Triaje): interpreta el problema, solicita proactivamente la información faltante al usuario y registra un ticket limpio con la prioridad correcta, dejando la resolución en manos del equipo humano.

---

## Arquitectura del Flujo

1. **Entrada (Trigger):** Interfaz de chat de n8n donde el usuario ingresa su consulta.
2. **Motor de Razonamiento (AI Agent):** Nodo avanzado configurado con OpenAI Chat Model (GPT) y un prompt estricto de sistema. Evalúa si el mensaje contiene los datos mínimos.
3. **Acción Física (Herramienta):** Integración con Google Sheets mediante la operación `Append`. Si la IA tiene los datos, extrae Nombre, Email, Empresa, Categoría y Prioridad para escribir una nueva fila.
4. **Log de Observabilidad:** Notificación automática en Slack enviando el resumen operativo generado por la IA al canal del equipo.

### Stack Tecnológico

| Categoría    | Herramienta | Rol en el sistema |
|---|---|---|
| Orquestador  |   n8n     | Define el flujo de trabajo y aloja el Agente IA. |
| Modelo de Lenguaje | OpenAI | Toma decisiones probabilísticas, clasifica y extrae entidades. |
| Base de Datos | Google Sheets | Actúa como registro maestro (ticketing) mediante inyección dinámica. |
| Observabilidad | Slack | Recibe el "Execution Log" para auditoría humana del triaje. |

---

## Guardrails y Seguridad de la IA

- **Prevención de bucles:** El Agente tiene un límite estricto de *Max Iterations* configurado en 5.
- **Detención por error:** El flujo está configurado con `Stop Workflow` en caso de fallas en el uso de herramientas.
- **Restricción de acciones:** El System Prompt prohíbe explícitamente modificar datos productivos, ejecutar acciones destructivas o comprometer tiempos de resolución.