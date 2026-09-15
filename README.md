# Nginx HTTPS/TLS Lab

Laboratório prático desenvolvido para estudar a implementação de **HTTPS utilizando TLS**, criação e gerenciamento de certificados digitais com **OpenSSL**, configuração de um servidor **Nginx** e funcionamento de uma **Certificate Authority (CA)**.

O projeto foi desenvolvido em ambiente Linux com foco em compreender, na prática, conceitos de **criptografia, PKI, certificados digitais, TLS e segurança de aplicações web**.

---

## Objetivos

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

# Arquitetura

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
```
## PKI

O projeto utiliza uma estrutura simplificada de Public Key Infrastructure (PKI).

A relação entre os certificados é:
```
MyLab Root CA
      │
      │ assina
      ▼
 server.crt
      │
      ▼
    Nginx
      │
      ▼
 HTTPS / TLS
```
A CA é responsável por emitir e assinar o certificado utilizado pelo servidor.
Durante o projeto foram utilizados os seguintes arquivos:
```
ca.key
ca.crt

server.key
server.csr
server.crt
Descrição
Arquivo	Descrição
ca.key	Chave privada da Certificate Authority
ca.crt	Certificado da Certificate Authority
server.key	Chave privada do servidor
server.csr	Certificate Signing Request
server.crt	Certificado digital do servidor
```
### 1. Criando a Certificate Authority

A primeira etapa consiste em criar uma CA própria.

Criando a chave privada
```
openssl genrsa -out ca.key 4096

A chave privada da CA é o componente mais sensível da estrutura de PKI.
```
Criando o certificado da CA
```
openssl req -x509 -new -nodes \
-key ca.key \
-sha256 \
-days 3650 \
-out ca.crt \
-subj "/C=BR/ST=Sao Paulo/O=MyLab/CN=MyLab Root CA"
```
A CA é autoassinada, portanto é esperado que:
```
Subject = MyLab Root CA
Issuer  = MyLab Root CA
```
Podemos verificar:
```
openssl x509 -in ca.crt -noout -subject -issuer
```
### 2. Criando a chave privada do servidor

A chave privada utilizada pelo Nginx foi criada com RSA 2048 bits:
```
openssl genrsa -out server.key 2048

Essa chave é utilizada pelo servidor durante o processo de TLS.

Assim como a chave da CA, ela deve permanecer privada.
```
### 3. Criando o Certificate Signing Request

O CSR contém informações que serão utilizadas para gerar o certificado do servidor.
```
openssl req -new
-key server.key
-out server.csr
-subj "/C=BR/ST=Sao Paulo/O=MyLab/CN=192.168.1.104"
```
O CSR pode ser visualizado com:
```
openssl req -in server.csr -noout -text
```
O fluxo é:
```
server.key
    │
    ▼
server.csr
    │
    │ enviado para a CA
    ▼
MyLab Root CA
```
### 4. Assinando o certificado do servidor

O certificado é assinado utilizando a CA:
```
openssl x509 -req 
-in server.csr 
-CA ca.crt 
-CAkey ca.key 
-CAcreateserial 
-out server.crt 
-days 365 
-sha256
```
Agora temos:
```
MyLab Root CA
      │
      │ assina
      ▼
 server.crt
```
Podemos verificar:
```
openssl x509 -in server.crt -noout -subject -issuer
```
O resultado deve demonstrar que:
```
Subject = (IP da sua máquina)
Issuer  = MyLab Root CA (Nome criado por mim)
```
Isso confirma que o certificado do servidor foi emitido por mim mesmo.

### 5. Configurando o Nginx

Os certificados foram armazenados em:
```
/etc/nginx/ssl/
```
Estrutura:
```
/etc/nginx/ssl/
├── server.crt
└── server.key
```
Configuração básica:
```
server {
    listen 443 ssl;

    ssl_certificate /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;

    location / {
        root /var/www/html;
        index index.html;
    }
}
```
O Nginx passa a aceitar conexões HTTPS através da porta:
```
443/TCP
```
### 6. Validando a configuração do Nginx

Antes de reiniciar o serviço, vamos testar se ele está funcionando corretamente e depois restarta-lo.
```
sudo nginx -t
Resultado esperado:
|syntax is ok      |
|test is successful|
Depois:
sudo systemctl restart nginx
Verifique:
sudo systemctl status nginx
```
### 7. Testando HTTPS

O servidor pode ser acessado utilizando:
```
http://localhost
```
Também é possível testar através do curl:
```
curl -k http://localhost

O parâmetro -k desabilita a validação do certificado.
```
Isso é útil porque a CA criada neste laboratório ainda não é uma autoridade confiável no sistema operacional.

### 8. Analisando TLS com OpenSSL

Uma das principais ferramentas utilizadas no projeto é:
```
openssl s_client
```
Para estabelecer uma conexão TLS:
```
openssl s_client -connect localhost:443
```
O comando permite analisar informações como:
```
versão do TLS;
cipher suite;
certificado apresentado pelo servidor;
cadeia de certificados;
chave pública;
algoritmo de assinatura;
resultado da validação do certificado.
TLS 1.3
```
Durante o teste foi possível observar:
```
New, TLSv1.3

