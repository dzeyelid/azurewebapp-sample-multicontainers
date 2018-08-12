[WIP]

A sample to deploy multi container to Azure Web App
====

References
----
- [Create a multi-container (preview) app in Web App for Containers | Microsoft Docs](https://docs.microsoft.com/en-us/azure/app-service/containers/tutorial-multi-container-app)


Prerequisite
----

- Azure Account
    - [Create your Azure free account today | Microsoft Azure](https://azure.microsoft.com/en-us/free/)
- Azure CLI
    - [Azure CLI 2.0 | Microsoft Docs](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest)

Roughly steps
----

1. Build containers
    - a wordpress container with plugins
    - a database container with initial data
2. Push these images to Azure Container Registry
3. Deploy them to Azure Web App

Notes
====

```bash
# Prepare variables
cp .env.sample .env
sed -i -E 's/^(DB_PASSWORD)=.*$/\1=<database password>/' .env
sed -i -E 's/^(DB_NAME)=.*$/\1=<database name>/' .env

# Put your dump file to initialize your database
cp <your dum file> ./services/db/dump.sql

# Prepare a volume
docker volume create --name=azurewebapp-sample-multicontainers-db-data

# Run containers
docker-compose up -d

# Access to the database
docker exec -it sample-db mysql -hlocalhost -p
# Enter password: <Enter your database password>

# Delete container instances
docker-compose down
```