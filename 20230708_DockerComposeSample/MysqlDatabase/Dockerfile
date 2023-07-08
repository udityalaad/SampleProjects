# Use the official MySQL 8.0 Image
FROM mysql:8.0
    # Collect Arguments passed to this DockerFile  -->  [Needs to be collected where it is used (E.g. 'runtime' here)]
ARG _containerPort
ARG _hostPort
    # Copy SQL script to initialize the DB
COPY init.sql /docker-entrypoint-initdb.d/
    # Log
RUN echo "Running on Port: ${_containerPort} inside container & exposed on port ${_hostPort} to others"