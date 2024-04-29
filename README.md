# LLM-project
## Demo

## Overview
This project introduces a novel business analytics system using LangChain and OpenAI's GPT-4 to translate natural language into SQL queries. Our system integrates a semantic layer with GPT-4, enabling users without SQL expertise to interact with databases simply by asking questions in natural language. 

## Data
Refer to the `dataset_flattened.csv` file. Our dataset comprises 50 distinct models, spanning regression, classification, and multi-class types, each with up to three versions to allow for comparative analysis over time. The schema includes:

* Model Name: An identifier for the 50 unique models.
* Model Type: The category of the model (regression, classification, or multi-class).
* Model Version: Up to three iterations of each model to analyze improvements or changes.
* Date: The range of dates representing the data ingested by the model.
* Daily Volume: The volume of data processed by the model on a given day.
* Company Name: Labels for three fictitious insurance companies, designed to vary the data application across different business scenarios.
* Performance Metrics: Specific metrics such as Mean Absolute Error (MAE) for regression, and Accuracy, Precision, and Recall for classification and multi-class models, calculated for each version.

## Installation
Clone the repository and install the necessary dependencies:

### Step 1: Install virtual environment
```bash
python -m pip install virtualenv
```
### Step 2: Use virtual environment #######
```bash
python -m venv myenv
```
### Step 3: Activate ######
```bash
.\myenv\Scripts\activate
```
### Step 4: Install requiremnets ##
```bash
# this txt file includes most of the package required for running the code
pip install -r requirements.txt
```
### Step 5: Enter your LLM API Key
```bash
import os
from getpass import getpass

if not (openai_api_key := os.getenv("OPENAI_API_KEY")):
    openai_api_key = getpass("🔑 Enter your OpenAI API key: ")

os.environ["OPENAI_API_KEY"] = openai_api_key
```
## Usage
### To use the system, run the provided notebook:
The following order is based on the sequence of our development. We recommend following this order to get a clearer understanding of our entire project.

* `NLP_to_SQL.ipynb` - Basic version for generating SQL from natural language.
* `NLP_to_SQL_GPT3.5.ipynb` - Advanced version using GPT-3.5 for enhanced accuracy.
* `model_tuning.ipynb` - Notebook for tuning and refining the model performance.
* `app.py` - Front-end Development

### Work Flow
All the input follow this flow to generate the final answer

<img width="413" alt="image" src="https://github.com/julio1185/IDS/assets/143755012/3d75267f-54c2-4cf5-8e1e-8a9dc38d6e3d">


## Prompt Engineering Example
```bash
# Prompt to generate SQL queries
SQl_prompt = PromptTemplate.from_template(
    """You are a SQLite expert. Given an input question, create a syntactically correct SQLite query to run, to answer the input question.
Remember, your query should aim to answer the question with a single SQL statement and limit the results appropriately. You can order the results to return the most informative data in the database.
You MUST double check your query before executing it. If you get an error while executing a query, rewrite the query and try again.
Never query for all columns from a table. Only query the columns that are needed to answer the question. Wrap each column name in double quotes (") to denote them as delimited identifiers.
Be careful not to query for columns that do not exist. Also, pay attention to which column is in which table.
Pay attention to use the date('now') function to get the current date if the question involves "today".

Only use the following tables:

{table_info}.

Question: {input}
Provide up to {top_k} SQL variations."""
)

# Prompt to answer the questions
answer_prompt = PromptTemplate.from_template(
    """Given the following user question, corresponding SQL query, and SQL result, answer the user question.
    
Question: {question}
SQL Query: {query}
SQL Result: {result}
Answer: """
)
```
## LangChain Example
### First Chain

```bash
#First Chain to take the user input and convert it to a SQL query
write_query = create_sql_query_chain(llm, db)
response = write_query.invoke({"question": "What is the volume of Model 1"})
response 
```
### Second Chain

```bash
#Second chain to excute the SQL queries
execute_query = QuerySQLDataBaseTool(db=db)

chain = write_query | execute_query
chain.invoke({"question": "What is the volume of Model 1"})
```

### Combine Two Chain
```bash
def execute_combined_chain(question):
    # Step 1: Generate SQL Query
    sql_query = generate_sql_chain.invoke({"question": question, "top_k": 1})
    
    # Step 2: Execute the SQL Query to get the result
    sql_result = execute_query(sql_query)  # Ensure this returns the result of executing the SQL query
    
    # Step 3: Pass the necessary inputs to the final chain and format the output to include both SQL query and result
    final_response = execute_sql_chain.invoke({"question": question, "query": sql_query, "result": sql_result})
    
    # Format the final answer to include both the SQL query and its result
    final_answer = f"SQL Query: {sql_query}\n Answer: {final_response}"
    return final_answer
```
