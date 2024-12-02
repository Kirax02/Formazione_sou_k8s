# Step 2

## Primo passo: Creare Dockerfile app di esempio Flask che esponga una pagina avente stringa "hello world"
ho creato il docker file app prendendo un grande spunto da github.

## Secondo passo: Scrivere pipeline dichiarativa Jenkins che effettui una build dell'immagine Docker e che effettui il push sul proprio account DockerHub con tag diverso in base al branch
Prima di creare la pipeline, ho modificato il playbook Ansible in modo da montare due volumi necessari per l'utilizzo di docker su entrambi i container. Come alternativa, si potrebbe anche costruire un'immagine Jenkins che includa l'installazione di Docker nel Dockerfile.

Nel playbook ho aggiunto delle task per creare il gruppo docker con lo stesso UID/GID della VM all'interno dei due container, per poi aggiungere l'utente jenkins a questo gruppo, in modo che abbia i permessi necessari per accedere al demone Docker.

Infine, ho creato il Jenkinsfile che esegue la build dell'immagine e il push con il tag corretto, a seconda del branch utilizzato.
