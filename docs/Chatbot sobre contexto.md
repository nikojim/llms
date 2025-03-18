# Chatbot sobre contexto

## Introducción
El propósito de este documento es describir los pasos realizadas para crear un chatbot cuya operativa sea sobre un contexto determinado. Para este propósito se utilizan los conceptos de LLMS para entrenar un modelo y RAG para que dicho modelo opere sobre el contexto dato. La estructura del documento será la siguiente: en primer lugar se describen las metodologías y herramientas que se usan para desarrollar el chatbot y luego se describen los diagramas y los flujos de los procesos implicados en dicho desarrollo.

## Herramientas y metodologías
A continuación se describen las herramientas y metodologías usadas para llevar a cabo la implementación de la solución.

### LLMs
Los Large Language Models, también conocidos como LLMs, son modelos de aprendizaje profundo de gran tamaño que se entrenan previamente con grandes cantidades de datos . El transformador subyacente es un conjunto de redes neuronales que consisten en un codificador y un decodificador con capacidades de autoatención.

### RAG
RAG es una técnica que permite que un modelo de lenguaje grande (LLM) genere respuestas enriquecidas aumentando la indicación de un usuario con datos auxiliares recuperados de un origen de información externo. Al incorporar esta información recuperada, RAG permite al LLM generar respuestas más precisas y de mayor calidad en comparación con no aumentar la indicación con contexto adicional.

### Ollama
Ollama es una herramienta que te permite ejecutar grandes modelos de lenguaje (LLMs) localmente en tu propia computadora. Simplifica el proceso de descargar, gestionar y ejecutar estos potentes modelos de IA sin depender de APIs basadas en la nube.

### LLama Embeddings
Los Llama Embeddings son representaciones numéricas de texto generadas por la familia de grandes modelos de lenguaje (LLMs) Llama desarrollados por Meta AI. Estos embeddings capturan el significado semántico y la información contextual del texto, lo que permite diversas tareas de procesamiento del lenguaje natural (PNL).

### Faiss 
FAISS es una biblioteca optimizada para realizar búsquedas de vecinos más cercanos (k-NN) y otros tipos de búsqueda de similitud en conjuntos de datos de alta dimensión. 
En términos generales, FAISS crea una estructura de índice sobre los vectores de datos. Esta estructura permite buscar los vecinos más cercanos de un vector consulta de forma mucho más rápida que una búsqueda lineal que compararía el vector consulta con cada vector en el conjunto de datos. Diferentes índices utilizan diferentes técnicas para organizar los datos y acelerar la búsqueda.

## SQLite
SQLite es un motor de base de datos relacional ligero y autónomo que se caracteriza por su simplicidad y facilidad de uso.  A diferencia de otros sistemas de gestión de bases de datos (SGBD) como MySQL o PostgreSQL, SQLite no necesita un servidor separado.  En cambio, la base de datos se almacena en un único archivo en el disco, lo que la hace muy portátil y fácil de usar.
Python tiene una excelente integración con SQLite a través del módulo sqlite3, que viene incluido en la biblioteca estándar de Python a partir de la versión 2.5. 

## Gradio
Gradio es una biblioteca de Python de código abierto que facilita la creación y el uso compartido de interfaces web interactivas para modelos de aprendizaje automático.  En esencia, te permite convertir tu código de Python (específicamente funciones que procesan datos y producen resultados, como las que se usan en machine learning) en una aplicación web sencilla y accesible.

## Streamlit
Streamlit es una biblioteca de Python de código abierto que facilita la creación y el uso compartido de aplicaciones web interactivas para ciencia de datos y aprendizaje automático.  Su principal fortaleza reside en su simplicidad y su capacidad para convertir scripts de Python en aplicaciones web funcionales con muy poco código adicional.  En otras palabras, puedes crear una aplicación interactiva a partir de tu análisis de datos o modelo de machine learning sin necesidad de conocimientos profundos de desarrollo web.


## Operativa de la solución
En este apartado se presentan y explican los diagramas correspondientes a las diferentes fases de la operativa de la solución realizada.

### Fase I - Vectorización del contexto

Se describe la fase de vectorización del contexto.

![Fase I](llms/docs/images/fase1.drawio.png)

En la Fase I se dividen los documentos del contexto proporcionado en chunks de un largo determinado. Luego mediante embeddings se vectorizan los chunks y se almacenan en la base de datos vectorial FAISS. De esta manera se facilita la búsqueda por similitud con las preguntas realizadas en el prompt.

### Fase II - Tuning del modelos y entrenamiento con casos del RT

Se describe la fase de tuning y entrenamiento a partir de casos del RT.

![Fase II](docs/images/fase2.drawio.png)
En la fase II se obtienen documentos Json, mediante consultas sql a la base de datos del RT, para luego entrenar al modelo con los resultados obtenidos. Además se realiza un fine tuning en la parametrización del modelo para generar respuestas más performantes. Luego de estas tareas se guarda el modelo entrenado y parametrizado para luego ser utilizado al ejecutar el prompt.

### Fase III - Carga del modelo entrenado

Luego de haber entrenado, tuneado y guardado el modelo, antes de comenzar a trabajar con el prompt se carga el modelo entrenado y tuneado.

![Fase III](docs/images/fase3.drawio.png)

### Fase IV - Carga de historial en sesión

En esta fase antes de comenzar a operar con el prompt se obtiene desde una base de datos SQLite el historial de consultas realizadas al modelo y se carga en la sesión del prompt. En el caso de repetirse alguna consulta, esta se toma del historial y no pasa por el modelo, de esta manera se optimiza la respuesta en casos de consultas ya realizadas con anterioridad.

![Fase IV](docs/images/fase4.drawio.png)

### Fase V - Prompt

Se describe la ejecución del prompt sobre el contexto determinado.

![Fase V](docs/images/fase5.drawio.png)

En esta fase mediante Gradio o Streamlit se genera una interfaz web para la interacción con el usuario. En primer lugar el usuario escribe una consulta en el panel de input de la interfaz, se hace una búsqueda en el historial de la sesión y si la consulta ya a sido realizada se brinda la respuesta dada, si no se pasa al siguiente paso. Si la consulta no está en el historial se tokeniza y se vectoriza mediante embeddings. Luego se hace una query por similitud sobre la base de datos vectorial donde están almacenados los chunks del contexto y se obtienen los 5 chunks con mayor similitud. Se le pasa la consulta y los chunks obtenidos del contexto al modelo para que elabore la respuesta más adecuada. Se brinda la respuesta al usuario y se almacena la consulta y la respuesta en el historial de la sesión.


