# Jenkins-Git

### Install Gitlab using Docker Compose 


<img width="550" alt="Zrzut ekranu 2023-07-24 o 21 48 52" src="https://github.com/eda6767/Jenkins-Git/assets/102791467/d00da96d-37a7-4caf-955c-c7fb8c774d4b">


<br/> 

<sub/>

<br/> 

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
docker-compose up -d
docker ps
```

Next step is to create group and projekt on gitlab https://gitlab.com/ 

<img width="500" alt="Zrzut ekranu 2023-07-25 o 14 33 36" src="https://github.com/eda6767/Jenkins-Git/assets/102791467/c3e156d1-b963-4e14-9581-f58ae4271e8d">

