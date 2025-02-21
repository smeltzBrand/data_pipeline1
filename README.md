This is a data pipeline using sample retail orders from Snowflake. We will be using Docker along with dbt for transformation and Airflow for orchestration of our DAG. 

We will create some staging tables using the source table. One of the staging tables will use a generated surrogate key. 

Then we will create some downstream fact tables in our data mart to transform the data, using macros and referencing some dimensional models. 

There will also be generic and singlular tests used.

### Step-by-Step Instructions:

1. Clone the repository
```bash
git clone https://github.com/smeltzBrand/data_pipeline1.git
```
2. Navigate to the project
```bash
cd data_pipeline1
```
3. Install dependencies
```bash
# Optional virtual environment
# python -m venv venv
# On Windows:
# venv\Scripts\activate
# On Mac:
# source venv/bin/activate
pip install dbt-snowflake
```
4. Set up Snowflake environment
- Run all code under "-- create accounts", only run "--clean up" code when finished
```sql
-- create accounts
use role accountadmin;

create warehouse dbt_wh with warehouse_size='x-small';
create database if not exists dbt_db;
create role if not exists dbt_role;

show grants on warehouse dbt_wh;

grant role dbt_role to user {your username here};
grant usage on warehouse dbt_wh to role dbt_role;
grant all on database dbt_db to role dbt_role;

use role dbt_role;

create schema if not exists dbt_db.dbt_schema;

-- clean up
use role accountadmin;

drop warehouse if exists dbt_wh;
drop database if exists dbt_db;
drop role if exists dbt_role;
```

5. Configure your dbt profile
- Add the following line to dbt_project.yml
```yml
profile: 'data_pipeline1'
```
- In terminal run
```bash
export DBT_USER="your_snowflake_username"
export DBT_PASSWORD="your_snowflake_password"
# for account, go to Admin->Accounts in your snowflake UI to find acount name and locator
# or this can be found in the Snowflake url: https://app.snowflake.com/<account_name>/<account_locator>/#/homepage
export DBT_ACCOUNT="<account_name>-<account_locator>"
export DBT_ROLE="dbt_role"
export DBT_DATABASE="dbt_db"
export DBT_WAREHOUSE="dbt_wh"
export DBT_SCHEMA="dbt_schema"
```
```bash
mkdir -p ~/.dbt
nano ~/.dbt/profiles.yml
```
- Modify the profiles.yml and save it
```yml
my_dbt_project:
  target: dev
  outputs:
    dev:
      type: snowflake
      account: "{{ env_var('DBT_ACCOUNT') }}"
      user: "{{ env_var('DBT_USER') }}"
      password: "{{ env_var('DBT_PASSWORD') }}"
      role: "{{ env_var('DBT_ROLE') }}"
      database: "{{ env_var('DBT_DATABASE') }}"
      warehouse: "{{ env_var('DBT_WAREHOUSE') }}"
      schema: "{{ env_var('DBT_SCHEMA') }}"
      threads: 10
      client_session_keep_alive: False
```
6. Create your models and transformations from the project
```bash
dbt run
```
7. Run the generic and singular tests in the project
```bash
dbt test
```
8. Install Astro and clone pipeline
```bash
# Make sure that Docker is installed:
# brew install --cask docker
brew install astro
cd..
git clone https://github.com/smeltzBrand/dbt-dag.git
cd dbt-dag
```
9. Start up Airflow using Astro
```bash
astro dev start
```
- A locahost Airflow window should start up in your browser, if not go to: localhost:8080/login/
- enter Username: admin, Password: admin
10. Create Connection in Airflow
- make sure project_config in the dbt_dag.py file is the correct name/location
- In Airflow UI, click Admin->Connections
- Click [+] icon, enter the following info:
```markdown
Connection Id: "snowflake_conn"
Connection Type: Snowflake
Account: "<account_locator>-<account_name>",
Warehouse: "dbt_wh",
Database: "dbt_db",
Role: "dbt_role",
Insecure_mode: false
```
11. Start Connection
- Click on DAGs near top of Airflow UI
- Click on the play icon under the Action column for the "dbt_dag" item in table
