# Jenkins-Git

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
    image: 'gitlab/gitlab-ee:latest'
    hostname: 'gitlab.example.com'
    ports:
      - '8090:80'
    volumes:
      - '$GITLAB_HOME/config:/etc/gitlab'
      - '$GITLAB_HOME/logs:/var/log/gitlab'
      - '$GITLAB_HOME/data:/var/opt/gitlab'
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

