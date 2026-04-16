## bor-workflow

> This project will create a workflow managment service implemented with the Prefect orchestration engine (https://docs.prefect.io/v3/get-started).

This project will create a workflow managment service implemented with the Prefect orchestration engine (https://docs.prefect.io/v3/get-started). 
A top requirement is that this is a light and simple to admin implementation of Prefect.
The context is an early stage project composed of docker container services, that has a next.js web application (docker container: bor-app), a node express api (docker container: bor-api), a mysql 8 database (docker container: bor-db), a filestore ftp service (docker container: bor-files), and this workflow service (docker container: bor-workflow). 
All docker containers are running in separate processes in separate docker containers and are connected by a docker network called bor-network. 
The bor-files, bor-db, and this bor-workflow share at least one docker persistent volume to be used in ETL and data processing workflows by this project. 
This bor-workflow service will be responsible for two sub-services: 
  1. bor-workflow-db: a postgresql database in a docker container using the standard best practice implementation of the database required to host the prefect server using a docker volume to persist the dataf. 
  2. bor-etl-agent: a service for executing prefect workflows and tasks. 

Initial workflow:
This service will be used initially in the following business case: a simple ETL ingestion will: 
1. check for the existence of an import file in the persistent docker volume shared with bor-files and bor-db. 
2. execute the 'LOAD DATA INFILE' command to ingest the file into a staging table in the 'borarch' database in the bor-db contanerized database called 'FundClassFee'. 
3. execute the stored procedure bormeta.usp_FundClassFee_Load . 
A simple and flexible implementation is required because the team has minimal experience with Prefect and just wants a basic but capable framework for scheduling and managing data processing related
workflows..
The service will run in peroduction in a docker container called bor-workflow

** Workflows implemented by this project are documented in /.workflows. ** 

https://docs.prefect.io/v3/deploy/infrastructure-examples/docker



## System port scheme:

container      dev   stage prod  docker-internal
-----------------------------------------------
bor-app        4400  4500  4600  3000
bor-api        4410  4510  4610  4000
bor-db         4420  4520  4620  3306
bof-message    4430  4530  4630  9092
bor-workflow   4440  4540  4640  4200 ** this container/service
    bor-workflow-db ...to be completed...
    bor-etl-agent   ...to be completed... (4450  4550  4650  8888)
bor-svc-calc   4460  4560  4660  5000
bor-files      4470  4570  4670  21


## prefect
The Prefect subsystem will use a containerized PostgreSQL database for orchestration state, following a simple, maintainable, and industry-standard architecture. The recommended production setup uses three containers:

### 1. Prefect Server/UI/API/CLI Container
- **Image:** Official Prefect image (`prefecthq/prefect:3-latest`)
- **Purpose:** Runs the Prefect server, UI, API, and CLI for orchestration, scheduling, and monitoring.
- **Ports:** Exposes 4640 (UI/API) to the host (e.g., `-p 4440:4200` for dev).
- **Management:** Start/stop with Docker (NOT Docker Compose). No custom ETL code or dependencies needed here.
- **Example:**
  ```bash
  docker run -d --rm \
    --name bor-workflow \
    --network bor-network \
    -p 4640:4200 \
    -e PREFECT_API_DATABASE_CONNECTION_URL="postgresql+asyncpg://prefect:prefect@bor-workflow-db:5432/prefect" \
    prefecthq/prefect:3-latest \
    prefect server start --host 0.0.0.0
  ```

### 2. ETL Agent/Worker Container (Custom Image)
- **Image:** Custom image based on `prefecthq/prefect:3-latest`, with your ETL dependencies (e.g., `mysql-client`, `pandas`, `mysql-connector-python`, etc.) installed.
- **Purpose:** Runs the Prefect agent, which picks up and executes your ETL flows and tasks.
- **Management:** Build and run as a separate container. Register this image with your Prefect work pool for flow execution.
- **Example Dockerfile:**
  ```dockerfile
  FROM prefecthq/prefect:3-latest
  RUN apt-get update && apt-get install -y mysql-client && \
      pip install pandas mysql-connector-python python-dotenv
  ```
- **Example run:**
  ```bash
  docker run -d --rm \
    --name bor-etl-agent \
    --network bor-network \
    -e PREFECT_API_URL="http://bor-workflow:4200/api" \
    custom-etl-image:latest \
    prefect agent start --work-pool 'default-agent-pool'
  ```

### 3. PostgreSQL Database Container (bor-workflow-db)
- **Image:** Official PostgreSQL image (e.g., `postgres:latest`)
- **Purpose:** Stores Prefect orchestration state and metadata.
- **Management:** Start/stop with Docker or Docker Compose. Data persisted via Docker volume.
- **Example:**
  ```bash
  docker run -d --name bor-workflow-db \
    --network bor-network \
    -v bor-workflow-db-data:/var/lib/postgresql/data \
    -e POSTGRES_USER=prefect \
    -e POSTGRES_PASSWORD=prefect \
    -e POSTGRES_DB=prefect \
    -p 5432:5432 \
    postgres:latest
  ```

### Management and Usage
- All containers are connected via the `bor-network` Docker network for secure, internal communication.
- The Prefect server and agent containers can be managed with Docker CLI or Docker Compose.
- Environment variables are used for all secrets and connection details, stored in `.env` files and passed to containers as needed.
- To deploy a flow:
  1. Build and apply the deployment from the agent container (or any container with the CLI and your code):
     ```bash
     docker exec -it bor-workflow bin/bash (or sh?)
     prefect deployment build src/workflows/file_ingestion.py:file_ingestion_workflow -n "File Ingestion" -q "default"
     prefect deployment apply file_ingestion_workflow-deployment.yaml
     ```
  2. Start the agent (if not already running):
     ```bash
     prefect agent start --work-pool 'default-agent-pool'
     ```
  3. Monitor and manage flows via the Prefect UI at `http://localhost:4440`.

### Summary Table
| Container           | Image                        | Purpose                        | Custom?         |
|---------------------|-----------------------------|--------------------------------|-----------------|
| Prefect Server      | prefecthq/prefect:3-latest  | UI, API, orchestration         | No              |
| ETL Agent/Worker    | Custom (from official base) | Runs your ETL flows/tasks      | Yes (for deps)  |
| bor-workflow-db     | postgres:latest             | Prefect state/metadata         | No              |

This architecture ensures a simple, maintainable, and production-ready Prefect deployment, with clear separation of concerns and easy extensibility for your ETL workflows.

## ########################################

Technology, general:
- This project will create a workflow management/orchestration service using the Prefect python library. 
- A docker network called 'bor-network' is used for interserice communication between this and other docker containers in the system.
- A docker volume called 'bor-files-data' will be used to persist Prefect's SQLite database, logs, and workflow state data between container restarts
- the project will use modern, best practices, and industry standard technologies for web application architecture and development.
- the project will use the latest version of Prefect, python, mysql client, and related technologies.


## technical details 
This project will be developed to a high standard of quality but will favour simplicity over sophistication in design and implementation.
Advice and input from AI will be given as an expert professional using industry best practices for the technologies used. 
All secrets will be stored in .env files using standard techniques for the technologies used. .env files will follow this naming convention:
- .env
- .env.development
- .env.development.local
- .env.production
- .env.production.local

## technologies used
- Typescript
- Python
- Prefect
- Mysql
- Docker
- Git
- Github and github actions for CI/CD to production

## developer details
You are a senior, experienced full stack system architect, designer and developer with years of production code in the above listed technologies. 
You have experience in data-centric environments including data management, etl, data validation, data processing. 
You follow industry best practices. 
You take pride in the quality of your work. 
You are technically competent but prefer simple implementations. 
You carefully consider tradeoffs to be made during design decisions. 


## RBAC considerations



## testing 
tbd 

## deployment 
 # to ensure prod has all of the .env files, some of which are .gitignored 
scp bor-db/.env* robmenning.com@xenodochial-turing.108-175-7-118.plesk.page:/var/www/vhosts/robmenning.com/bor/bor-db/
scp bor-api/.env* robmenning.com@xenodochial-turing.108-175-7-118.plesk.page:/var/www/vhosts/robmenning.com/bor/bor-api/
scp bor-app/.env* robmenning.com@xenodochial-turing.108-175-7-118.plesk.page:/var/www/vhosts/robmenning.com/bor/bor-app/
scp bor-files/.env* robmenning.com@xenodochial-turing.108-175-7-118.plesk.page:/var/www/vhosts/robmenning.com/bor/bor-files/
scp bor-workflow/.env* robmenning.com@xenodochial-turing.108-175-7-118.plesk.page:/var/www/vhosts/robmenning.com/bor/bor-workflow/


# DEV ITERATIONS

## deploying to the Prefect server (locally, on container)
./scripts/deploy-wf.sh


DEV:
1. clear; echo "------------------ $(date)";  ./scripts/manage-containers.sh stop; ./scripts/manage-containers.sh clear;  ./scripts/manage-containers.sh start dev; docker ps;

1.prod
clear; echo "------------------ $(date)";  ./scripts/manage-containers.sh stop; ./scripts/manage-containers.sh clear;  ./scripts/manage-containers.sh start prod; docker ps; echo "sleep 4"; sleep 4; docker logs bor-workflow

# deploy all workflows
docker exec -it bor-workflow prefect deploy --all (20250525: included in 'manage-containers.sh start')

# to copy a file into the volume accessible by bor-db: 
docker cp tests/data/holdweb-20241231.csv bor-db:/var/lib/mysql-files/ftpetl/incoming/holdweb-20241231.csv
# Check if the file exists in the expected location
docker exec bor-db ls -la /var/lib/mysql-files/ftpetl/incoming/holdweb-20241231.csv
# Check the file contents to ensure it copied correctly
docker exec bor-db head -10 /var/lib/mysql-files/ftpetl/incoming/holdweb-20241231.csv
docker exec bor-db head -10 /var/lib/mysql-files/ftpetl/incoming/holdweb-20241231.csv


# verify table
docker exec bor-db mysql -u borETLSvc -pu67nomZyNg -e "USE borarch; DESCRIBE holdweb;"


# verify sproc
docker exec bor-db mysql -u borETLSvc -pu67nomZyNg -e "SHOW CREATE PROCEDURE bormeta.usp_holdweb_process\G"
                                           

# to check if file is in the correct place for the intended flow:
docker exec -it bor-db ls -l /var/lib/mysql-files/ftpetl/incoming/holdweb-20241231.csv

to deploy a flow with the cli:
   prefect deployment build src/workflows/file_ingestion.py:file_ingestion_workflow -n "File Ingestion" -q "default"
   

This creates a deployment YAML file (e.g., file_ingestion_workflow-deployment.yaml).
apply the deployment
    prefect deployment apply file_ingestion_workflow-deployment.yaml

start an agent:
To actually run flows, you need a Prefect agent running and connected to your server.
In your container, you can start an agent with:
    prefect agent start --work-pool 'default-agent-pool'
(You may need to create a work pool in the UI or via CLI if it doesn't exist.)

4. Result
Once deployed, your flow will appear in the "Flows" and "Deployments" sections of the UI.
You can then trigger runs, schedule them, and monitor their status.

2. git push origin <branch>

PROD:
3. logged in as robmenning.com@xenodochial-turing...
3.a. cd ~/bor/bor-workflow
3.b. git pull origin <branch>
4. logged in as root@xenodochial-turing
4.a. cd /var/www/vhosts/robmenning.com/bor/bor-workflow
4.b. clear; *stop and start service...*; docker ps; docker logs bor-workflow



## running and monitoring in different environments:


### Development Scripts


### Docker Container Scripts


### Usage Instructions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/robmenning) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
