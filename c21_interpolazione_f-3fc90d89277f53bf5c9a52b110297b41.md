# Capitolo 2: Interpolazione, filtraggio e compressione
Come visto in precedenza, il primo passo dopo l’acquisizione dell’immagine biomedica è l’esecuzione di operazioni a basso livello che hanno lo scopo di migliorare l’immagine. Queste operazioni corrispondono al secondo livello nella classificazione degli algoritmi di computer vision.

## Interpolazione
Una immagine biomedica ha dimensioni definite a livello di acquisizione o del processo di ricostruzione implementato nello scanner. Le dimensioni sono tipicamente legate alla risoluzione spaziale, nel senso che a parità di campo di vista (FOV), modalità con una migliore risoluzione spaziale saranno in grado di acquisire pixel di dimensioni minori e quindi la dimensione dell’immagine sarà maggiore. Le dimensioni tipiche di una immagine radiologica possono andare da 64x64 (immagini SPECT) a 256x256 (MR cardiaca) fino a 512x512 (CT e MRI) o 1024x1024 (CT). La radiografia classica ha risoluzioni più alte, fino a 5000x5000 per la mammografia.  
Nell’elaborazione delle immagini biomediche, può essere necessario cambiare la dimensione delle immagini, aumentando o diminuendo il numero di pixel componenti (Figura 2.1).

Un tipico esempio è la visualizzazione dell’immagine su di uno schermo per la visualizzazione e refertazione. Tipicamente la dimensione di una immagine radiologica è minore (o comunque diversa) della dimensione del supporto digitale (stampa o schermo) su cui deve essere visualizzata. Un monitor tipico per refertazione radiologica ha risoluzione di 1600x1200 pixel, che è comunque una risoluzione usuale anche nei monitor di uso generale. È necessario quindi interpolare l’immagine in modo da permetterne la visualizzazione in maniera corretta adattando le dimensioni delle immagini a quelle dello schermo (o della porzione di schermo) in cui vanno visualizzate. 
Un altro esempio, come sarà visto nel seguito, è il cambio di risoluzione spaziale nelle immagini 3D per ottenere dei voxel cubici e quindi delle immagini isotrope.
Come già visto in precedenza, l’immagine biomedica può essere vista come una serie di campioni di un processo fisico acquisiti in uno spazio discreto, la cui risoluzione è uguale alla dimensione del pixel. In pratica l’immagine ci fornisce i campioni in NxM locazioni spaziali poste al centro dei pixel dell’immagine (seguendo la convenzione DICOM). 
Consideriamo l’immagine in Figura 2.1 con FOV 340x340 mm e dimensioni 256x256 e il suo ingrandimento a destra. Il pixel size è 340/256=1.32 mm, e questo è anche l’intervallo di campionamento spaziale. 
Ogni pixel corrisponde ad un punto della griglia (le crocette in figura). Se vogliamo aumentare il numero di righe e colonne dell’immagine dobbiamo aumentare la risoluzione della griglia. Supponiamo di raddoppiare la risoluzione interpolando l’immagine a 512x512.
 
<img src="./images/image-1.png" alt="Interpolazione" style="width:100%;">

*Figura 2.1. L'operazione di interpolazione prevede di stimare il valore dell'intensità del pixel su una nuova grigila.*

In base alla nuova griglia dovremo definire dei nuovi pixel nella posizione corrispondente ai nuovi punti della griglia. A seconda della procedura usata per definire dei nuovi valori avremo vari metodi di interpolazione. Tipicamente il nuovo valore dei pixel verrà dedotto da quello dei pixel vicini, nell'esempio di  Figura 2.1 dobbiamo ricampionare l’immagine sulla griglia rossa utilizzando i valori dei pixel originali (in bianco). Tipicamente per ottimizzare il tempo di elaborazione si utilizzano solo i pixel più “vicini” al pixel da ricomputare.    

### Nearest neighbor
Un primo approccio (NN, nearest neighbor)  assegna al nuovo pixel il valore del pixel più vicino (Figura 2.2). Il metodo è molto veloce e ha il vantaggio di non introdurre nuovi livelli di grigio, cosa che può essere rilevante da un punto di vista diagnostico. D’altra parte, il metodo NN tende a creare dei gruppi di pixel con lo stesso valore, dando l’impressione visiva di una immagine “pixellata”. 

<img src="./images/image-2.png" alt="InterpolazioneNN" style="width:100%;">

*Figura 2.2. Interpolazione NN.*

### Interpolazione bilineare
Un metodo più perfezionato è la cosiddetta interpolazione bilineare (Figura 2.3), nella quale il nuovo valore del pixel viene stimato come la media pesata dei valori dei 4 (o 8 in 3D) pixel più vicini. Il valore del pixel nel punto $P$ è la somma dei valori dei pixel in $Q_{ij}$ pesati per l’area normalizzata del rettangolo opposto a $Q_{ij}$. 

<img src="./images/image-3.png" alt="Interpolazione_bilineare" style="width:100%;">

*Figura 2.3. Interpolazione bilineare.*

Formalmente si ha: 

$$
\begin{aligned}
f(x,y) \approx {} &
\frac{f(Q_{11})}{(x_2 - x_1)(y_2 - y_1)} (x_2 - x)(y_2 - y) \\
&+ \frac{f(Q_{21})}{(x_2 - x_1)(y_2 - y_1)} (x - x_1)(y_2 - y) \\
&+ \frac{f(Q_{12})}{(x_2 - x_1)(y_2 - y_1)} (x_2 - x)(y - y_1) \\
&+ \frac{f(Q_{22})}{(x_2 - x_1)(y_2 - y_1)} (x - x_1)(y - y_1).
\end{aligned}
$$

L’interpolazione bilineare offre solitamente una qualità visiva migliore dell’interpolazione NN. 
L’interpolazione bilineare può essere vista come una stima del valore atteso del segnale in $P$ modellando la transizione del segnale tra pixel come una retta ed utilizzando solo i due campioni più vicini per stimare i parametri della retta stessa. 

### Interpolazione bicubica e spline
L’idea può essere estesa utilizzando modelli non lineari (ad esempio funzioni polinomiali di grado crescente o funzioni sinusoidali) ed aumentando il numero di campioni utile a stimare dette funzioni. Un esempio è l’interpolazione bicubica (bicubic interpolation) che utilizza polinomi di terzo grado e una griglia di 16 pixel.  
Esistono poi metodi ancora più perfezionati (spline) in cui l’intera distribuzione dei pixel sull’immagine viene modellata come una superficie ed i valori interpolanti ottenuti di conseguenza.
Nella visualizzazione di immagini mediche, oltre alla qualità visiva percepita dell’interpolazione è importante anche il tempo di elaborazione, in quanto il sistema software deve rispondere in tempo reale ai comandi dell’utente, ad esempio quando viene effettuato uno zoom.   

### Reslicing
Il processo di interpolazione è di particolare rilevanza nell’elaborazione di immagini 3D, nelle quali tipicamente la distanza inter-fetta (asse z) e maggiore della risoluzione del pixel sulla singola fetta (piano x-y). In questo caso per ottenere un volume isotropo (cioè con risoluzione spaziale uguale su tutti gli assi) è necessario eseguire una operazione di interpolazione 3D che crei delle nuove slice con distanza inter-fetta uguale alla risoluzione del pixel. Tale operazione è detta reslicing (Figura 2.4). L’operazione di interpolazione deve mantenere costante il campo di vista (FOV), cioè la regione di spazio coperta dal volume.

<img src="./images/image-4.png" alt="Reslicing" style="width:100%;">

*Figura 2.4. Reslicing.*

Supponiamo di avere un volume definito da $Nz$ fette con un $FOV = [dimX, dimY, dimZ]$, dove \
$dimX = dx Nx$   (pixel spacing x colonne) \
$dimY = dy Ny$	 (pixel spacing x righe) \
$dimZ = dz Nz$	 (slice spacing x numero slice)	

se $dz > dx=dy$ come avviene di solito, dobbiamo creare delle nuove slice con il reslicing in modo da avere la distanza inter-slice $dz1$ del nuovo volume uguale a $dx/dy$ senza cambiare il $FOV$. Quindi:

$dimZ = dz Nz = dz1 Nz1 = dx Nz1$ 

e quindi   

$Nz1 = dz Nz/dx$ 
 
