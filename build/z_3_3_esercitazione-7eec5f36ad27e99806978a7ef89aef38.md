# Esercitazione 5: segmentazione del grasso sottocutaneo e viscerale

## Contesto clinico
Obesità e sovrappeso sono universalmente riconosciuti come fattori di rischio per le principali malattie croniche: malattie cardiovascolari, ictus, diabete di tipo 2, alcuni tumori (endometriale, colon-rettale, renale, della colecisti e della mammella in post-menopausa), malattie della colecisti, osteoartriti. Altri problemi associati a un eccesso di grasso corporeo sono: ipertensione, ipercolesterolemia, apnea notturna e problemi respiratori, asma, aumento del rischio chirurgico, complicanze in gravidanza, irsutismo e irregolarità mestruali. L'obesità è il principale fattore di rischio nei paesi occidentali, la più importante causa indiretta di morte e di invalidità non accidentale. L’insieme dei disturbi associati all’obesità o al sovrappeso prende in nome di sindrome metabolica.
L’eccesso di tessuto adiposo in particolari sedi anatomiche rappresenta un indice di rischio rilevante per diverse patologie. In particolare, l’accumulo di grasso a livello intra-addominale è correlato con un rischio elevato di sviluppare malattie cardiovascolari e disturbi metabolici. È quindi importante sviluppare tecniche di imaging che siano in grado di misurare con precisione la massa grassa e di valutarne in modo quantitativo la distribuzione.
Il tessuto adiposo può essere individuato e misurato con varie tecniche, che possono essere suddivise in tecniche dirette e tecniche indirette. Le tecniche indirette di più comune utilizzo sono: 
1. l’indice di massa corporea (BMI) che si ottiene dividendo il peso corporeo per l’altezza al quadrato e fornisce indicazioni sulla presenza di eventuali alterazioni del peso corporeo, dovute tipicamente ad un eccesso di grasso. L’OMS fornisce una tabella (per dire la verità abbastanza conservativa) che consente sulla base del BMI di valutare la presenza di uno stato patologico associato al peso corporeo. Il computo del BMI richiede l’uso solo di una bilancia e di un metro lineare. 
2. la misura della circonferenza della vita e il rapporto tra la circonferenza della vita e quella dei fianchi (specifica per valutare la quantità di grasso addominale). 
3. La plicometria, che stima la percentuale di grasso corporeo derivandola dallo spessore delle pliche cutanee a livello del bicipite, del tricipite, della zona sottoscapolare e soprailiaca; la plicometria richiede l’uso di un apposito strumento detto plicometro. 
4. l’analisi dell’impedenza bioelettrica attraverso bilance impedenziometriche, ormai disponibili anche a costi molto contenuti per quanto la precisione della misura in strumenti economici può essere limitata. La bioimpedenziometria si basa sostanzialmente sulla misura della resistenza elettrica del corpo, che viene ridotta dalla presenza di grasso. Le tecniche bioimpedenziometriche includono evidentemente un modello semplificato del corpo umano che rende la misura intrinsecamente imprecisa. 