Cipher is TLS_AES_256_GCM_SHA384

Isso demonstra que a comunicação foi estabelecida utilizando TLS 1.3.

A cipher suite utilizada foi:

TLS_AES_256_GCM_SHA384
```
### 9. Validando a CA

Inicialmente, executamos o comando:
```
openssl s_client -connect localhost:443
```
pode gerar:
```
verify error:num=20:unable to get local issuer certificate
```
Isso ocorre porque o sistema não conhece nossa CA.
Podemos fornecer explicitamente o certificado da CA:
```
openssl s_client \
-connect localhost:443 \
-CAfile ca.crt

O resultado esperado é:

Verify return code: 0 (ok)

Isso demonstra que o certificado apresentado pelo servidor pode ser validado utilizando nossa CA.
```
#### Cadeia de confiança

A cadeia de confiança utilizada neste projeto é:
```
┌─────────────────────┐
│   MyLab Root CA     │
│                     │
│      ca.crt         │
└──────────┬──────────┘
           │
           │ assina
           ▼
┌─────────────────────┐
│     Server Cert     │
│                     │
│     server.crt      │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│       Nginx         │
│                     │
│       :443          │
└─────────────────────┘
```
10. Visualizando o certificado no navegador

O certificado pode ser analisado diretamente pelo navegador acessando:
```
https://localhost
```
As informações disponíveis incluem:
```
Subject;
Issuer;
período de validade;
chave pública;
algoritmo de assinatura;
cadeia de confiança;
Subject Alternative Name, quando configurado.
```
<img width="672" height="718" alt="image" src="https://github.com/user-attachments/assets/ba239ca7-7a45-490f-8e66-8f7c074bc78c" />

A relação esperada é:
```
Subject:
    192.168.1.104 (Ip da minha máquina)

Issuer:
    MyLab Root CA
Self-Signed vs CA-Signed
```
Durante o projeto foi possível observar a diferença entre os dois modelos.
```
Self-Signed
Certificate
    │
    ├── Subject: Server
    └── Issuer: Server

O próprio certificado assina a si mesmo.

CA-Signed
MyLab Root CA
      │
      │ assina
      ▼
Server Certificate

Nesse caso:

Subject: Server
Issuer:  MyLab Root CA
```
Esse modelo representa uma estrutura simplificada de PKI.

Comandos utilizados
```
Ver certificado
openssl x509 -in server.crt -noout -text

.Ver Subject e Issuer
openssl x509 -in server.crt -noout -subject -issuer
.Verificar chave privada
openssl rsa -in server.key -check
.Ver CSR
openssl req -in server.csr -noout -text
.Testar Nginx
sudo nginx -t
.Testar TLS
openssl s_client -connect localhost:443
.Testar TLS utilizando a CA
openssl s_client \
-connect localhost:443 \
-CAfile ca.crt
.Testar HTTPS
curl -k https://localhost
```
Segurança

As seguintes práticas foram consideradas durante o laboratório:
```
Chaves privadas não devem ser publicadas.
ca.key deve ser mantida em local seguro.
server.key deve possuir permissões restritivas.
Certificados públicos podem ser compartilhados.
A CA deve ser adicionada explicitamente ao trust store quando necessário.
```
Exemplo de .gitignore:
```
*.key
*.csr
*.srl
Estrutura do projeto
nginx-tls-lab/
│
├── README.md
│
├── images/
│   ├── nginx-https.png
│   ├── certificate.png
│   ├── openssl-handshake.png
│   └── certificate-chain.png
│
└── docs/
    └── troubleshooting.md
```

Próximos passos
```
 Configurar Subject Alternative Name (SAN)
 Adicionar a CA ao trust store do Linux
 Adicionar a CA ao Firefox
 Testar TLS 1.2
 Comparar TLS 1.2 e TLS 1.3
 Capturar o handshake com Wireshark
 Analisar pacotes TLS
 Configurar HTTP → HTTPS redirect
 Configurar HSTS
 Estudar CRL
 Estudar OCSP
 Configurar uma cadeia com Root CA e Intermediate CA
```
Conceitos estudados
```
SSL/TLS
HTTPS
TLS 1.2
TLS 1.3
PKI
Certificate Authority
Certificados digitais
Chaves públicas e privadas
CSR
Certificate Chain
Self-Signed Certificates
CA-Signed Certificates
RSA
AES-GCM
TLS Handshake
OpenSSL
Nginx
Linux
```
Resultado

Ao final do projeto foi configurado um servidor Nginx utilizando HTTPS e TLS, com um certificado digital emitido por uma Certificate Authority criada localmente.

O projeto permitiu compreender na prática o fluxo:
```

Private Key
     │
     ▼
    CSR
     │
     ▼
Certificate Authority
     │
     ▼
Server Certificate
     │
     ▼
Nginx
     │
     ▼
HTTPS / TLS
     │
     ▼
Client
```
O laboratório demonstra, de forma prática, como certificados digitais e uma infraestrutura de confiança podem ser utilizados para proteger a comunicação entre clientes e servidores.