Il reslicing viene anche utilizzato per ricostruire piani di vista diversi da quelli acquisiti, ad esempio per ottenere un piano sagittale, coronale o obliquo da una acquisizione assiale come viene fatto usualmente nella TAC.
Si noti che gli algoritmi standard di interpolazione presenti nei software generici dedicati alla elaborazione di immagini (come Matlab/python) presuppongono che i pixel dell’immagine siano quadrati e i voxel cubici. Nel caso di immagini biomediche questa assunzione può essere errata, specialmente nel caso di immagini 3D. Il processo di interpolazione deve quindi tener conto della posizione reale dei pixel o voxel nello spazio, cosa che si può fare accedendo ai campi DICOM opportuni.

In MATLAB è possibile utilizzare la funzione `interp3` in combinazione con la funzione `meshgrid`. La funzione `interp3` permette anche di scegliere il metodo di interpolazione da utilizzare attraverso opportune opzioni.
Nel caso di immagini isotrope e fattore di interpolazione uguale per tutti gli assi è possibile utilizzare le funzioni `imresize` e `imresize3`.  

In Python è possibile utilizzare la funzione `interpn` della libreria **SciPy** (Notebook XXX). 

La funzione `interpn` permette anche di scegliere il metodo di interpolazione da utilizzare attraverso opportune opzioni.
Nel caso di immagini isotrope e fattore di interpolazione uguale per tutti gli assi è possibile utilizzare la funzione `RegularGridInterpolator`.  

Come detto in precedenza un’applicazione tipica degli algoritmi di interpolazione è l’estrazione da volumi di dati 3D di una immagine con orientamento diverso da quello di acquisizione (reslicing). Nella visualizzazione triplanar (proiezione ortogonale) dal volume dei dati vengono estratti tre piani tra loro perpendicolari attraverso una operazione di interpolazione. L’origine dei tre piani può essere definita in modo interattivo dall’operatore, che così può esplorare il volume dei dati. La visualizzazione triplanar è tipicamente disponibile su tutte le workstation per l’analisi di bioimmagini. Un esempio è riporato in Figura 2.5. 

<img src="./images/image-5.png" alt="triplanar_view" style="width:100%;">

*Figura 2.5. Triplanar view di un fantoccio ottenuta con imagej.*

In python la visualizzazione triplanar è implementata nella libreria **napari**. 
 
## Filtraggio dell’immagine biomedica
Gli algoritmi di filtraggio consentono l’elaborazione dell’immagine biomedica in modo da modificarne le caratteristiche. Non è banale definire in modo univoco cosa si intende per operazione di filtraggio dal punto di vista radiologico. Seguendo l’approccio della computer vision, possiamo definire filtraggio una operazione che cambia il contenuto informativo dell’immagine senza estrarre informazioni topologiche (in questo differente dal processo di segmentazione). Quindi un filtro potrà evidenziare i contorni dell’immagine, mentre un processo di segmentazione convertirà i contorni evidenziati in una entità geometrica, come una curva o una maschera. 
Il filtraggio ha essenzialmente due possibili scopi:

1. Migliorare la visualizzazione dell’immagine per ottimizzare l’analisi visiva della stessa.
2. Predisporre l’immagine per ottimizzare una elaborazione successiva, tipicamente una segmentazione.

Per quanto riguarda il secondo punto, l’operazione di filtraggio avviene tipicamente in modo trasparente rispetto all’utente, che osserva solo il risultato dell’operazione successiva che rappresenta il fine del processo di elaborazione. Il primo caso è invece critico rispetto al processo diagnostico, in quanto un errore nell’algoritmo di filtraggio può causare la perdita di informazione utile e fini clinici e quindi indurre un errore diagnostico. L’introduzione di algoritmi di filtraggio in fase diagnostica è quindi effettuato con estrema cautela.    

Le operazioni di filtraggio sull’immagine si possono classificare in tre categorie principali:

1. Puntuali: Un’operazione puntuale opera solo sul singolo pixel dell’immagine, trasformandolo in base ad una funzione $g(v)$ dove $v$ è il valore dell’intensità del pixel. Esempio di operazioni puntuali sono il negativo dell’immagine e in generale le lookup table compresa l’operazione di windowing e l’applicazione della gamma function. In una operazione puntuale il valore del pixel sull’immagine filtrata dipende solo dal valore dello stesso pixel nell’immagine originale.
2. Locali: una operazione locale elabora un pixel in base al valore del pixel stesso e dei pixel circostanti. Esempio di operazione locale è la convoluzione spaziale che vedremo in dettaglio nel seguito.
3. Globali: Le operazioni globali operano sull’immagine nel suo complesso. Esempio di operazioni globali sono le operazioni sull’istogramma, come l’equalizzazione.

### Operazioni puntuali
Le operazioni puntuali sono equivalenti ad una trasformazione dei livelli di grigio. In pratica un livello di grigio $v$ dell’immagine viene trasformato in un nuovo livello u secondo una trasformazione $u=t(v)$. $t$ può essere utilmente codificata in una lookup-table, cioè una tabella con una sola colonna ed un numero di righe uguale alla profondità dell’immagine. Il vantaggio della lookup-table è che il filtraggio può essere eseguito in modo molto efficiente da un punto di vista computazionale. Una volta definita la tabella come un array $T$ di $N$ elementi dove $N$ sono i possibili valori che può assumere $v$, il valore di $u$ sarà semplicemente $u=T(v)$, e quindi il nuovo valore di $u$ verrà computato senza nessuna operazione algebrica ma attraverso un semplice accesso in memoria. Le color map per la visualizzazione a falsi colori sono un esempio di una lookup-table a tre colonne dove la tripletta $(R,G,B)$ è data da $(R,G,B) = LT(:,v)$ dove $v$ è il livello di grigio. 
  
Esempi di trasformazioni puntuali sono (si consideri una immagine a 256 livelli): 
1. l’inversione dei livelli di grigio $u = (255-v)$
2. il Windowing 
3. le operazioni di tipo gamma $u=v^{\gamma}$
4. la binarizzazione. Viene decisa una soglia T e l’immagine viene ridotta ad una immagine bimodale ($u = A$ se $v>T$, $u = B$ se $v <T$)

La Figura 2.6 riassume alcune operazioni di filtraggio puntuale. Come si osserva, una operazione di filtraggio puntuale può essere sempre espressa come una curva (non necessariamente continua) in un piano bidimensionale con due assi di dimensioni pari alla profondità dell’immagine originale (in questo caso 255, 8 bit) e dell’immagine filtrata (in questo caso normalizzata a 1).   

<img src="./images/image-6.png" alt="filtraggio_puntuale" style="width:100%;">

*Figura 2.6. Esempi di filtri locali puntuali.*

#### Windowing
Esaminiamo in dettaglio l’operazione di Windowing che è di cruciale importanza nella visualizzazione delle immagini biomediche. Una immagine biomedica è in generale codificata su 16 bit, cioè ogni pixel dell’immagine può assumere $2^{16}= 65536$ valori, corrispondenti ad altrettanti livelli di grigio. Di fatto il numero di livelli che vengono utilizzati è più basso, ad esempio nella risonanza magnetica solitamente qualche migliaio. Ad ogni modo, gli strumenti di visualizzazione utilizzati nella pratica clinica, come schermi video tradizionali o LCD, sono in grado di visualizzare solo 256 livelli di grigio. D’altra parte, risoluzioni superiori sarebbero inutili in quanto l’occhio umano è in grado di distinguere un numero limitato di livelli di grigio contemporaneamente, dell’ordine di qualche decina. Poiché l’occhio è invece capace di separare un numero molto maggiore di colori diversi, la rappresentazione delle immagini in falsi colori, cioè facendo corrispondere ad ogni livello di grigio un determinato colore attraverso una mappa prestabilita può essere utilizzata per caratterizzare immagini mediche. Questa tecnica trova però applicazione solo in medicina nucleare e nella creazione di mappe parametriche.
Tutti gli strumenti di visualizzazione devono quindi affrontare il problema di rappresentare un numero di livelli di grigio superiore a quello che lo strumento di visualizzazione può sostenere. Le immagini visualizzate o stampate sono quindi fortemente caratterizzate dal modo in cui questo problema è risolto, ed essendo la soluzione del problema tutt’altro che univoca, bisogna tener conto del fatto che ogni immagine stampata o visualizzata su schermo può rappresentare solo parzialmente l’informazione globale contenuta nei dati effettivamente acquisiti dallo scanner.

