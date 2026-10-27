# Flask app for Azure App Service

A minimal Flask application with a home page and a health-check endpoint.

## Run locally

From this folder, create and activate a virtual environment, then install dependencies:

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python app.py
```

Open <http://127.0.0.1:5000>. The health endpoint is at <http://127.0.0.1:5000/health>.

## Run tests

```powershell
python -m unittest discover -s tests
```

## Azure DevOps build and deployment flow

The Azure DevOps pipeline in [`azure-pipelines.yml`](./azure-pipelines.yml) builds, tests, packages, and deploys this Flask application to Azure App Service.

### Azure and GitHub setup

1. Create the Azure resource group and Linux App Service that will host the application.
2. Create an Azure DevOps service connection for the Azure subscription/resource access. The pipeline refers to the connection as `Azure-Service-Connection`; update the `azureServiceConnection` variable in the YAML if your connection has a different name.
3. Connect the GitHub repository to Azure DevOps and configure the pipeline to use this repository and its `azure-pipelines.yml` file.
4. Create an Azure Artifacts feed and grant the pipeline permission to publish packages to it. The current pipeline publishes to `firstproject/azurelearning-feeed` under the package name `azurelearning-app`.
5. Configure the App Service name in the pipeline. The current `webAppName` value is `vamshi`.

Use the actual resource group, App Service, feed, and service-connection names from your Azure DevOps and Azure setup. Do not put credentials in this repository; keep authorization in service connections and pipeline settings.

### Pipeline behavior

- Both CI builds and pull-request validation are configured for the `dev` branch.
- The **Build** stage selects Python 3.12, installs `requirements.txt` and `pytest`, compiles the Python files, and runs the tests with `pytest`.
- The application is packaged as `app.zip` and published as the `python-app` pipeline artifact.
- The package contents are also published to Azure Artifacts as a Universal Package. The feed and package names are configured in the YAML.
- After a successful build, the **Deploy** stage downloads the `python-app` artifact and deploys it to the Linux App Service using ZIP deploy. The pipeline enables `SCM_DO_BUILD_DURING_DEPLOYMENT`.

The YAML currently sets the Azure DevOps deployment environment to `production`. If the intended environment is `development`, update `environment: production` in the Deploy stage to the exact Azure DevOps environment name before relying on this documentation as the active deployment configuration.

### Work item and pull request linking

Connect GitHub to the Azure Boards project, then use the work item's **Development links** to associate the repository. When a pull request is raised against `dev`, Azure Boards can attach the pull request to the work item, keeping the code change and work item linked.

## Deploy to Azure App Service

For a manual deployment outside the Azure DevOps pipeline, install and sign in to the [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), then run these commands from this folder:

```powershell
az login
az webapp up --name <unique-app-name> --resource-group <resource-group> --location <region> --runtime "PYTHON:3.11"
```

Replace the placeholders with your Azure resource group, an available globally unique web app name, and an Azure region. `az webapp up` creates the App Service resources if needed and deploys the current folder.

If you configure the app manually in the Azure portal, use a Linux App Service with Python 3.11, deploy this folder's contents, and set the startup command to:

```text
gunicorn --bind=0.0.0.0 --timeout 600 app:app
```

After deployment, open `https://<unique-app-name>.azurewebsites.net`. The health endpoint is `https://<unique-app-name>.azurewebsites.net/health`.
