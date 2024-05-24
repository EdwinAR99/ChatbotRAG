# RAG CHATBOT

## Descripcion

En este proyecto se desarrollo un chatbot RAG en LangChain que utiliza Neo4j para recuperar datos sobre los pacientes, las reseñas de los pacientes, las ubicaciones de los hospitales, las visitas, las compañías de seguros y los médicos del sistema hospitalario.

## Estructura de directorios

 * [`data`](https://github.com/EdwinAR99/ChatbotRAG/tree/master/data): En este directorios se encuentran los datos sin procesar del sistema hospitalario, almacenados como archivos CSV. Explorará estos datos. Estos datos a una base de datos Neo4j que su chatbot consultará para responder preguntas.
 * [`chatbot_api`](https://github.com/EdwinAR99/ChatbotRAG/tree/master/chatbot_api): Aqui esta core del chatbot, tambien contiene los subdirectorios [`chatbot_api/src/agents/`](https://github.com/EdwinAR99/ChatbotRAG/tree/master/chatbot_api/src/agents) y [`chatbot_api/src/chains/`](https://github.com/EdwinAR99/ChatbotRAG/tree/master/chatbot_api/src/chains) donde estan los objetos LangChain que componen el chatbot, ademas de un directorio [`chatbot_api/src/tools/`](https://github.com/EdwinAR99/ChatbotRAG/tree/master/chatbot_api/src/tools) donde se implementa una herramienta de simulacion para tiempos de espera.
 * [`hospital_neo4j_etl`](https://github.com/EdwinAR99/ChatbotRAG/tree/master/hospital_neo4j_etl): Este directorio contiene un script que carga los datos sin procesar de [`data/`](https://github.com/EdwinAR99/ChatbotRAG/tree/master/data) en una base de datos Neo4j basada en grafos. Este script se ejecuta antes de crear el chatbot.

 ## Directorio data/

Este dataset simula un gran sistema hospitalario. El cual contiene informacion sobre pacientes, reseñas de los pacientes, visitas, médicos, hospitales y las compañías de seguros. Todos los datos que se usaron en este proyecto se generaron sintéticamente de manera manual y tambien con GPT, ademas muchos de ellos se derivaron de un conjunto de datos [health care dataset](https://www.kaggle.com/datasets/prasad22/healthcare-dataset) en Kaggle.

Como se menciono antes, todos los registros estan guardados en archivos CSV, En esta sección se brindará una descripción detallada de cada archivo CSV.

* ### hospitals.csv
    En [`hospitals.csv`](https://github.com/EdwinAR99/ChatbotRAG/blob/master/data/hospitals.csv) se registro información de cada hospital. Hay 30 hospitales y tres columnas en este archivo.

    * _hospital_id_: ID única de un hospital.
    * _hospital_state_: Estado en el que se encuentra el hospital.

    Por ejemplo, que el hospital Walton, LLC tiene un ID de 2 y está ubicado en el estado de Florida, FL.

* ### physicians.csv
    En [`physicians.csv`](https://github.com/EdwinAR99/ChatbotRAG/blob/master/data/physicians.csv) contiene datos sobre los médicos que trabajan para el sistema hospitalario. Este conjunto de datos tiene los siguientes campos:

    * _physician_id_: ID único de cada médico.
    * _physician_name_: Nombre del médico.
    * _physician_dob_: Fecha de nacimiento del médico.
    * _physician_grad_year_: El año en que el médico se graduó en la facultad de medicina.
    * _medical_school_: Escuela de medicina a la que médico asistio.
    * _salary_: El salario del médico.

    Por ejemplo, Heather Smith tiene ID 3, nació el 15 de junio de 1965, se graduó de la escuela de medicina el 15 de junio de 1995, asistió a la Facultad de Medicina Grossman de la Universidad de Nueva York y su salario es de aproximadamente $ 295,239.

* ### payers.csv
    [`payers.csv`](https://github.com/EdwinAR99/ChatbotRAG/blob/master/data/payers.csv) se registro información sobre las compañías de seguros a las que sus hospitales facturan por las visitas de los pacientes. Similar a [`hospitals.csv`](https://github.com/EdwinAR99/ChatbotRAG/blob/master/data/hospitals.csv), es un archivo pequeño con un par de campos:

    * _payer_id_: ID única de cada compañía.
    * _payer_name_: Nombre de la compañía de seguros.

    Las únicas cinco compañías de seguros incluidas en los datos son Medicaid, UnitedHealthcare, Aetna, Cigna y Blue Cross.

* ### reviews.csv
    [`reviews.csv`](https://github.com/EdwinAR99/ChatbotRAG/blob/master/data/reviews.csv) contiene las reseñas de los pacientes sobre su experiencia en el hospital. Tiene estos campos:

    * _review_id_: ID único de una reseña.
    * _visit_id_: ID del paciente que realizó la reseña.
    * _review_: Esta es la reseña en texto que dejó el paciente.
    * _physician_name_: El nombre del médico que trató al paciente.
    * _hospital_name_: El hospital donde estuvo el paciente.
    * _patient_name_: El nombre del paciente.

    Por ejemplo, la reseña con ID 9 corresponde a la visita ID 8138, y las primeras palabras son “The hospital’s commitment to pat…”.

* ### patients.csv
    [`patients.csv`](https://github.com/EdwinAR99/ChatbotRAG/blob/master/data/patients.csv) cotiene informacion general de los pacientes. Estos son los campos:

    * _patient_id_: ID único del paciente.
    * _patient_name_: Nombre del paciente.
    * _patient_sex_: Genero del paciente, puede ser "MALE" o "FEMALE".
    * _patient_dob_: Fecha de nacimiento del paciente.
    * _patient_blood_type_: Tipo de sangre al que corresponde el paciente como "AB+", "AB-", "O+"y  "O-".

* ### visits.csv
    El último archivo, [`visits.csv`](https://github.com/EdwinAR99/ChatbotRAG/blob/master/data/visits.csv), se registro los detalles de cada visita al hospital. En [`visits.csv`](https://github.com/EdwinAR99/ChatbotRAG/blob/master/data/visits.csv) como una tabla de hechos que conecta hospitales, médicos, pacientes y pagadores. Estos son los campos:

    * _visit_id_: ID único de una visita al hospital.
    * _patient_id_: ID del paciente asociado con la visita.
    * _date_of_admission_: La fecha en que el paciente ingresó al hospital.
    * _room_number_: El número de habitación del paciente.
    * _admission_type_: Tipo de admision "Elective", "Emergency" o "Urgent".
    * _chief_complaint_: Cadena que describe el motivo principal del paciente para estar en el hospital.
    * _primary_diagnosis_: Cadena que describe el diagnóstico principal realizado por el médico.
    * _treatment_description_: Un resumen en texto del tratamiento administrado por el médico.
    * _test_results_: Un tipo de resultado "Inconclusive", "Normal" o "Abnormal".
    * _discharge_date_: La fecha en que el paciente fue dado de alta del hospital.
    * _physician_id_: ID del médico que trató al paciente.
    * _hospital_id_: ID del hospital en el que estuvo el paciente.
    * _payer_id_: ID de la compañia de seguros utilizado por el paciente.
    * _billing_amount_: Cantidad de dinero facturada a la compañia de seguros por la visita.
    * visit_status: Estado de la visita "OPEN" o "DISCHARGED".

## Directorio hospital_neo4j_etl/

### Descripcion
Este directorio contiene un proceso básico de extracción, transformación y carga (ETL) que mueve datos a Neo4j, por lo que sus únicas dependencias son neo4j y retry. El script principal para el ETL es [`hospital_neo4j_etl/src/hospital_bulk_csv_write.py`](https://github.com/EdwinAR99/ChatbotRAG/blob/master/hospital_neo4j_etl/src/hospital_bulk_csv_write.py).

Se definio un archivo [`entrypoint.sh`](https://github.com/EdwinAR99/ChatbotRAG/blob/master/hospital_neo4j_etl/src/entrypoint.sh) que se ejecutará en caso de iniciar un contenedor Docker.

### Directorio
```
hospital_neo4j_etl/
│
├── src/
│   ├── entrypoint.sh
│   └── hospital_bulk_csv_write.py
│
└── pyproject.toml
```
### Base de Datos
Las bases de datos de gráficos, como Neo4j, son bases de datos diseñadas para representar y procesar datos almacenados como un gráfico. Los datos del gráfico constan de nodos, aristas o relaciones y propiedades. Los nodos representan entidades, las relaciones conectan entidades y las propiedades proporcionan metadatos adicionales sobre nodos y relaciones. Por ejemplo, Paciente y Visita están conectados por la relación HAS, lo que indica que un paciente del hospital tiene una visita. De manera similar, Visita y Pagador están conectados por la relación COVERED_BY, lo que indica que un pagador de seguro cubre una visita al hospital.

![Diseño de base de datos en grafos](https://github.com/EdwinAR99/ChatbotRAG/blob/images/readme_images/graph.png)

Este diagrama muestra todos los nodos y relaciones en los datos del sistema hospitalario. Una forma útil de pensar en este diagrama de flujo es comenzar con el nodo Paciente y seguir las relaciones. Un paciente tiene una visita en un hospital y el hospital contrata a un médico para tratar la visita que está cubierta por un pagador de seguro.
Aquí están las propiedades almacenadas en cada nodo:

![Propiedades de los nodos](https://github.com/EdwinAR99/ChatbotRAG/blob/images/readme_images/properties.png)

Ademas estas son las propiedades de relaciónes:

![Propiedades de las relaciones](https://github.com/EdwinAR99/ChatbotRAG/blob/images/readme_images/relations.png)

## Directorio chatbot_api/

### Descripcion
Como se menciono antes, aqui es donde esta core del chatbot, que esta dividido en estos subdirectorios: 
* [`chatbot_api/src/agents/`](https://github.com/EdwinAR99/ChatbotRAG/tree/master/chatbot_api/src/agents): Aqui esta el agente, que define que herramienta debe usar dependiendo de la peticion del usuario.
* [`chatbot_api/src/chains/`](https://github.com/EdwinAR99/ChatbotRAG/tree/master/chatbot_api/src/chains): En esta parte estan los LLM que se encargan de recibir las peticiones reenviadas por el agente, estas son procesadas y respondidas con ayuda de la conexion a la BD instanciada en Neo4j.
* [`chatbot_api/src/tools/`](https://github.com/EdwinAR99/ChatbotRAG/tree/master/chatbot_api/src/tools): Aqui se simula una herramienta para tiempos de espera, ademas de implementar una funcion para verificar que determinado hospital exista en los registros.

### Directorio
```
chatbot_api/
│
├── src/
│   │
│   ├── agents/
│   │   └── hospital_rag_agent.py
│   │
│   ├── chains/
│   │   ├── hospital_cypher_chain.py
│   │   └── hospital_review_chain.py
│   │
│   └── tools/
│       └── wait_times.py
│
└── pyproject.toml
```

### hospital_review_chain.py
Este script se encargar de la creación de una cadena que responde preguntas sobre las experiencias de los pacientes utilizando sus reseñas usando Neo4j como índice de vector. El indice del vector permite ejecutar consultas semánticas directamente en los gráfos. Haciendo que el chatbot pueda almacenar embeddings de reseñas. Con LangChain, se usa Neo4jVector para crear embeddings de reseñas y un retriever necesario para la cadena que contiene la peticion del usuario.

### hospital_cypher_chain.py
Este script se usa para hacer consultas mas complejas con la BD, en resumen este script aceptará la consulta en lenguaje natural de un usuario, convertirá la consulta en una consulta Cypher y ejecutará la consulta Cypher en Neo4j y utilizará los resultados para responder a la consulta del usuario. Usando  una instancia de un Neo4jGraph para permitir a los LLM ejecutar consultas en su instancia de Neo4j, este script utiliza 2 templates, primero `cypher_generation_template` que proporcionando al LLM instrucciones muy específicas sobre lo que debe y no debe hacer al generar consultas Cypher y `qa_generation_template` que le indica al LLM que utilice los resultados de la consulta Cypher para generar una respuesta con un formato agradable a la consulta del usuario.

### wait_times.py
Este script le da al chatbot la capacidad responder preguntas sobre los tiempos de espera en los hospitales, esta informacion no se almacena en ningún lugar, por lo que su chatbot usa dos funciones para esto: una simula encontrar el tiempo de espera actual en un hospital y la otra encuentra el hospital con el tiempo de espera más corto.

### hospital_rag_agent.py
Este script crea un agente que actuará como su chatbot. Dependiendo de la consulta que se le haga, el agente debe decidir entre las funciones de una cadena Cypher, cadena de reseñas y tiempos de espera. Aqui se está importando `reviews_vector_chain`, `hospital_cypher_chain`, `get_current_wait_times()` y `get_most_available_hospital()`. El agente los utilizará directamente como herramientas. _HOSPITAL_AGENT_MODEL_ (Variable de entorno) es el LLM que actuará como el cerebro de su agente, decidiendo qué herramientas llamar y qué entradas pasarles.

Al final agente tiene cuatro herramientas disponibles: **_Reseñas_**, **_Grafos_**, **_Tiempos de espera_** y **_Tiempos de disponibilidad_**. Las herramientas **_Reseñas_** y **_Grafos_** se llaman desde su metodo `.invoke()` con sus respectivas cadenas, mientras que **_Tiempos de espera_** y **_Tiempos de disponibilidad_** llaman a las funciones de tiempo de espera. Ademas, las herramientas tienen descripciones con indicaciones breves que le indican al agente cuándo debe usar la herramienta y le brindan un ejemplo de qué entradas pasar.

## Instalacion

Si se desea recrear este chatbot en un entorno local, puede seguir los pasos que se mostraran a continuacion, aunque primero debe cumplir con los siguiente requisitos:

### Requisitos
* Contar con una version de _python 3.12_ o superior.
* Instancia de Neo4j: Antes que nada, es necesario crear una cuenta en [Neo4j](https://neo4j.com/), despues de crear la cuenta solo debe crear una instancia en su la plataforma de [aura](https://console.neo4j.io/) de Neo4j, no es necesaria una configurar especial, todo fue creado con opciones por defecto.
* API_KEY de OpenAI: Todo el procesamiento de cadenas es potenciado por modelos GPT de OpenAI, por lo tanto es necesario contar con una clave de acceso a su API, para ello puede crear una cuenta desde cero en la [pagina de OpenAI](https://openai.com/index/openai-api/), con lo cual se le daran creditos gratis, en caso de ya tener una cuenta, puede revisar que sus creditos no esten vencidos, si es asi, puede recargar o crear una nueva cuenta. Despues de crear una cuenta, puede ir a este [enlace](https://platform.openai.com/api-keys) para crear su propia API_KEY.

> NOTA: Tanto las credenciales para Neo4j como la api_key para OpenAI, solo se muestran al momento de crearlas, por lo tanto guardelas donde no se puedan perder.

### 1. Clonar Repositorio.
Primero debe clonar este repositorio o tambien lo puede descargar directamente desde este [enlace](https://github.com/EdwinAR99/ChatbotRAG/archive/refs/heads/master.zip), para clonarlo puede seguir estos pasos.
* En una consola de git
```
git clone https://github.com/EdwinAR99/ChatbotRAG.git
```
Con esto ya tendra todos los directorios que necesta.

### 2. Configurar archivo de variable de entorno.
Para evitar cambiar ingresar directamente algun api_key en el codigo, todas estas variables se asignan en un archivo `.env` que tendra que crear en la carpeta raiz del proyecto (ChatbotRAG), y debera configurarlo de la siguiente manera.

```
OPENAI_API_KEY=<TU_OPENAI_API_KEY>

NEO4J_URI=<TU_NEO4J_URI>
NEO4J_USERNAME=<TU_NEO4J_URI>
NEO4J_PASSWORD=<TU_NEO4J_PASSWORD>

HOSPITALS_CSV_PATH=https://raw.githubusercontent.com/EdwinAR99/ChatbotRAG/master/data/hospitals.csv
PAYERS_CSV_PATH=https://raw.githubusercontent.com/EdwinAR99/ChatbotRAG/master/data/payers.csv
PHYSICIANS_CSV_PATH=https://raw.githubusercontent.com/EdwinAR99/ChatbotRAG/master/data/physicians.csv
PATIENTS_CSV_PATH=https://raw.githubusercontent.com/EdwinAR99/ChatbotRAG/master/data/patients.csv
VISITS_CSV_PATH=https://raw.githubusercontent.com/EdwinAR99/ChatbotRAG/master/data/visits.csv
REVIEWS_CSV_PATH=https://raw.githubusercontent.com/EdwinAR99/ChatbotRAG/master/data/reviews.csv

HOSPITAL_AGENT_MODEL=gpt-3.5-turbo-1106
HOSPITAL_CYPHER_MODEL=gpt-3.5-turbo-1106
HOSPITAL_QA_MODEL=gpt-3.5-turbo-0125
```
> NOTA: Las rutas de los archivos CSV estan dirigidas a este repositorio, porque para que la BD de Neo4j pueda acceder a los archivos, estos deben estar en la nube, ya que no podria acceder a tu entorno local.

Una vez creado el archivo, debe tener su directorio asi.

```
ChatbotRAG/
│
├── chatbot_api/
|   └─ ...
|
├── hospital_neo4j_etl/
|   └─ ...
│
└── .env
```
### 3. Crear entorno virtual de python (Opcional).
Como se muestra, este paso es opcional, se recomiendan crear un entorno virtual para evitar conflictos entre librerias del entorno global y las necesarias para este proyecto.

Para crear el entorno solo siga estos pasos:
* Definir el entorno
```
python -m venv <nombre del entorno>
```
* Activar entorno virtual
```
<nombre del entorno>\Scripts\activate
```

Ahora puede instalar librerias sin tener complicaciones con el entorno global de python.

### 4. Enviar CSV a la instancia en Neo4j.
Como se menciono antes [`hospital_bulk_csv_write.py`](https://github.com/EdwinAR99/ChatbotRAG/blob/master/hospital_neo4j_etl/src/hospital_bulk_csv_write.py) contiene el script donde se procesan todos los archivos CSV que se encuentran en data, se envian a la instancia en Neo4j, para ello solo debe ejecutar estos comandos desde la carpeta raiz:

* Instalar de librerias
```
python -m pip install hostpital_neo4j_etl
```
* Ejecutar el script
```
hostpital_neo4j_etl/src/entrypoint.sh
```

Si todo funciono bien, no deberia salir ningun error terminar el proceso de forma correcta, mostrando todos los registros de las conexiones con la base de datos.

### 5. Ejecutar chatbot_api.
Ya para terminar, para ejecutar el bot puede hacerse de dos formas, aunque primero es necesario ejecutar la instalacion de librerias, desde la carpeta raiz escribir el siguiente comando:
```
python -m pip install chatbot_api/
```
Automaticamente se instalaran las librerias que utilizan los scripts, ahora para probar el bot, puede crear un archivo .py o bien, desde el interprete de python.
* Creando un archivo .py
Primero se debe instalar la librera de `python-dotenv` para poder cargar las variables de entorno, ejecute el siguiente comando:\
```
pip install python-dotenv
```
Es posible crear un archivo con la logica para ejecutar el agente o chatbot, para ello debe asegurarse de crearlo en la carpeta raiz, puede usar el siguiente script para probar el bot.

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Custom Code Highlighting</title>
    <style>
        body {
            font-family: Arial, sans-serif;
        }
        pre {
            padding: 10px;
            border-radius: 5px;
            overflow: auto;
            white-space: pre-wrap; /* Permite el ajuste de línea automático */
        }
        .keyword { color: #007acc; font-weight: bold; }
        .function { color: #795548; }
        .string { color: #d32f2f; }
        .comment { color: #757575; font-style: italic; }
        .indent { margin-left: 20px; }
    </style>
</head>
<body>
<pre><code class="python">
<span class="keyword">import</span> dotenv
dotenv.<span class="function">load_dotenv</span>()

<span class="keyword">from</span> chatbot_api.src.agents.hospital_rag_agent <span class="keyword">import</span> hospital_rag_agent_executor

<span class="keyword">while</span> <span class="keyword">True</span>:
    <span class="indent">input_text = <span class="function">input</span>(<span class="string">"Introduce tu pregunta (o escribe 'salir' para terminar): "</span>)</span>
    <span class="indent"><span class="keyword">if</span> input_text.lower() == <span class="string">'salir'</span>:</span>
        <span class="indent"><span class="indent"><span class="function">print</span>(<span class="string">"Saliendo del programa."</span>)</span></span>
        <span class="indent"><span class="indent"><span class="keyword">break</span></span></span>
    <span class="indent">response = hospital_rag_agent_executor.<span class="function">invoke</span>({<span class="string">"input"</span>: input_text})</span>
    <span class="indent"><span class="function">print</span>(response.<span class="function">get</span>(<span class="string">"output"</span>))</span>
</code></pre>
</body>
</html>

De esta forma puede simular la conversacion con el bot.

* Con el interprete de python
Esta forma simplificada del proceso, aunque engorrosa si tiene quiere repetirlo, ya que tendra que escribir lo mismo una y otra vez para importar las funciones. Para hacerlo simplemente ejecute los siguientes comandos:
    * Abrir interprete de python.
```
python
```
Notara que apareceran este simbolo en su terminal `>>>`, ahora debe importar las librerias y digitar linea a linea e invocar al bot, ejecute los siguientes comandos
```
import dotenv
```
```
dotenv.load_dotenv()
```
```
from chatbot_api.src.agents.hospital_rag_agent import hospital_rag_agent_executor
```
Aqui se invoca al bot, pasandole un diccionario con una clave `input` y un valor `"Muéstrame reseñas escritas por el paciente 7674."` (Puede cambiarlo por la pregunta que guste), la terminal le mostrara la funcion que invoca el agente, su consulta y un diccionario con informacion sobre su peticion.
```
response = hospital_rag_agent_executor.invoke({"input": "Muéstrame reseñas escritas por el paciente 7674."})

> Entering new AgentExecutor chain...

Invoking: `Experiences` with `Show me reviews written by patient 7674.`

{'query': 'Show me reviews written by patient 7674.', 'result': 'I\'m sorry, but there are no reviews provided by a patient with the identifier "7674" in the context given. If you have any other questions or need information about the reviews provided, feel free to ask.'}Lo lamento, pero no hay reseñas proporcionadas por un paciente con el identificador "7674" en el contexto dado. Si tienes alguna otra pregunta o necesitas información sobre las reseñas proporcionadas, no dude en preguntar.

> Finished chain.
```
Con este comando puede extraer la respuesta del bot.
```
response.get("output")

'Lo lamento, pero no hay reseñas proporcionadas por un paciente con el identificador "7674" en el contexto dado. Si tienes alguna otra pregunta o necesitas información sobre las reseñas proporcionadas, no dude en preguntar.'
```

Con esto habra terminado de montar el bot en su repositorio local.

## Trabajos futuros
Este repositorio puede ser usado de forma libre, una posibilidad a futuro es implementar el bot en un APIREST para consumir desde un frontend funcional, y simular un chat con un usuario real.
