# Esercitazione 6.1: CNN (classificazione)

Lo scopo dell’esercitazione è utilizzare una convolutional neural network (CNN) per la classificazione di immagini biomediche. In particolare, il set di dati comprende immagini di risonanza magnetica cardiaca, acquisite in asse corto con diverse sequenze di acquisizione (Figura 6.e1). 

<img src="./images/Figura_ese_6_1.png" alt="Immagini MR cardiache" style="width:100%;">

*Figura 6.e1. Immagini MR cardiache.*

Le sequenze utilizzate mirano alla valutazione della perfusione cardiaca con primo passaggio del mezzo di contrasto, alla valutazione della funzione cardiaca (sequenza fast-cine), alla misura del valore di T2* con sequenze multi-echo, ed alla valutazione dell’infarto miocardico attraverso sequenze con contrasto ritardato (LGE). La dimensione delle immagini può essere diversa in dipendenza dalla sequenza MR utilizzata.

Nell’elaborazione delle immagini cardiache è necessario identificare la posizione della fetta in asse corto rispetto all’asse ventricolare, in particolare, ad ogni fetta deve essere assegnata una label (basale, media o apicale) in modo da poter “riempire” il modello di riferimento AHA (Figura 6.e2) mediando le informazioni delle varie fette appartenenti alla stessa macro-regione. Questa operazione può avvenire in due modi. Nel primo caso le tre fette vengono definite in fase di acquisizione dal tecnico radiologo (immagini di perfusione e T2*). Nel secondo caso vengono acquisite più fette (tipicamente 10-12) che coprono tutto il ventricolo e la localizzazione delle varie fette viene definita manualmente da un operatore esperto. Ambedue gli approcci sono operatore-dipendenti, per cui il reference sarà affetto da variabilità inter- e intra-osservatore. 

<img src="./images/Figura_ese_6_2.png" alt="Modello AHA a 16/17 segmenti" style="width:100%;">

*Figura 6.e2. Modello AHA a 16/17 segmenti.*

Quello che vogliamo è realizzare una CNN che riconosca in modo automatico la classe (basale, media, apicale) a cui appartengono le fette.     
La base di conoscenza è quindi formata da tre gruppi di immagini contenute in tre cartelle che ne definiscono la posizione. Le immagini sono classificate secondo la loro posizione lungo il ventricolo, come basali, medie e apicali, seguendo la classificazione standard AHA.  
Le immagini sono contenute nella cartella TEST_MR che contiene tre cartelle Base, Middle, Apical con le immagini DICOM corrispondenti. Quindi il reference come al solito, è definito dalla posizione delle immagini nelle cartelle. Il dataset comprende circa 250 immagini per ogni classe. 
Le immagini MR cardiache hanno tipicamente un FOV molto più grande del cuore, che è l’oggetto di interesse, per evitare la generazione di artefatti da ribaltamento. Quindi è opportuno personalizzare la funzione di lettura delle immagini in modo che comprenda una fase di pre-processing, in cui converrà eseguire un cropping su tutte le immagini in modo da isolare l’area centrale in cui è presente il cuore stesso. Inoltre, le immagini date in input alla rete devono avere tutte le stesse dimensioni, quindi dovremo eseguire un'interpolazione. Quindi in fase di pre-processing dobbiamo implementare le seguenti operazioni:

•	Lettura delle immagini in formato DICOM\
•	Cropping delle immagini DICOM in modo da includere solo il cuore in tutte le immagini\
•	Interpolazione in modo da ricondurre tutte le immagini alla stessa dimensione. 

Per l’operazione di cropping è opportuno ragionare in termini di dimensioni anatomiche, per cui definiremo in modo conservativo un’area in mm sulla base dell’anatomia cardiaca  (ad esempio 150x150 mm) e opereremo il cropping conoscendo la dimensione del pixel dai campi DICOM opportuni.
L’operazione di pre-processing potrebbe essere implementata attraverso un programma che carica le immagini originali e produce un nuovo dataset con le immagini elaborate. Questo approccio però ha alcuni importanti svantaggi:

•	Viene occupato spazio su disco, cosa che per database grandi può essere un grosso svantaggio.\
•	Se voglio cambiare i parametri di pre-processing (ad esempio la dimensione finale delle immagini) devo rigenerare tutto il database.

Per questi motivi tipicamente si adotta un approccio on-the-fly, nel quale il pre-processing viene effettuato ”al volo” quando le immagini vengono lette dal disco.   