Il processo mediante il quale i livelli di grigio dell’immagine reale vengono rappresentate sullo schermo o stampate viene solitamente detto windowing. I pixel che giacciono in un certo intervallo di valori (una finestra o window), che può essere impostato dall’utente, vengono rappresentati utilizzando tutti i livelli disponibili (tipicamente 256). I valori al di sotto della finestra selezionata vengono tutti posti a zero, e verranno quindi rappresentati in nero. A tutti i valori superiori verrà invece assegnato il valore massimo visualizzabile (tipicamente 255) e quindi essi appariranno bianchi. La Figura 2.7 mostra un esempio delle differenze di visualizzazione ottenibili attraverso diverse scelte della finestra da utilizzare. Nell’immagine superiore i livelli di grigio vengono mappati in modo proporzionale. Quindi i livelli visualizzati (da 0 a 255, asse y) vengono ottenuti normalizzando i valori originali rispetto al valor massimo dell’immagine originale. Nell’immagine in basso è stata effettuata una opportuna operazione di windowing per visualizzare al meglio la zona di interesse. Si può notare come alcune zone abbiano perso notevolmente di risoluzione a causa dello schiacciamento dei livelli di grigio relativi.    

<img src="./images/image-7.png" alt="windowing" style="width:100%;">

*Figura 2.7. Effetto dell’operazione di windowing sulla visualizzazione di una immagine.*

L’operazione di Windowing è implementata in tutte le stazioni radiografiche e viene tipicamente pilotata dal movimento del mouse. Ad esempio il movimento orizzontale del mouse può pilotare la larghezza della finestra e il movimento verticale la posizione del centro della finestra sull’asse x. Comunque il metodo con cui viene implementato il filtraggio è in realtà molto raffinato e mira a permettere una regolazione del windowing veloce ma allo stesso tempo precisa. La transizione dal livello 0 al livello massimo della finestra non è necessariamente lineare, ma può assumere varie forme. Il formato DICOM contiene alcuni parametri che pilotano l’operazione di windowing. 

Group Element	| Title	| Esempio
---|---|---
[0028-0106]	| Smallest Image Pixel Value	| 0
[0028-0107]	| Largest Image Pixel Value	| 4177
[0028-0150]	| Window Centre	| 163
[0028-0151]	| Window Width	| 327
   
I primi due parametri, “Smallest Image Pixel Value” e “Largest Image Pixel Value” permettono di leggere il massimo e minimo valore dell’immagine e di creare una funzione di windowing lineare, evitando di comprendere nell’operazione di windowing valori al di fuori del campo di definizione dell’immagine, anche se compresi nella massima profondità teorica. I parametri successivi “Window Centre” e “Window Width” definiscono una finestra di Windowing da utilizzare nella visualizzazione dell’immagine. Questa finestra viene tipicamente salvata nel DICOM al momento dell’acquisizione in base ad una ottimizzazione fatta dal produttore. Il programma di visualizzazione leggerà dal DICOM la finestra e produrrà una visualizzazione iniziale dell’immagine che l’utente potrà poi modificare.
Come illustrato in Figura 2.8, l’operazione di Windowing per quanto semplice in via di principio è estremamente importante nella pratica clinica, e necessita quindi di una accurata implementazione.
Vale la pena di notare che le lastre radiografiche tradizionali o le immagini digitali in formato standard (jpeg, tiff) allegate ad un referto elettronico rappresentano il prodotto di una precisa operazione di Windowing eseguita dal medico refertante, con l’obiettivo di fornire la migliore informazione iconografica al paziente ed al medico inviante. Esse non hanno però valore legale, in quanto l’operazione di Windowing può cancellare alcune componenti delle immagini se eseguita in modo inappropriato. L’unica fonte certa sono quindi le immagini digitali in formato DICOM, che vengono quindi immagazzinate e sempre più spesso consegnate al paziente su supporto digitale (tipicamente un DVD contenente un visualizzatore DICOM).     
 
<img src="./images/image-8.png" alt="windowing_book" style="width:100%;">

*Figura 2.8 (tratto da R Rangayyan, Biomedical Image Analysis, CRC Press 2004); esempio di Windowing a fini diagnostici su immagini CT.*

### Operazioni locali e filtraggio spaziale
L’operazione di base nel filtraggio locale delle immagini è l’operazione di convoluzione spaziale detta anche a finestra mobile. Definiamo una matrice KxK, detta anche kernel o nucleo del filtro. Di solito il kernel è quadrato e di dimensioni dispari. Siano $w_{ij}$ gli elementi del kernel e fmn gli elementi dell’immagine di dimensioni MxN.  Allora il punto gmn dell’immagine filtrata sarà dato da:

$$
g_{mn} =
\sum_{i=-\frac{k-1}{2}}^{\frac{k+1}{2}}
\sum_{j=-\frac{k-1}{2}}^{\frac{k+1}{2}}
f_{m+i,\,n+j}\, w_{i,j}
$$

In pratica l’operazione di convoluzione spaziale consiste nel far scorrere il kernel sull’immagine e sostituire volta per volta il pixel dell’immagine corrispondente al pixel centrale del kernel con un valore che è la somma dei pixel dell’immagine coperti dal kernel pesati per gli elementi del kernel stesso. Solitamente il kernel è normalizzato, e quindi la somma degli elementi del kernel è unitaria in modo da non mutare sostanzialmente la dinamica dell’immagine dopo il filtraggio. Questo punto è importante nell’imaging medico in quanto il valore del segnale può avere un significato diagnostico (come nella TAC). Inoltre essendo le immagini codificate come interi a 16 bit mutare la dinamica può causare problemi di overflow numerico. 
Il risultato di un’operazione di convoluzione spaziale è un’immagine di dimensione pari alla somma dell’immagine più la dimensione del kernel. Solitamente si considera solo la parte centrale dell’immagine risultante per ottenere una immagine delle stesse dimensioni dell’immagine originale.
L’operazione di filtraggio spaziale è implementata in MATLAB tramite la funzione `conv2` e, nel caso tridimensionale, tramite la funzione `conv`.  
In Python, le operazioni equivalenti sono fornite dalla libreria **SciPy** mediante le funzioni
`scipy.signal.convolve2d` e `scipy.signal.convolve`.
L’opzione `mode='same'` consente di mantenere la dimensione dell’immagine filtrata.
In alternativa, è possibile utilizzare la funzione `scipy.ndimage.convolve`, che presenta
caratteristiche analoghe a `imfilter` di MATLAB.
I principali filtri possono essere definiti in modo automatico tramite funzioni dedicate,
come `scipy.ndimage.gaussian_filter`, oppure mediante kernel definiti manualmente.

Cambiando il kernel, si possono ottenere i vari tipi di filtraggio. Il primo esempio è il filtro di smoothing, che è in grado di ridurre il rumore. Il filtro di smoothing KxK è definito come:

$$
W_{ij}=\frac{1}{K^{2}} \quad \forall \quad i,j
$$
Un esempio 3x3 è
 
$$
\frac{1}{9}*\begin{bmatrix}
1 & 1 & 1 \\
1 & 1 & 1 \\
1 & 1 & 1 
\end{bmatrix}
$$

Notiamo che per semplicità si preferisce esprimere il filtro con valori interi, spesso sottintendendo l’operazione di normalizzazione che comunque viene sempre eseguita.
Il filtro effettua una operazione di media mobile, sostituendo ad un pixel dell’immagine il valore della media dei pixel in un intorno. Il filtro riduce efficacemente il rumore nelle regioni omogenee, mentre introduce un “ammorbidimento” dei contorni nelle regioni di confine tra i pattern, cosa non desiderabile nelle immagini biomediche. I due effetti sono tanto più rilevanti quanto più è grande il Kernel. 
Una variante del filtro a media mobile è il filtro *pillbox* di Figura 2.9 (opzione `disk` della funzione
`fspecial` in MATLAB), che realizza una media mobile limitata a una regione circolare.
In Python, un filtro equivalente può essere ottenuto tramite un kernel circolare normalizzato,
oppure utilizzando la funzione `scipy.ndimage.uniform_filter` in combinazione con una maschera
circolare.
    
<img src="./images/image-9.png" alt="filtro_pillbox" style="width:100%;">

*Figura 2.9. Filtro "pillbox".*

Un altro esempio di filtri dedicati alla riduzione del rumore sono i filtri di tipo gaussiano, dove il kernel è definito come una gaussiana bidimensionale. Un esempio di filtro gaussiano 3x3 il seguente: 

$$
\frac{1}{16}*\begin{bmatrix}
1 & 2 & 1 \\
2 & 4 & 2 \\
1 & 2 & 1 
\end{bmatrix}
$$

I filtri gaussiani riducono il rumore riducendo l’effetto di sfuocaσmento dell’immagine. Esempi di filtri gaussiani con diversa deviazione standard sono:

