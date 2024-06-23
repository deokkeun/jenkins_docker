<img width="1440" alt="스크린샷 2024-06-23 오후 10 37 29" src="https://github.com/deokkeun/jenkins_docker/assets/84825191/bacdbd78-2a8a-49c5-aa74-6de7c3d2d3e0">
<br>

<h1>Downloading and running Jenkins in Docker</h1>
<a href="https://www.jenkins.io/doc/book/installing/docker/#downloading-and-running-jenkins-in-docker">Docker에서 Jenkins 다운로드 및 실행</a>
<br>
<h2>On macOS and Linux</h2>
다음 명령을 사용하여 Docker에서 브리지 네트워크를 만듭니다

```
  docker network create jenkins
```
<br>
Jenkins 노드 내에서 Docker 명령을 실행하려면 docker:dind다음 docker run명령을 사용하여 Docker 이미지를 다운로드하고 실행하십시오.

```
docker run \
  --name jenkins-docker \
  --rm \
  --detach \
  --privileged \
  --network jenkins \
  --network-alias docker \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-docker-certs:/certs/client \
  --volume jenkins-data:/var/jenkins_home \
  --publish 2376:2376 \
  docker:dind \
  --storage-driver overlay2
```
<br>
위 명령 조각을 복사하여 붙여넣는 데 문제가 있는 경우 아래 주석이 없는 버전을 사용하세요.

```
docker run --name jenkins-docker --rm --detach \
  --privileged --network jenkins --network-alias docker \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-docker-certs:/certs/client \
  --volume jenkins-data:/var/jenkins_home \
  --publish 2376:2376 \
  docker:dind --storage-driver overlay2
```
<br>

다음 두 단계를 실행하여 공식 Jenkins Docker 이미지를 사용자 지정합니다.

  - a. 다음 콘텐츠로 Dockerfile을 만듭니다.

```
FROM jenkins/jenkins:2.452.2-jdk17
USER root
RUN apt-get update && apt-get install -y lsb-release
RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
  https://download.docker.com/linux/debian/gpg
RUN echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
  https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
RUN apt-get update && apt-get install -y docker-ce-cli
USER jenkins
RUN jenkins-plugin-cli --plugins "blueocean docker-workflow"
```

  - b. 이 Dockerfile에서 새 Docker 이미지를 빌드하고 이미지에 "myjenkins-blueocean:2.452.2-1"과 같은 의미 있는 이름을 할당합니다.

```
docker build -t myjenkins-blueocean:2.452.2-1 .
```

다음 명령을 사용하여 Docker에서 자체 myjenkins-blueocean:2.452.2-1이미지를 컨테이너로 실행합니다

```
docker run \
  --name jenkins-blueocean \
  --restart=on-failure \
  --detach \
  --network jenkins \
  --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client \
  --env DOCKER_TLS_VERIFY=1 \
  --publish 8080:8080 \
  --publish 50000:50000 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  myjenkins-blueocean:2.452.2-1
```

alpine/socat
<a href="https://stackoverflow.com/questions/47709208/how-to-find-docker-host-uri-to-be-used-in-jenkins-docker-plugin">alpine/socat</a>

```
docker run -d --restart=always -p 127.0.0.1:2376:2375 --network jenkins -v /var/run/docker.sock:/var/run/docker.sock alpine/socat tcp-listen:2375,fork,reuseaddr unix-connect:/var/run/docker.sock
docker inspect <container_id> | grep IPAddress
```

Docker Host URI

```
tcp://IPAddress:2375
```


