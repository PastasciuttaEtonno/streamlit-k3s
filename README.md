# Streamlit K3s Application

Questo progetto contiene un'applicazione Streamlit containerizzata e configurata per il deployment su Kubernetes (K3s). Di seguito la descrizione dei componenti principali.

## Struttura del Progetto

### MyApp.py
Application Streamlit principale molto semplice.

## Build e Distribuzione

### Build dell'immagine Docker
```bash
docker build -t <username>/streamlit-k3s:latest .
```

### Test locale
```bash
docker run -p 8501:8501 <username>/streamlit-k3s:latest
```

### Push su Docker Hub
```bash
docker login
docker push <username>/streamlit-k3s:latest
```

## Deployment su K3s

L'immagine è pronta per il deployment su Kubernetes K3s. Il container espone la porta 8501 e include health check per facilitare l'orchestrazione e il load balancing.

I manifest si trovano nella directory /k3s


### Comandi di Deployment

Applicare i manifest sul control plane di K3s nell'ordine seguente:

```bash
# 1. Applicare la configurazione di Nginx
kubectl apply -f nginx-config.yaml

# 2. Applicare l'applicazione Streamlit e il suo servizio
kubectl apply -f streamlit-v1.yaml

# 3. Applicare il proxy Nginx che userà la ConfigMap
kubectl apply -f nginx-deploy.yaml

# 4. Applicare l'Ingress per aprire le porte al mondo
kubectl apply -f ingress.yaml
```

### Comandi di Verifica

Verificare lo stato del cluster e dell'applicazione:

```bash
# Verificare i nodi del cluster
kubectl get nodes

# Verificare i namespace
kubectl get namespaces

# Verificare il Deployment
kubectl get deployment streamlit-app

# Verificare i Pod
kubectl get pods -l app=streamlit

# Visualizzare il dettaglio dei Pod
kubectl describe pod -l app=streamlit

# Verificare il Service
kubectl get service streamlit-service

# Verificare gli endpoint del Service
kubectl get endpoints streamlit-service

# Visualizzare i log di un Pod specifico
kubectl logs <pod-name>

# Visualizzare i log in tempo reale
kubectl logs -f <pod-name>

# Eseguire un comando all'interno di un Pod
kubectl exec -it <pod-name> -- /bin/bash

# Ottenere informazioni dettagliate su un Deployment
kubectl describe deployment streamlit-app

# Monitorare gli eventi del cluster
kubectl get events --sort-by='.lastTimestamp'
```

### Accesso all'Applicazione

Una volta applicati i manifest, accedere all'applicazione:

```bash
# Ottenere l'indirizzo IP esterno (EXTERNAL-IP)
kubectl get service streamlit-service

# L'applicazione sarà disponibile su http://<EXTERNAL-IP>
# Se si usa LoadBalancer locale, potrebbe essere localhost o l'IP assegnato da K3s
```

### Gestione del Deployment

Comandi utili per la gestione:

```bash
# Scalare il numero di repliche
kubectl scale deployment streamlit-app --replicas=3

# Applicare aggiornamenti al manifest
kubectl apply -f deployment.yaml

# Editare il deployment direttamente
kubectl edit deployment streamlit-app

# Applicare una patch al deployment
kubectl patch deployment streamlit-app -p '{"spec":{"replicas":3}}'

# Forzare il riavvio di tutti i Pod
kubectl rollout restart deployment/streamlit-app

# Aggiornare l'immagine
kubectl set image deployment/streamlit-app streamlit=<username>/streamlit-k3s:v2.0

# Monitorare lo stato del rollout durante l'aggiornamento
kubectl rollout status deployment/streamlit-app

# Rollback all'immagine precedente
kubectl rollout undo deployment/streamlit-app

# Visualizzare la cronologia degli aggiornamenti
kubectl rollout history deployment/streamlit-app

# Eliminare il Deployment
kubectl delete deployment streamlit-app

# Eliminare il Service
kubectl delete service streamlit-service
```

## Requisiti

- Docker installato per il build e il test locale
- Docker Hub account per il push delle immagini
- K3s cluster configurato e alive
- kubectl installato e configurato per accedere al cluster K3s