$$ \sigma = 0.391, \quad dim=3*3 \\
\begin{bmatrix}
1 & 4 & 1 \\
4 & 12 & 4 \\
1 & 4 & 1 
\end{bmatrix}
$$

$$ \sigma = 0.625, \quad dim=5*5 \\
\begin{bmatrix}
 1 & 2 & 3 & 2 & 1 \\
 2 & 7 & 11 & 7 & 2 \\
 3 & 11 & 17 & 11 & 3 \\
 2 & 7 & 11 & 7 & 2 \\
 1 & 2 & 3  & 2 & 1 \\
\end{bmatrix}
$$

$$ \sigma = 1, \quad dim = 9*9 \\
\begin{bmatrix}
 0 &0 &1 & 1 & 1 & 1 &1 &0 &0 \\
 0 &1 &2 & 3 & 3 & 3 &2 &1 &0 \\
 1 &2 &3 & 6 & 7 & 6 &3 &2 &1 \\
 1 &3 &6 & 9 &11 & 9 &6 &3 &1 \\
 1 &3 &7 &11 &12 &11 &7 &3 &1 \\
 1 &3 &6 & 9 &11 & 9 &6 &3 &1 \\
 1 &2 &3 & 6 & 7 & 6 &3 &2 &1 \\
 0 &1 &2 & 3 & 3 & 3 &2 &1 &0 \\
 0 &0 &1 & 1 & 1 & 1 &1 &0 &0
 \end{bmatrix}
 $$

Nel filtro gaussiano, la somma del valore del peso centrale è maggiore degli altri e i pesi sono proporzionali alla distanza dal centro secondo l’equazione della gaussiana.
Un filtro gaussiano è definito quindi da due fattori. Il primo è la grandezza del kernel, il secondo è il valore della deviazione standard σ che definisce il valore degli elementi del kernel stesso. Si noti che per σ molto più grande delle dimensioni del kernel il filtro gaussiano collassa in un filtro a media mobile, per σ molto piccolo si ottiene un kernel diverso da zero solo nel punto centrale che lascia l’immagine inalterata.  

Come visto in precedenza, l’effetto dei filtri di smoothing (media mobile o gaussiano) è simile al partial volume effect (PVE) che avviene a livello di acquisizione. Tali filtri possono quindi essere utilizzati per simulare il PVE nel modello dell’immagine biomedica. 

I filtri a convoluzione spaziale possono essere applicati efficacemente anche per evidenziare i contorni in una immagine. Il gradiente di un’immagine $f(x,y)$ nel punto $(x,y)$ è il vettore:

$$
\nabla f = 
\begin{bmatrix}
G_{x} \\
G_{y} 
\end{bmatrix} =
\begin{bmatrix}
\frac{\partial f}{\partial x} \\
\\
\frac{\partial f}{\partial y} 
\end{bmatrix}
$$

E’ noto che il vettore gradiente punta nella direzione di massima velocità di variazione di $f$ nei punti $(x,y)$. Pertanto nel problema della rivelazione dei bordi è importante l’ampiezza di questo vettore data da:

$$
\nabla f =
\begin{bmatrix}
G_{x}^{2} & G_{y}^{2} 
\end{bmatrix}^{1/2}
$$                     

Anche la direzione del gradiente $\alpha(x,y)$ è una quantità importante:

$$
\alpha(x,y) = tan^{-1} 
\begin{bmatrix}
\frac{G_{x}}{G_{y}} 
\end{bmatrix}
$$
dove l’angolo è misurato rispetto all’asse x. Notare che il calcolo del gradiente di un’immagine è basato sul calcolo delle derivate $\frac{\partial f}{\partial x}$ e $\frac{\partial f}{\partial y}$ per ogni pixel dell’immagine. Quindi il calcolo del gradiente di un’immagine deve essere fatto in due passi utilizzando due kernel: uno per la direzione $x$ ed uno per la direzione $y$ (immagini 2D) o tre passi (immagini 3D). E’ possibile anche definire un gradiente temporale in caso di immagini 2D+T o 3D+T.
In calcolo del gradiente implica il calcolo del valore della derivata da campioni discreti vista la natura discreta delle immagini. La derivata può essere approssimata in diversi modi. Per una maschera 3x3 il modo più semplice è il gradiente di Sobel:

$$
\frac{\partial f}{\partial x} = 
\begin{bmatrix}
-1 & 0 & 1 \\
-2 & 0 & 2 \\
-1 & 0 & 1 
\end{bmatrix}
$$

$$
\frac{\partial f}{\partial y} = 
\begin{bmatrix}
1 & 2 & 1 \\
0 & 0 & 0 \\
-1 & -2 & -1 
\end{bmatrix}
$$


L’operazione di derivata basata sull’operatore di Sobel è data da:

$$
G_y = (w_{31} + 2w_{32} + w_{33}) - (w_{11} + 2w_{12} + w_{13})
$$
$$
G_x = (w_{13} + 2w_{23} + w_{33}) - (w_{11} + 2w_{12} + w_{13})
$$

Notiamo che la formulazione precedente dei kernel è valida se δx=δy, cioè se la risoluzione spaziale sull’asse x è uguale a quella sull’asse y (pixel quadrato). Questa ipotesi è di solito verificata per immagini 2D, mentre se espandiamo il filtro in 3D non è in generale vera e si deve esplicitare nel kernel il valore della dimensione del voxel o interpolare il volume prima del filtraggio. 

Un'altra implementazione del filtro derivativo è il filtro di prewitt. 

$$
\frac{\partial f}{\partial x} = 
\begin{bmatrix}
-1 & 0 & 1 \\
-1 & 0 & 1 \\
-1 & 0 & 1 
\end{bmatrix}
$$

$$
\frac{\partial f}{\partial y} = 
\begin{bmatrix}
1 & 1 & 1 \\
0 & 0 & 0 \\
-1 & -1 & -1 
\end{bmatrix}
$$

Analogamente si può definire il Laplaciano che implementa la derivata seconda dell’immagine: 

$$
\nabla^2 f(x,y) = \frac{\partial^2 f}{\partial x^2} +
\frac{\partial^2 f}{\partial y^2}
$$

che corrisponde al kernel discreto:

$$
\nabla^2 =
\begin{bmatrix}
0 & -1 & 0 \\
-1 & 4 & -1 \\
0 & -1 & 0
\end{bmatrix}
$$

L’immagine di gradiente ha la caratteristica di avere un valore elevato sui contorni dell’immagine e valore nullo nelle regioni ad intensità costante. Per eliminare il problema di pixel negativi che possono essere introdotti dal filtro possiamo normalizzare l’immagine o sommare un valore costante. Uno dei possibili usi dell’immagine di gradiente/laplaciano è di fungere da guida nel filtraggio con un filtro di smoothing, in pratica il filtraggio viene effettuato solo nelle regioni nelle quali il valore dell’immagine di gradiente è basso come vedremo in seguito.
Se sommiamo all’immagine il laplaciano abbiamo un filtro che produce una maggiore definizione dei contorni (sharpening operator): 

$$
\nabla =
\begin{bmatrix}
0 & -1 & 0 \\
-1 & 5 & -1 \\
0 & -1 & 0
\end{bmatrix}
$$
L’efficacia di questo filtro è dovuta alla capacità del sistema occhio-cervello di concentrarsi sui bordi eliminando le zone a intensità costante. 
Filtri laplaciani più complessi sono i filtri “a sombrero”.

<img src="./images/image-10.png" alt="filtri_sombrero" style="width:100%;">

*Figura 2.10. Esempi di filtri a sombrero.*

In MATLAB i filtri derivativi possono essere ottenuti tramite la funzione `fspecial`.
Il gradiente dell’immagine può inoltre essere calcolato direttamente mediante la funzione
`imgradient`, che consente di utilizzare diversi metodi per il calcolo delle derivate.

In Python non esiste una funzione unica equivalente a `imgradient`; tuttavia, funzionalità
analoghe sono fornite dalle librerie **SciPy** e **scikit-image**. In particolare, gli operatori
di gradiente più comuni sono disponibili in `scipy.ndimage` e `skimage.filters`, tra cui:

- **Sobel** (`scipy.ndimage.sobel`)
- **Prewitt** (`scipy.ndimage.prewitt`)
- **Roberts** (`skimage.filters.roberts`)
- **Differenze centrali**, calcolabili tramite derivate discrete o convoluzioni
- **Differenze in avanti**, ottenibili mediante operatori di differenza finita

Il modulo e la direzione del gradiente possono essere calcolati a partire dalle componenti
lungo le direzioni orizzontali e verticali, rispettivamente come

