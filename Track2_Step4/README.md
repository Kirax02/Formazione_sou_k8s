# Jenkins Pipeline con Helm e Kubernetes

Questo progetto configura una pipeline Jenkins per installare Helm e `kubectl`, e consente l'uso di Helm per distribuire applicazioni su un cluster Kubernetes creato con Minikube.

## Creazione del cluster di Kubernetes su MAC.

Per creare il cluster su Mac, sono necessari i seguenti strumenti: **Minikube** e **HyperKit**, che possono essere installati con i comandi:  
`brew install minikube`  
`brew install hyperkit`  
Una volta installati, eseguire il comando:  
`minikube start --listen-address=0.0.0.0 --ports=8443:8443 --driver=hyperkit`  
Questo comando avvia un cluster Kubernetes con Minikube utilizzando tutte le interfacce di rete disponibili e configurando HyperKit come driver.

## Configurazione di Jenkins per interagire con il cluster ed installazione di Helm

Ho fatto alcune modifiche al playbook cosi che il container jenkins possa interagire con il cluster e possa utilizzare helm.  

Ho aggiunto al playbook le seguenti task che scaricano ed estraggono l' archivio di helm.
```yaml
    - name: Download Helm binary
      community.docker.docker_container_exec:
        container: jenkins_master
        user: root
        command: >
          curl -fsSL -o /var/jenkins_home/helm.tar.gz https://get.helm.sh/helm-v3.16.3-linux-amd64.tar.gz

    - name: Extract Helm binary
      community.docker.docker_container_exec:
        container: jenkins_master
        user: root
        command: >
          tar -xzf /var/jenkins_home/helm.tar.gz -C /var/jenkins_home --strip-components=1 linux-amd64/helm
```
Con queste task invece spostiamo l' eseguibile di helm in /usr/local/bin e gli impostiamo i permessi di esecuzione
```yaml
    - name: Move Helm binary to /usr/local/bin
      community.docker.docker_container_exec:
        container: jenkins_master
        user: root
        command: >
          mv /var/jenkins_home/helm /usr/local/bin/helm

    - name: Add execution permission to helm
      community.docker.docker_container_exec:
        container: jenkins_master
        user: root
        command: >
          chmod +x /usr/local/bin/helm
```
Una volta installato helm bisogna copiare i file utili per la configurazione del cluster di k8s dal MAC alla VM con le seguenti task
```yaml
    - name: Copy kubeconfig to VM
      copy:
        src: /Users/lorenzomoro/.kube/config
        dest: /home/vagrant/config

    - name: Copy client.crt to VM
      copy:
        src: /Users/lorenzomoro/.minikube/profiles/minikube/client.crt
        dest: /home/vagrant/client.crt

    - name: Copy client.key to VM
      copy:
        src: /Users/lorenzomoro/.minikube/profiles/minikube/client.key
        dest: /home/vagrant/client.key
```
Bisogna modificare il file sostituendo i path corretti per i certificati e aggiungendo una linea che permette di ignorare il certificato con le seguenti task

```yaml
    - name: Replace client.crt
      ansible.builtin.replace:
        path: /home/vagrant/config
        regexp: 'client-certificate: /Users/lorenzomoro/.minikube/profiles/minikube/client.crt'
        replace: 'client-certificate: /var/jenkins_home/client.crt'

    - name: Replace client.key
      ansible.builtin.replace:
        path: /home/vagrant/config
        regexp: 'client-key: /Users/lorenzomoro/.minikube/profiles/minikube/client.key'
        replace: 'client-key: /var/jenkins_home/client.key'

    - name: Replace certificare
      ansible.builtin.replace:
        path: /home/vagrant/config
        regexp: 'certificate-authority: /Users/lorenzomoro/.minikube/ca.crt'
        replace: 'certificate-authority:'

    - name: Add lines
      ansible.builtin.lineinfile:
        path: /home/vagrant/config
        insertafter: '^    certificate-authority:.*'
        line: '    insecure-skip-tls-verify: true'
 ```
Con le seguenti task invece andiamo a creare la directory che conterrà il config e copiamo i file necessari dalla nostra VM al container jenkins
```yaml
    - name: Create .kube dir in jenkins_home
      community.docker.docker_container_exec:
        container: jenkins_master
        user: jenkins
        command: mkdir -p /var/jenkins_home/.kube

    - name: Copy kubeconfig to Jenkins container
      ansible.builtin.command:
        cmd:  docker cp /home/vagrant/config jenkins_master:/var/jenkins_home/.kube/config

    - name: Copy client.crt to Jenkins container
      ansible.builtin.command:
        cmd:  docker cp /home/vagrant/client.crt jenkins_master:/var/jenkins_home/client.crt

    - name: Copy client.key to Jenkins container
      ansible.builtin.command:
        cmd:  docker cp /home/vagrant/client.key jenkins_master:/var/jenkins_home/client.key
```
Invece con le seguenti task scarichiamo il comando kubectl
```yaml
    - name: Download kubectl binary in Jenkins container
      community.docker.docker_container_exec:
        container: jenkins_master
        user: root
        command: > 
          curl -LO https://dl.k8s.io/release/v1.27.3/bin/linux/amd64/kubectl

    - name: Move kubectl binary in /usr/local/bin
      community.docker.docker_container_exec:
        container: jenkins_master
        user: root
        command: >
          mv kubectl /usr/local/bin/

    - name: Add execution permission to kubectl
      community.docker.docker_container_exec:
        container: jenkins_master
        user: root
        command: >
          chmod +x /usr/local/bin/kubectl
```
## Lancio pipeline 
Una volta finito il setup per jenkins bisogna lanciare la pipeline che effettuerà l' helm install sul namespace formazione-sou
