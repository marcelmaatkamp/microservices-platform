# Microservices Development Platform

Containes:

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

```
$ # add user 'test'
$ docker-compose run nodepki ash -c "cd /root/nodepki && node /root/nodepki/nodepkictl.js useradd --username thomas --password test"
$ docker-compose up -d nodepki
```

goto http://nodepki:5000
 * import root
 * trust root

Request new certificate for servers
 - nexus
 - gitlab
 - gitlab-runner

/certs/hostname/
 * cacertfile -> chained.pem
 * certfile -> signed.crt
 * keyfile -> domain.key

# Nexus

Start nexus:
```
$ docker-compose run nodepki ash -c 'cd /certs/nexus && openssl pkcs12 -export -in signed.crt -inkey domain.key -chain -CAfile chained.pem  -name "nexus" -out nexus.p12'
$ docker-compose run nexus ash -c 'cd /certs/nexus && keytool -importkeystore -deststorepass password -destkeystore /nexus-data/keystore.jks -srckeystore nexus.p12 -srcstoretype PKCS12'
$ docker-compose up -d nexus
```
goto https://nexus:8443

# Gitlab

Start gitlab:
```
docker-compose up -d gitlab
```

gitlab-dind:
 root.crt -> /etc/ssl/certs/ca-certificates.crt
 $ update-ca-certificates
 $ docker pull alpine

Gitlab-runner
