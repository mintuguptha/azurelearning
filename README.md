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

## Deploy to Azure App Service

Install and sign in to the [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), then run these commands from this folder:

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

## Adding the ending.....
