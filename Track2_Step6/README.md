# Configurazione Ingress per Accedere a una Pagina "Hello World"

Con helm install si installa anche l' ingress controller.  
Successivamente, bisogna attivarlo con il seguente comando: `minikube addons enable ingress`.  
Bisogna acnche andare a modificare il file: `/etc/hosts` associando al dominio formazionesou.local l indirizzo IP di minikube, che si ottiene con il segunete comnado `minikube ip`