$$
|\nabla f| = \sqrt{G_x^2 + G_y^2}, \qquad
\theta = \arctan\left(\frac{G_y}{G_x}\right).
$$

#### Filtraggio adattivo
I filtri visti finora agiscono nello stesso modo su tutta l’immagine. Una classe di filtri più complessa è quella dei filtri anisotropici, che funzionano in modo diverso in regioni diverse dell’immagine. Questi filtri vengono detti anche adattivi in quanto adattano il loro comportamento sulla base del contenuto dell’immagine. Tipicamente un filtro anisotropico utilizza una informazione di gradiente per localizzare i bordi dell’immagine che devono essere preservati nell’operazione di filtraggio. Il filtraggio viene modulato in base al valore locale del gradiente.  
Un esempio semplice di un filtro adattivo è un filtro di smoothing che effettua lo smoothing solo dove il gradiente locale dell’immagine è inferiore ad un certo valore di soglia, mentre dove il valore è superiore lascia l’immagine inalterata. 
Nella pratica per realizzare un semplice filtro adattivo si computa il gradiente dell’immagine $I_{OR}$ attraverso un filtraggio locale come descritto in precedenza. Si ottiene così una immagine $G$ di dimensioni uguale all’originale che contiene il gradiente dell’immagine stessa. Stabilita una soglia $T_{g}$ si applica a $G$ un filtro binario puntuale con soglia $T_{g}$, ottenendo una immagine $mask_{HG}$, che vale uno se il gradiente è maggiore di $T_{g}$ e zero altrove. Una immagine binaria di questo tipo è spesso detta maschera (mask). Applicando un filtro puntuale di inversione a $mask_{HG}$ si ottiene la sua inversa $mask_{LG}$, che rappresenta la maschera delle regioni dell’immagine con gradiente inferiore o uguale a $T_{g}$. 
A questo punto si applica un filtraggio locale a media mobile (smoothing) o gaussiano all’immagine $I_{OR}$ ottenendo l’immagine filtrata $I_{MM}$, dove lo smoothing è applicato su tutta l’immagine. L’immagine filtrata in modo adattivo $I_{FA}$ si ottiene dalla formula:

$$
I_{FA} = I_{OR} * mask_{HG} + I_{MM} * mask_{LG}
$$
Infatti essendo le due maschere mutuamente esclusive (la loro somma da uno su tutti i pixel) l’immagine risultante è uguale all’immagine originale nelle zone ad alto gradiente ed alla immagine filtrata nelle zone a basso gradiente.  
Le prestazioni di un filtro adattivo così realizzato dipendono ovviamente dal valore di $T_{g}$. Se $T_{g}$ è troppo alto alcuni contorni verranno sfumati, se $T_{g}$ è troppo basso regioni uniformi dell’immagine non verranno filtrate. Per individuare il valore ottimo di $T_{g}$ è utile esprimere tale soglia in funzione delle statistiche del rumore associato all’immagine. Facciamo riferimento ad un modello semplificato di immagine biomedica dove l’immagine reale è data dalla somma dell’immagine reale e di rumore gaussiano con media nulla e deviazione standard $\sigma$.

$$
I(x,y)=I_{0}(x,y)+n(x,y)
$$

Su un pattern omogeneo dell’immagine (dove vogliamo che venga applicato il filtro) il segnale sarà distribuito il modo gaussiano con media $Sp$ (valore del segnale sul pattern) e deviazione standard $\sigma$. Il valore del gradiente sul pattern intuitivamente dipenderà da $\sigma$ e dal tipo di filtro utilizzato per calcolare le due componenti del gradiente stesso. Nel caso semplificato in cui si computi la derivata come differenza tra il valore di due pixel adiacenti (kernel [1 0 -1]) il valore di $Gx$ sara $Ga(Sp,\sigma)–Gb(Sp, \sigma)$ dove $Ga$ e $Gb$ sono due realizzazioni indipendenti di un processo gaussiano con  media $Sp$ e deviazione standard $\sigma$. Sapendo che il $95%$ dei valori generati da un processo gaussiano è compreso nella finestra $[-1.96\sigma,1.96\sigma]$ approssimabile a $[-2\sigma,2\sigma]$ nel caso peggiore $|Gx|$ potrà assumere il valore $4\sigma$ e così per $Gy$, dando un gradiente complessivo massimo $G = |Gx| + |Gy| = 8\sigma$. Si tratta evidentemente di un caso limite molto improbabile, specialmente nel caso che il kernel usato per computare il gradiente sia di dimensioni non ridottissime, ma che giustifica l’idea di imporre: 

$$
T_{g} = k \sigma
$$

Eliminando quindi la dipendenza di $T_{g}$ dal rumore. Il valore di $\sigma$ può essere calcolato in modo manuale o automatico utilizzando le tecniche viste nel calcolo del $SNR$ e $CNR$. Il valore di $k$ consente di regolare il funzionamento del filtro, il valore di $k$ varia tipicamente tra 1 e 4.
Un filtro adattivo così definito ha lo svantaggio principale di transire in modo brusco dalla zona in cui viene effettuato lo smoothing alla zona in cui l’immagine originale viene preservata, a causa dell’utilizzo di un filtro a gradino. L’altro svantaggio è di richiedere la definizione di $k$ per stabilire la soglia. 

Un’alternativa più evoluta è rappresentata dal **filtro di Wiener**.
In MATLAB tale filtro è implementato tramite la funzione `wiener2`.

In Python, un filtro equivalente è disponibile nella libreria **SciPy**
mediante la funzione `scipy.signal.wiener`.
Il metodo si basa sul calcolo delle statistiche locali dell’immagine:
l’immagine viene inizialmente filtrata con un filtro a media mobile,
ottenendo $I_{MM}$, che rappresenta la media locale per ciascun pixel.
Successivamente viene calcolata la mappa della varianza locale
$I_{VAR}$, stimata sullo stesso kernel utilizzato per il filtro a media mobile.

L’immagine filtrata risulta quindi:

$$
I_W = I_{MM} +
\frac{I_{VAR} - \sigma^2}{I_{VAR}}
\left( I_{OR} - I_{MM} \right)
$$

Se la varianza dell’immagine è uguale a quella del rumore (pattern uniforme) il secondo termine si annulla e l’immagine risultante è uguale a quella filtrata con il filtro a media mobile. Se $I_{VAR}$ è molto grande rispetto al rumore dell’immagine l’immagine filtrata risulta uguale all’immagine originale. Nei due casi estremi il filtro di Wiener si comporta quindi come il filtro adattivo visto prima. Il vantaggio del filtro di Wiener è che gestisce in modo graduale la transizione tra le zone omogenee e le zone ad alto gradiente dove sono presenti i contorni. Inoltre non è richiesto di definire il valore di $k$ ma solo di valutare il rumore sull’immagine $\sigma$.

Un possibile approccio per la stima automatica di $\sigma$ consiste nel far scorrere
sull’immagine un kernel (ad esempio $5 \times 5$) e nel calcolare il valore della
deviazione standard $SD$ sulla regione di immagine coperta dal kernel.
In MATLAB tale operazione è implementata tramite la funzione `stdfilt`.

In Python, una funzionalità equivalente può essere ottenuta utilizzando la libreria
**SciPy**, mediante la funzione `scipy.ndimage.generic_filter`, oppure calcolando la
deviazione standard locale a partire da media e varianza locali.

L’operazione produce una mappa di valori di $SD$ avente la stessa dimensione
dell’immagine di partenza. Nelle regioni corrispondenti a pattern uniformi,
il valore di $SD$ approssima $\sigma$ nel caso di rumore gaussiano additivo.
Al contrario, nelle regioni in prossimità dei bordi tra pattern differenti,
il valore di $SD$ risulta maggiore di $\sigma$. Un esempio è riportato in Figura 2.11. 

<img src="./images/image-11.png" alt="mappaSD" style="width:100%;">

*Figura 2.11. Sinistra: immagine originale. Destra: mappa di deviazione standard.*

In Figura 2.11. osserviamo una immagine formata da pattern omogenei con rumore gaussiano a media nulla e $\sigma=10$ e la corrispondente mappa della $SD$. Come si nota dalla figura il valore di $SD$ sulla mappa è massimo i corrispondenza dei bordi ed assume valori bassi e variabili nelle regioni omogenee. Questo è confermato dall’istogramma della mappa SD riportato in Figura 2.12.

<img src="./images/image-12.png" alt="isto_mappaSD" style="width:100%;">

*Figura 2.12. Istogramma della mappa di deviazione standard.*

