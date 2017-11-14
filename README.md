Platform

proxy
`docker-compose up -d proxy`

opens a SOCKS5 proxy on port :1080

nodepki:
`docker-compose up -d nodepki` and `docker-compose run nodepki ash -c "cd /root/nodepki && node /root/nodepki/nodepkictl.js useradd --username thomas --password test"`

goto nodepki:5000
import root
trust root

Request new certificate
 - nexus
 - gitlab
 - gitlab-runner

cacertfile -> chained.pem
certfile -> signed.crt
keyfile -> domain.key

gitlab-dind:
 root.crt -> /etc/ssl/certs/ca-certificates.crt
 $ update-ca-certificates
 $ docker pull alpine
