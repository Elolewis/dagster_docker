# Dagster Repository User Guide

This repository is designed to help you deploy and manage Dagster-compatible code locations into a central Dagster server. Follow the instructions below to add your code locations and get started.

## Prerequisites

- [Docker](https://docs.docker.com/engine/install/)
- [Docker Compose](https://docs.docker.com/compose/install/)

## Repository Structure

- `docker-compose.yml`: Defines the services required for the Dagster setup, including PostgreSQL, user code containers, Dagster webserver, and Dagster daemon.
- `dagster.yaml`: Configuration file for Dagster instance.
- `Dockerfile_user_code`: Dockerfile for building user code containers.
- `Dockerfile_dagster`: Dockerfile for building Dagster webserver and daemon containers.
- `entrypoint.sh`: Entrypoint script for user code containers.
- `workspace.yaml`: Defines the gRPC servers for Dagster to load user code.
- `repositories/`: Directory to store individual code locations.

## Adding a New Code Location

1. **Create a New Directory**:
   - Create a new directory under `repositories/` for your code location, e.g., `repositories/my_new_repo`.

2. **Add Your Code**:
   - Add your Dagster-compatible code to the new directory. Ensure there is a single entry point file, e.g., `my_new_repo.py`.

3. **Create a Requirements File**:
   - If your code has dependencies, create a `requirements.txt` file in the new directory.

4. **Update `docker-compose.yml`**:
   - Add a new service for your code location in the `docker-compose.yml` file. Use the existing `dagster_user_code_a` and `dagster_user_code_b` services as templates.
   - Ensure the `REPO_FILE` environment variable points to your entry point file, and the `REQ_FILE` environment variable points to your `requirements.txt` file.
   - Ensure that the first number in the port mapping (e.g. "4001" in ports: "4001:4000") specified for your new service does not conflict with existing services.

5. **Update `workspace.yaml`**:
   - Add a new entry under `load_from` in the `workspace.yaml` file for your new gRPC server. Use the existing entries as templates.
    - NOTE: the port refers to the internal port and should be set at 4000 to remain compatible with the attached dockerfile.

## Running the Services

1. **Build and Start the Services**:
   - Run the following command to build and start the services:
     ```sh
     docker-compose up --build
     ```

2. **Access Dagster Webserver**:
   - Open your browser and navigate to `http://localhost:3000` to access the Dagster webserver.

3. **Verify Your Code Location**:
   - Ensure your new code location is loaded and available in the Dagster webserver.

## Example

Here is an example of adding a new code location:

1. **Create Directory**:
   ```sh
   mkdir -p repositories/my_new_repo
   ```

2. **Add Code** (repositories/my_new_repo/my_new_repo.py):
   ```python
   from dagster import job, op

   @op
   def example_op():
       return "Hello, Dagster!"

   @job
   def example_job():
       example_op()
   ```

3. **Create Requirements file** (repositories/my_new_repo/requirements.txt):
   ```txt
   # Add any dependencies here
   ```
   OR
   within a virtual environment export your current requirements to file
   ```
    dasda
   ```
 
4. **Update docker-compose.yml:**
    ```
    services:
        ...

        dagster_user_code_c:
        build:
            context: .
            dockerfile: ./Dockerfile_user_code
        restart: always
        container_name: dagster_user_code_c
        image: dagster_user_code_image_c
        ports:
            - "4002:4000" # confirm that 4002 is not in use
        dns:
            - 8.8.8.8
        environment:
            DAGSTER_POSTGRES_USER: "postgres_user"
            DAGSTER_POSTGRES_PASSWORD: "postgres_password"
            DAGSTER_POSTGRES_DB: "postgres_db"
            REPO_FILE: "/opt/dagster/app/my_new_repo.py"
            REQ_FILE: "/opt/dagster/app/requirements.txt"
            DAGSTER_CURRENT_IMAGE: "dagster_user_code_image_C"
            DAGSTER_HOME: "/opt/dagster/dagster_home"
        volumes:
            - ./repositories/my_new_repo:/opt/dagster/app
        networks:
            - dagster_network

        ...
    ```


5. **Update workspace.yaml:**
    ```
    - grpc_server:
        host: dagster_user_code_c
        port: 4000
        location_name: "dagster_user_code_c"
    ```

6. **Build and Start Services:**
    ```
    docker-compose up --build
    ```
    you can now visit http://localhost:3000/ to see and manage your pipeline using the dagster webserver.

Now your new code location should be deployed and managed by the Dagster server.

## Conclusion
This guide provides the steps to add and manage Dagster-compatible code locations in this repository. Follow the instructions carefully to ensure your code is properly deployed and accessible through the Dagster webserver.
