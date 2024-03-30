## Componentes importantes do ecossistema Kubernetes    

O conteúdo abaixo foi extraído e adaptdo da LinuxTips (Descomplicando Kubernetes) e partes da doc do Kubernetes.


* __Container Engine__ - gerencia imagens e volumes, responsável por garantir que os recursos que os containers estão utilizando, estão devidamente isolados. Exemplo de Container Engine: Docker, CRI-0 e Podman.    
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


