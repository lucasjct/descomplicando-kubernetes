## StateFulSet e StateHeadless

[Documentação StateFulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)  

O stateFulSet garante que os pods subirão em ordem, garante o load balance, o DNS para acessar o pod, caso esteja associado com o ao Headless Service, etc. 

Após criar as réplicas com StateFulSet, criamos o Service Headless, que serão associados através da label nginx.  O Service Headless, garantirá o DNS para o stateFulSet.  
Exemplo:  
`nginx-1.nginx.default.svc.cluster.local`  
O termo headless, diz respeito que o ClusterIP do Service Headless, deve ser informado como 0.  

Após criados nossos pods, podemos testá-los da seguinte maneira:  

* Entrar no container que foi criado:  
    ` kubectl.exe exec -ti nginx-1 -- bash `

* Exectuar o comando bash para criar um texto no arquivo do index.html do nginx:   
`echo Segundo Pod > /usr/share/nginx/html/index.html`

* Dentro do container, executar o curl e verificar se o dns está funcionando:  
`curl nginx-1.nginx.default.svc.cluster.local`