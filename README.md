# Microservices Development Platform

Development platform - Contains a PKI infrastructure (NodePKI), Code/Source Repository for code/wiki and isssues (Gitlab), CI to build libraries and/or docker containers (Gitlab-CI and Gitlab-runners), Library/Docker Repository to store (Nexus)

Contains:

 * Socks5 Proxy
 * NodePKI
 * Sonatype Nexus
 * Gitlab 
	- Gitlab server	- Git repository
	- Gitlab CI	- Automatic build
	- Gitlab runner - Automatic builder
	- Gitlab Dind	- Gitlab Docker Container builder

# Proxy 

Start proxy, opens a SOCKS5 proxy on port :1080

```
docker-compose up -d proxy
```

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

Start nexus:
```
$ docker-compose run nodepki ash -c 'cd /certs/nexus && openssl pkcs12 -export -in signed.crt -inkey domain.key -chain -CAfile chained.pem  -name "nexus" -out nexus.p12'
$ docker-compose run nexus ash -c 'cd /certs/nexus && keytool -importkeystore -deststorepass password -destkeystore /nexus-data/keystore.jks -srckeystore nexus.p12 -srcstoretype PKCS12'
$ docker-compose up -d nexus
```
goto https://nexus:8443

<img src="https://raw.githubusercontent.com/marcelmaatkamp/microservices-platform/master/.gitlab/nexus_green_ssl.png" width="250" >

# Gitlab

## Start gitlab:

| - | - | 
| root.pem | /etc/gitlab/trusted-certs/ | 
| intermediate.pem | /etc/gitlab/trusted-certs/ |
| domain.key | /etc/gitlab/ssl/gitlab.key |
| chained.pem | /etc/gitlab/ssl/gitlab.crt |

where chained.pem = 
 * signed.crt +
 * root.crt +
 * intermediate.crt

```
docker-compose up -d gitlab
```

[[ SCREENSHOT GREEN SSL CONNECTION GITLAB]]

<img src="" width="250" >

## gitlab-dind:

```
$ docker-compose up -d gitlab-dind
```

Add root.crt -> /etc/ssl/certs/ca-certificates.crt

```
$ docker-compose exec gitlab-dind update-ca-certificates
```

Test with a pull of alpine:

```
$ docker-compose exec gitlab-dind docker pull alpine
```

## Gitlab-runner

[[ SCREENSHOT ADMIN GENERIC RUNNER ID IN GITLAB ]]

<img src="" width="250" >

Create the certificate file at: 
 - `/etc/gitlab-runner/certs/gitlab.crt` on *nix systems when gitlab-runner is executed as root
 - `~/.gitlab-runner/certs/gitlab.crt` *nix systems when gitlab-runner is executed as non-root
 - `./certs/gitlab.crt` on other systems

```
$ docker-compose exec gitlab-runner update-ca-certificates
$ docker-compose run gitlab-runner register -n \
 --url https://gitlab:8443 \
 --registration-token <<GENERIC_TOKEN_ID>> \
 --executor docker \
 --docker-image docker:17.06.0-ce \
 --docker-volumes /var/run/docker.sock:/var/run/docker.sock
$ docker-compose up -d gitlab-runner
```

## Validate connection

```
$ docker-compose exec gitlab-runner openssl s_client -connect gitlab:8443
```

