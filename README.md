WghaQZlbCKhGnCiQJDJN

##--openssl-legacy-provider
--max-old-space-size=2048



bin/logstash -e 'input { stdin { } } output {elasticsearch {hosts => ["http://localhost:9200"] index => "logstash-%{+YYYY.MM.dd}"}}'

kibanasystempassword
logstashsystempassword: Ri9aK2NJQpzlo8PBSoQ_
vaultpassword

/usr/share/elasticsearch/bin/elasticsearch-certutil cert --silent --pem --ca-cert /etc/elasticsearch/certs/ca/ca.crt --ca-key /etc/elasticsearch/certs/ca/ca.key --name http --out /etc/elasticsearch/certs/http/http-cert.zip