La fase di pre-prorocessing si può implementare attraverso una funzione **transform** in **torchvision** e componendo con **compose** una serie di trasformazioni in monai, come visto nei notebook di esempio.

Definito il dataset come visto nei notebook di esempio, questo andrà diviso in una parte di training (la maggioranza delle immagini, ad esempio il 90%) ed una parte di test che verrà usata alla fine per verificare le prestazioni della rete (10%). Inoltre è opportuno crearsi un validation set che verrà usato nella fase di training (cross-validation) per validare iterativamente la rete. 
Avremo quindi una fase di tuning della rete utilizzando un singolo set di validazione, nel quale ottimizziamo gli iperparametri di addestramento, ed eseguiremo la fase di k-fold alla fine sul modello ottimizzato per verificare che le prestazioni sono indipendenti dai dati. Ci attendiamo un valore di accuratezza alto, intorno al 95%.

Nella fase di cross-validation useremo un approccio **k-fold** con k=5, quindi divideremo il dataset di training in k=5 parti ottenendo k set di validazione e k di training (**ConcatDataset**) ed effettueremo k addestramenti verificando che le k reti addestrate abbiano prestazioni comparabili sui k set di validazione.   
Dopo il k-fold sceglieremo uno dei 5 modelli addestrati (se la cross-validation dà un risultato positivo sono equivalenti) per l'analisi del test set. Tipicamente viene scelto il modello con prestazioni migliori o, più "onestamente", il mediano. 

Per la classificazione possiamo usare una rete VGG (Figura 6.e3) di cui dovremo definire gli iperparametri di configurazione (N, M, K). L'input sarà definito dalla funzione di pre-processing mentre l'output sarà su tre classi. 

<img src="./images/Figura_ese_6_3.png" alt="Esempio di rete VGG" style="width:100%;">

*Figura 6.e3. Esempio di rete VGG.*

Il modello base per la classificazione delle immagini è il modello VGG mostrato in figura, composto da una serie di strati convolutivi che riducono la dimensione dell’immagine fino ad arrivare allo stadio di classificazione. In generale è opportuno che la riduzione di risoluzione dall’input all’output avvenga in modo continuo attraverso blocchi POOL con lo stesso stride. La profondità degli strati CONV di solito è dell’ordine del numero di classi, è inutile avere profondità molto più grandi del numero di classi. Altezza e larghezza degli strati CONV corrispondono alla regione di immagine “vista” dalla rete, quindi dipendono dalla grandezza delle immagini e dall’ordine dello strato. Comunque, conviene usare più strati CONV in cascata (N>1) piuttosto che ingrandire troppo il singolo strato CONV. 

Una volta addestrato il modello, vanno misurate le prestazioni sul test set. In particolare, valuteremo l’accuratezza della classificazione dalla confusion matrix (Figura 6.e4) che misura la capacità della rete di riconoscere correttamente le classi. Dalla confusion matrix è possibile valutare la sensitività, specificità ed accuratezza della classificazione per le tre classi che sono i parametri clinici di riferimento. Ad esempio per le immagini apicali avremo:

VP = Fette apicali riconosciute come tali\
VN = Fette non apicali riconosciute come basali o medie\
FP = Fette non apicali riconosciute come apicali\
FN = Fette apicali non riconosciute come apicali \

<img src="./images/Figura_ese_6_4.png" alt="Valutazione di accuratezza, sensitività e specificità dalla confusion matrix" style="width:100%;">

*Figura 6e4. Valutazione di accuratezza, sensitività e specificità dalla confusion matrix.*

Questi valori si possono ricavare dalla confusion matrix come in figura. Clinicamente sono importanti i valori di sensitività e specificità:

Sensitività (Apice) = VP/(VP+FN)\ 
Specificità (Apice) = VN/(VN+FP)\
Accuratezza (Apice) = (VN+VP)/(VN+VP+FN+FP) 

Il valore di accuratezza atteso sul test set è maggiore del 90%

Infine, è utile visualizzare le immagini classificate in modo errato in modo da verificare il motivo di un'incorretta classificazione. Essendo il reference affetto da possibili errori dovuti alla variabilità inter- e intra-osservatore, è possibile in questo modo individuare eventuali errori o casi borderline nel reference.  
        

Come esperimento finale possiamo usare la rete addestrata su un “external test set”, in questo caso costituito da immagini cardiache CT con contrasto (EXTERNAL_TEST_CT). Questa prova ci consente di verificare la capacità della rete di generalizzare la classificazione ad altre modalità di imaging.   








