# Helm    

[Documentação Oficial](https://helm.sh/docs/)

### Instalação 

```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```   

Helm é um gerenciador de pacotes Kubernetes, instala, atualiza aplicações e serviços no Kubernetes.  
Ele possui um __Chart__ que é um conjunto de arquivos, um pacote com um conjunto de templates que definem a aplicação.

Arquivo `values.yaml` é onde podemos parametrizar nossa instalação. Nele contém os valores default para um chart. Esses valores podem ser sobrescritos durante o uso por  comandos `helm install` ou `helm upgrade`.    

O arquivo `values.yaml` pode ser consutado neste [link](../dia-19/chart/values.yaml).  
Ao declarar os valroes em `values.yaml`, estes serão utilizado nos templates utilizando a seguinte sintaxe:  `{{ .Values.giropopsSenha.name }}` 

### Exemplo completo


```{{- range $component, $config := .Values.deployments }} # 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ $component }}
  labels:
    app: {{ $config.labels.app }}
spec:
  replicas: {{ $config.replicas | default 2 }} # função default
  selector:
    matchLabels:
      app: {{ $config.labels.app }}
  template:
    metadata:
      labels:
        app: {{ $config.labels.app }} 
    spec: 
      containers:
      - name: {{ $component }}
        image: {{ $config.image }} 
        ports:
        {{- range $config.ports }}
        - containerPort: {{ .port }}
        {{- end }}
        resources:
          requests:
            memory: {{ $config.resources.requests.memory }}
            cpu: {{ $config.resources.requests.cpu }}
          limits:
            memory: {{ $config.resources.limits.memory }}
            cpu: {{ $config.resources.limits.cpu }}
---
{{- end }}       
``` 

Para fazer comentários dentro de arquivos Helm, podemos utiliza a sintaxe abaixo:  

```
{{/*
Definir as portas dos containers
*/}}
```

### Deploy  

Para deployar a aplicação de exemplo, considerando que está no repositório local, no diretório que contém o __chart__, podemo usar o comando para instalar:  

`helm install giropops-senhas ./`    

Para atualizar:  
`helm upgrade giropops-senhas ./`


Para desinstalar, utilizamos o comando:  
`helm uninstall giropops-senhas`  


## Funções do Helm  

* `default` para definir valores, como exemplo em que definimos um valor default de replicas:  `replicas: {{ $config.replicas | default 2 }}`   
* `toYaml` para converter um campo para yaml. como exemplo defimos um valor para o configMap: `{{- toYaml $config | nindent 4}}` e `nindent 4` é a definição de identação para o yaml.    
* `toJspm` para converter um campo para json. como exemplo defimos um valor para o configMap: `{{- toJson $config}}`.  

Para verificar estes outputs, podemos executar respectivamente:    
`k get cm giropops-senha-observablity-config -o yaml`   

`k get cm giropops-senha-db-config -o yaml`  

###   Helpers  

Uma forma de reduzir a quantidade de código repetido e a complexidade nos templates.   

o arquivo de exemplo:  

[_helpers.tpl](/chart/templates/_helpers.tpl)  

Dentro dos Helpers, podemos utilizar funções que vimos até então.
