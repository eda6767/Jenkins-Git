# Jenkins-Git & DSL & Pipelines

### Install Gitlab using Docker Compose 

<p align="center">
<img width="550" alt="Zrzut ekranu 2023-07-24 o 21 48 52" src="https://github.com/eda6767/Jenkins-Git/assets/102791467/d00da96d-37a7-4caf-955c-c7fb8c774d4b">
</p>

<br/> 

<sub/>

Based on documentation on the website : https://docs.gitlab.com/ee/install/docker.html#install-gitlab-using-docker-compose  we are gonna extend docker-compose.yml
<br/> 

file:

```
  git:
    container_name: git-server
    image: 'gitlab/gitlab-ce:latest'
    hostname: 'gitlab.example.com'
    ports:
      - '8090:80'
    volumes:
      - '$GITLAB_HOME/config:/etc/gitlab'
      - '$GITLAB_HOME/logs:/var/log/gitlab'
      - '$GITLAB_HOME/data:/var/opt/gitlab'
    networks:
      - net
```

Then, we can create new service:

```
export GITLAB_HOME=$HOME/gitlab
docker-compose up -d
docker ps
```

Next step is to create group and projekt on gitlab https://gitlab.com/ 
<p align="center">
<img width="400" alt="Zrzut ekranu 2023-07-25 o 14 33 36" src="https://github.com/eda6767/Jenkins-Git/assets/102791467/c3e156d1-b963-4e14-9581-f58ae4271e8d">
</p>



Let's go into our virtual machine. If command git doesn't work, install git:

```
git --version
git clone https://github.com/jenkins-docs/simple-java-maven-app.git
ls simple-java-maven-app
```

Next, clone the new project in git:

```
git clone https://gitlab.com/jenkins4982930/maven.git
cd maven
cp -r ../simple-java-maven-app/* .
```


```
git add .
git status
git commit -m "Adds maven files"
git push -uf origin main
```


### Integrate Git server to your maven job

Add new credentials: 

<p align="center">
<img width="650" alt="Zrzut ekranu 2023-07-30 o 16 28 13" src="https://github.com/eda6767/Jenkins-Git/assets/102791467/3f5f8ff6-4af7-4143-af23-d6ee30ab0e18">
</p>


Then, we can create job: called maven-job and add git repository as source for the code.

<p align="center">
<img width="650" alt="Zrzut ekranu 2023-07-30 o 17 03 10" src="https://github.com/eda6767/Jenkins-Git/assets/102791467/e1365bc7-7d54-45fa-8836-b29fb707172c">
</p>


### Git Hooks 

Git hooks works like a trigger - whenever a developer pushes to master branch - we could trigger a script.

```
cd /var/opt/gitlab/git-data/repositories/jenkins/maven.git/
mkdir custom_hooks
cd custom_hooks
vi post-receive
```

```
if ! [ -t 0 ]; then
  read -a ref
fi
IFS='/' read -ra REF << "${ref[2]}"
branch="${REF[2]}"

if [ $branch == "master"]; then
crumb=$(curl -u "jenkins:1234" -s http://jenkins:8080/crumbIssuer/api/xml?path=concat(//crumbRequestField,":",//crumb)')
curl -u "jenkins:1234" -H $crumb -X POST http://jenkins:8080/job/maven-job/build?delay=0sec

  if [ $? -eq 0]; then
    echo " *** ok"
  else
    echo " *** Error"
  fi
 fi
```

```
chmod +x post-receive
chown git:git custom_hooks/ -R
```


```
cd maven
cat git/.config
```
<p align="center">
  
<img width="529" alt="Zrzut ekranu 2023-07-30 o 19 02 10" src="https://github.com/eda6767/Jenkins-Git/assets/102791467/8052e3a0-f51e-4b97-9c9f-f6bb22bdd488">

</p>


Now, we can modify some file in maven folder:

<p align="center">
<img width="550" alt="Zrzut ekranu 2023-07-30 o 19 06 32" src="https://github.com/eda6767/Jenkins-Git/assets/102791467/b20b6b6d-9d3f-4304-a459-41763d5d53d2">
</p>

```
git status
git add src/
git commit -m "Test git hook trigger"
git push -uf origin main
```



### Jenkins & DSL

DSL lets you create jobs using code instead of Jenkins' console. To use this we nedd to install plugins.

