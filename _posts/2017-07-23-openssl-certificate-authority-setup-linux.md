OpenSSL Root Certificate / Chrome missing_subjectAltName Error

https://jamielinux.com/docs/openssl-certificate-authority/introduction.html

install openssl
Save config files


Create root key


Create root cert
`openssl req -config openssl.cnf -key private/ca.key.pem -new -x509 -days 7300 -sha256 -extensions v3_ca -out certs/ca.cert.pem`

Create intermediate key


Create intermediate CSR
`openssl req -config intermediate/openssl.cnf -new -sha256 -key intermediate/private/intermediate.key.pem -out intermediate/csr/intermediate.csr.pem`

Create intermediate cert
`openssl ca -config openssl.cnf -extensions v3_intermediate_ca -days 3650 -notext -md sha256 -in intermediate/csr/intermediate.csr.pem -out intermediate/certs/intermediate.cert.pem`

Create root/intermediate cert bundle
`cat intermediate/certs/intermediate.cert.pem certs/ca.cert.pem  > intermediate/certs/ca-chain.cert.pem`

Create site CSR
`openssl req -config intermediate/openssl.cnf -key intermediate/private/brains.lan.key.pem -new -sha256 -out intermediate/csr/brains.lan.csr.pem`

Create site cert
`openssl ca -config intermediate/openssl.cnf -extensions server_cert -days 3650 -notext -md sha256 -in intermediate/csr/brains.lan.csr.pem -out intermediate/certs/brains.lan.cert.pem`

Verify site cert
`openssl x509 -noout -text -in intermediate/certs/brains.lan.cert.pem `