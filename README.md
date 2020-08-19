# CSW Concourse Pipeline

For access requirements go to [Prerequisites](#Prerequisites).

If you want to build a Docker image, follow the [Docker](#Docker) section.

For information about how the Concourse jobs are set up, go to [Concourse](#Concourse)

If you want to write or amend a Concourse pipeline, follow the [Implementing Concourse pipeline](#Implementing-Concourse-pipeline) section.

For information about the IAM role and policies that have been implemented go to [IAM role](#IAM-role).

### Prerequisites
Access to [DockerHub](https://hub.docker.com/) - Ask the Tooling team to share credentials for the gdscyber account via LastPass.

Access to [csw-concourse GitHub repo](https://github.com/alphagov/csw-concourse)

Access to [Concourse](https://cd.gds-reliability.engineering/teams/cybersecurity-tools/) - Alphagov GitHub single sign-on

Read general info on [deploying into PaaS using Concourse](https://cyber-security-team-manual.cloudapps.digital/How-To-Build-and-Deploy-an-App-to-PaaS-using-Concourse.html#set-pipeline-on-concourse-and-run)

### Docker
Note: The Docker images that are defined in [dockerfiles](dockerfiles) are now automatically built
using the `cyber-security-concourse-base-image` pipeline on the RE Concourse. There is no need to
manually build and deploy these images.

The CSW Concourse implementation has two docker images in the Docker Hub:

  - **cyber-chalice**

    *The docker image is a ubuntu 18.04 installation with Python 3.6, Python virtual environment, nodejs and terraform installed.*

    Dockerfile for chalice:
      * https://github.com/alphagov/csw-concourse/blob/master/dockerfiles/chalice/Dockerfile

    The command below builds a docker image, which is tagged as latest version, the -t flag also tags it as version 2.0:

    ```
    docker build --no-cache -t gdscyber/cyber-chalice -t gdscyber/cyber-chalice:2.0 .
    ```

    Check DockerHub for the latest version tag.

    Afterwards to push to DockerHub run this command:

    ```
    docker push gdscyber/cyber-chalice:2.0
    ```

    You also have to separately run either of the following for AWS ECS to pick up that this is the latest version:

    ```
    docker push gdscyber/cyber-chalice
    docker push gdscyber/cyber-chalice:latest
    ```

  - **csw-concourse-worker**

    *The docker image is a ubuntu 18.04 installation with Geckodriver and Firefox and Python environment inherited from cyber-chalice image*

    Dockerfile for csw-concourse-worker:
     * https://github.com/alphagov/csw-concourse/blob/master/dockerfiles/csw/Dockerfile


    The command below builds a docker image for the csw-concourse-worker and tags it simultaneously as latest and version 1.3.1:

    ```
    docker build --no-cache -t gdscyber/csw-concourse-worker -t gdscyber/csw-concourse-worker:1.3.1 .
    ``` 

    Then to push to DockerHub:

    ```
    docker push gdscyber/csw-concourse-worker:1.3.1
    ```
    and
    ```
    docker push gdscyber/csw-concourse-worker:latest 
    or
    docker push gdscyber/csw-concourse-worker
    ```

   #### Scripts
   
   Except for getsshkey and aws-assume-role, the below are all [expect scripts](https://en.wikipedia.org/wiki/Expect).

  * [aws-assume-role](https://github.com/alphagov/csw-concourse/tree/master/dockerfiles/csw/bin/aws-assume-role)
    * Allows the concourse worker to assume a given role to deploy to the staging / production accounts

  * [buildcsw](https://github.com/alphagov/csw-concourse/tree/master/dockerfiles/csw/bin/buildcsw)
    * Accepts arguments and answers to yes/no prompts from the gulp environment.build task. Currently not in use as we are using existing environments.

  * [deploycsw](https://github.com/alphagov/csw-concourse/tree/master/dockerfiles/csw/bin/deploycsw)
    * Accepts the "yes/no" prompt from the ```usr/local/bin/gulp environment.deploy``` command

  * [getsshkey](https://github.com/alphagov/csw-concourse/tree/master/dockerfiles/csw/bin/getsshkey)
    * Loads ssh keys from SSM into ```/root/.ssh/${ENVIRONMENT}```  & gives it the correct permissions in the docker container

  * [loadcsw](https://github.com/alphagov/csw-concourse/tree/master/dockerfiles/csw/bin/loadcsw)
    * This script takes the arguments provided and loads them into the gulp script prompts.


### Concourse
https://cd.gds-reliability.engineering/teams/cybersecurity-tools/pipelines/csw


  * Deploy job (incl. load)
    * Loads prefix and account ID from settings.json for the specified environment.
    * Does installation of dependencies and runs ```aws-assume-role```, ```getsshkey```, ```loadcsw``` and ```deploycsw``` scripts as above.
    * Notifies Slack #cst-tooling-team channel on successful exit or failure.

    * uat-deploy job
      * task ```csw-unit-test``` - installs and activates virtual environment, installs wheel and requirements-dev.txt. Then runs unittest.
      * task ```csw-uat-deploy``` - loads 'uat' environment and runs deploy job.
      * task ```csw-e2e-test``` - runs e2e test on the 'uat' environment as ```e2etest.user```

    * prod-deploy job
      * only runs if ```uat-deploy``` passed
      * task ```csw-prod-deploy``` - runs deploy job in the 'prod' environment.
      
      
  * Build job - currently NOT IN USE
    * The build job starts by installing latest Linux updates and generating RSA key pair.
    * It installs and activates virtual environment and package dependencies and runs unittest.
    * It then assumes the ```concourse``` role and saves private and public keys to SSM parameter store.
    * It runs the ```buildcsw``` script and notifies Slack if successful or failing.    
    
    
  * Destroy job - currently NOT IN USE
    * After assuming AWS Concourse role and fetching SSH keys this job activates virtual environment and installs dependencies.
    * It runs ```loadcsw``` script then ```gulp environment.cleanup``` for the specified environment and notifies Slack on success/failure.
    
CSW Concourse pipeline uses variables which are uploaded into Concourse hosting account SSM parameter store.
We have hidden from the pipeline code some variables like cyber staging and production account numbers, concourse role name and slack webhook url.

    The cyber staging account ID:
    ```aws ssm put-parameter \
    --name "/cd/concourse/pipelines/cybersecurity-tools/cyber-staging" \
    --value "103495720024" \
    --type SecureString \
    --key-id "9044a24d-2e69-4058-ba72-52c43dec4979" \
    --overwrite \
    --region eu-west-2
    ```

    Concourse role:
    ```aws ssm put-parameter \
    --name "/cd/concourse/pipelines/cybersecurity-tools/cd-role" \
    --value "cd-cybersecurity-tools-concourse-worker" \
    --type SecureString \
    --key-id "9044a24d-2e69-4058-ba72-52c43dec4979" \
    --overwrite \
    --region eu-west-2
    ```
    
    Slack webhook configuration:

    ```
    aws ssm put-parameter --cli-input-json '{"Type": "SecureString", "KeyId": "9044a24d-2e6
    9-4058-ba72-52c43dec4979", "Name": "/cd/concourse/pipelines/cybersecurity-tools/slack-webhook-cyber", "Value"
    : "https://hooks.slack.com/services/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" }'  --overwrite --region eu
    -west-2
    ```

### Implementing Concourse pipeline

The csw Concourse pipeline yaml file can be found at ```csw-concourse/pipelines/csw-pipeline.yml```

Once you've made changes to the yaml file you login to concourse and set your target, in this case "cd":

    fly login --target cd -c https://cd.gds-reliability.engineering -n cybersecurity-tools    

The following command will create a new pipeline or amend the existing pipeline script: 

    fly -t cd sp -c csw-pipeline.yml -p csw


### IAM role
  We restricted the policy assigned to the Concourse IAM role. It is likely that this will need to change in the future as new functionalities are added.

  The policy lives in ```csw-infra/tools/concourse/main.tf``` file. To make changes to it you need to:
  * git clone the csw-infra repo
  * cd into tools/concourse
  * get ```apply.tfvars``` and ```backend.tfvars``` files from csw-configuration/tools/concourse/ and copy them to the csw-infra/tools/concourse folder where ```main.tf``` file is.
  * make the needed changes in ```main.tf``` file.
  * amend bucket name in ```apply.tfvars``` and ```backend.tfvars``` depending on whether you are implementing the changes in staging or production.
  * then run Terraform as follows **for both staging and production accounts**:
  
    ```
    aws-vault exec <prod or staging account> -- terraform init -backend-config=backend.tfvars -reconfigure
    ```
    ```
    aws-vault exec <prod or staging account> -- terraform plan -var-file apply.tfvars
    ```
    ```
    aws-vault exec <prod or staging account> -- terraform apply -var-file apply.tfvars
    ```
    !!! Staging and production should always have the same policy.
