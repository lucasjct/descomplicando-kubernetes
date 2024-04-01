# Helm    

[Documentação Oficial](https://helm.sh/docs/)

Helm é um gerenciador de pacotes Kubernetes, instala, atualiza aplicações e serviços no Kubernetes.  
Ele possui um __Chart__ que é um conjunto de arquivos, um pacote com um conjunto de templates que definem a aplicação.

Arquivo `values.yaml` é onde podemos parametrizar nossa instalação. Nele contém os valores default para um chart. Esses valores podem ser sobrescritos durante o uso por  comandos `helm install` ou `helm upgrade`.    

O arquivo `values.yaml` pode ser consutado neste [link](../dia-19/chart/values.yaml).  
Ao declarar os valroes em `values.yaml`, estes serão utilizado nos templates utilizando a seguinte sintaxe:  `{{ .Values.giropopsSenha.name }}` 

Exemplo completo


```apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: giropops-senhas
  name: {{ .Values.giropopsSenha.name }} # valor declarado no values.yaml
spec:
  replicas: 2
  selector:
    matchLabels:
      app: giropops-senhas
  template:
    metadata:
      labels:
        app: giropops-senhas
    spec:
      containers:
      - image: {{ .Values.giropopsSenha.image }} # valor declarado no values.yaml
        name: giropops-senhas
        ports:
        - containerPort: 5000
        imagePullPolicy: Always
``` 

### Instalação 

```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```   

### Deploy  

Para deployar a aplicação de exemplo, considerando que está no repositório local, no diretório que contém o __chart__, podemo usar o comando:  

`helm install giropops-senhas ./`  


Para desinstalar, utilizamos o comando:  
`helm uninstall giropops-senhas`