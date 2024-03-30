## Componentes importantes do ecossistema Kubernetes    

O conteúdo abaixo foi extraído e adaptdo da LinuxTips (Descomplicando Kubernetes) e partes da doc do Kubernetes.


 ### Container Engine   
 Gerencia imagens e volumes, responsável por garantir que os recursos que os containers estão utilizando, estão devidamente isolados e sendo executados. Gerencia o ciclo de vida dos containers. Exemplo de Container Engine: Docker, CRI-0 e Podman.      

* __Container Runtime:__  o Container Engine faz utiliza algum Container Runtime.  
* __Tipo de Container Runtime__:  
    
    * __Low-level:__ executados diretamente pelo Kernel, exemplo: runc, crun, runsc.    
    * __High-level:__ executados por um *Container Engine*, como containerid, CRI-O e o Podman.    
    * __Sandbox:__ executado por um *Container Engine*, executado de forma mais segura em unikernels ou utilizando proxy para comunicar-se com Kernel.  *Gvisor* é um exemplo.   
    * __Virtualized:__  executado por *Container Engine*, responsaveil por executar containers de maneira segura em máquinas virtuais. *Kata Containers* é um exemplo.   


## Arquitetura do Kubernetes  

### Control Planes e Workers.

Cluster organizados em modelo __Control Plane__/__Workers__. Recomendado 3 __nodes__ sendo um *Control Plane* e outro *Worker*. O control plane é voltado para o gerenciamento do cluster. Os workers para executar as tarefas sobre o cluster.   

### API Server  

Componente do Control Plane que expõe a API do Kubernetes. É uma API REST então quando executamos comando com o `kubectl` estamos enviando requisições HTTP para API Server que nos retornar o que foi solicitado em um JSON ou YAML.

### etcd  

É um data store que armazena dados utilizando estrutura chave-valor, utilizadno para armazenar especificações, status e configurações do cluster. Os dados podem ser manipulados apenas através da API. Veja mais na [doc oficial](https://etcd.io/docs/v3.5/quickstart/).   

### Scheduler  

É um componente do control plane que selecionará o nó que irá hospedar um determinado pod. Os fatores que serão levados em conta as decisões do scheduler são: hardware/software/policy, workload.  

### Controller Manager  

É quem garante que o cluster esteja no último estado definido no *etcd*. O controler manager sempre olhara para o etcd para garantir que a conformidade do cluster esteja de acordo com o que foi definido.    

### Kubelet   

É um agente primário que será executado em cada node worker do cluster. Ele é responável por de fato gerenciair os __pods__, que foram direicionados pelo controller do cluster. Ele pode iniciar, manter ou parar os containers em funcionamento de acordo com as instruções recebidas. Veja mais na [documentação oficial](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/).   

### Kube-proxy  

Atua como um *load-balancer*. Responsável por efetuar o roteamento de requisições para os pods corretos, como também cuidar da parte de redes do nó.  



## Conceitos Chaves Kubernetes   

### Pod  
Menor objeto do Kubernetes. Os pods organizam os containers. Como os containers são organizados dentro de Pods, eles dividem recursos de memória e cpu, volumes, endereços de rede. Pode existir varios containers dentro de um pod. Exemplo de configuração de um Pod com um úncio container: [ubuntu](../day-2/pod-limitado.yml). Exemplo de pod com mais de um container: [conjunto de containers](../day-2/pod-parametros.yaml). 


### Deployment  

É um conjunto de __ReplicaSet__. Garante que um número determinado de *replicas* de pods sejam executados dentro workers do cluster. Exemplo de [deployment](../day-3/deployment.yaml). Podemos observar as replicas definidas para um pod, bem como as estratégias de deploy e rollback, além de recursos do pod no exemplo do link acima.   

### ReplicaSets  

Objeto responsável por garantir a quantidade de pods em execução em um node. 

### Service  

É um método de expor uma uma rede da aplicação para executar com um ou mais pods dentro de um cluster. Para isso, utiliza-se: *ClusterIP*, *NodePort* ou *LoadBalancer*. Veja exemplos para: [ClusterIP](../day-7/services/nginx-clusterIP-service.yaml), [NodePort](../day-7/services/nginx-nodePort-service.yaml), [LoadBalancer](../day-7/services/load-balancer-service.yaml).   