<p align="center">
<img width="650" alt="Zrzut ekranu 2023-07-30 o 19 13 45" src="https://github.com/eda6767/Jenkins-Git/assets/102791467/3d43cf84-30f8-4c9d-96eb-84b2d0038b03">
</p>


Next you have to create a seed job, and in jenkins container create dsl folder with a first job file:

```
mkdir dsl
nano job.j2
cat job.j2
```


<img width="435" alt="Zrzut ekranu 2023-07-30 o 19 23 49" src="https://github.com/eda6767/Jenkins-Git/assets/102791467/9168ca10-733b-4847-aeee-dc24d1381898">

<p align="center">
<img width="650" alt="Zrzut ekranu 2023-07-30 o 19 26 47" src="https://github.com/eda6767/Jenkins-Git/assets/102791467/dca776f3-6eeb-48c2-8e9b-de0ca2e335df">
</p>


We can also create jobs with description and parameters:

```
job('job_dsl_example') {

description('This is my first job')

parameters {
    stringParam('Planet', defaultValue='world', description='This is the world')
    booleanParam('FLAG', 'true')
    choiceParam('OPTION', ['option1 (default)', 'option2', 'option3'])
}
}
```


#### SCM, triggers, steps, mailer

SCM point on the repository, where our code is located.

```
job('job_dsl_example') {

description('This is my first job')

parameters {
    stringParam('Planet', defaultValue='world', description='This is the world')
    booleanParam('FLAG', 'true')
    choiceParam('OPTION', ['option1 (default)', 'option2', 'option3']) }


scm {
    git('https://github.com/jenkins-docs/simple-java-maben-app', 'master')
}
triggers {
      cron('H 5 * * 7')
}

steps {
    shell("""
          echo 'Hello world'
          echo 'Running script'
          /tmp/script.sh
          """)
}

publishers {
        mailer('me@example.com', true, true)
}
}
```


#### Versioning your DSL code using GIT

<br/> 

Lets create a new project called dsl-job

<br/> 


<p align="center">
<img width="650" alt="Zrzut ekranu 2023-07-30 o 20 18 16" src="https://github.com/eda6767/Jenkins-Git/assets/102791467/b6134864-feda-4ce1-a24f-b325720fa2d0">
</p>

<br/> 

Next, let's clone git repository on our jenkins container.

```
cd dsl-job
vi jobs
git status
git add job
git commit -m "DSL jobs"
git push -uf origin main
```

Next, we need to create Git Hook:

```
docker exec -ti git-server bash
cd /var/opt/gitlab/git-data/repositories/jenkins/dsl-job.git/
mkdir custom_hooks
cd custom_hooks
vi post_receive
```

```
if ! [ -t 0 ]; then
  read -a ref
fi
IFS='/' read -ra REF << "${ref[2]}"
branch="${REF[2]}"

if [ $branch == "master"]; then
crumb=$(curl -u "jenkins:1234" -s http://jenkins:8080/crumbIssuer/api/xml?path=concat(//crumbRequestField,":",//crumb)')
curl -u "jenkins:1234" -H $crumb -X POST http://jenkins:8080/job/dsl-job/build?delay=0sec

  if [ $? -eq 0]; then
    echo " *** ok"
  else
    echo " *** Error"
  fi
 fi
```

```
chown git:git custom_hooks/ -R
```


After creating hooks, we can change code in jobs file - push changes to repository and dsl-job on Jenkins should work.


## Pipeline

```
mkdir pipeline
cd pipeline
nano first_pipeline
```


```
pipeline {
  agent any 
    stages {
      stage('Deploy') {
        steps {
          retry(3) {
            sh 'echo Hello'
          }
          timeout(time: 3, unit: 'SECONDS') {
                sh 'sleep 5' }
              }
            }
        }
  }

```

<p align="center">
<img width="650" alt="Zrzut ekranu 2023-08-3 o 22 10 17" src="https://github.com/eda6767/Jenkins-Git/assets/102791467/e9f386da-d6ea-4202-a88f-e2a7c7655aab">
</p>


<p align="center">
<img width="577" alt="Zrzut ekranu 2023-08-3 o 22 12 56" src="https://github.com/eda6767/Jenkins-Git/assets/102791467/6cb29ac4-dd31-4418-bc87-b759001b6564">

</p>


