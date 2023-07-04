# Como executar Prometheus e Grafana para monitorar um cluster no EKS   


**Pré-requisito:**  

* Instalar e configuraro Kubectl para executar os comandos do Kubernetes.   

* Ter uma conta na AWS e instalar e configurar o [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) com as credencias geradas no AWS IAM.  

* Instalar o [EKSCTL](https://eksctl.io/) que é a ferramenta CLI oficial para criar e gerenciar o cluster na AWS EKS. 

* Clonar o projeto do Kube-Prometheus, para utilizar seus plugins e aplicar seus manifestos que garantem um  conjuntos de ferramentas para montiroamento e alertas.    
***   

* Cumprido os requisitos acima, vamos começar informando o **kubectl** que iremos gerenciar um cluster no EKS:  

```
aws eks --region us-east-1 update-kubeconfig --name eks-cluster
```

* Agora podemos começar a criar o cluster no EKS, com o seguinte comando no terminal:  

```
eksctl create cluster --name=eks-cluster --version=1.24 --region=us-east-1 --nodegroup-name=eks-cluster-nodegroup --node-type=t3.medium --nodes=2 --nodes-min=1 --nodes-max=3 --managed
```   

* Após alguns minutos, o cluster será criado no EKS.  Para confirmar a criação, vamos executar o seguinte comando:  

    `kubectl get nodes`   

Como foram criados 2 nodes, a saída do comando mostrará estes nodes do EKS.   

* Depois de ter os cluster rodando na AWS, precisamos clonar o projeto do Kube-Prometheus, para isso, vamos executar o comando:   
`git clone https://github.com/prometheus-operator/kube-prometheus`    

Após clonar o projeto, vamos acessá-lo e executar o seguinte comando:   

`kubectl create -f manifests/setup`  

* Os recursos que estão dentro dos manifestos do kube-prometheus passarão a ser reconhecidos pelo Kubernetes, agora teremos que aplicá-los:   

`kubectl apply -f manifests/`   

Após a conclusao do apply, pode ver se o ServicesMonitors foi concluído:   

` kubectl get servicemonitors -n monitoring`   

Ele é responsável é um recurso do Prometheus operator responsável por monitorar os recursos Kubernetes e as Aplicações.    


* Finalmente, podemos acessar o Grafana e ter acesso a todas métricas default que podem ser visualizadas no dashboards do Grafana já configurados por default após a instalação do Kube Prometheus. Para isso, precisamos expor a porta do Grafana para acessá-lo remotamente através da máquina local.   

  `kubectl port-forward -n monitoring svc/grafana 33000:3000`   

* Agora podemos acessar o Grafana a partir do browser com o seguinte endereço:  

  `http://localhost:33000`   

* O usuário e senha padrão para o grafana é `admin`. Feito isto, pedirá para ser alterada a senha.    

* Acessando a página principal do Grafana, podemos ir no menu lateral, selecionar Dashboards, escolher a opção Default, selecionar o Prometheus e visualizar as diversas opção de métricas que estão disponíveis.    

* Para deletar o cluster no EKS e todos os recursos associados a ele, podemos executar:  
`eksctl delete cluster --name=eks-cluster -r us-east-1`     

