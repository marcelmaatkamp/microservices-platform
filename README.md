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

# nodepki

Add user thomas/test:

```
$ docker-compose run nodepki ash -c "cd /root/nodepki && node /root/nodepki/nodepkictl.js useradd --username thomas --password test"
$ docker-compose up -d nodepki
```

goto http://nodepki:5000 and 

 * import root and
 * trust root in your keystore

[[ SCREENSHOT APPLE KEYSTORE ]]

Request new certificate for servers

 - nexus
 - gitlab
 - gitlab-runner

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

# Gitlab

## Start gitlab:

```
docker-compose up -d gitlab
```

## gitlab-dind:

 root.crt -> /etc/ssl/certs/ca-certificates.crt
 $ update-ca-certificates
 $ docker pull alpine

## Gitlab-runner
