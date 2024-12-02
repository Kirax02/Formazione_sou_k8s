# Step 3

Questo Helm chart effettua il deploy di un'applicazione Docker presente nel proprio Docker Hub su un cluster Kubernetes all'interno del namespace `formazione-sou`.

## Struttura del Chart

Ho utilizzato il comando `helm create charts` che ha creato una cartella con vari file di default. Ho mantenuto solo quelli necessari per il mio scopo, ovvero:

- `values.yaml`
- `Chart.yaml`
- Nella cartella `templates`:
  - `deployment.yaml`
  - `service.yaml`
  - `serviceaccount.yaml`
  - `ingress.yaml`

### `values.yaml`
il file `values.yaml` contiene le configurazioni generali per il deploy (immagine, risorse, porte, etc.), che saranno richiamate nei file dei template.

### `deployment.yaml`
Il file `deployment.yaml` definisce come viene gestito il ciclo di vita del pod, la configurazione delle repliche, le risorse e le probe per garantire che l'applicazione sia sempre attiva e pronta.

### `service.yaml`
il file `service.yaml` crea un service che espone l'applicazione all'interno del cluster Kubernetes, rendendola raggiungibile tramite un nome di servizio e una porta.

### `serviceaccount.yaml`
Il file `serviceaccount.yaml` crea un ServiceAccount per l'applicazione, utile per assegnare permessi di accesso a risorse Kubernetes.  
Questo file ci servirà per lo step 5.

### `ingress.yaml`
Il file `ingress.yaml` crea un ingress che consente l'accesso esterno all'applicazione tramite un Ingress, instradando il traffico HTTP verso il servizio.  
Questo file ci servirà per lo step 6
