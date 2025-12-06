# Overview
Special certificate for azurite that doesn't use a multipart hostname (since azurite doesn't support it)
```bash
[alt_names]
DNS.1 = localhost
DNS.2 = *.localhost
DNS.3 = azurite
DNS.4 = azurite-docker-local
IP.1 = 127.0.0.1
IP.2 = ::1
```


```bash
# 1. Generate the certificate with Subject Alternative Names (SANs):
openssl req -x509 -nodes -days 9999 -newkey rsa:2048 -keyout azurite-local.key -out azurite-local.crt -config azurite-local.conf -extensions v3_req

# 2. Create PKCS12 keystore for Kafka brokers:
openssl pkcs12 -export -out azurite-local.p12 -inkey azurite-local.key -in azurite-local.crt -name azurite-local -passout pass:password

# 3. Convert PKCS12 to JKS keystore (Java KeyStore format):
keytool -importkeystore -srckeystore azurite-local.p12 -srcstoretype PKCS12 -srcstorepass password \
  -destkeystore azurite-local.keystore.jks -deststoretype JKS -deststorepass password -destkeypass password

# 4. Create truststore with the CA certificate:
keytool -import -trustcacerts -alias azurite-local-ca -file azurite-local.crt -keystore azurite-local.truststore.jks \
  -storepass password -noprompt

# 5. Install to system trust store (Linux):
sudo cp ./azurite-local.crt /usr/local/share/ca-certificates/azurite-local.crt
sudo update-ca-certificates

# 6. Install to Chrome's certificate database (NSS):
sudo apt update && sudo apt install -y libnss3-tools
certutil -d sql:$HOME/.pki/nssdb -D -n "azurite-local" 2>/dev/null || true  # Remove old if exists
certutil -d sql:$HOME/.pki/nssdb -A -t "CP,CP," -n "azurite-local" -i ./azurite-local.crt

# 7. Convert to a pfx for use with .net and kestrel
openssl pkcs12 -export -out azurite-local.pfx -inkey azurite-local.key -in azurite-local.crt -passout pass:password

# 8. Verify it was added:
certutil -d sql:$HOME/.pki/nssdb -L | grep azurite-local

# 9. Make sure everything's readable by all users:
chmod 644 ./azurite-local.key
chmod 644 ./azurite-local.p12
chmod 644 ./azurite-local.pfx
chmod 644 ./azurite-local.keystore.jks
chmod 644 ./azurite-local.truststore.jks


# Create credential files
touch azurite-local-key-creds && chmod 644 azurite-local-key-creds && echo -n "password" > azurite-local-key-creds
touch azurite-local-keystore-creds && chmod 644 azurite-local-keystore-creds && echo -n "password" > azurite-local-keystore-creds
touch azurite-local-truststore-creds && chmod 644 azurite-local-truststore-creds && echo -n "password" > azurite-local-truststore-creds
```