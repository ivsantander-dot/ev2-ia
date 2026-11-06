# Sistema de Agente Inteligente para Automatización Organizacional - Aseguradora del Pacífico

## 1. Descripción General del Proyecto
Este repositorio contiene la implementación técnica correspondiente a la **Evaluación Parcial 2 (EP2)** de la asignatura *Ingeniería de Soluciones con IA (ISY0101)*. El proyecto evoluciona desde un prototipo básico hacia un **agente inteligente autónomo y funcional**, diseñado para resolver requerimientos organizacionales en el rubro asegurador (*Aseguradora del Pacífico*). 

El sistema integra capacidades avanzadas de consulta transaccional, recuperación semántica de normativas (RAG), escritura/registro en bases de datos y persistencia conversacional en flujos prolongados, utilizando arquitecturas basadas en grafos de estado (**LangGraph**), procesamiento de lenguaje de alta velocidad (**Groq** con `openai/gpt-oss-20b`) y frameworks modernos de IA.

---

## 2. Arquitectura del Sistema y Módulos
La solución se ha estructurado bajo un enfoque modular desacoplado, lo que garantiza alta escalabilidad, mantenimiento simplificado y compatibilidad técnica:

```text
/proyecto-aseguradora-ep2
│
├── 1. Módulo de Inicialización y Configuración Segura (Secrets)
├── 2. Módulo de Memoria de Largo Plazo / RAG (FAISS & FakeEmbeddings)
├── 3. Módulo de Herramientas de Negocio (Tools: Consulta, Escritura y Razonamiento)
├── 4. Módulo de Orquestación y Grafo de Estados (LangGraph & MemorySaver)
└── 5. Módulo de Ejecución y Pruebas (Validación de Sesiones Multiturno)
```

### Detalle de Integración entre Módulos:
1. **Módulo de Seguridad (`google.colab.userdata`):** Centraliza la ingestión de credenciales sensibles (`GROQ_API_KEY`) mediante secretos encriptados, aislando la lógica de negocio de la exposición de claves en texto plano.
2. **Módulo de Recuperación Semántica - RAG (Largo Plazo):** Utiliza un sistema vectorial local basado en `FAISS` y `CharacterTextSplitter`. Indexa los documentos institucionales y normativos de la aseguradora para que el agente pueda responder consultas complejas sobre coberturas de pólizas.
3. **Módulo de Herramientas Funcionales (`@tool`):** Define las capacidades operativas del agente:
   * `buscar_normativa_documental`: Conecta la consulta del usuario con el motor vectorial RAG.
   * `consultar_estado_poliza`: Actúa como pasarela hacia el sistema transaccional de la compañía para verificar estados de clientes (ej. `CLI-001`).
   * `registrar_modificacion_sistema`: Capacidad de escritura para registrar incidencias o actualizaciones organizacionales.
4. **Módulo de Orquestación y Memoria de Corto Plazo (`LangGraph` + `MemorySaver`):** Modela el comportamiento del agente como un ciclo iterativo de razonamiento (*ReAct*). El uso de `MemorySaver` asociado a un `thread_id` permite conservar el historial conversacional entre distintos turnos de interacción sin perder el hilo conductor.

---

## 3. Decisiones de Diseño Técnico

* **Elección del Proveedor de Inferencia (Groq):** Se seleccionó Groq para optimizar la latencia de inferencia y garantizar respuestas en tiempo real durante la demostración en vivo, manteniendo total compatibilidad con la interfaz de llamadas a herramientas (*tool-calling*) de LangChain.
* **Gestión de Embeddings Locales (`FakeEmbeddings`):** Para evitar dependencias externas de costos o limitaciones de red en entornos académicos controlados, se implementó una vectorización local optimizada que alimenta eficientemente el almacén FAISS.
* **Persistencia de Estado Basada en Grafos:** A diferencia de los chatbots basados en cadenas lineales (*Chains*), LangGraph permite un control determinista sobre las transiciones (decidir si se debe invocar una herramienta o finalizar la respuesta), ofreciendo robustez ante fallos y condiciones cambiantes del entorno.

---

## 4. Instrucciones de Instalación y Ejecución

### Prerrequisitos
* Entorno de ejecución de Python (Google Jupyter Notebook / Google Colab).
* Clave de API de Groq configurada en los secretos del entorno bajo la variable `GROQ_API_KEY`.

### Pasos de Ejecución:
1. **Instalación de Dependencias:**
   ```bash
   !pip install -q langchain langchain-groq langchain-community faiss-cpu python-dotenv langgraph
   ```
2. **Configuración del Entorno y Carga de Secretos:**
   Ejecute el bloque de inicialización para importar las librerías y enlazar la clave de API de forma segura.
3. **Construcción del Grafo y Definición de Herramientas:**
   Ejecute sucesivamente los bloques de RAG, Herramientas y Orquestación.
4. **Validación de la Demo en Vivo:**
   Ejecute el bloque final de pruebas multiturno para comprobar la persistencia de la sesión y la autonomía en la selección de herramientas:
   ```python
   config = {"configurable": {"thread_id": "sesion-evaluacion-ep2-groq-01"}}
   ```
