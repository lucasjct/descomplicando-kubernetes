# Secrets e ConfigMaps  

Secretes guardam informações sensíveis para serem utilizadas por recursos do Kubernetes. Por default, secrets não garantem segurança com criptografia, e para isso, precisamos usar serviços externos.   

[documentação sobre Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)   


> [!IMPORTANT]
> O base64 não é criptografia, é apenas uma maneira de criar uma cadeia de strings que torna mais rápido o tráfego do dado. Para criptografia, devemos considerar serviços externos.   

O base64 pode ser facilmente decriptado, como podemos ver abaixo.   

`echo -n "c3lzdGVtOmJvb3RzdHJhcHBlcnM6a3ViZWFkbTpkZWZhdWx0LW5vZGUtdG9rZW4=" | base64 -d`    

Para tornar um dado como base64, no Linux podemos utilizar o comando echo que direciona o output para base64:  
`echo "lucasjct" | base64`

### Tipos de Secret  

Na documentação oficial do kubernetes, temos os tipos de Secrets e o exemplo de como aplica-las: [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/#secret-types)  


## Criando um Secret tipo Opaque  
Opaque é o tipo mais comum, ele é utilizado para criar secrets em geral, como informação de usuário e senha

Pela linha de comando, podemos criar secrets como o exemplo abaixo.  
`k create secret generic giropops-test --from-literal=username=bHVjYXNqY3QK --from-literal=password=dGVzdGVzZWNyZXRlCg==`    
Este comando acima, codifica dados em Base64 automaticamente.  Podemos armazenar os dados sensível em vairáveis de ambientes ou em arquivos, utilizando respectivamente:  `--from-env-file` ou `--from-file`.

Podemos criar pelo arquivo Yaml como no exemplo abaixo:  

```yaml
apiVersion: v1
kind: Secret 
metadata: 
  name: giropops
type: opaque
data:
  username: bHVjYXNqY3QK
  password: dGVzdGVzZWNyZXRlCg== 
```
## Utilizando o Secret dentro de um Pod  

Dado o [arquivo da secret](/secret/opaq-secret.yaml) criado, podemos aplicar a secret no pod da seguinte maneira abaixo:

```yaml
apiVersion: v1
kind: Pod  
metadata:
  name: giropops-pod
spec:
  containers:
  - name: giropops-container
    image: nginx
    env: # aqui se inicia a configuração da Secret
    - name: USERNAME  # nome da variavel de ambiente que recebera o usaname definido na Secret e será usada no pod
      valueFrom:
        secretKeyRef:
          name: giropops-secret  # nome da Secret
          key: username # esta variavel esta definida na Secret
    - name: PASSWORD # nome da variavel de ambiente que recebera o usaname definido na Secret e será usada no Pod
      valueFrom:
        secretKeyRef:
          name: giropops-secret # nome da Secret
          key: password # esta variavel esta definida na Secret
```

Verificar como a variável de ambiente foi armazenada no container do pod:  

`kubectl exec -ti giropops-pod --env`  

Comando para acessar o container:  

`kubectl exec -ti giropops-pod -- bash`     


## Criando uma Secret para autenticar no Docker Hub do tipo dockerconfigjson e usar imagens privadas    

Guardar os dados para autenticar e lidar com as imagens hospedadas no Docker Hub

>[!IMPORTANT]  
> Como pré requisito, devemos ter um conta no Docker Hub.  


1. Primeiro passo, localizar o diretório `.docker/config.json`. Digite `docker login` e a saída será o diretorio que precisamos. Se dermos um comando cat, teremos a seguinte informação:  

  ```yaml
  },
  "https://index.docker.io/v1/": {
	  		"auth": "bHVjYXNqY3Q6THVjYXMjMjAyNA=="
	  	}
  }
  ```
  
  2. Vamos utilizar o arquivo acima para fazer nossa Secret, utilizando o base64 vamos colar a saída do comando abaixo para utilizar na secret:   
  `base64 docker/config.json`  

  3. Agora precisamos começar a definir nosso Secret utilizando a o base64 gerado acima como valor da propriedade: `dockerconfigjson`:

```yaml
apiVersion: v1
kind: Secret 
metadata:
  name: dockerhub-secret
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: |
    ewoJImF1dGhzIjogewoJCSJodHRwczLmRvY2tlci5pby92MS8iOiB7ChdXRo
    IjogImJIVmpZWE5x1qTWpBeU5BPT0iQp9
  ```      


## Criando Secret do tipo TLS (Transport Layer Security) 

O tipo Secret `kubernetes.io/tls` são utilizados para garantir segurança na comunicação entre os serviços Kubernetes. O exemṕlo conhecido, é garantir a segurança através da comunicação HTTPS.   

1. Para gerar um certificado, pode utilizar o `openssl`:   
`openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout chave-privada.key -out certificado.crt`  
2. Podemos observar que o comando gera dois arquivos como saída: `chave-privada.key` e `certificado.crt`. Estes dois arquivos serão armazenados na Secret.
3. Após gerados, podemos iremos utilizá-los agora para criar nossa Secret através da linha de comando:  
`kubectl create secret tls meu-servico-web-tls-secret --cert=certificado.crt --key=chave-privada.key`   
4. Feito isso, a Secret está criada. Podemos utilizar `kubectl get secret meu-servico-web-tls-secret -o yaml` a Secret estará configurada com o certificado e a chave-privada.    

```yaml
apiVersion: v1
data:
  tls.crt: LS0tLS1CRUdJTiBDRVSJUSUZJQ0FURS0tLS0tCk1JSUR1VENDQXFHZ0F3SUJBZ0lVSkVxdXFJaDlhOStQQlZDZGQrWjlOHMEx
  tls.key: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0t4LS0tCk1JSUV2UUlCQURBTkJna3Foa2lHOXcwQkFRRUZBHQVNDQktjd2dnU2pBZ
kind: Secret
metadata:
  name: meu-servico-wev-tls-secret
  namespace: default
  resourceVersion: "28957"
type: kubernetes.io/tls
```  
  

# ConfigMaps   

Seu proposito é armazenar informações relacionadas a configurações.    

Abaixo, é o arquivo de configuração do configMap:  

```yaml
apiVersion: v1
data:
  nginx.conf: | 
    events {}

    http{
      server{
        listen 80;
        listen 443 ssl;

        ssl_certificate /etc/nginx/tls/certificado.crt;
        ssl_certificate_key /etc/nginx/tls/chave-privada.key;
        
        location / {
          return 200 'ola terraqueos e extraterrestres"\n';
          add_header Content-Tyoe text/plain;
        } 
      }
    }
kind: ConfigMap
metadata:
  name: nginx-config
  namespace: zora-system
```  
No configMap acima contém o arquivo de configuração do `nginx.conf`.  

Para aplicar o manifesto:  
`kubectl apply -f nginx-config.yaml`   

Para acessar o conteúdo do configMap e garantir que a configuração está correta:   
`kubectl get configmap nginx-config -o yaml`   

Para ver como o ConfigMap foi aplicado ao Pod, check a configuração do Pod abaixo:   
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: giropops-pod
  labels:
    app: nginx
spec:
  containers:
    - name: giropops-container 
      image: nginx
      ports:
        - containerPort: 80 
        - containerPort: 443 
      volumeMounts: 
        - name:  nginx-config-volume # nome do volume em que será montado o arquivo de config do nginx
          mountPath: /etc/nginx/nginx.conf # Caminho que será montado o arquivo de configuração do nginx
          subPath: nginx.conf  # nome do arquivo de configuração do Nginx
        - name: nginx-tls # nome do volume que será montado o arquivo tls e a chava gerada.
          mountPath: /etc/nginx/tls # caminho em que será montado o certificado tls e a chave privada gerada.
  volumes:
  - name: nginx-config-volume # nome do volume
    configMap:
      name:  nginx-config # nome da configMap
  - name: nginx-tls #  nome do volume que vamos usar para montar o certificado TLS e a chave privada.
    secret: # volume utilizado, tipo secret
      secretName: meu-servico-web-tls-secret # nome do Secret criada
      items:
        - key: tls.crt # nome do arquivo dentro da Secret criada
          path: certificado.crt # nome do arquivo gerado após gerar o certificado com openssl
        - key: tls.key # nome do arquivo dentro da Secret criada
          path: chave-privada.key # nome do arquivo gerado após gerar o certificado com openssl
           
```  

Aplicar o manifesto acima:  
`kubectl apply -f nginx.yaml`  

Criar um Service para expor o Pod criado:  
`kubectl expose po nginx`  

Garantir que o Service foi criado:  
`kubectl get services`   

Fazer o port-foward para garantir que a requisição para o nginx está funcionando, conforme foi configurado no configMap:  
`kubectl port-forward service/nginx 4443:443`