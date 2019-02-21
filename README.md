# Codeship CI/CD pipeline
``` steps:
# Build the Docker image.
- name: 'gcr.io/cloud-builders/docker'
  entrypoint: 'bash'
  args: [ '-c','docker build -t us.gcr.io/$PROJECT_ID/authentication:latest  --build-arg DB_PASS=$$DATABASE_PASS .']
  secretEnv: ['DATABASE_PASS']

# Push it to GCR.
- name: 'gcr.io/cloud-builders/docker'
  args: ['push', 'us.gcr.io/$PROJECT_ID/authentication']    

secrets:             
- kmsKeyName: projects/elite-ceremony-228616/locations/global/keyRings/cloudBuild-test/cryptoKeys/buildcryptest
  secretEnv:
      DATABASE_PASS: CiQAnwhTc/1vS/JMsey2WRv1ojKu5a+qgg2NXf7eFegpmjoTzmUSNQDoG6N0lMHT2wyEhVqmWTZa/JugBCPvZ+Ccn9+Kdd19cCSjEV6W2vbcS6NDgTo/0x8n61rb



      #'build',  '-t', 'us.gcr.io/$PROJECT_ID/authentication', '--build-arg', 'DATABASE_PASS=$$DATABASE_PASS',  '.' ```