Che presenta un picco evidente in corrispondenza della $SD$ misurata nelle zone omogenee con il massimo in corrispondenza della σ del rumore gaussiano imposto. Dalla mappa $SD$ è possibile quindi valutare il valore di $\sigma$ utile per la configurazione del filtro adattivo. L’approccio più semplice trascura la $SD$ delle regioni di transizione e stima σ come la media delle $SD$ sulla mappa (nel caso in figura si ottiene $\sigma=11.4$). Volendo eliminare gli outliers dovuti ai bordi è preferibile computare la mediana della mappa $SD$  (nel caso in Figura 2.12 si ottiene $\sigma=9.2$). Un approccio più accurato estrae il primo picco dell’istogramma e ne computa la posizione del massimo.
Se il rumore non è gaussiano si dovrà tenere conto di ciò nel calcolo di $\sigma$. Ad esempio nelle immagini MR ci attendiamo che l’istogramma della $SD$ map presenti due picchi, uno in corrispondenza dello sfondo ed un altro spostato a destra di un fattore 1.526 in corrispondenza alle regioni omogenee ad alto SNR. In questo caso il valore di σ andrà valutato direttamente sul secondo picco o sul primo introducendo l’opportuno fattore correttivo. 

#### Adaptive Template Filtering
Per “Adaptive Template Filtering” si indica un filtraggio basato sulla scelta adattiva di un determinato template (cioè un kernel di filtraggio) in base alle caratteristiche dell’immagine sottostante. Una implementazione possibile è quella proposta da C. B. Ahn.  L’idea di base è quella di avere una collezione di possibili template e di scegliere per ogni locazione dell’immagine un particolare template ottimizzato. L’obiettivo dell’algoritmo è quello di migliorare l’SNR evitando allo stesso tempo la perdita di definizione dei contorni. Il numero dei possibili template dipende dalla grandezza del template stesso ed è dato da: 

$$
NT = \sum_{k=2}^{N} C_N^{k}
   = \sum_{k=2}^{N} \frac{n!}{k!(n-k)!}
$$

per un template $3 \times 3$ abbiamo chiaramente  $N=9$ e un numero totale di $NT=255$ configurazioni possibili. La Figura 2.13 mostra la collezione dei template 3x3. Essendo m il numero di 1 nel template, si ha un template per m=9, 8 template per m=8, 28 template per m=7, etc. 

<img src="./images/image-13.png" alt="adaptive_filters" style="width:100%;">

*Figura 2.13. Esempio di banco di filtri per filtraggio adattivo.*

La scelta del template ottimo viene effettuata attraverso il calcolo di un opportuno indice. In particolare, viene calcolata la deviazione standard $(SD)$ del valore dei pixel sul template, data da: 

$$
\sigma(k,l) =
\sqrt{
\frac{1}{m-1}
\sum_{i,j \in T}
\left[ x(i,j) - \bar{x}(k,l) \right]^2
}
$$

$$
\bar{x}(k,l) =
\frac{1}{m}
\sum_{i,j \in T} x(i,j)
$$

dove $x(i,j)$ sono i valori dei pixel nel template, $(k,l)$ sono le coordinate del pixel da computare (e quindi le coordinate del centro del template), $T_{j}$ è il template corrente e $N_{j}$ è la dimensione m del template $T_{j}$ . Per ogni pixel dell’immagine da filtrare, la $SD$ viene calcolata per ogni possibile template. I template vengono divisi in due classi in base alla $SD$: I template con $SD$ minore di una certa soglia (plane templates) e quelli con $SD$ superiore alla soglia (edge templates). 
Se vengono riconosciuti uno o più plane template, viene scelto quello con dimensione maggiore. Se non ci sono plane template, viene selezionato  l’edge template con $SD$ minima. 
In realtà per diminuire i tempi di calcolo si procede esaminando i template in ordine di dimensione e fermandosi appena viene trovato un plane template. Nella sostanza l’algoritmo individua per ogni pixel la distribuzione spaziale dei pixel circostanti “simili” al pixel corrente, ed effettua l’operazione di smoothing tenendo conto solo di detti pixel. Il punto fondamentale dell’algoritmo è la scelta della soglia sulla $SD$. Similmente a quanto visto in precedenza la soglia è definita come:

$$
\tau = a\sigma_{n}
$$

dove è la $SD$ del rumore sull’immagine e a è un fattore di scala, che assume un valore tra 1.2 e 1.6. La stima di $\sigma_{n}$ può essere effettuata come visto in precedenza per il filtro adattivo.
 
### Operatori globali ed equalizzazione dell’istogramma 
Un esempio di filtraggio globale dell’immagine è rappresentato della procedura di equalizzazione dell’istogramma, che ha lo scopo di aumentare il contrasto percepito dell’immagine. Nella procedura di equalizzazione l’istogramma dell’immagine viene modificato in modo da divenire costante. Quindi se consideriamo l’immagine $f$ con istogramma $h(f)$, il filtro produrrà una immagine $g$ con istogramma $h(g)$ tale che $h(g) = C$ per tutti i valori di livello di grigio di $g$. Si ha chiaramente $C = N/p$  dove $N$ è il numero di pixel dell’immagine e $p$ la profondità dell’immagine stessa.
Inoltre, deve essere conservato l’ordinamento dei livelli di grigio, per cui se $f1< f2$ deve risultare $g1<g2$, quindi la trasformazione deve essere monotona. 
In pratica il filtro deve trovare “l’inversa” dell’istogramma di $f$, in modo che la funzione monotona $T$ che caratterizza il filtro sia tale che:

$$
T(h(f)) = h^{-1}(f)h(f)  = C 
$$

Tale funzione è la distribuzione cumulativa (CDF) dell’istogramma da equalizzare, definita come:

$$
g_k = \frac{p-1}{N} \sum_{j=0}^{k} f_k,
\qquad k = 0,1,\ldots,p-1
$$

Consideriamo il fantoccio TAC in Figura 2.14 ed il relativo istogramma (in blu). Per chiarezza è stata eliminata la prima riga dell’istogramma che risulta molto alta contenendo la zona di zero padding.
 
<img src="./images/image-14.png" alt="cdf" style="width:100%;">

*Figura 2.14. Sinistra: fantoccio TAC. Destra: Istogramma (blu) e relativa CDF (rosso).*

La CFD relativa all’istogramma è visualizzata in rosso. Come si osserva la CDF è “piatta” nelle regioni dove l’istogramma ha valori bassi mentre cresce ripidamente dove l’istogramma ha valori alti. Il filtro usa la CDF come “hash table” per definire la trasformazione implementata. Quindi ad esempio, i valori di livello di grigio dell’immagine tra 2000 e 3000 verranno sostituiti da valori molto simili dati dal valore della CDF che varia lentamente tra i due picchi a 2000 e 3000, “appiattendo” l’istogramma.
L’immagine filtrata risulta come in Figura 2.15. 

<img src="./images/image-15.png" alt="equalizzazione" style="width:100%;">

*Figura 2.15. Esempio di equalizzazione dell'istogramma.*

Si osserva come l’equalizzazione dell’istogramma “allarghi” i picchi e introduca una quantizzazione
dei livelli di grigio, concentrandoli nelle regioni in cui la CDF risulta pressoché costante.

In Python l’equalizzazione dell’istogramma può essere implementata tramite la libreria
**scikit-image** usando la funzione `skimage.exposure.equalize_hist`.
A differenza di `histeq` di MATLAB, l’operazione può essere applicata anche a immagini a
profondità maggiore (ad esempio 16 bit), a patto di gestire correttamente la normalizzazione
dell’intervallo di intensità. L’esempio è implementato nel Notebook XXX. 

