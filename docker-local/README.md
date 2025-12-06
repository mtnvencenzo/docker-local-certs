# Overview
Common local dev cert supporting these adns aliases
```bash
DNS.1 = localhost
DNS.2 = *.localhost
DNS.3 = docker.local
DNS.4 = *.docker.local
DNS.5 = host.docker.internal
IP.1 = 127.0.0.1
IP.2 = ::1
```

## Regenerate ssl
This contains fully functional dev certs but if you would like to trust other host names you can regenerate them. First, update the docker-local.conf file with any additions or changes you would like to make, then run the below script using the root of the repo as your working directory.


```bash
# 1. Generate the certificate with Subject Alternative Names (SANs):
openssl req -x509 -nodes -days 9999 -newkey rsa:2048 -keyout docker-local.key -out docker-local.crt -config docker-local.conf -extensions v3_req

# 2. Create PKCS12 keystore for Kafka brokers:
openssl pkcs12 -export -out docker-local.p12 -inkey docker-local.key -in docker-local.crt -name docker-local -passout pass:password

# 3. Convert PKCS12 to JKS keystore (Java KeyStore format):
keytool -importkeystore -srckeystore docker-local.p12 -srcstoretype PKCS12 -srcstorepass password \
  -destkeystore docker-local.keystore.jks -deststoretype JKS -deststorepass password -destkeypass password

# 4. Create truststore with the CA certificate:
keytool -import -trustcacerts -alias docker-local-ca -file docker-local.crt -keystore docker-local.truststore.jks \
  -storepass password -noprompt

# 5. Install to system trust store (Linux):
sudo cp ./docker-local.crt /usr/local/share/ca-certificates/docker-local.crt
sudo update-ca-certificates

# 6. Install to Chrome's certificate database (NSS):
sudo apt update && sudo apt install -y libnss3-tools
certutil -d sql:$HOME/.pki/nssdb -D -n "docker-local" 2>/dev/null || true  # Remove old if exists
certutil -d sql:$HOME/.pki/nssdb -A -t "CP,CP," -n "docker-local" -i ./docker-local.crt

# 7. Convert to a pfx for use with .net and kestrel
openssl pkcs12 -export -out docker-local.pfx -inkey docker-local.key -in docker-local.crt -passout pass:password

# 8. Verify it was added:
certutil -d sql:$HOME/.pki/nssdb -L | grep docker-local

# 9. Make sure everything's readable by all users:
chmod 644 ./docker-local.crt
chmod 644 ./docker-local.key
chmod 644 ./docker-local.p12
chmod 644 ./docker-local.pfx
chmod 644 ./docker-local.keystore.jks
chmod 644 ./docker-local.truststore.jks

# Create credential files
touch docker-local-key-creds && chmod 644 docker-local-key-creds && echo -n "password" > docker-local-key-creds
touch docker-local-keystore-creds && chmod 644 docker-local-keystore-creds && echo -n "password" > docker-local-keystore-creds
touch docker-local-truststore-creds && chmod 644 docker-local-truststore-creds && echo -n "password" > docker-local-truststore-creds
```