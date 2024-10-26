## Obbiettivo
##### L’obiettivo principale di questo progetto è dimostrare l’implementazione di un cluster Kubernetes per servizi di caching e memorizzazione dei dati utilizzando Redis e PostgreSQL. Il lavoro esplora come l’integrazione di queste tecnologie possa ottimizzare la gestione dei dati in ambienti cloud. Redis nel cluster sarà configurato con tre nodi: un master per le operazioni di lettura e scrittura, e due slave per mantenere le repliche. L’uso di Redis Sentinel gestirà gli eventi di failover, garantendo alta disponibilità e consistenza dei dati con un impatto minimo sul servizio. Sarà inoltre configurato un servizio di memorizzazione dei dati con PostgreSQL, assicurando resilienza e continuità operativa, sfruttando le funzionalità di Kubernetes per una gestione ottimale. Infine, verrà presentato un caso d’uso per illustrare l’integrazione e l’efficacia dei servizi implementati di Redis e PostgreSQL in un cluster Minikube, evidenziando i benefici e le potenzialità della soluzione proposta.


#### Avvio Minikube Single Node
```minikube start```

#### Creazione Namespace
```kubectl apcreate ns postgres-ns```
```kubectl create ns redis-ns```
```kubectl create ns client```

### Inizio Configurazione di Redis
#### Sposto verso il namespace creato
```kubectl config set-context --current --namespace=redis-ns```

#### Una volta fatto ciò, spostarsi nella cartella in cui vi sono i manifesti di configurazione di Redis, fatto ciò applicare tutti i manifesti per la sua configurazione
```kubectl apply -f .```

#### Spostarsi nella cartella sentinel e porre lo stesso comando. Una volta applicati tutti i manifesti previsti, porre il controllo se tutto è correttamente configurato Check dello stato dei pod all'interno del nostro cluster Minikube andando ad indicare anche l'IP
```kubectl get pods -o wide -n redis```

#### Controllo Service e Persistent Volume
```kubectl get pvc -n redis-ns```
```kubectl get svc -n redis-ns```

#### Una volta eseguito che tutto è sotto controllo ed ad un corretto funzionamento, poniamo in analisi la configurazione di Redis e dei suoi pod, iniziando ad interrogare il pod numero 0 di default settato a master e notando anche i suoi slave
```kubectl exec -it redis-0 -n redis-ns -- redis-cli```

#### Poniamo il comando di autenticazione al fine di accedere al nostro database
```AUTH```

#### Interrogo la sua configurazione
```INFO REPLICATION```

#### Tale comando al fine di un check può essere posto anche per quanto riguarda i pod redis-1 e redis-2. Posta l'attenzione su redis-0 e andando a controllare che gli slave sono correttamente configurati, si passa all'interrogazione delle sentinel
```kubectl exec -it sentinel-0 -n redis-ns -- redis-cli```

#### A seguire poniamo l'interrogazione al fine di comprendere il suo stato effettivo
```INFO SENTINEL```

### Inizio Configurazione di PostgreSql
#### Sposto verso il namespace creato
```kubectl config set-context --current --namespace=postgres-ns```

#### Una volta fatto ciò, spostarsi nella cartella in cui vi sono i manifesti di configurazione di Postgres, fatto ciò applicare tutti i manifesti per la sua configurazione
```kubectl apply -f .```

#### Applicati i manifesti sarà posssibile accertarsi dello stato dei pod e porre gli stessi controlli su svc e pvc  
```kubectl get pods -o wide -n postgres-ns ```
```kubectl get pvc -n postgres-ns```
```kubectl get svc -n postgres-ns ```

#### Svolta la configurazione di postgres vi srà possibile accedere al pod mediante il comando 
```kubectl exec -it postgres-0 -n postgres-ns -- sh -c 'PGPASSWORD=tren7979  psql -U postgres -d mydatabase'```

#### Una volta svolta l'impostazione dei database nei ruoli desiderati sarà possibile mediante dei job inserire dei dati e porre una loro interrogazione 
#### Sposto verso il namespace creato
```kubectl config set-context --current --namespace=client-ns```

#### Applico il secret posta nella cartella destianta a tali manifesti 
```kubectl apply -f j-secret.yaml```

#### Ora sarà possibile applicare i due job 
```kubectl apply -f job.yaml ```
```kubectl apply -f job1.yaml ```

#### Applicate i due job al interno del namespace dedicato ad esso sarà possibile notare lo stato colmpleted dei due pod con il comado
```kubectl get pods```

#### In segui per notare al  meglio (gli id sono solo per l'esempio volta per volta che si applicano i manifesti dei job cambieranno pertano assicurarsi sempre di eseguire il comado precedente per assumere e corrette info)
```kubectl logs data-insert9ht90k ```
```kubectl logs data-checker90htk```

#### L'esecuzione del job data-checker esprime una richiesta di una serie di ID di articoli che sono presenti nel database primario di postgres la richiesta in primo luogo viene espressa nei confonti di redis che a cui apparterrano gli ultimi dati caricati in postgres se essi sarranno presenti in redis si potra notare il messaggio di cache HIT altrimenti 



#### Una volta appreso ed eseguito l'inserimento e l'interrogazione dei nostri database sarà possibile anche eseguire le stesse iterrogazioni anche nei database di Redis e POstgres (ovviamente si perga di eseguire l'accesso con i comadi eseguiti in precedenza e  porre le due seguenti query )

#### Query Redis  
```KEYS * ```

#### Query Postgresql  che mi assicura che la tabbella è stata creata 
``` \dt ```

#### Query Postgresql 
```SELECT * FROM articles;```

#### La configurazione proposta garantisce la resilienza dei dati. È possibile verificarla eliminando manualmente i pod:
```kubectl delete pod redis-0 -n redis-ns```
```kubectl delete pod postgres-0 -n postgres-ns```

#### Questa eliminazione manuale simula il mancato funzionamento di un pod in un ambiente di produzione, dimostrando i meccanismi di recupero automatico implementati.










