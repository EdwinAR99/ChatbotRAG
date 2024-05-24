# RAG CHATBOT

## Descripcion

En este proyecto se desarrollo un chatbot RAG en LangChain que utiliza Neo4j para recuperar datos sobre los pacientes, las reseñas de los pacientes, las ubicaciones de los hospitales, las visitas, las compañías de seguros y los médicos del sistema hospitalario.

## Estructura de directorios

 * [`data`](https://github.com/EdwinAR99/ChatbotRAG/tree/master/data): En este directorios se encuentran los datos sin procesar del sistema hospitalario, almacenados como archivos CSV. Explorará estos datos. Estos datos a una base de datos Neo4j que su chatbot consultará para responder preguntas.
 * [`chatbot_api`](https://github.com/EdwinAR99/ChatbotRAG/tree/master/chatbot_api): Aqui esta core del chatbot, tambien contiene los subdirectorios [`chatbot_api/src/agents/`](https://github.com/EdwinAR99/ChatbotRAG/tree/master/chatbot_api/src/agents) y [`chatbot_api/src/chains/`](https://github.com/EdwinAR99/ChatbotRAG/tree/master/chatbot_api/src/chains) donde estan los objetos LangChain que componen el chatbot.
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
Las bases de datos de gráficos, como Neo4j, son bases de datos diseñadas para representar y procesar datos almacenados como un gráfico. Los datos del gráfico constan de nodos, aristas o relaciones y propiedades. Los nodos representan entidades, las relaciones conectan entidades y las propiedades proporcionan metadatos adicionales sobre nodos y relaciones. Por ejemplo, Paciente y Visita están conectados por la relación HAS, lo que indica que un paciente del hospital tiene una visita. De manera similar, Visita y Pagador están conectados por la relación COVERED_BY, lo que indica que un pagador de seguro cubre una visita al hospital.

## Instalacion

1. Clonar Repo(TO DO especificar)
2. Crear instancia de BD en Neo4j
3. Ejecutar chatbot_api