## Compressione dell’immagine biomedica
Accenniamo infine al problema della compressione delle immagini biomediche. La compressione ha lo scopo di ridurre il numero di bit necessari per l’immagazzinamento dell’immagine biomedica. Il vantaggio è quello di ridurre la dimensione dei supporti informatici necessari all’immagazzinamento a breve e lungo termine e di migliorare la velocità di trasmissione delle immagini stesse nelle reti di comunicazione. Quest’ultima applicazione è quella oggi più rilevante. Lo svantaggio della compressione è che per usufruire dell’immagine compressa è necessario un algoritmo di decompressione che ripristini l’immagine originaria. In altri termini il software con cui si utilizza l’immagine deve implementare le caratteristiche DICOM di compressione, altrimenti l’immagine sarà inutilizzabile. La compressione viene quindi solitamente utilizzata o quando strettamente necessaria (telemedicina) o quando tutte le operazioni sull’immagine sono sotto il controllo di una singola struttura (ad esempio backup di un PACS). L’argomento verrà trattato più in dettaglio nel seguito quando si descriveranno i sistemi PACS.  
L’efficacia di una operazione di compressione si misura attraverso il rapporto  di  compressione che è  il rapporto tra le dimensioni dell’immagine dopo la compressione e le dimensioni originali. Il rapporto di compressione si può indicare in termini percentuali (25% o 0.25 indica che la dimensione dell’immagine compressa è il 25% di quella originale) o come X:1, dove X indica quante volte l’immagine originale è più grande di quella compressa (ad esempio un rapporto di compressione 4:1 indica che l’immagine compressa è 4 volte più piccola e corrisponde ad un rapporto percentuale del 25%).
Una distinzione fondamentale è quella tra compressione  senza  perdite (lossless), che è reversibile nel senso che dall’immagine compressa è possibile ricostruire l’immagine originale, e compressione con perdite nella quale una parte dell’informazione associata all’immagine viene persa nel processo di conversione. La compressione con perdite permette rapporti di compressione molto maggiori, di almeno un ordine di grandezza. Un esempio di compressione senza perdite sono gli archivi digitali (zip, rar, etc). L’esempio tipico di compressione con perdite è il formato mp3 per i file audio e MPEG per i file video.  
In ogni caso perché sia possibile una compressione deve esistere una ridondanza, cioè il numero di bit che codificano l’immagine deve essere maggiore del numero di bit che descrivono il contenuto informativo dell’immagine stessa. Questo accade pressoché sempre nell’imaging biomedico, si pensi ad esempio al segnale di fondo che non apporta nessun contenuto informativo. Inoltre nelle immagini biomediche pixel vicini hanno spesso valori simili e questa caratteristica può utilmente essere usata in fase di compressione. Questo è vero anche in senso temporale nelle immagini dinamiche, dove si possono utilizzare algoritmi come l’MPEG.
Esistono innumerevoli algoritmi di compressione con perdite e senza perdite. Tra i classici ricordiamo l’RLE (Run Lenght Encoding) che sfrutta le ripetizioni di byte uguali, l’LZW (Lempel Ziv Welch) che impiega le ripetizioni di stringhe di byte uguali, l’algoritmo di Huffman che utilizza codici di rappresentazione più brevi per i pixel che appaiono più frequentemente.
Tra gli algoritmi di compressione con perdite ricordiamo la DCT (Discrete Cosine Transform) e la compressione Wavelets che sfruttano la scomposizione dell’immagine in sub immagini con un diverso grado di dettaglio. Le immagini con grado di dettaglio molto elevato (corrispondenti a variazioni tra pixel molto vicine) vengono considerate rumore ed eliminate.
Nella compressione con perdite si definisce l’errore di compressione come l’errore quadratico medio misurato tra l’immagine originale (I) e l’immagine dopo la compressione/decompressione (D):

$$
\overline{e^2} =
\frac{1}{N^2}
\sum_{i=1}^{N}
\sum_{j=1}^{N}
\left( I(i,j) - D(i,j) \right)^2
$$

si definisce anche il rapporto segnale/errore di compressione (Figura 2.16) come:
 
$$
\mathrm{PSNR} =
10 \log_{10}
\left(
\frac{
\displaystyle
\sum_{i=1}^{N}
\sum_{j=1}^{N}
I^2(i,j)
}{
\overline{e^2}
}
\right)
$$
 
Di solito esisterà un rapporto inverso tra errore percentuale di compressione (PSNR) e rapporto di compressione (CR).
 
<img src="./images/image-16.png" alt="errore_compressione" style="width:100%;">

*Figura 2.16. Rapporto segnale/errore di compressione.*

Naturalmente nell’imaging biomedico è fondamentale anche il giudizio “visivo” dell’operatore. Il formato DICOM supporta JPEG, JPEG Lossless, JPEG 2000, and Run-length encoding (RLE). Il Lossless JPEG (compressione JPEG senza perdite) viene usato tipicamente in immagini agiografiche che sono di dimensioni elevate e sostanzialmente bimodali. Ottiene rapporti di compressione di qualche unità (fino a 4:1). Il formato JPEG 2000 supporta la compressione con perdite e senza perdite con tecnica DCT o Wavelet. Inoltre permette di scegliere livelli di compressione diversi in diverse regioni dell’immagine. È usato soprattutto in applicazioni di telemedicina.
Nell’ambito dell’imaging biomedico è importante notare come la compressione con perdite comporti la perdita di una parte del contenuto informativo, e va quindi utilizzata con estrema cautela. 
 
## Super-Resolution
Gli algoritmi di interpolazione descritti in precedenza possono ridurre la dimensione del voxel ma non possono ovviamente migliorare la risoluzione “fisica” dell’immagine che è limitata dall’effetto volume parziale. Modelliamo il caso di un nodulo di piccole dimensioni nel fegato e studiamo le conseguenze del PVE sulla possibilità di riconoscere il modulo in un sistema di imaging. Consideriamo una sfera di raggio R (ad esempio 1mm), formata da un materiale che produca un segnale S1 in un certo sistema di imaging, immersa in un altro materiale con associato un segnale S2, ed eseguiamo una scansione con voxel cubico di dimensione 2Rx2Rx2R. Il valore di CNR tra sfera e tessuto contenente come visto in precedenza sarà:

$$
CNR =
\frac{|M_1 - M_2|}{\sigma}
=
\frac{|p S_1 + (1-p) S_2 - S_2|}{\sigma}
=
\frac{p\,|S_1 - S_2|}{\sigma}
$$

Dove $M_1$ e $M_2$ sono i segnali misurati. Infatti $M_2$ sarà uguale a $S_2$ mentre $M_1$ sarà dato dal contributo dei due tessuti nel voxel in dipendenza da $p$. La relazione ci dice che il $CNR$ misurato si ottiene moltiplicando il valore di $CNR$ ideale per il fattore p di riempimento del voxel. Nel caso migliore della sfera perfettamente inscritta nel cerchio  $p=\pi/6$ mentre nel caso peggiore di sfera inclusa in modo simmetrico in otto voxel $p=\pi/48$. Quindi il $CNR$ rispetto al valore teorico si riduce di circa il 50% nel caso migliore fino a diventare circa il 6% nel caso peggiore. Se la soglia di $CNR$ che assicura la visibilità della sfera è intermedia tra i due casi, non essendo prevedibile a livello di acquisizione il valore di $p$, la sfera risulterà visibile o meno in modo casuale in dipendenza dalla posizione della sfera stessa nel sistema di imaging. Consideriamo l'esempio semplificativo di Figura 2.17.

<img src="./images/image-17.png" alt="superresolution" style="width:100%;">

*Figura 2.17. Esempio di diversi valori di riempimento di un pixel per una cerchio di raggio $r$.*

A sinistra osserviamo una immagine ad alta risoluzione (HR) 2048x2048 di un fantoccio circolare. Il cerchio ha valore di segnale $S_1=1000$ mentre il fondo ha valore di segnale $S=100$. Il raggio del cerchio $r$ è 128. Se interpoliamo a bassa risoluzione (LR) con passo 256 otteniamo una immagine 8x8, con dimensioni del pixel dell’immagine LR uguale alle dimensioni del cerchio. Se il pixel LR corrisponde esattamente al cerchio il cerchio si riduce ad un unico pixel di valore 
$\frac{\pi r^{2}}{4\pi r^{2}}S_{1} + \frac{(4-\pi)r^{2}}{4\pi r^{2}}S_{2} \approx 806$ (immagine centrale). Se il cerchio è equi-diviso in quattro pixel LR avremo quattro pixel LR con valore diverso dal fondo pari a  valore $\frac{\pi r^{2}}{16\pi r^{2}}S_{1} + \frac{(4-\pi/4)r^{2}}{4\pi r^{2}}S_{2} \approx 276$  (immagine a destra). 
Facendo variare lo “sfasamento” tra voxel LR e cerchio, si ottengono tutti i possibili valori di contrasto in dipendenza dallo sfasamento come riportato in Figura 2.18. 

<img src="./images/image-18.png" alt="contrasto_vs_sfasamento" style="width:100%;">

*Figura 2.18. Andamento del contrasto al variare dello sfasamento.*

Tornando al caso reale, un nodulo con dimensioni dell’ordine della dimensione del voxel potrà essere visibile o meno sulla base della posizione del paziente rispetto al sistema di imaging in modo del tutto imprevedibile. La “certezza” di osservare l’oggetto si ha solo quando le sue dimensioni sono tali da saturare con certezza almeno un intero voxel. In teoria, se ripetiamo l’acquisizione un numero grande di volte variando la posizione dell’oggetto e/o la posizione della griglia di acquisizione, prima o poi otterremo il caso ottimale di sfera inscritta nel voxel e quindi visibile almeno in un set di immagini. Questa idea è alla base del concetto di super-imaging.    

