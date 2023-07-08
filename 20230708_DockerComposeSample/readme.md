## What is Docker-Compose ? [[1]](#1)
Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration.

Compose works in all environments: production, staging, development, testing, as well as CI workflows. It also has commands for managing the whole lifecycle of your application:
- Start, stop, and rebuild services
- View the status of running services
- Stream the log output of running services
- Run a one-off command on a service

The key features of Compose that make it effective are:
- Have multiple isolated environments on a single host
- Preserves volume data when containers are created
- Only recreate containers that have changed
- Supports variables and moving a composition between environments

<br>

## Purpose of the Sample Project
- Learn how to deploy mutliple services in dedicated linux-container-images using **docker-compose** & **DockerFile**.
- In this project, we to deploy the following 2 services:
  1. **MySql Database** service
  2. **Dotnet Core Web App**  that consumes the database service

<br>

## Requirements
- **OS:** Windows/Mac/Linux
- **IDE:** Visual Studio (Preferred) / VS Code
- .NET 6.0 (LTS), MySQL 8.0
- Docker

<br>

## Instructions / Steps
### A. Web App
1. Visual Studio  &rarr;  Create New Project
2. Provide Project Name
3. Select DotNet 6 (LTS)

### B. Directory for MySQL Database
1. **Go to:** `<Project-Parent-Directory>`
2. **Add New Directory:** “MysqlDatabaseInstance”

We wish to host the above 2 services in 2 separates linux-container-images

### C. Environment Variables (For all services)
1. **Go to:**  `<Project-Parent-Directory>`
2. Create **.env** file, with the following content:
    ```
    _MYSQL_DATABASE="DB_DockerCompose"
    _MYSQL_USER="useCase_db_user"
    _MYSQL_PASSWORD="useCase_db_password"
    _MYSQL_ROOT_PASSWORD="useCase_db_root_password"
    _MYSQL_DB_HOST_PORT=1111
    _MYSQL_DB_CONTAINER_PORT=3306
    
    _WEBAPP_HOST_PORT=2222
    _WEBAPP_CONTAINER_PORT=80
    ```
    
### D. Configure docker-compose
1. **Go to:**  `<Project-Parent-Directory>`
2. Create **docker-compose.yml** file with the following content:
    ```
    version: '1'

    services:     # Services to run (in dedicated images)
      web:                  # Service-1:  Web Application --> <Free to use any name>
        build:                    # Build-Application Reference (via Dockerfile)
          context:  ./MyApp
          dockerfile: Dockerfile
          args:                       # Arguments to pass to the <dockerFile>
            - _containerPort=${_WEBAPP_CONTAINER_PORT}
            - _hostPort=${_WEBAPP_HOST_PORT}
        ports:
          - "${_WEBAPP_HOST_PORT}:${_WEBAPP_CONTAINER_PORT}"
        depends_on:               # PRE-CONDITION:  'db' service [Refer Below] should be up & running
          - db
        
      db:                   # Service-2:  SQL Database --> <Free to use any name>
        build:                      # Database Configuration
          context: ./MysqlDatabase
          dockerfile: Dockerfile
          args:                       # Arguments to pass to the <dockerFile>
            - _containerPort=${_MYSQL_DB_CONTAINER_PORT}
            - _hostPort=${_MYSQL_DB_HOST_PORT}
        environment:                # DB Environment Variables set using --> Using: values set in .env
          - MYSQL_DATABASE = ${_MYSQL_DATABASE}
          - MYSQL_USER = ${_MYSQL_USER}
          - MYSQL_PASSWORD = ${_MYSQL_PASSWORD}
          - MYSQL_ROOT_PASSWORD = ${_MYSQL_ROOT_PASSWORD}
          - MYSQL_DB_HOST_PORT = ${_MYSQL_DB_HOST_PORT}
          - MYSQL_DB_CONTAINER_PORT = ${_MYSQL_DB_CONTAINER_PORT}
        volumes:                    # Makes DB persistent (even after restarts)   -   [Otherwise data is lost on each launch]
          - db-data:/var/lib/mysql 
        ports:
          - "${_WEBAPP_HOST_PORT}:${_WEBAPP_CONTAINER_PORT}"
        
    volumes:      # Volumes used (for persistence puspose)   -   [Otherwise data is lost on each launch]
      db-data:        # Helps make DB persistent (even after restarts)   -   [Otherwise data is lost on each launch]
    ```

### E. Configure dockerfile(s)
Create dedicated dockerfile for each service as follows:
1. **`<WebApp>` / DockerFile**
    ```
    # Use the official .NET 6.0 SDK as the build image
    FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
        # Set the working directory in the container
    WORKDIR /src
        # Copy the project file and restore dependencies
    COPY MyApp.csproj .
    RUN dotnet restore
        # Copy the remaining source code files
    COPY . .
        # Build the application
    RUN dotnet build -c Release --no-restore
        # Publish the application
    RUN dotnet publish -c Release --no-restore --output /app
    
    
    # Use the official ASP.NET Core runtime as the base image
    FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS runtime
        # Collect Arguments passed to this DockerFile  -->  [Needs to be collected where it is used (E.g. 'runtime' here)]
    ARG _containerPort
        # Set the working directory in the container
    WORKDIR /app
        # Copy the published output of the web application from the build image
    COPY --from=build /app .
        # Log
    RUN echo "Running on Port: ${_containerPort} inside container & exposed on port ${_hostPort} to others"
        # Start the web application when the container starts
    CMD ASPNETCORE_URLS=http://*50 dotnet MyApp.dll
    ```

2. **`<WebApp>` / DockerFile**
    ```
    # Use the official MySQL 8.0 Image
    FROM mysql:8.0
        # Collect Arguments passed to this DockerFile  -->  [Needs to be collected where it is used (E.g. 'runtime' here)]
    ARG _containerPort
        # Copy SQL script to initialize the DB
    COPY init.sql /docker-entrypoint-initdb.d/
        # Log
    RUN echo "Running on Port: ${_containerPort} inside container & exposed on port ${_hostPort} to others"
    ```

<br>

## Bibliographies
[1] <a id='1'> [Docker Compose](https://docs.docker.com/compose/) </a>

