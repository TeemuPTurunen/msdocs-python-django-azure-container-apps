# Deploy a Python (Django) app to Azure to Azure Containers Apps

This Python app is a simple restaurant review application built with the [Django](https://www.djangoproject.com/) framework. The app stores application data in PostgreSQL with environment variables defining the connection info.

This repo was created to be built to a Docker image and run as a container instance in [Azure Container Apps](https://azure.microsoft.com/services/container-apps/). For more information, see the tutorial [Deploy a Python web app on Azure Container Apps with GitHub Actions](https://learn.microsoft.com/azure/developer/python/tutorial-deploy-python-web-app-azure-container-apps-01).

This Python web app repo can also be used in other ways:

* You can run the web app locally in a virtual environment. Make sure to define *.env* file with environment settings.

  * In the `.env` file, fill in a secret value for `SECRET_KEY`. You can use this command to generate an appropriate value for your product deployment:

    ```shell
    python -c 'import secrets; print(secrets.token_hex())'
    ```

* You can create a container locally and run it in Docker locally. You need Docker Desktop installed. For this scenario, set REMOTE_POSTGRESQL=1 and other environment variables in a *.env* file to point to a PostgreSQL server instance in Azure. You also need to create an application service principal, add it as a PostgreSQL ROLE to the server, and grant it necessary permissions on the database. See the *.env.example* file for details.

  ```bash
  docker build --file Dockerfile --tag pythoncontainer:latest .
  docker run -it --env-file .env --publish 8000:8000/tcp pythoncontainer:latest
  ```

  If you want to use PostgreSQL instance locally, you add `--add-host` to the Docker command. For more information, see the [Docker run](https://docs.docker.com/engine/reference/commandline/run/) command. For an example of how to do this with MongoDB, see [Build and test a containerized Python web app locally](https://docs.microsoft.com/azure/developer/python/tutorial-containerize-deploy-python-web-app-azure-02).

* You can deploy the code (not a container) to App Service. For guidance on how to deploy code, see [Quickstart: Deploy a Python (Django or Flask) web app to Azure App Service](https://docs.microsoft.com/azure/app-service/quickstart-python) and [Overview: Deploy a Python web app to Azure with managed identity](https://docs.microsoft.com/azure/developer/python/tutorial-python-managed-identity-01).

* You can create a Docker image from this repo and host the container image in Web Apps for Containers (App Service). See [Containerized Python web app on Azure](https://docs.microsoft.com/azure/developer/python/tutorial-containerize-deploy-python-web-app-azure-01).

If you need an Azure account, you can [create on for free](https://azure.microsoft.com/free/).

A Flask sample application with similar functionality is at https://github.com/Azure-Samples/msdocs-python-flask-azure-container-apps.

## Requirements

The [requirements.txt](./requirements.txt) has the following packages:

| Package | Description |
| ------- | ----------- |
| [Django](https://pypi.org/project/Django/) | Web application framework. |
| [pyscopg2-binary](https://pypi.org/project/psycopg-binary/) | PostgreSQL database adapter for Python. |
| [gunicorn](https://pypi.org/project/gunicorn/) | WSGI HTTP Server for UNIX. Required for running containers in [VS Code](https://code.visualstudio.com/docs/containers/quickstart-python#_gunicorn-modifications-for-djangoflask-apps).|
| [python-dotenv](https://pypi.org/project/python-dotenv/) | Read key-value pairs from .env file and set them as environment variables. In this sample app, environment variables describe how to connect to the database and storage resources. Because managed identity is used no sensitive information is included in environment variables. <br><br> This package is used in the [manage.py](./manage.py) file to load environment variables. |
| [requests](https://pypi.org/project/requests/) | Python HTTP for Humans. |
| [whitenoise](https://pypi.org/project/whitenoise/) | Static file serving for WSGI applications, used in the deployed app. <br><br> This package is used in the [azureproject/production.py](./azureproject/production.py) file, which configures production settings. |

## Deploying to Azure Container Apps

The steps to do this are covered more completely in the tutorial [Deploy a Python web app on Azure Container Apps with GitHub Actions](https://learn.microsoft.com/azure/developer/python/tutorial-deploy-python-web-app-azure-container-apps-01). Briefly, here are the steps:

1. Fork and then clone locally.
1. Create an Azure Container Registry.
1. Build a container image from the repo.
1. Create an Azure Database for PostgreSQL Flexible Server instance.
1. Make your Azure user account a Microsoft Entra administrator on the server.
1. Create a database on the server.
1. Create a user-assigned managed identity.
1. Add the managed identity as a ROLE on the PostgreSQL server.
1. Assign the managed identity PostgreSQL permissions on the database.
1. Deploy the web app container to Azure Container Apps.
1. Migrate and create database schema by connecting to the container app and issuing the `python manage.py migrate` command.
1. Configure continuous deployment.

See the [create_resources.sh](./create_resources.sh) Bash script for automating the creation of these steps. For best results, run the script in Azure Cloud Shell.