Le tecniche indirette risultano adeguate per la maggior parte delle esigenze cliniche, tuttavia non consentono di verificare la localizzazione dei depositi di grasso. Per raggiungere questo obiettivo sono necessarie tecniche dirette che sono tipicamente tecniche di imaging. Le principali tecniche dirette sono:
1. La DEXA (Dual Energy X-ray Absorbitiometry) è una modalità di imaging che permette la valutazione del contenuto minerale osseo (BMC), della massa di tessuto magro (LTM) e della massa di tessuto adiposo (FM). La tecnica è basata sul principio fisico secondo il quale raggi X a differenti energie sono attenuati in modo diverso quando attraversano il corpo umano. Ha un ruolo dominante in diagnosi di osteopenia e osteoporosi ed è usata in un largo spettro di studi e ricerche mediche, anche grazie al limitato rischio radiologico legato alle basse energie utilizzate. La DEXA è una modalità di tipo radiologico non tomografico (proiettiva), e quindi non permette di valutare la distribuzione volumetrica dei depositi di grasso. 
2. La TAC o CT (Computed Tomography) è una modalità di imaging molto veloce con un’accuratissima discriminazione tra i vari tessuti, che in base alla loro natura, sono individuati da un numero di Hounsfield. Tuttavia l’esposizione a radiazioni ionizzanti rende questa tecnica poco utilizzabile in studi a lungo termine, studi su bambini e adolescenti. Nella CT il grasso avendo un assorbimento inferiore appare più scuro del tessuto muscolare. 
3. La Risonanza Magnetica è il metodo di elezione per la misura diretta della distribuzione del grasso corporeo, soprattutto per il fatto di non presentare un rischio radiologico. Il grasso e l’acqua si diversificano nelle immagini di risonanza, perché, pur contenendo entrambi molti nuclei di idrogeno, acqua e grasso hanno tempi di rilassamento abbastanza diversi. In particolare, il grasso ha un T1 più lungo rispetto a quello degli altri tessuti e dei liquidi corporei e produce quindi un segnale di alta intensità rispetto agli altri tessuti utilizzando un tempo di eco elevato. Le immagini MR per la valutazione del grasso sono quindi tipicamente pesate T1. Sono poi disponibili sequenze MR specializzate (DIXON, IDEAL) basate sul fenomeno del chemical shift che permettono di separare il grasso dagli altri tessuti. 

Una immagine addominale MR T1w è del tipo in Figura 3.63. Sono evidenziati il Tessuto Adiposo Sottocutaneo (SAT) e il Tessuto Adiposo Viscerale (VAT). Poiché il metabolismo dei due depositi differisce è importante la loro quantizzazione separata.

<img src="./images/image-63.png" alt="esempio-vat-sat" style="width:100%;">

*Figura 3.63. Immagine addominale MR T1w con evidenziati il tessuto adiposo sottocutaneo (SAT) ed il tessuto adiposo viscerale (VAT).*
 
Tipicamente viene acquisito un volume 3D composto da una sequenza di fette assiali che coprono tutto l’addome del paziente. Le immagini acquisite vengono elaborate in modo da identificare il volume complessivo di SAT e VAT ed il loro rapporto rispetto al volume addominale complessivo.
Il problema della segmentazione del grasso addominale costituisce un esempio tipico dei problemi che si incontrano nello sviluppo di algoritmi utili al supporto della pratica clinica. Infatti, abbiamo due tessuti (SAT e VAT) che hanno lo stesso livello di segnale ma sono topologicamente distinti. La struttura dei due tessuti è profondamente diversa, in quanto il SAT è un tessuto compatto con una forma ben definita (approssimabile ad un anello o una struttura toroidale in 3D) mentre il VAT è una struttura ramificata non necessariamente connessa. SAT e VAT possono essere connessi tra loro, almeno al livello di risoluzione delle immagini MR. Esistono vari metodi in uso nella pratica clinica e scientifica (Bliton 2017). La procedura di segmentazione classica può essere schematizzata con la flow-chart seguente (Positano V, JMRI 2004) (Figura 3.64).

<img src="./images/image-64.png" alt="flowchart" style="width:100%;">

*Figura 3.64. Flowchart per la segmentazione di SAT e VAT.*
 
Il primo passo nella segmentazione è separare i tre componenti principali dell'immagine, cioè lo sfondo (aria), il grasso e gli altri tessuti che presentano il tipico segnale dell'acqua. In dipendenza dal FOV usato nel campo di vista possono comparire le braccia del paziente che devono essere eliminate dall'analisi. Questo passo può essere effettuato attraverso un algoritmo di clustering *k-means* o *FCM*. Imponendo $K=3$ (numero di cluster) si ottengono le mappe di distribuzione dei tessuti in figura, dove (a) è l'immagine originale, (b) la mappa dell'aria, (c) la mappa dell'acqua e (d) la mappa del grasso.

Per eliminare la regione delle braccia di può applicare un algoritmo di labeling alla maschera dello sfondo, dopo averla binarizzata con un filtro a soglia se ottenuta con l’algoritmo *FCM* che produce mappe di appartenenza non binarie ed invertita. Con un algoritmo di labeling si estrae la parte del tronco che è quella di maggiori dimensioni. Otteniamo così la maschera della regione addominale.

