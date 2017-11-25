# Microservices Development Platform

Development platform - Contains a PKI infrastructure (NodePKI), Code/Source Repository for code/wiki and isssues (Gitlab), CI to build libraries and/or docker containers (Gitlab-CI and Gitlab-runners), Library/Docker Repository to store (Nexus)

Contains:

 * Socks5 Proxy		- Socks5 proxy
 * NodePKI 		- PKI infrastructure
 * Sonatype Nexus	- Repository manager for docker, npm and java
 * Gitlab:
	- Gitlab server	- Git repository
	- Gitlab CI	- Automatic build
	- Gitlab runner - Automatic builder
	- Gitlab Dind	- Gitlab Docker Container builder
	
# Installation

```
$ git pull && \
  cd microservices-platform &&\
  git submodule update --recursive --remote
```

# Proxy 

Start proxy, opens a SOCKS5 proxy on port :1080

```
docker-compose up -d proxy
```

Install [Proxy Switch Omega](https://chrome.google.com/webstore/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif) for Google Chrome for example and connect the proxy software to the machine where docker runs on port socks5://docker_machine:1080

# NodePKI

Add user thomas/test:

```
$ docker-compose run nodepki ash -c \
  "cd /root/nodepki && node /root/nodepki/nodepkictl.js useradd --username thomas --password test"
$ docker-compose up -d nodepki
```

goto http://nodepki:5000 and login:

<img src="https://raw.githubusercontent.com/marcelmaatkamp/microservices-platform/master/.gitlab/nodepki_user.png" width="250" >

<img src="https://raw.githubusercontent.com/marcelmaatkamp/microservices-platform/master/.gitlab/nodepki_loggedin.png" width="250" >

## Trust root certificate 

 * import root and
 * trust root in your keystore

<img src="https://raw.githubusercontent.com/marcelmaatkamp/microservices-platform/master/.gitlab/nodepki_root.png" width="250" >

[[ SCREENSHOT OSX KEYSTORE ]]
<img src="" width="250" >

## Generate server certificates 

Request new certificate for servers

<img src="https://raw.githubusercontent.com/marcelmaatkamp/microservices-platform/master/.gitlab/nodepki_new_cert.png" width="250" >

 - nexus
 - gitlab
 - gitlab-runner

<img src="https://raw.githubusercontent.com/marcelmaatkamp/microservices-platform/master/.gitlab/nodepki_all_certs.png" width="250" >

This generates in /certs/hostname/ like /certs/gitlab the following files:

 * cacertfile -> /certs/gitlab/chained.pem
 * certfile -> /certs/gitlab/signed.crt
 * keyfile -> /certs/gitlab/domain.key

Now we can start configuring and running the individual servers like nexus and gitlab

# Nexus

Nexus needs a java keystore for its SSL certificates. Before we can start Nexus we need to create one where the keys from NodePKI can be found in `/certs/nexus` and are imported in the keystore `nexus.p12`. Do not worry about all those certificates and where to find them, these scripts take care of everyting which is the main motivation for writing this repository.

## Generate Java Keystore

```
$ docker-compose run nodepki ash -c 'cd /certs/nexus && openssl pkcs12 -export -in signed.crt -inkey domain.key -chain -CAfile chained.pem  -name "nexus" -out nexus.p12'
$ docker-compose run nexus ash -c 'cd /certs/nexus && keytool -importkeystore -deststorepass password -destkeystore /nexus-data/keystore.jks -srckeystore nexus.p12 -srcstoretype PKCS12'
```

## Start nexus

```
$ docker-compose up -d nexus
```
goto https://nexus:8443

<img src="https://raw.githubusercontent.com/marcelmaatkamp/microservices-platform/master/.gitlab/nexus_green_ssl.png" width="250" >

# Gitlab

## Start gitlab:

file | location
-- | --
root.pem | /etc/gitlab/trusted-certs/ 
intermediate.pem | /etc/gitlab/trusted-certs/
domain.key | /etc/gitlab/ssl/gitlab.key 
chained.pem | /etc/gitlab/ssl/gitlab.crt 

where chained.pem = 
 * signed.crt +
 * root.crt +
 * intermediate.crt

```
docker-compose up -d gitlab
```

[[ SCREENSHOT GREEN SSL CONNECTION GITLAB]]

<img src="" width="250" >

## Gitlab-runner

[[ SCREENSHOT ADMIN GENERIC RUNNER ID IN GITLAB ]]

<img src="" width="250" >

Create the certificate file at: 
 - `/etc/gitlab-runner/certs/gitlab.crt` on *nix systems when gitlab-runner is executed as root
 - `~/.gitlab-runner/certs/gitlab.crt` *nix systems when gitlab-runner is executed as non-root
 - `./certs/gitlab.crt` on other systems

```
$ docker-compose up -d gitlab-runner
docker-compose exec gitlab-runner gitlab-runner register -n \
  --url https://greup2.pirod.nl:8443 \
  --registration-token N7yPskZskU46wzyQoEVc \
  --executor docker  \
  --description "My Docker Runner" \ 
  --docker-image "docker:latest" \
  --docker-volumes /var/run/docker.sock:/var/run/docker.sock \
  --docker-volumes /root/.docker/config.json:/root/.docker/config.json
```

## Example `.gitlab-ci.yml`
```
image:
  name: docker/compose:1.17.1

stages:
 - build

# before_script:
# - docker login -u gitlab-ci-token -p $CI_BUILD_TOKEN gitlab.myhostname.com

build:
 stage: build
 variables:
   CI_DEBUG_TRACE: "false"
 script:
  - docker-compose build

 only:
  - master
```

## Validate connection

```
$ docker-compose exec gitlab-runner openssl s_client -connect gitlab:8443
```

