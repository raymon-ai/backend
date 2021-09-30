# Adding a TLS endpoint
If you cannot use a managed TLS endpoint (like a load balancer) you can deploy one yourself and forward requests to the Raymon services. You can then use any certificate you want, including a self-signed one or, if possible, one using [Let's Encrypt](https://letsencrypt.org/) and [Certbot](https://certbot.eff.org/). We'll outline how to use your self-signed certificate below, but this guide should be extensible for use with certbot, as outlined [here](https://dzone.com/articles/nginx-and-https-with-lets-encrypt-certbot-and-cron).

## 1. Generating a self-signed certificate

Run the following commands to (1) become your own Certificate Authority (CA) and (2) use that CA to create a certificate for your domain. 
**Important:** ALL machines that will access need to import and TRUST the root certificate (`myCA.pem`), or warnings will be generated in web browsers. To do so (on mac), double click the file, and it will be imported in KeyChain. Then, right click on the certificate in KeyChain -> get info -> Trust -> Always Trust.

```bash
# Taken from:  https://stackoverflow.com/questions/7580508/getting-chrome-to-accept-self-signed-localhost-certificate

######################
# Become a Certificate Authority
######################

# Generate private key
openssl genrsa -des3 -out myCA.key 2048
# Generate root certificate
openssl req -x509 -new -nodes -key myCA.key -sha256 -days 825 -out myCA.pem

######################
# Create CA-signed certs
######################

NAME=mydomain.com # Use the URL where Raymon will be accessible.
# Generate a private key
openssl genrsa -out $NAME.key 2048
# Create a certificate-signing request
openssl req -new -key $NAME.key -out $NAME.csr
# Create a config file for the extensions
>$NAME.ext cat <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = $NAME # Be sure to include the domain name here because Common Name is not so commonly honoured by itself
EOF
# Create the signed certificate
openssl x509 -req -in $NAME.csr -CA myCA.pem -CAkey myCA.key -CAcreateserial \
-out $NAME.crt -days 825 -sha256 -extfile $NAME.ext
```

## 2. Deploy an extra ingress / load balancer container

Copy your generated keys into the extra/keys folder and name them `cert.crt` and `cert.key`. Then, add the following to the deployment yaml file and deploy it.

```yaml
  ingress:
    image: nginx:1.21.0
    volumes:
      - ${PWD}/extra/nginx.conf:/etc/nginx/nginx.conf
      - ${PWD}/extra/keys:/run/secrets/keys

    ports:
      - 443:443
      - 5001:5001

```
The API should now be accessible over HTTPS, on port 5001.

Problems? Please [contact us](mailto:hello@raymon.ai)!