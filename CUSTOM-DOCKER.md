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

# 2. Set your Google Cloud project
gcloud config set project apps-n8n

# 3. Build n8n locally first
pnpm install --frozen-lockfile
pnpm build > build.log 2>&1

# 4. Build the Docker image and push to Google Container Registry
# Build the application
node scripts/build-n8n.mjs

# Build and tag the Docker image
docker build -f docker/images/n8n/Dockerfile -t gcr.io/YOUR_PROJECT_ID/n8n-custom:latest .

# Push to GCR
docker push gcr.io/YOUR_PROJECT_ID/n8n-custom:latest

# 5. Deploy to Cloud Run
gcloud run deploy n8n \
  --image gcr.io/YOUR_PROJECT_ID/n8n-custom:latest \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --port 5678 \
  --memory 2Gi \
  --cpu 2 \
  --min-instances 1 \
  --max-instances 10 \
  --set-env-vars "NODE_ENV=production,N8N_HOST=0.0.0.0,N8N_PORT=5678"
```

Here are some commands we've used so far in the deployment:

```
export PROJECT_ID=apps-n8n
export REGION=us-east1
export SERVICE_URL="https://n8n-325859163001.us-east1.run.app/"

gcloud run services update n8n     --region=$REGION     --update-env-vars="N8N_HOST=$(echo $SERVICE_URL | sed 's/https:\/\///'),WEBHOOK_URL=$SERVICE_URL,N8N_EDITOR_BASE_URL=$SERVICE_URL"


```