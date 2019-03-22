# Readme
## Codeship CI/CD pipeline 
 > all verbiage are lowercase for codeship to interate within there environment
### codeship-services.yaml 
> Google App Engine

 ```
googleclouddeployment:
  image: codeship/google-cloud-deployment
  encrypted_env_file: ae.env.encrypted
  add_docker: true
  working_dir: /google-deploy.sh
  volumes:
    - ./:/deploy

#codeship_gcr_dockercfg:
  #image: codeship/gcr-dockercfg-generator
  #add_docker: true
  #encrypted_env_file: ae.env.encrypted
``` 
### codeship-steps.yaml
```
- name: google-cloud-deployment
  service: googleclouddeployment
  command: bash /deploy/google-deploy.sh
```

### google-deploy.sh 
```#!/bin/bash
set -e
# Authenticate with the Google Services
codeship_google authenticate
echo "Setting default project $GOOGLE_PROJECT_ID"
gcloud config set project $GOOGLE_PROJECT_ID



# switch to the directory containing your app.yml (or similar) configuration file
# note that your repository is mounted as a volume to the /deploy directory
cd /deploy/

# deploy the application
gcloud app deploy  app.yaml  --quiet
```
### Codeship Encrypting Google Service Account Credentials  
Login into Codeship and select project. Upper right hand corner select `Project Setting` and click `General`
Copy AES key and save to file within root directory of application as `codeship.aes`
Create a Google Service Account in Google IAM and name it codeship-deploy or of the like. Click create json key and download 
and save in different folder other than application being used. 
### Create your_env_file.encrypted
make sure Docker for desktop is running and logged in. Create a folder inside of application root directory dot file .your_env_file
and copy and paste into dot file
`GOOGLE_AUTH_JSON=...
GOOGLE_AUTH_EMAIL=...
GOOGLE_PROJECT_ID=...
`
In terminal, enter into the folder and file of service account json key and copy the json.key file name. 
In command line type in `tr '\n' ' ' < name of json.key`. This will remove of lines in json.key and reformate the json.key. 
Copy and paste this in `GOOGLE_AUTH_JSON=` below `GOOGLE_AUTH_EMAIL=` email of service account `codeship-deploy@blank.blank.com`
`GOOGLE_PROJECT_ID=` will be the Project_ID displayed on home page of Google Cloud Console. 

With file constructed with Google service account creditionals, in the commandline type in 
`jet encrypt your_env_file your_env_file.encrypted` after file is encrypted, delete dot file of plaintext Google account creditionals
The your_env_file.encrypted will be using in codeship-services.yaml under `encrypted_env_file`

