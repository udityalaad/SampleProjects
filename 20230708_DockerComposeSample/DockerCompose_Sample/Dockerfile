# Use the official .NET 6.0 SDK as the build image
FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
    # Set the working directory in the container
WORKDIR /src
    # Copy the project file and restore dependencies
COPY DockerCompose_Sample.csproj .
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
ARG _hostPort
    # Set the working directory in the container
WORKDIR /app
    # Copy the published output of the web application from the build image
COPY --from=build /app .
    # Log
RUN echo "Running on Port: ${_containerPort} inside container & exposed on port ${_hostPort} to others"
    # Start the web application when the container starts
CMD ASPNETCORE_URLS=http://*80 dotnet DockerCompose_Sample.dll