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
git push origin master
```


