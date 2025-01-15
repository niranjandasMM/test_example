**Local Mode:**
- All dependent services are started as part of the Compose setup and are accessed through exposed host ports.
- Hatbot services are started in local mode via provided scripts, which utilize the appropriate `local` environment for each setting.

**Dev/Prod Mode:**
- All dependent services are started as part of the Compose setup and are accessed within the Docker network.
- Hatbot services are started as part of the Compose setup and use the `dev` / `prod` configuration environment, which maps all required service hosts to appropriate container names based on the YAML configuration.


### Code Package Structure

The codebase is organized under the main package `hatbot`, which includes the following key components:

- **`hatbot.app`**: This package manages all logic related to FastAPI or REST API endpoints, structured following the MVC pattern for clear separation of concerns:
  - **`hatbot.app.routes`**: Acts as the entry point for external communication, handling incoming requests.
  - **`hatbot.app.controllers`**: Receives requests from endpoints and processes business logic.
  - **`hatbot.app.model_handlers`**: Provides abstraction classes with added functionalities to interact with SQLAlchemy models.
  - **`hatbot.app.models`**: Contains all database table definitions using SQLAlchemy.
- **`hatbot.cli`**: Command-line interface for executing the `hatbot-pipeline`.
- **`hatbot.config`**: Manages backend configuration settings.
- **`hatbot.pipeline`**: Contains logic for data ingestion pipelines and chatbot service processing.
- **`hatbot.langchain`**: Includes utilities for LangChain functionality.
- **`hatbot.queue`**: Provides services for Redis job queuing.
- **`hatbot.pubsub`**: Manages Redis-based publish-subscribe services.


## 2. Prerequisites

Please ensure that the following tools are installed on your system:

- [pyenv](https://github.com/pyenv/pyenv)
  - Version: `2.4.8`
- [Poetry](https://python-poetry.org/docs/#installing-with-pipx)
  - Version: `1.8.3`
- [Docker](docs/docker/README.md)
  - Version: `27.3.1 (build ce12230)`
  - [Docker Compose](https://docs.docker.com/compose/install/)
    - Version: `Docker Compose version v2.29.7-desktop.1`  
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
  - Version: `4.35.1 (173168)`
- [MakeFile](https://www.gnu.org/software/make/)
- Code Quality
  - [Pre commit Hooks](https://pre-commit.com/hooks.html)
  - [Pyre](https://pyre-check.org/docs/getting-started/)


## 3. Machine Setup

Clone the project
```bash
git clone https://github.com/HashAgileTech/gen-ai/
```

### 3.1 Python Virtual Environment Setup

#### 3.2.2 Using Poetry

When installing Poetry via `pipx`, it establishes its own isolated Python environment. 
However, it still requires an active Python executable. Below are the steps to install `pyenv` and 
set up the Python environment.

**For Ubuntu:**

```bash
sudo apt install -y make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev

curl https://pyenv.run | bash
echo -e 'export PYENV_ROOT="$HOME/.pyenv"\nexport PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo -e 'eval "$(pyenv init --path)"\neval "$(pyenv init -)"' >> ~/.bashrc
exec "$SHELL"
```

**For macOS:**

```bash
brew update
brew install pyenv

echo -e 'export PATH="$HOME/.pyenv/bin:$PATH"' >> ~/.zshrc
echo -e 'eval "$(pyenv init --path)"\neval "$(pyenv init -)"' >> ~/.zshrc

source ~/.zshrc
```

#### System Dependencies

**macOS:**
```bash
brew install libpq 
```

**Ubuntu:**
```bash
sudo apt-get build-dep python-psycopg2
sudo apt install python3-psycopg2 
```

#### Poetry Commands

The libraries are categorized into groups based on their purpose or functionality:
- **dev**: Development-related libraries to enhance code development and quality.
- **common**: Libraries shared across all Python modules.
- **app**: Dependencies specific to the FastAPI backend.
- **langchain**: Libraries for working with LLMs (Language Learning Models).
- **security**: Libraries for application security.
- **datapipeline**: Dependencies specific to run data pipelines.
- **test**: Libraries that support unit and integration testing.


To manage the Python environment and dependencies, use the following Poetry commands:

```bash
# Install Python version using pyenv
pyenv install 3.12.4

# Set the Python version for the current project root
pyenv local 3.12.4
pyenv shell 3.12.4
poetry env use python3.12

# Configure Poetry to create a virtual environment within the project root
poetry config virtualenvs.create true
poetry config virtualenvs.in-project true

# Switch to the project Python virtual environment
poetry shell

poetry lock

# Install all dependencies
poetry install --all-extras
# Or
poetry install --with <group1,group2>

# Add or update dependencies using Poetry commands within the virtual environment:
poetry add <new_package> --group {common/app/pipeline}

# Sync the local environment with the project’s library versions
poetry install --sync
```

Always use `poetry add <library> --group <group name>` to add any new dependencies. 
Ensure that you add the library to the correct group.  If you're unsure, consult your lead before proceeding.

#### 3.2.3 Without Poetry

While Poetry is the recommended method for managing the Python virtual environment, you may use your existing setup. Install the necessary dependencies using the following commands:

Activate your virtual environment and proceed with the installation:

```bash
pip install -r requirements_poetry.txt # Assuming the `requirements_poetry.txt` reflects the latest changes
```

If the `requirements_poetry.txt` file is outdated, update the dependencies as follows:

```bash
pip install poetry 
poetry export --dev --with langchain,vecdb,app,ui,ray,tools,test --without-hashes --format=requirements.txt > requirements_poetry.txt
```

Although not recommended, you may also use:

```bash
pip install poetry 
poetry install --all-extras
```

## 4. Git 

Commands to setup Pre commit hooks

```
# Install pre commit hooks
pre-commit install

# run to update hooks (for admins)
pre-commit autoupdate
```

Command to fix the source code formatting 

```
# run this command to fix code before each commit
pre-commit run --all-files
```

## 5. Run on Local Machine

```
make up-dep-services
./scripts/start.sh
```

Commands to create default table entries (use a separate terminal):

```
python ./scripts/create_super_user.py
python ./scripts/index_url.py
```

Command to reset Database:

```
./scripts/db_schema_reset.sh;
```

## 6. Unit Test

The testing framework used is `pytest`. For SQLAlchemy models, `pytest` fixtures are set up to initialize the database environment within a separate `test` schema, 
applying migrations similarly to the development environment. This approach replicates the development setup closely, 
ensuring that migrations run smoothly on `test` schema tables, helping to catch any migration issues early on before deployment.

For REST endpoint integration testing, `FastAPI`'s `TestClient` is utilized, as outlined in the FastAPI documentation [here](https://fastapi.tiangolo.com/tutorial/testing/#using-testclient).

```
cd /path/to/gen-ai/

export HATBOT_APP_ENV_FOR_DYNACONF=test
export HATBOT_DB_SCHEMA_NAME=test

pytest -s tests/hatbot_tests/unit/app/model_handler/test_role_handler.py;
TODO: validate other DB model test cases

 pytest -s tests/hatbot_tests/unit/app/model_handler/test_role_handler.py
TODO : fix pytest -s tests/unit/app/crud/test_roles.py; and more test cases
```

## 7. Run with Docker Containerization

### Docker Image Build

- [Dockerfile](ops/docker/api/cpu/Dockerfile)

Command to build the image locally (compulsory step!)

```bash
make build-api-cpu-image-dev
```

Command to access docker image bash for debugging
```bash
make debug-api-cpu-image-dev
```

Command to run with local Image  
```bash
make run-api-cpu-image-dev
```

Command to access running container bash
```bash
make exec-postgres
```

### Compose

Command to start FastAPI Backend with dependent services like Redis, Postgresql and PgAdmin
```
make up-dep-services
make up-backend

# or

make up
```

Command to stop
```
make down-dep-services
make down-backend

#or
 
make down
```

Command to remove along with volume
```
make down-dep-services-volume
make down-backend-volume

# or

make down-volume
```

## Migrations

The Alembic migration setup defaults to the `shared` schema for all application tables. 
For test environments, however, the `test` schema is used. Therefore, after generating a migration script, 
you may need to manually adjust the schema to match the intended environment.

To auto-generate a migration script, use:
```bash
alembic revision --autogenerate -m "Initial migration"
```

To apply the migration, run:
```bash
alembic upgrade head
```

## 9. Service URLs

- **FastAPI Docs**: http://0.0.0.0:8088/docs
- **PgAdmin**: http://localhost:5050/
  - Username: genaiteam@hashagile.com
  - Password: hashagile
- **Qdrant:** http://0.0.0.0:6333/dashboard
- **Elastic** : http://localhost:5601
  - Username: elastic
  - Password: hashagile

## 10. Code Development Workflow

Always rebase your main branch before starting a new task. When creating a merge request (MR), 
tag the corresponding GitHub issue (e.g., #12). This helps us track changes related to the issue effectively.

```bash
# Switch to the main branch
git checkout main
# Pull the latest changes
git pull 

# Create and switch to a new feature branch
git checkout -b <branch_name>
# Make changes and add new files
git add <files>

# Run pre-commit hooks
pre-commit run --all-files

# Stage the modified files
git add <files>
# Commit the changes with a descriptive message
git commit -m "#<git issue number>: message"
```
