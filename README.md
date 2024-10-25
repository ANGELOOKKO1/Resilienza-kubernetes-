## Avvio Minikube Single Node
```minikube start```

### Creazione Namespace
```kubectl apcreate ns postgres-ns```
```kubectl create ns redis-ns```
```kubectl create ns client```

### Inizio Configurazione di Redis
### Sposto verso il namespace creato
```kubectl config set-context --current --namespace=redis-ns```

### Una volta fatto ciò, spostarsi nella cartella in cui vi sono i manifesti di configurazione di Redis, fatto ciò applicare tutti i manifesti per la sua configurazione
```kubectl apply -f .```

### Spostarsi nella cartella sentinel e porre lo stesso comando. Una volta applicati tutti i manifesti previsti, porre il controllo se tutto è correttamente configurato
### Check dello stato dei pod all'interno del nostro cluster Minikube andando ad indicare anche l'IP
```kubectl get pods -o wide -n redis```

### Controllo Service e Persistent Volume
```kubectl get pvc -n redis-ns```
```kubectl get svc -n redis-ns```

### Una volta eseguito che tutto è sotto controllo ed ad un corretto funzionamento, poniamo in analisi la configurazione di Redis e dei suoi pod, iniziando ad interrogare il pod numero 0 di default settato a master e notando anche i suoi slave
```kubectl exec -it redis-0 -n redis-ns -- redis-cli```

### Poniamo il comando di autenticazione al fine di accedere al nostro database
```AUTH```

### Interrogo la sua configurazione
```INFO REPLICATION```

### Potremmo notare il seguente stato
```Replication```
```role:master```
```connected_slaves:2```
```slave0:ip=10.244.0.8,port=6379,state=online,offset=207267,lag=1```
```slave1:ip=10.244.0.9,port=6379,state=online,offset=207267,lag=1```
```master_failover_state:no-failover```
```master_replid:9858ee1cebf87b304531bfcc2434b74ab5a6e76a```
```master_replid2:0000000000000000000000000000000000000000```
```master_repl_offset:207433```
```second_repl_offset:-1```
```repl_backlog_active:1```
```repl_backlog_size:1048576```
```repl_backlog_first_byte_offset:1```
```repl_backlog_histlen:207433```
```127.0.0.1:6379> ```

### Tale comando al fine di un check può essere posto anche per quanto riguarda i pod redis-1 e redis-2
### Posta l'attenzione su redis-0 e andando a controllare che gli slave sono correttamente configurati, si passa all'interrogazione delle sentinel
```kubectl exec -it sentinel-0 -n redis-ns -- redis-cli```

### A seguire poniamo l'interrogazione al fine di comprendere il suo stato effettivo
```INFO SENTINEL```
