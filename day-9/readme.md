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


1. Primeiro passo, localizar o diretóri `.docker/config.json`. Se dermos um comando cat, teremos a seguinte informação:  

  ```docker
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
