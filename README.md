# Build and Deploy a Modern YouTube Clone Application in React JS with Material UI 5

![YouTube](https://i.ibb.co/4R5RkmW/Thumbnail-5.png)

## Some Steps for the process of gitlab CI/CD pipeline
--> Clone the repository into your github.

==> We need api key for this youtube project. we need to go into this website for creating the api-key. URL is https://rapidapi.com/

--> Signup into account, and login into account
--> search for Youtube v3 and copy the APY-Key. Example: X-RapidAPI-Key': '9b0ab58964msh154f05eb8c1f38dp1ac6eajsn8cfd4e5f83fe',

--> Go to GitLab account and create a new project.   
--> Import this project and go into this project.

--> Login into aws account and create an EC2 instance with "t2.medium" and 10GB of storage for docker and SonarQube.
--> Install docker and run sonar container on that EC2 Instance. the installation commands will be:

    - sudo apt-get update
    - sudo apt-get install docker.io -y
    - sudo usermod -aG docker $USER   #my case is ubuntu
    - newgrp docker
    - sudo chmod 777 /var/run/docker.sock

--> Run this command on your EC2 instance to create a SonarQube container:

    - docker run -d --name sonar -p 9000:9000 sonarqube:lts-community

    - docker ps or docker container ls 

--> Now copy the IP address of the ec2 instance with port 9000

-->  Go to gitlab account and create a new file name as: .gitlab-ci.yml
write the code here
    
    stages:
        - npm

    install dependecy:
        stage: npm    
        image:
            name: node:16
        script:
            - npm install

Note: this will be running on "Shared Runners".

--> To run this job, on left side click on Builds --> click on pipelines --> here the build already started.

--> go to sonarqube --> click on manual --> give name as "youtube" --> select your branch name like main/master --> click on GitLab CI

--> Create a "sonar-project.properties" file in your Project and paste the following code:
    sonar.projectKey=youtube-app
    sonar.qualitygate.wait=true

--> Add environment variables

--> Define the SonarQube Token environment variable.

--> In GitLab, go to Settings > CI/CD > Variables to add the following variable and make sure it is available for your project:
--> In the Key field, enter "SONAR_TOKEN"
--> In the Value field, enter an existing token, or a newly generated one: "Generate a token"
--> Uncheck the "Protect Variable" checkbox.
--> Check the "Mask Variable" checkbox.

--> Define the SonarQube URL environment variable.

--> Still in Settings > CI/CD > Variables add a new variable and make sure it is available for your project:
--> In the Key field, enter "SONAR_HOST_URL" 
--> In the Value field, enter "http://3.110.218.0:9000" 
--> Uncheck the "Protect Variable" checkbox.
--> Leave the "Mask Variable" checkbox unchecked.

stages:
    - npm
    - sonar

install dependency:
    stage: npm
    image: 
        name: node:16
    script: 
        - npm install 

sonarqube-check:
    stage: sonar
    image: 
        name: sonarsource/sonar-scanner-cli:latest
        entrypoint: [""]
    variables:
        SONAR_USER_HOME: "${CI_PROJECT_DIR}/.sonar"  # Defines the location of the analysis task cache
        GIT_DEPTH: "0"  # Tells git to fetch all the branches of the project, required by the analysis task
    cache:
        key: "${CI_JOB_NAME}"
        paths:
            - .sonar/cache
    script: 
        - sonar-scanner
    allow_failure: true
    only:
        - master

--> Go to Docker hub --> click on settings --> click on security --> generate a Token here.

--> Go to GitLab Settings --> Click on CI/CD --> Click on Variables --> add variables here like 

    DOCKER_USERNAME this is key --> provide user name in value --> please make uncheck protect variable and ckeck masked variable
    DOCKER_PASSWORD this is key --> provide token in value --> please make uncheck protect variable and ckeck masked variable

stages:
    - npm
    - sonar
    - trivy
    - docker
    - image scan

install dependency:
    stage: npm
    image: 
        name: node:16
    script: 
        - npm install 

sonarqube-check:
    stage: sonar
    image: 
        name: sonarsource/sonar-scanner-cli:latest
        entrypoint: [""]
    variables:
        SONAR_USER_HOME: "${CI_PROJECT_DIR}/.sonar"  # Defines the location of the analysis task cache
        GIT_DEPTH: "0"  # Tells git to fetch all the branches of the project, required by the analysis task
    cache:
        key: "${CI_JOB_NAME}"
        paths:
            - .sonar/cache
    script: 
        - sonar-scanner
    allow_failure: true
    only:
        - master

Trivy scanner:
    stage: trivy
    image:
        name: aquasec/trivy:latest
        entrypoint: [""]
    script:
        - trivy fs .

Docker Build and Push:
    stage: docker
    image:
        name: docker:latest
    services:
        - docker:dind
    script:
        - docker build --build-arg REACT_APP_RAPID_API_KEY=9b0ab58964msh154f05eb8c1f38dp1ac6eajsn8cfd4e5f83fe -t youtube . 
        - docker tag youtube $DOCKER_USERNAME/youtube:latest
        - docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
        - docker push $DOCKER_USERNAME/youtube:latest

Image Scanner:
    stage: image scan
    image:
        name: aquasec/trivy:latest
        entrypoint: [""]
    script:
        - trivy image $DOCKER_USERNAME/youtube:latest


--> Go to docker hub and check image is pushed successfully or not
 
--> Connect EC2 Instance with Git bash for to create a runner 
--> Goto Gitlab settings --> click on CICD --> Click on Runners Expand --> go to project runners --> click on 3 dots and --> click on show runners installation. --> select Linux and arm64 .

--> copy the commands --> goto git bash --> 
    - sudo vim gitlab-runner-installation
    - Paste all the commands here
    - sudo chmod +x gitlab-runner-installation
    - ./gitlab-runner-installation
    - sudo gitlab-runner start

--> Now run the below command or your command to register the runner
--> Update the token is enough
    - sudo gitlab-runner register --url https://gitlab.com/ --registration-token <token>

--> Provide Enter for GitLab.com

-->  For token we already added with token, so click on Enter again

--> Description as your wish like youtube

--> Tags also and you can use multiple tags by providing a comma after each tag like youtube, Ravi

--> Maintenance note is just optional

--> For executors use Shell

after completion of this we need to execute
    - sudo gitlab-runner start
    - sudo gitlab-runner run

==> go  back to project runner here we can see the runner in "Green" colour it is  successfully completed

--> click on pencil mark and check the run untagged jobs

go back to .gitlab-ci.yml file

stages:
    - npm
    - sonar
    - trivy
    - docker
    - image scan
    - deploy

install dependency:
    stage: npm
    image: 
        name: node:16
    script: 
        - npm install 

sonarqube-check:
    stage: sonar
    image: 
        name: sonarsource/sonar-scanner-cli:latest
        entrypoint: [""]
    variables:
        SONAR_USER_HOME: "${CI_PROJECT_DIR}/.sonar"  # Defines the location of the analysis task cache
        GIT_DEPTH: "0"  # Tells git to fetch all the branches of the project, required by the analysis task
    cache:
        key: "${CI_JOB_NAME}"
        paths:
            - .sonar/cache
    script: 
        - sonar-scanner
    allow_failure: true
    only:
        - master

Trivy scanner:
    stage: trivy
    image:
        name: aquasec/trivy:latest
        entrypoint: [""]
    script:
        - trivy fs .

Docker Build and Push:
    stage: docker
    image:
        name: docker:latest
    services:
        - docker:dind
    script:
        - docker build --build-arg REACT_APP_RAPID_API_KEY=9b0ab58964msh154f05eb8c1f38dp1ac6eajsn8cfd4e5f83fe -t youtube . 
        - docker tag youtube $DOCKER_USERNAME/youtube:latest
        - docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
        - docker push $DOCKER_USERNAME/youtube:latest

Image Scanner:
    stage: image scan
    image:
        name: aquasec/trivy:latest
        entrypoint: [""]
    script:
        - trivy image $DOCKER_USERNAME/youtube:latest

Deploy the container:
    stage: deploy
    tags: 
        - youtube
    script:
        docker run --name youtube -d -p 3000:3000 $DOCKER_USERNAME/youtube:latest

--> It will automatically run the pipeline

--> here we face some error for deploying the container. so we need  to rectify that.
--> goto git bash
    - sudo -i
    - sudo vi /home/gitlab-runner/.bash_logout

==> uncomment like this 

#if [ "$SHLVL" = 1 ]; then
#    [ -x /usr/bin/clear_console ] && /usr/bin/clear_console -q
#fi

    - sudo gitlab-runner restart
    - exit #from root
    - sudo gitlab-runner start
    - sudo gitlab-runner run

--> Now go to GitLab --> Build --> Pipelines --> Click on Run Pipeline

==> Now it is  successfully deployed --> goto git bash 
    - docker ps

--> copy public ip and paste it in browser:3000

--> here we can access the youtube application

