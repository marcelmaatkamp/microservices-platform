Platform

Start proxy, opens a SOCKS5 proxy on port :1080
```docker-compose up -d proxy```

nodepki:
```
$ docker-compose up -d nodepki
$ docker-compose run nodepki ash -c "cd /root/nodepki && node /root/nodepki/nodepkictl.js useradd --username thomas --password test"
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


Start nexus:
```
$ docker-compose run nodepki ash -c 'cd /certs/nexus && openssl pkcs12 -export -in signed.crt -inkey domain.key -chain -CAfile chained.pem  -name "nexus" -out nexus.p12'
$ docker-compose run nexus ash -c 'cd /certs/nexus && keytool -importkeystore -deststorepass password -destkeystore /nexus-data/keystore.jks -srckeystore nexus.p12 -srcstoretype PKCS12'
```

Start gitlab:
```
docker-compose up -d gitlab
```

gitlab-dind:
 root.crt -> /etc/ssl/certs/ca-certificates.crt
 $ update-ca-certificates
 $ docker pull alpine
