# Configurando o minikube

```
// primeiro comando
minikube start

// ativar ingress
minikube addons enable ingress
```

# Comandos básicos
```
// Consultar pods
kubectl get pods
kubectl get pods --all-namespaces
kubectl get pods -n X

// Consultar services
kubectl get svc

// Consultar ingress
kubectl get ingress
```

## Configurando os arquivos
1. Criar um servidor simples http
#### server.js

```
var http = require('http');

var handleRequest = function(request, response) {
  console.log('Received request for URL: ' + request.url);
  response.writeHead(200);
  response.end('Hello World!');
};var www = http.createServer(handleRequest);
www.listen(8080);
```

2. Criar o Dockerfile para gerar a imagem do servidor http
#### Dockerfile

```
FROM node:6.9.2
EXPOSE 8080
COPY server.js .
CMD node server.js
```

3. Comando para gerar a imagem
```
docker build -t hello-kube:v1 .
```

4. Criando arquivo principal do Kubernetes para poder subir a aplicação
#### deployment.yml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: hello-kube
  name: hello-kube
spec:
  replicas: 1 // copias rodando
  selector:
    matchLabels:
      app: hello-kube
  template:
    metadata:
      labels:
        app: hello-kube
    spec:
      serviceAccountName: default
      containers:
        - name: hello-kube // name simples
          image: "hello-kube:v1" // nome da imagem gerada
```

5. Criando arquivo do serviço para expor a aplicação
#### service.yml

```
apiVersion: v1
kind: Service
metadata:
  name: hello-kube
spec:
  selector:
    app: hello-kube
  ports:
    - name: http
      port: 80 // porta exposta no kubernetes
      protocol: TCP
      targetPort: 8080 // porta dentro do container docker
```      

6. Criando arquivo ingress para poder configurar o DNS para acessar a aplicação
#### ingress.yml
```      
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-kube-ingress
spec:
  rules:
    - host: qualquerdns.local
      http:
        paths:
            - path: /
              pathType: Prefix
              backend:
                service:
                  name: hello-kube
                  port: 
                    number: 80
```      
## Rodando os comandos
```
kubectl apply -f deployment.yml
kubectl apply -f service.yml
kubectl apply -f ingress.yml
```

## Configurando o Ingress
```
//Identificar qual é o IP do minikube
minikube ip
```

## Configurando o DNS interno no minikube
```
// digitar o comando e pegar o IP
kubectl get svc kube-dns -n kube-system

// entrar no minikube
minikube ssh

// instalar o nano
sudo apt-get update
sudo apt-get install nano

// editar o arquivo abaixo. Este arquivo é a confogração do linux para resolver o DNS
sudo nano /etc/resolv.conf

//comentar a linha e add no arquivo a linha abaixo
nameserver IP "QUE VEIO NO PRIMEIRO COMANDO"

// saior do arquivo e digitar o comando
nslookup hello-kube.default.svc.cluster.local
// o resultado deve ser o IP e não um erro

```

## Adicionar no /etc/hots dentro do ssh do minikube
```
// comand
sudo nano /etc/hosts

// add linhas no arquivo.  NO LUGAR DO IP ABAIXO, COLOCAR O QUE APARECEU ALI EM CIMA
172.17.0.2      qualquerdns.local
```

## Adicionar no /etc/hots na mpaquina do vagrant
```
// comand
sudo nano /etc/hosts

// add linhas no arquivo.  NO LUGAR DO IP ABAIXO, COLOCAR O IP QUE APARECE NO COMANDO minikube ip
172.17.0.2      qualquerdns.local
```

## Para testar
```
curl http://qualquerdns.local
```
