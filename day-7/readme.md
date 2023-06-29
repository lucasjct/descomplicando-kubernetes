## StateFulSet e StateHeadless

O stateFulSet garante que os pods subirão em ordem, garante o load balance, o DNS para o pod, caso esteja associado com o StateHeadless, etc. 

Após criar as réplicas com StateFulSet, criamo o StateHeadles que será associado ao primeiro através da label nginx.  O stateles, garantira o dns para o stateFulSet. Após criados nossos pods, podemos testá-los da seguinte maneira:  

* Entrar no container que foi criado:  
    ` kubectl.exe exec -ti nginx-1 -- bash `

* Exectuar o comando bash para criar um texto no arquivo do index.html do nginx:   
`echo Segundo Pod > /usr/share/nginx/html/index.html`

* Dentro do container, executar o curl e verificar se o dns está funcionando:  
`curl nginx-1.nginx.default.svc.cluster.local`