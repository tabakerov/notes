# ALL HAIL ROBOTS! NEED MORE ROBOTS! LET'S DELEGATE ALL WORK TO ROBOTS!

## at least the kind of work which requires high level of precision, concentration or just dumb, boring, repetitive.

For example, I do not want to build my code by hands. So, let's try to create a setup where robots will 

## Creating the project

    mkdir MyAwesomeThing && cd MyAwesomeThing
    dotnet new globaljson --sdk-version 5.0.301
    dotnet new blazorserver -o BlazorProject
    dotnet new sln
    dotnet sln add BlazorProject/BlazorProject.csproj
    touch Dockerfile

## Dockerfile
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

      # Runs a set of commands using the runners shell
      # - name: Build the docker thing
      #   run: docker build .
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
          # labels: 111

```

