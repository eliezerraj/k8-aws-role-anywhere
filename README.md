# Aws Role Anywhere

   Run a workload to access aws services via aws role anywhere

## create a root-ca.cnf
    see $/assets/certs/root-ca.cnf

## create key

    openssl genrsa -out root-ca.key 4096

## validate key

    openssl rsa -check -noout -in root-ca.key

## create certificate

    openssl req -x509 -new -nodes \
        -key root-ca.key \
        -sha256 -days "3650" \
        -extensions ca_ext \
        -config root-ca.cnf \
        -out root-ca.crt

## Create the certificate signing request
    openssl req -new \
      -config root-ca.cnf \
      -out root-ca.csr \
      -keyout ./root-ca.key

## Sign our request
    openssl ca -selfsign \
      -config root-ca.cnf \
      -in root-ca.csr \
      -out root-ca.crt \
      -extensions ca_ext

## see cert

    openssl x509 -in root-ca.crt -text -noout

# Client CA

## create client-01.cnf
    see $/assets/certs/client-01.cnf

## create key
    openssl genrsa -out client-01.key 4096

## validate key
    openssl rsa -check -noout -in client-01.key

## create a csr
    openssl req -new \
    -key client-01.key \
    -subj "/C=BR/L=SP/ST=SAO PAULO/O=Dock Tech/OU=Client-01/CN=on-premisse-local-machine/" \
    -out client-01.csr

## inspect
    openssl req -text -noout -verify -in client-01.csr

# Root

## signed client certificate for 365 days.
    openssl x509 -req \
        -in client-01.csr \
        -CA root-ca.crt -CAkey root-ca.key \
        -CAcreateserial \
        -out client-01.crt \
        -days 365 -sha256 \
        -extensions client_ext \
        -extfile root-ca.cnf

or

    openssl ca \
      -config root-ca.cnf \
      -in client-01.csr \
      -out client-01.crt \
      -extensions client_ext

# Client

## install aws_signing_helper (https://github.com/aws/rolesanywhere-credential-helper/releases)

    curl https://rolesanywhere.amazonaws.com/releases/1.7.1/X86_64/Linux/Amzn2023/aws_signing_helper --output aws_signing_helper
    chmod 755 aws_signing_helper

## test
    ./aws_signing_helper credential-process \
      --certificate ../client-01.crt --private-key ../client-01.key \
      --role-arn arn:aws:iam::9099999993:role/eliezer-role-anywhere-arch-root-ca-01 \
      --trust-anchor-arn arn:aws:rolesanywhere:us-east-2:9099999993:trust-anchor/6317b3ca-5218-40a3-8120-0241512ba484 \
      --profile-arn arn:aws:rolesanywhere:us-east-2:9099999993:profile/246eb787-621f-4b20-ba3e-f9fba176dc52 