Eliminate le braccia, si procede quindi alla segmentazione del grasso subcutaneo (SAT). Si definisce un contorno circolare che sia esterno al tronco e si utilizza la mappa dell'aria invertita o quella del grasso come campo di forze esterne. Il contorno settato con parametri opportuni verrà attirato sul bordo esterno del SAT. A questo punto si crea un nuovo contorno, duplicando il contorno ottenuto nel passo precedente eventualmente rimpicciolendolo per facilitare la convergenza. In questo secondo passaggio si utilizza come mappa delle forze esterne la mappa del grasso. Il contorno convergerà al bordo interno del SAT. Nel caso di utilizzo di un algoritmo di tipo Snake, dovrà essere usato un parametro alfa alto per favorire la convergenza all'interno e un parametro beta alto per evitare che lo snake debordi nel VAT. In questo modo è facile ottenere la misura del grasso subcutaneo (SAT). E' sufficiente calcolare il numero di pixel contenuti tra il contorno esterno e interno del SAT (tipicamente sottraendo le due maschere) e moltiplicarlo per il volume di un voxel ottenuto dai campi DICOM.

A questo punto sottraendo la maschera del SAT dalla maschera del grasso addominale potremmo ottenere la maschera del VAT e quindi la misura del VAT stesso. Tuttavia, questa soluzione non è opportuna a causa dell’effetto volume parziale, che è molto rilevante nel VAT a causa della sua struttura complessa e molto variabile da paziente a paziente a causa della struttura paziente-specifica del VAT. Inoltre all’interno della maschera del VAT possono comparire strutture con segnale molto simile ma non assegnabili al VAT. E quindi opportuno utilizzare un algoritmo che comprenda un modello del segnale atteso dal VAT. Si estraggono quindi i pixel all'interno del secondo contorno definito nella valutazione del SAT e si costruisce l'istogramma (Figura 3.65).

<img src="./images/image-65.png" alt="istogramma" style="width:100%;">

*Figura 3.65. Istogramma per la stima del VAT.*

L'istogramma avrà due picchi, uno a sinistra che rappresenta il segnale dell'acqua e uno a destra che rappresenta il segnale del grasso che è quello che ci interessa. L'istogramma viene fittato con due gaussiane (in Figura 3.65 è visualizzata solo quella del grasso) attraverso un algoritmo *EM-GMM* e il numero di pixel del VAT viene valutato come area della seconda gaussiana.

Nel caso di immagini CT la procedura è analoga, con la differenza che essendo in CT il segnale del grasso inferiore a quello dell’acqua il picco da individuare nell’analisi del VAT sarà quello di sinistra.

## Esercitazione
I dati dell’esercitazione sono rappresentati da una fetta assiale CT che descrive l’addome di un paziente affetto da obesità (Figura 3.66). I risultati attesi dell’elaborazione sono la superfice del $SAT$, la superfice del $VAT$, il loro rapporto percentuale $100*VAT/SAT$ che è un indicatore del rischio clinico e la percentuale totale di grasso $P=100*(VAT+SAT)/VOL$ dove $VOL$ è la superfice totale dell’addome. l’analisi di una singola fetta invece che il volume addominale è accettabile perché esiste una forte correlazione tra la misura effettuata su una singola slice a livello ombelicale ed un’analisi volumetrica, anche se sarebbe sempre preferibile effettuare un’analisi in 3D.

<img src="./images/image-66.png" alt="esercitazione" style="width:100%;">

*Figura 3.66. Fetta assiale CT.*

Nell’immagine CT si riconosce il lettino CT, che va evidentemente escluso dall’analisi, e la regione della colonna con un segnale alto rispetto al grasso ed al tessuto addominale (segnale dell’acqua).

Le immagini CT sono tipicamente codificate come valori di Hounsfield (HU). Nelle immagini CT il segnale del grasso ha valore H intorno a $-100$ mentre i segnale dell’acqua è circa $50$. Il fondo ha il segnale tipico dell’aria $-1000$ mentre le regioni di zero padding assumono il valore convenzionale $HU=-3024$.

Anche se sarebbe possibile memorizzare valori negativi nel DICOM utilizzando un formato “signed 16-bit integer, tipicamente per le immagini CT il valore del livello di grigio è memorizzato come “unsigned 16 bit integer” (interi positivi) e vengono utilizzati due campi del DICOM per codificare i valori $HU$. Tali campi sono: 

