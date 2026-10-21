# Chatbot de Agendamiento con Agente LangGraph
**Centro MedImage — Resonancia y Scanner Valparaíso | EP2 ISY0101 | Octubre 2026**

Chatbot conversacional para gestionar citas de exámenes de imágenes médicas (resonancia magnética y TAC) en el Centro MedImage, Av. Colón 2383, Valparaíso. Desarrollado como entregable del EP2 de ISY0101.

---

## Stack

| Componente | Tecnología |
|---|---|
| LLM | Groq — `openai/gpt-oss-120b` |
| Agente | LangGraph `create_react_agent` |
| Herramientas | LangChain Core 0.3 `@tool` |
| Memoria corto plazo | LangGraph `MemorySaver` (por `thread_id`) |
| Memoria largo plazo | SQLite — tabla `historial_conversacion` |
| RAG / Vector store | FAISS + Gemini Embeddings (`gemini-embedding-001`) |
| Trazabilidad | LangSmith — proyecto `chatbot-resonancia-v2` |
| Entorno | Google Colab |

---

## Estructura del notebook

```
chatbot_agendamiento_v2.ipynb
├── c01  Presentación
├── c02  Imports y variables de sesión
├── c03  Instalación de dependencias (pip)
├── c04  Claves API (Google Colab userdata)
├── c05  LangSmith — activación de trazabilidad
├── c06  Inicialización del LLM (Groq)
├── c07  Base de datos SQLite (citas + historial)
├── c08  Reglas de negocio y datos del centro
├── c09  IE1 — Herramienta: consultar_horarios_disponibles
├── c10  IE1 — Herramienta: agendar_cita
├── c11  IE1 — Herramienta: cancelar_cita / consultar_cita
├── c12  IE1 — Herramienta: buscar_faq_resonancia (RAG FAISS)
├── c13  IE3 — MemorySaver (checkpointer LangGraph)
├── c14  IE1 — Herramienta: verificar_feriados_chile
├── c15  IE4 — FAISS: carga de FAQ interna
├── c16  IE4 — FAISS: indexación de documentos
├── c17  IE4 — FAISS: prueba de búsqueda semántica
├── c18  IE6 — LangSmith: verificación de trazabilidad
├── c19  SQLite: inicialización de tablas
├── c20  Registro de herramientas (lista `herramientas`)
├── c21  IE2 — Agente LangGraph (create_react_agent)
├── c22  Prueba de herramientas individuales
├── c23  Prueba de consulta RAG
├── c24  Prueba de agendamiento completo
├── c25  IE5 — Demo ReAct (inspección de messages)
├── c26  IE4 — Demo RAG (fuentes recuperadas)
└── c27  IE3/IE5 — Loop de chat interactivo
```

---

## Indicadores EP2

| IE | Descripción | Implementación |
|---|---|---|
| IE1 | 6 herramientas `@tool` | `langchain_core.tools` con tipado y docstring |
| IE2 | Framework LangChain/LangGraph | `create_react_agent` de `langgraph.prebuilt` |
| IE3 | Memoria dual | `MemorySaver` (corto plazo) + SQLite (largo plazo) |
| IE4 | RAG con trazabilidad de fuentes | FAISS + metadato `fuente` (FAQ_interna / API_feriados_cl) |
| IE5 | Ciclo ReAct | Automático vía grafo LangGraph |
| IE6 | Documentación técnica | LangSmith + celdas demo + `VERBOSE_AGENTE` |

---

## Ejecución en Google Colab

### 1. Crear secretos en Colab

En el panel izquierdo → 🔑 **Secrets**, agregar:

| Nombre | Descripción |
|---|---|
| `GROQ_API_KEY` | Clave API de [console.groq.com](https://console.groq.com) |
| `GEMINI_API_KEY` | Clave API de [aistudio.google.com](https://aistudio.google.com) |
| `LANGSMITH_API_KEY` | Clave API de [smith.langchain.com](https://smith.langchain.com) |

### 2. Ejecutar las celdas en orden

```
Menú → Runtime → Run all  (Ctrl+F9)
```

Las celdas c01–c20 instalan dependencias e inicializan el sistema (~2–3 min).

### 3. Interactuar con el chatbot

Ejecutar la celda **c27** (Loop de chat). Comandos disponibles durante la conversación:

| Comando | Acción |
|---|---|
| `salir` | Termina la sesión |
| `memoria` | Muestra el historial completo del `MemorySaver` |
| `historial` | Muestra los últimos 10 registros de SQLite |
| _(cualquier texto)_ | Envía un mensaje al agente |

---

## Uso del agente desde código

```python
from langchain_core.messages import HumanMessage

# Configuración de sesión
config = {"configurable": {"thread_id": "mi-sesion-001"}}

# Invocación sincrónica
resultado = agente.invoke(
    {"messages": [HumanMessage(content="¿Hay hora disponible el 15 de noviembre?")]},
    config=config,
)
respuesta = resultado["messages"][-1].content
print(respuesta)

# Streaming token a token
for chunk in agente.stream(
    {"messages": [HumanMessage(content="Quiero agendar una resonancia")]},
    config=config,
    stream_mode="messages",
):
    msg, metadata = chunk
    if hasattr(msg, "content"):
        print(msg.content, end="", flush=True)

# Inspeccionar memoria de la sesión
estado = agente.get_state(config)
for m in estado.values.get("messages", []):
    print(f"[{type(m).__name__}] {m.content[:80]}")
```

---

## Reglas de negocio

- **Horario:** Lunes–Viernes 8:00–19:00 / Sábado 9:00–14:00
- **Feriados:** Sin atención (verificado en tiempo real vía feriados.cl)
- **Bloque por examen:** 60 minutos
- **Orden médica:** Obligatoria para agendar
- **TAC con contraste IV:** Ayuno 4 h + resultado de creatinina reciente

---

## Archivos generados en ejecución

| Archivo | Contenido |
|---|---|
| `medimage_citas.db` | Base de datos SQLite con citas e historial |
| `faiss_index/` | Índice FAISS del FAQ interno |

---

## Autor

Sebastián — ISY0101 EP2, Duoc UC Valparaíso, Octubre 2026
