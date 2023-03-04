# A continuous integration and continuous deployment to Google App Engine with Ruby on Rails


 I have the time to gain knowledge in deploying a Ruby on Rails app to Google Cloud Platform. Creating a CI/CD pipeline with CloudBees Codeship to Google App Engine (GAE). GAE is a very simplistic way of deploying services that. More information on GAE can be found here. [GAE Docs](https://cloud.google.com/appengine/)
GAE is very easy to do a blue/green canary upgrade and many other features. There's quite a bit of working gears per say in this deployment but once done, any changes to the code will be simplistically. Only down side I have to this is that the cloudbuild takes forever to build and if you had to change one line of code, you'll be waiting for at least two hours.
 
To start we select GAE app engine flex. I ended up building my own Ruby image through Docker and pushed it to GCR (Google Container Registry). The current app engine flex was not compatible with our version of RoR. Other, services included GCP Sql (postgres) and GCP memorystore (redis). 
I also added sidekiq to create an instance to connect with cloud sql.(detailed below)
```yml
cloudsql sidekiq
beta_settings:
  cloud_sql_instances: GCP_Project_ID:us-central1:Name_CloudSql

network:
  name: default

skip_files:
  - .env
  - .bundle
  - .byebug_history
  - .vscode/
  - .idea/
  - storage/
  - vendor/
  - log/
  - tmp/   
```
I use CloudBees Codeship services for a secure CI/CD pipeline and GCP KMS to encrypt credentials. 

With deploying to GAE you will need to add Dockerfile which is then built on cloudbuild. Let's get into the trenches and build something cool. 
(examples only) 

```yaml
FROM gcr.io/GCP_Project_Name/Name_of_Image

EXPOSE 8080
ARG ruby_version=2.5.3
ARG bundler_version=1.17.1

# Ruby
ENV DEFAULT_RUBY_VERSION=${ruby_version}

# Install Ruby, set default Ruby version, and install Bundler
RUN rbenv global 2.5.3

# Workdir
ADD . /workspace/
WORKDIR /workspace/

# CloudSQL
VOLUME /cloudsql

# Set up environment variables used in production
ENV RACK_ENV=production \
RAILS_ENV=production \
RAILS_SERVE_STATIC_FILES=true

# Run bundle
RUN gem install bundler 
RUN rbenv exec gem install bundler
RUN bundle install --deployment --without="development test"

#ARG for authentication docker image 
ARG DB_PASS
ARG SECRET_B_KEY 


#env-variable for app-beta.yaml and worker-beta.yaml
ENV SECRET_KEY_BASE=${SECRET_B_KEY}
ENV RAILS_ENV="production"
ENV RACK_ENV="production"
ENV SERVICE_NAME="authentication"
ENV REDIS_HOST="10.0.0.3"
ENV REDIS_PORT=6379
ENV DATABASE_USER="postgres"
ENV DATABASE_PASS=${DB_PASS}
ENV DATABASE_NAME="postgres"
ENV DATABASE_HOST="/cloudsql/GCP-project:us-central1:nameofdb"
 ENV RAILS_LOG_TO_STDOUT: enabled
 ENV RAILS_SERVE_STATIC_FILES: enabled
 ENV LANG: en_US.UTF-8

# Entrypoint
CMD bundle exec puma -p 8080 -e production && bundle exec sidekiq -t 120 -C config/sidekiq.yml 
```
 
 As you can see the _ENV-Var_ are not what you expect. This is because I have encrypted them until they reach GAE where GCP KMS decrypts them automatically (More on this later) 

 Creating own docker ruby image and pushing to Google Container Registery
  ```dockerfile
 # Use the base image provided by Google
FROM gcr.io/gcp-runtimes/ruby/ubuntu16

# Ruby 2.4.5
ARG ruby_version=2.4.5
ARG bundler_version=1.17.1
RUN rbenv install -s ${ruby_version} \
    && rbenv global ${ruby_version} \
    && rbenv rehash \
    && (bundle version > /dev/null 2>&1 \
        || gem install bundler --version ${bundler_version}) \
    && rbenv rehash && gem cleanup

# Ruby 2.5.3
ARG ruby_version=2.5.3
ARG bundler_version=1.17.1
RUN rbenv install -s ${ruby_version} \
    && rbenv global ${ruby_version} \
    && rbenv rehash \
    && (bundle version > /dev/null 2>&1 \
        || gem install bundler --version ${bundler_version}) \
    && rbenv rehash && gem cleanup

# Ruby 2.6.1
ARG ruby_version=2.6.1
ARG bundler_version=2.0.1
RUN rbenv install -s ${ruby_version} \
    && rbenv global ${ruby_version} \
    && rbenv rehash \
    && (bundle version > /dev/null 2>&1 \
        || gem install bundler --version ${bundler_version}) \
    && rbenv rehash && gem cleanup

# ENV
ENV DEFAULT_RUBY_VERSION=${ruby_version}

```
I also wanted to point out cloudsql is proxied already within sidekiq
add the configuration within -config -database.yml and in the  Dockerfile ENV DATABASE_HOST=.
 
## Now moving on to CloudBees CODESHIP 

CloudBees Codeship has a very simple and secure way for CI/CD pipeline. In order to do this you can get a free service or paid service. In the example below is by a pro subscription. [CloudBees Codeship GCP]({% link 'https://documentation.codeship.com/pro/continuous-deployment/google-cloud/' %})
[my examples] ({% link https://github.com/Arconapalus/GCP-example-solutions %})

*We need to authenticate CloudBees Codeship to GCP* 
this will be done by going to projects and then projects setting and getting an aes key so that jet cli (which is run on docker) to encrypt the env_var of GCP project credentials. 


*Create service and steps for CloudBees Codeship to execute to GCP* 
Codeship GCP service should look something like this
```yml
googleclouddeployment:
  image: codeship/google-cloud-deployment
  encrypted_env_file: env_e.encrypted
  add_docker: true
  working_dir: /google-deploy.sh
  volumes:
    - ./:/deploy
```
though depending if you are deploying on more than one GCP service such as GKE, then the service will be more detailed. 

Codeship GCP steps should look something like this
```yml
- name: google-cloud-deployment
  service: googleclouddeployment
  command: bash /deploy/google-deploy.sh
```

and finally Codeship GCP google-deploy.sh 
```binbash

#!/bin/bash
set -e
# Authenticate with the Google Services
codeship_google authenticate
echo "Setting default project $GOOGLE_PROJECT_ID"
gcloud config set project *Project_name

# switch to the directory containing your app.yml (or similar) configuration file
# note that your repository is mounted as a volume to the /deploy directory
cd /deploy/
# deploy the application
gcloud builds submit --config cloudbuild.yaml --verbosity debug

```

## CloudBuild.yaml which is doing to build the docker and push to GAE 

```yml 
steps:
# Build the Docker image.
- name: 'gcr.io/cloud-builders/docker'
  entrypoint: 'bash'
  args: [ '-c','docker build -t gcr.io/$PROJECT_ID/_service_:latest --build-arg DB_PASS=$$DATABASE_PASS --build-arg SECRET_B_KEY=$$SECRET_KEY
.']
  secretEnv: [ 'DATABASE_PASS', 'SECRET_KEY']

# Build Docker authentication-worker image 
- name: 'gcr.io/cloud-builders/docker'
  entrypoint: 'bash'
 gcr.io/$PROJECT_ID/appengine/_Service_:latest --build-arg DB_PASS=$$DATABASE_PASS --build-arg SECRET_B_KEY=$$SECRET_KEY --build-arg  .']
 
  args: [ '-c','docker build -t gcr.io/$PROJECT_ID/_Service_:latest --build-arg DB_PASS=$$DATABASE_PASS --build-arg SECRET_B_KEY=$$SECRET_KEY --build-arg  .']
  secretEnv: [ 'DATABASE_PASS', 'SECRET_KEY']

  # Push it to GCR.
- name: 'gcr.io/cloud-builders/docker'
  args: ['push', 'gcr.io/$PROJECT_ID/_Service_']   

- name: 'gcr.io/cloud-builders/docker'
  args: ['push', 'gcr.io/$PROJECT_ID/_Service_']  

# build to google app engine 
- name: 'gcr.io/cloud-builders/gcloud'
  args: ['app', 'deploy', 'app.yaml', '--image-url=gcr.io/$PROJECT_ID/_Service_']


- name: 'gcr.io/cloud-builders/gcloud'
  args: ['app', 'deploy', 'worker.yaml', '--image-url=gcr.io/$PROJECT_ID/_Service_']
timeout: 1600s


#KMS secrets 
secrets:             
- kmsKeyName:  projects/_Project-name_/locations/global/keyRings/KR_NAME/cryptoKeys/CK_name
  secretEnv:
      DATABASE_PASS: KMS encyption base_64
      SECRET_KEY: KMS encyption base_64
     
```
A lot is going on here but I want to point out that the KMS is decrypted as soon as it hits GCP network, so simple for automation. 

The way I automated this was creating a script to convert this from plain text to encrypted. 

```script
#!/bin/bash


gcloud kms encrypt --plaintext-file=filename.txt --ciphertext-file=filename.enc.txt --location=global --keyring=KR_Name --key=CK_Name

base64 filename.enc.txt -i 0 > filename.enc64.txt

```
(this is for macos bash terminal, the base64 will be different for other os. Also, gcloud sdk was installed) 



Here is the file structure 
