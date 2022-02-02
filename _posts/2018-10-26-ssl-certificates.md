---
layout: post
title: "SSL Certificates"
author: pratyush
categories: [ Security, Tutorial ]
#tags: [red, yellow]
image: assets/images/ssl-certificates.jpg
description: "How to generate SSL Certificates"
featured: true
hidden: true
#rating: 4.5
---

Steps to generate SSL Certificates for Java based servers. Learn how to convert various certificate types.

### What is SSL
SSL stands for Secure Sockets Layer. It is a cryptographic protocol designed to provide security over a computer network widely used to secure websites. 
#### There are 3 versions of SSL
1. SSL v1.0
2. SSL v2.0
3. SSL v3.0

SSL is now depricated and the successor to SSL is TLS (Transport Layer Security). 
#### There are 4 versions of TLS.
1. TLS v1.0
2. TLS v1.1
3. TLS v1.2
4. TLS v1.3

TLS v1.3 is considered to be most secure.

### How to create a Java Keystore
A Java KeyStore (JKS) is a repository of security certificates which contains public key certificates plus their corresponding private keys used for TLS encryption. Java Development Kit provides a utility called "keytool" to manage the certificates and keys present in the Keystore.

#### STEP-1 Create a 2048 bit java keystore
keytool -genkey -alias tomcat -keyalg RSA -keystore keystore.ks -keysize 2048

#### STEP-2 Create a CSR using the keystore
keytool -certreq -alias tomcat -keyalg RSA -file certificate.csr -keystore keystore.ks -sigalg SHA1withRSA

#### STEP-3 Create a single p7b file from server certificare and intermediate certificate.
openssl crl2pkcs7 -nocrl -certfile server.crt -certfile intermediate.crt -out outfile.p7b

#### STEP-4 IMPORTING PURCHASED CERTIFICATE (in p7b format) INTO KEYSTORE
keytool -import -alias tomcat -trustcacerts -file outfile.p7b -keystore keystore.ks

NOTE: A p7b certificate file contains Certificates & Chain Certificates. 
This file is used if there are a collection of certificates to be imported into the keystore instead of one single certificate file.

### Conversion of x509 certificates
Binary to Text
openssl x509 -inform der -in certificate.der -out certificate.crt

Text to Binary
openssl x509 -outform der -in certificate.crt -out certificate.cer
