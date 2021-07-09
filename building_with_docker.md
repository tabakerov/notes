# ALL HAIL ROBOTS!
or automate as much as you can

## at least the kind of work which requires a high level of precision, concentration, or just dumb, boring, repetitive

For example, I do not want to build my code by hand. So, let's try to create a setup where I (as creative and lazy human) will take a fun part while all other boring stuff will be handled by robots. In a consistent and reproducible way.

## Creating the project

I'll create a basic .NET 5 project for this example

    # create directory for the sample
    mkdir MyAwesomeThing && cd MyAwesomeThing
    # fix SDK version as I have a lot of them installed
    dotnet new globaljson --sdk-version 5.0.301
    # create dotnet project in BlazorProject/ directory
    dotnet new blazorserver -o BlazorProject
    # create dotnet solution...
    dotnet new sln
    # ...and add the project to the solution
    dotnet sln add BlazorProject/BlazorProject.csproj
    # add empty dockerfile
    touch Dockerfile

## Dockerfile

Below is the source for multi-stage Dockerfile. It uses .NET SDK image from Microsoft to restore dependencies and build the given solution and then put the result into a runtime container. That way, everythnig happens within a strictly defined containers environment, so no weird side effects from the developer's computer possible. Almost...

```
# https://hub.docker.com/_/microsoft-dotnet
FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build
WORKDIR /source

# copy csproj and restore as distinct layers
COPY *.sln .
COPY BlazorProject/*.csproj ./BlazorProject/
RUN dotnet restore

# copy everything else and build app
COPY BlazorProject/. ./BlazorProject/
WORKDIR /source/BlazorProject
RUN dotnet publish -c release -o /app --no-restore

# final stage/image
FROM mcr.microsoft.com/dotnet/aspnet:5.0
WORKDIR /app
COPY --from=build /app ./
ENTRYPOINT ["dotnet", "BlazorProject.dll"]
```

## Github Action

...but still, it is expected to have Docker running on a developer's computer. And that developer should have access to someplace where build artifacts should be stored. So the developer should have some additional secret keys from the artifact storage, should not forget to push... let's automate this.

Github stores the code, but it also has a way to build it (Github Actions) and store artifacts (Github Container Registry).

And there is the Github Action to:
- log in to Github Container Registry, 
- get metadata for tagging the artifact, 
- build the project (with existing Dockerfile) and push it to the registry

```
# This is a basic workflow to help you get started with Actions

name: CI

# Controls when the workflow will run
on:
  # Triggers the workflow on push or pull request events but only for the master branch
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:


  
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v2

      # Runs a single command using the runners shell
      - name: Run a one-line script
        run: echo Hello, world!
        
      - name: Log in to GHCR
        uses: docker/login-action@v1.10.0
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Add meta
        id: meta
        uses: docker/metadata-action@v3.4.0
        with:
          flavor: 
            latest=true
          images: ghcr.io/tabakerov/mysampleapp
          tags: type=sha

      - name: Build and push Docker image
        uses: docker/build-push-action@v2.6.1
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
```

You can take a look at [the sample project sources](https://github.com/tabakerov/SampleDockerizedBuildStudy)