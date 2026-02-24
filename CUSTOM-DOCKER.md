# Custom Docker Image

This project contains our custom Docker build for n8n.

What motivated this fork is in order to use EXIF tools in n8n, we need to install a community n8n node which depends on Perl being installed on the underlying system. In order to achieve this, the Docker image for n8n needs to be customized to add this manually. This fork will allow us to keep up to date with the latest n8n dockerfiles as a base, since they are updated relatively frequently.

https://www.npmjs.com/package/n8n-nodes-exif-data

In order to deploy a new version of n8n, we must take the docker files and build them manually in gcloud. We then use gcloud to replace our Cloud Run service with the new artifact.

Be sure to upload all related files to the gcloud terminal. These files can be found in `docker/images`

```
# 1. Clone your fork and checkout the custom-docker branch
git clone https://github.com/LeadsForwardLLC/n8n.git
cd n8n
git checkout custom-docker

## If already checked out
git pull origin custom-docker

# 2. Set your Google Cloud project
gcloud config set project apps-n8n

# 3. Perform cloud build
gcloud builds submit --config=cloudbuild.yaml

# 4. Deploy to Cloud Run
gcloud run deploy n8n
    --image=gcr.io/apps-n8n/n8n-custom:latest
    --command="/bin/sh"
    --args="-c,sleep 5;n8n start"
    --region=$REGION
    --allow-unauthenticated
    --port=5678
    --memory=2Gi
    --cpu 2
    --min-instances 1
    --max-instances 5
    --no-cpu-throttling
    --set-env-vars="NODE_ENV=production,N8N_PORT=5678,N8N_PROTOCOL=https,N8N_HOST=$(echo $SERVICE_URL | sed 's/https:\/\///'),WEBHOOK_URL=$SERVICE_URL,N8N_EDITOR_BASE_URL=$SERVICE_URL,DB_TYPE=postgresdb,DB_POSTGRESDB_DATABASE=n8n,DB_POSTGRESDB_USER=n8n-user,DB_POSTGRESDB_HOST=/cloudsql/$PROJECT_ID:$REGION:n8n-db,DB_POSTGRESDB_PORT=5432,DB_POSTGRESDB_SCHEMA=public,GENERIC_TIMEZONE=UTC,QUEUE_HEALTH_CHECK_ACTIVE=true"
    --set-secrets="DB_POSTGRESDB_PASSWORD=n8n-db-password:latest,N8N_ENCRYPTION_KEY=n8n-encryption-key:latest"
    --add-cloudsql-instances=$PROJECT_ID:$REGION:n8n-db
    --service-account=n8n-service-account@$PROJECT_ID.iam.gserviceaccount.com

```

Here are some commands we've used so far in the deployment:

```
export PROJECT_ID=apps-n8n
export REGION=us-east1
export SERVICE_URL="https://n8n-325859163001.us-east1.run.app/"

# Updating
gcloud run services update n8n
    --region=$REGION
    --update-env-vars="N8N_HOST=$(echo $SERVICE_URL | sed 's/https:\/\///'),WEBHOOK_URL=$SERVICE_URL,N8N_EDITOR_BASE_URL=$SERVICE_URL"

# Creating
gcloud run deploy n8n
    --image=n8nio/n8n:latest
    --command="/bin/sh"
    --args="-c,sleep 5;n8n start"
    --region=$REGION
    --allow-unauthenticated
    --port=5678
    --memory=2Gi
    --cpu 2
    --min-instances 1
    --max-instances 5
    --no-cpu-throttling
    --set-env-vars="N8N_PORT=5678,N8N_PROTOCOL=https,N8N_HOST=$(echo $SERVICE_URL | sed 's/https:\/\///'),WEBHOOK_URL=$SERVICE_URL,N8N_EDITOR_BASE_URL=$SERVICE_URL,DB_TYPE=postgresdb,DB_POSTGRESDB_DATABASE=n8n,DB_POSTGRESDB_USER=n8n-user,DB_POSTGRESDB_HOST=/cloudsql/$PROJECT_ID:$REGION:n8n-db,DB_POSTGRESDB_PORT=5432,DB_POSTGRESDB_SCHEMA=public,GENERIC_TIMEZONE=UTC,QUEUE_HEALTH_CHECK_ACTIVE=true"
    --set-secrets="DB_POSTGRESDB_PASSWORD=n8n-db-password:latest,N8N_ENCRYPTION_KEY=n8n-encryption-key:latest"
    --add-cloudsql-instances=$PROJECT_ID:$REGION:n8n-db
    --service-account=n8n-service-account@$PROJECT_ID.iam.gserviceaccount.com

```