tag|description
---|---
0028,1052 |Rescale Intercept 
0028,1053 |Rescale Slope 

Se questi campi sono presenti, una libreria DICOM correttamente configurata leggerà i valori interi positivi dell’immagine I e calcolerà i valori corretti Ic come: 

$$
Ic = RescaleIntercept + RescaleSlope*I 
$$

In pratica viene definita una funzione lineare che implementa una hash table che converte valori interi positivi a 16 bit (quelli del DICOM) in un qualsiasi range di uscita. Il campo RescaleType se presente ci fornisce il tipo di rescaling utilizzato, se assente si presume che venga applicata la scala HU. 

La funzione `dicomread` di MATLAB ignora erroneamente questi campi (almeno fino alla versione
2021b); pertanto, è preliminarmente necessario implementare la conversione corretta
utilizzando i campi indicati in precedenza, in modo da ottenere un array contenente i valori
HU corretti. È inoltre opportuno convertire il tipo di dato da `int16` a `float` (double)
per le elaborazioni successive. In Python, la lettura dei file DICOM può essere effettuata con la libreria **pydicom**.
La conversione in unità Hounsfield (HU) può essere ottenuta utilizzando i campi
`RescaleSlope` e `RescaleIntercept`, e successivamente convertendo l’array in `float64`
per le elaborazioni successive.

Analogamente a quanto visto per immagini MR, la procedura di analisi delle immagini CT dovrà tipicamente includere i seguenti passi:

1. Import dell’immagine CT e conversione in valori HU. Essendo presente una regione di zero padding (la regione non ricostruita, valore $-3024$) a valore noto, è opportuno escluderla per evitare di dover inserire un altro cluster nell’analisi *FCM*. Ad esempio possiamo porre i pixel di zero padding uguali al valore dell’aria $(-1000)$ o escluderli dal clustering 
2. Applicazione dell’algoritmo *FCM* all’immagine per identificare i pixel appartenenti all’aria, acqua e grasso (3 cluster). Ci aspettiamo un risultato come quello di Figura 3.67, in cui i pixel della colonna hanno un valore di appartenenza “ibrido”.

<img src="./images/image-67.png" alt="fcm" style="width:100%;">

*Figura 3.67. Risultato atteso da algoritmo FCM.*

3. Escludere dall’immagine il lettino CT, analogamente a quanto previsto per le braccia in MR, con un algoritmo di labeling che operi sull’inverso della mappa dell’aria. 
4. Definire automaticamente un contorno seed esterno all’addome (ad esempio un cerchio iscritto nell’immagine). Con un algoritmo a contorni attivi segmentare il bordo esterno del $SAT$ utilizzando la mappa del grasso. 
5. Partendo dal contorno esterno del $SAT$ usato come seed, con un algoritmo a contorni attivi segmentare il bordo interno del $SAT$. Come guida per l’algoritmo si può usare la mappa del grasso e quella dell’acqua. Potrebbe essere utile “rimpicciolire” il seed iniziale con imerode o funzioni simili (ad esempio un filtro di thinning). Dovremmo ottenere dei contorni simili a queli di Figura 3.68.

<img src="./images/image-68.png" alt="contorni" style="width:100%;">

*Figura 3.68. Risultato atteso contorni attivi.*

6.	Calcolare la superfice del $SAT$ in $cm^2$ come sottrazione del maschere del contorno $SAT$ esterno e interno. 
7.	Calcolare il volume del $VAT$ applicando un algoritmo *GMM* alla porzione di immagine interna al contorno interno del $SAT$. All’interno della regione avremo due picchi principali di segnale (grasso e acqua) ed un picco secondario (tessuto osseo della colonna). E’ da valutare se per la convergenza del *GMM* sia opportuno usare due gaussiane o tre. 
8.	Calcolare il volume del $VAT$ in $cm^2$ dai parametri del *GMM* e quindi il rapporto $VAT/SAT$ e grasso/addome. I risultati di massima sono: 

$SAT$ circa $290 cm^2$, $VAT$ circa $140 cm^2$, $VAT/SAT$ circa $50\%$, grasso/addome circa $60\%$.
    
