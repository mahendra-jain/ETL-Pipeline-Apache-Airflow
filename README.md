# ETL-Pipeline-Apache-Airflow

Initial ubuntu commands:

# create the virtual env
source ~/airflow-env/bin/activate

# Go to the dir /airflow/dags
cd ~/airflow/dags

#create a new py file using the vs code
code etl_pipeline.py

#ls to see the file in the folder
ls

#run the scheduler
airflow scheduler

#run the webserver (in different terminal)
airflow webserver -p 8080


# etl_pipeline.py (code):

	from airflow import DAG
	from airflow.operators.python import PythonOperator
	from datetime import datetime, timedelta
	import pandas as pd

	def extract():
	    print("Extracting data...")
	    df = pd.read_csv("/mnt/d/Data Analyst/AirFlow ETL Pipeline/Analyzing Employee Trends.csv")
	    return df.to_dict()
	
	def transform(**context):
	    print("Transforming data...")
	    df = pd.DataFrame(context['ti'].xcom_pull(task_ids="extract"))
	    df = df.dropna()
	    return df.to_dict()
	
	def load(**context):
	    print("Loading data...")
	    df = pd.DataFrame(context['ti'].xcom_pull(task_ids="transform"))
	    output_path = "/mnt/d/Data Analyst/Airflow ETL Pipeline/Cleaned.csv"
	    df.to_csv(output_path, index=False)
	    print(f"File saved to: {output_path}")
	
	default_args = {
	    "owner": "mahendra",
	    "start_date": datetime(2025, 10, 1),
	    "retries": 1,
	    "retry_delay": timedelta(minutes=5),
	}
	
	with DAG(
	    dag_id="etl_pipeline_employee_trends",
	    default_args=default_args,
	    description="Simple CSV ETL pipeline to Analyze the employee trends",
	    schedule_interval="@hourly",   # run every hour
	    catchup=False,
	) as dag:
		t1 = PythonOperator(task_id="extract", python_callable=extract)
	    t2 = PythonOperator(task_id="transform", python_callable=transform, provide_context=True)
	    t3 = PythonOperator(task_id="load", python_callable=load, provide_context=True)

	    t1 >> t2 >> t3