L’idea alla base delle tecniche di Super-Resolution è quella di combinare immagini diverse della stessa struttura anatomica per ottenere una migliore risoluzione spaziale. Consideriamo  l'esempio di Figura 2.19, dove viene riportata una tipica acquisizione cardiaca MRI, nella quale vengono ottenute immagini cardiache lungo gli assi principali del cuore (asse corto, due camere, tre camere, quattro camere).   

<img src="./images/image-19.png" alt="acq_cardiaca_MRI" style="width:100%;">

*Figura 2.19. Immagini multipiano del cuore in MR (4C, 2C, 3C, SA).*

Ogni tipo di immagine è acquisita su di un piano diverso e presenta una risoluzione spaziale fortemente anisotropica (tipicamente 1.7x.17x8 mm). Nelle tecniche di super-resolution si cerca di combinare i voxel anisotropi in modo da migliorare la risoluzione spaziale sull’asse perpendicolare al piano di acquisizione (tecnicamente in MRI la slice-select direction). In Figura 2.20 sono riportati due esempi per tre acquisizioni su piani perpendicolari e per tre acquisizioni “interallacciate” in cui la distanza inter-slice è una frazione del thickness. 

<img src="./images/image-20.png" alt="superresolution" style="width:100%;">

*Figura 2.20. Concetto di super-resolution (E van Reeth et al, Concepts in Magnetic Resonance part A 2012;40A:306-325).*

Lo stesso concetto si può applicare ad una serie temporale di immagini, con la fondamentale differenza che si ha una deformazione della struttura dell’organo che va compensata con algoritmi di registrazione come sarà descritto nel seguito. 
In generale il problema della super-resolution si può descrivere come $(k=1,...,N)$:

$$
Y_k=D_k T_k h_k X+n_k
$$

$Y_k$ sono le $N$ immagini a bassa risoluzione che abbiamo disponibili dal sistema di imaging (le immagini reali nel modello di immagine biomedica introdotta nel primo capitolo). $D_k$ rappresenta in processo di “downsampling” che riduce il numero di pixel dell’immagine $X$. $T_k$ rappresenta la trasformazione geometrica che descrive la posizione della griglia di acquisizione più l’eventuale deformazione indotta dal processo di acquisizione, ad esempio per i movimenti del paziente tra una acquisizione e l’altra. hk rappresenta l’effetto volume parziale che può essere modellato come la convoluzione con un kernel gaussiano in 2D o 3D. Nel caso di slice spacing nullo $h_k$ è un filtro gaussiano 3D con kernel anisotropico con dimensione legata alla risoluzione spaziale lungo i tre assi. $X$ è l’immagine “ideale” ad alta risoluzione. $n_k$ è il rumore indotto dal sistema di imaging per il quale possono essere fatte le considerazione viste in precedenza per il modello dell’immagine biomedica. 
Lo scopo di un algoritmo di super-resolution è stimare $X$ note le immagini a bassa risoluzione $Y_k$. 
Se la risoluzione spaziale delle immagini $Y_k$ è la stessa, $h_1=h_2=...=h_N = h$ e $D_1=D_2=...=D_N=D$. Tipicamente anche il rumore avrà la stessa distribuzione in tutte le acquisizioni, quindi avremo:

$$
Y_k= D T_k h X+n
$$

In generale il problema di stimare $X$ da  $Y_k$ è un tipico problema inverso, che è tipicamente “mal posto” e quindi non ha una soluzione univoca. Il modo più semplice di risolvere il problema è attraverso un algoritmo di “iterated back-projection”, in cui viene fatta una stima di $X$ e si cerca di minimizzare in modo iterativo la differenza tra gli $Y_k$ prodotti dalla stima di $X$ e gli $Y_k$ misurati (M Irani et al CVGIP 1991, doi: 10.1016/1049-9652(91)90045-L).
Preliminarmente alla soluzione del problema inverso, è necessario stimare il disallineamento tra le immagini $Y_k$. In alcuni casi il disallineamento può essere noto, ad esempio dalle informazioni DICOM. Altrimenti il disallineamento deve essere stimato attraverso un algoritmo di registrazione di immagini che verrà introdotto nei capitoli successivi. 
L’idea di base dell’algoritmo è quella di partire da una stima dell’immagine HR $X0$ (ottenuta ad esempio dall’interpolazione di una immagine LR). A questo punto da $X0$ si stimano le immagini LR sulla base della conoscenza di $D_k$, che come prima detto si considera nota, e di $h$ che è un filtro opportuno che tiene conto del PVE. Si ottiene così la stima di $Y_k$ al passo 1 come:

$$
\hat{Y}_k^{(1)}=D T_k h X_0
$$

Questo passaggio simula il processo di acquisizione delle immagini LR, quindi $h$ e $D$ devono essere definiti in modo congruente al rapporto tra le risoluzioni spaziali delle immagini HR e LR. 
A questo punto l’algoritmo calcola la differenza tra l’immagine LR stimata e quella reale:

$$
G_k=Y_k-\hat{Y}_k^{(1)}
$$

Il gradiente G utilizzato per ottenere la nuova stima di X è:

$$
G = \sum_{k=1}^{N} D T_k^{-1} G_k
$$

Dove $T_k^{-1}$ rappresenta la trasformazione geometrica inversa che riallinea le mappe $G$. Nota $G$, si calcola il nuovo valore di X come:

$$
\hat{X}_1 = \hat{X}_0 + \lambda G*H
$$

Dove $\lambda$ determina la velocità della convergenza (un valore alto rischia di non assicurare la convergenza, un valore basso aumenta il numero di iterazioni necessarie ad ottenere una soluzione). $H$ rappresenta il kernel di un filtro che ottimizza la convergenza, e che può essere scelto in modo arbitrario. Per le immagini biomediche standard la scelta ottima è di solito $H = h$.
Il processo prosegue in modo iterativo 

$$
\hat{X}^{(n+1)} = \hat{X}^{(n)} + \lambda G^{(n)}*H
$$

Fino a quando la differenza percentuale tra i valori di $X$ stimati a due passi successivi non scende sotto una certa soglia. Si dimostra che l’algoritmo minimizza l’errore quadratico medio tra le $Y_k$ misurate e quelle stimate:

$$
\varepsilon^{(n)} =
\sqrt{
\sum_{k=1}^{N}
\left( Y_k - \hat{Y}_k^{(n)} \right)^2
}
$$
 
Essendo il problema mal posto, la soluzione non è unica e quindi può dipendere dalla scelta delle condizioni iniziali $\hat{X}^{(0)}$. Tipicamente $\hat{X}^{(0)}$ viene ottenuta interpolando una immagine a bassa risoluzione $Y_k$ riportandola alle dimensioni dell’immagine ad alta risoluzione. Per risolvere il problema si possono inserire delle condizioni di regolarizzazione, che introducono delle ipotesi a priori sul tipo di soluzione che si desidera raggiungere. L’ipotesi tipica è che $X$ sia “smooth”, cioè che non siano possibili transizioni troppo brusche tra i livelli di grigio. In questo caso viene minimizzata la metrica:
 
$$
\varepsilon =
\sum_{k=1}^{N}
\left( Y_k - \hat{Y}_k^{(n)} \right)^2
+ \gamma \lVert C \hat{X} \rVert^2
$$

Dove C rappresenta un filtraggio di tipo passa-alto, quindi ad esempio il gradiente dell’immagine, e γ è una costante che rappresenta il peso della regolarizzazione nel processo di minimizzazione della metrica. Chiaramente la presenza del filtro C penalizza le soluzioni con bruschi cambiamenti di segnale ottenendo la regolarizzazione. 
 
Sono stati proposti molti altri algoritmi per la soluzione del problema inverso in super-resolution, quali il  Deterministic Regularized Approach,  lo Statistical Regularized Approach e altri. Per maggiori informazioni si può fare riferimento alle review di Van Reeth et al (doi: 10.1002/cmra.21249) e H Greenspan et al (doi:10.1093/comjnl/bxm075).

## Bibliografia
1. R Rangayyan, Biomedical Image Analysis, CRC Press 2004
2. M Irani et al CVGIP 1991, doi: 10.1016/1049-9652(91)90045-L
3. Van Reeth et al (doi: 10.1002/cmra.21249) e H Greenspan et al (doi:10.1093/comjnl/bxm075)