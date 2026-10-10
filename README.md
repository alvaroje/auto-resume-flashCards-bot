# 🧠 AI Study Assistant: Generador Automatizado de Resúmenes y Flashcards

[![n8n](https://img.shields.io/badge/n8n-Workflow_Automation-FF6D5A?style=flat-square&logo=n8n)](https://n8n.io/)
[![Gemini](https://img.shields.io/badge/Google_Gemini-LLM-4285F4?style=flat-square&logo=google)](https://deepmind.google/technologies/gemini/)
[![JavaScript](https://img.shields.io/badge/JavaScript-Data_Transformation-F7DF1E?style=flat-square&logo=javascript)](https://developer.mozilla.org/es/docs/Web/JavaScript)
[![Google Drive API](https://img.shields.io/badge/Google_Drive-Cloud_Storage-4285F4?style=flat-square&logo=googledrive)](https://developers.google.com/drive)

Un orquestador de datos documental que transforma automáticamente apuntes universitarios en bruto en material de estudio optimizado. El sistema ingesta archivos PDF, procesa su contenido mediante Inteligencia Artificial y exporta simultáneamente resúmenes maquetados y baterías de tarjetas de memoria (Flashcards) compatibles con software de repetición espaciada como Anki.

## 🎯 Objetivo del Proyecto

Optimizar drásticamente el tiempo de preparación de material de estudio y gestión del conocimiento. Mediante la automatización del flujo de trabajo, cualquier documento subido a la nube es procesado instantáneamente, extrayendo la "señal del ruido" y generando dos entregables listos para consumir: un resumen ejecutivo en formato PDF y un dataset en CSV con pares de preguntas/respuestas.

## 📂 Arquitectura del Flujo

El pipeline de datos está construido en **n8n** y opera de forma asíncrona basada en eventos (Event-Driven Architecture):

1. **Triggering & Ingestion:** Un `Google Drive Trigger` monitoriza una carpeta específica mediante *polling*. Al detectar la creación de un nuevo archivo, el sistema lo descarga en formato binario.
2. **Data Extraction:** El binario pasa por un nodo de extracción de PDF que convierte la capa de datos en texto plano y lo inyecta en el flujo JSON.
3. **LLM Processing:** Google Gemini actúa como motor de análisis cognitivo. Mediante un prompt estructurado (*System Prompting*), se fuerza al modelo a devolver un objeto JSON estricto que contiene un resumen del texto y un array de 15 flashcards técnicas.
4. **Data Transformation (JavaScript):** Un nodo de código intermedio procesa la salida de la IA:
   - Limpia posibles anomalías del Markdown.
   - Itera sobre el array de flashcards para construir un archivo delimitado por tuberías (`Pregunta|Respuesta`), ideal para Anki.
   - Inyecta el resumen en una plantilla HTML5/CSS predefinida para su correcta maquetación.
   - Empaqueta ambas cadenas de texto en objetos binarios (Buffers).
5. **Compilation & Routing:** El archivo CSV se sube directamente a Google Drive. Simultáneamente, el HTML es procesado por un motor de conversión a PDF antes de ser almacenado en la nube en su versión final.

## 🛠️ Stack Tecnológico

*   **Orquestación e Integración:** n8n
*   **Procesamiento de Lenguaje Natural (NLP):** Google Gemini (PaLM API)
*   **Transformación de Datos:** JavaScript (ES6), Node.js Buffers
*   **Almacenamiento Cloud:** Google Drive API
*   **Formatos:** JSON, CSV, HTML, PDF

## ⚠️ Retos Técnicos Superados

*   **Estructuración Rígida de Salidas LLM:** Garantizar que Gemini devuelva siempre un JSON válido en lugar de texto narrativo es un desafío común. Se implementó una lógica de limpieza con expresiones regulares (`replace(/```json/g, "")`) en la etapa de transformación JS para manejar de forma segura las variaciones en la respuesta del modelo y evitar rupturas en el parseo (`JSON.parse()`).
*   **Manipulación de Datos Binarios *On-The-Fly*:** n8n maneja los datos de texto y binarios de forma paralela. Se utilizó `this.helpers.prepareBinaryData` junto con `Buffer.from()` en JavaScript para codificar dinámicamente las cadenas de texto (CSV y HTML) generadas en memoria en archivos binarios listos para ser subidos mediante API, evitando la necesidad de almacenarlos temporalmente en disco local.
*   **Generación Dinámica de UI en Backend:** Para asegurar que el PDF final no fuera un simple bloque de texto sin formato, se programó una inyección de estilos CSS (*Inline Style Sheet*) dentro del propio nodo de código, garantizando que el motor de conversión PDF recibiera un documento web semánticamente completo y atractivo de manera autónoma.

## ⚙️ Instalación / Importación

Para analizar el código fuente:
1. Clona este repositorio o descarga el archivo `workflow.json`.
2. En tu instancia de n8n, utiliza la opción `Import from File`.
3. Configura tus credenciales OAuth2 para **Google Drive** y tu API Key de **Google Gemini**.
4. Ajusta los IDs de las carpetas de origen y destino de Google Drive en los parámetros de los nodos.

---
*Desarrollado por [Álvaro Pérez-Moreno Callejo](https://alvaroje.github.io/portfolio/)*
