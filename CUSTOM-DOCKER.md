# Custom Docker Image

This project contains our custom Docker build for n8n.

What motivated this fork is in order to use EXIF tools in n8n, we need to install a community n8n node which depends on Perl being installed on the underlying system. In order to achieve this, the Docker image for n8n needs to be customized to add this manually. This fork will allow us to keep up to date with the latest n8n dockerfiles as a base, since they are updated relatively frequently.

https://www.npmjs.com/package/n8n-nodes-exif-data

In order to deploy a new version of n8n, we must take the docker files and build them manually in gcloud. We then use gcloud to replace our Cloud Run service with the new artifact.

Be sure to upload all related files to the gcloud terminal. These files can be found in `docker/images`