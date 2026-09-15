# 🔐 Nginx HTTPS/TLS Lab

Laboratório prático desenvolvido para estudar a implementação de **HTTPS utilizando TLS**, criação e gerenciamento de certificados digitais com **OpenSSL**, configuração de um servidor **Nginx** e funcionamento de uma **Certificate Authority (CA)**.

O projeto foi desenvolvido em ambiente Linux com foco em compreender, na prática, conceitos de **criptografia, PKI, certificados digitais, TLS e segurança de aplicações web**.

---

## 🎯 Objetivos

- Configurar um servidor web Nginx.
- Habilitar HTTPS.
- Criar uma Certificate Authority (CA) própria.
- Gerar chaves privadas utilizando OpenSSL.
- Criar um Certificate Signing Request (CSR).
- Assinar um certificado de servidor utilizando uma CA.
- Configurar o Nginx para utilizar o certificado.
- Testar o handshake TLS utilizando OpenSSL.
- Validar a cadeia de confiança.
- Analisar certificados através do navegador.
- Compreender a diferença entre certificados self-signed e certificados assinados por uma CA.

---

# 🏗️ Arquitetura

A estrutura do projeto é composta por:

```text
┌─────────────────────┐
│       Client        │
│                     │
│  Browser / OpenSSL  │
└──────────┬──────────┘
           │
           │ HTTPS
           │ TCP/443
           ▼
┌─────────────────────┐
│       Nginx         │
│                     │
│     HTTPS :443      │
└──────────┬──────────┘
           │
           │ server.crt
           │
           ▼
┌─────────────────────┐
│    MyLab Root CA    │
│                     │
│     ca.crt          │
└─────────────────────┘
