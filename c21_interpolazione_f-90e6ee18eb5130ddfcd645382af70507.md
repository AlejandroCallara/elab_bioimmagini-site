# Capitolo 2: Interpolazione, filtraggio e compressione

Come visto in precedenza, il primo passo dopo l’acquisizione dell’immagine biomedica è l’esecuzione di operazioni a basso livello che hanno lo scopo di migliorare l’immagine. Queste operazioni corrispondono al secondo livello nella classificazione degli algoritmi di computer vision.

---

## Interpolazione

Una immagine biomedica ha dimensioni definite a livello di acquisizione o del processo di ricostruzione implementato nello scanner. Le dimensioni sono tipicamente legate alla risoluzione spaziale, nel senso che a parità di campo di vista (FOV), modalità con una migliore risoluzione spaziale saranno in grado di acquisire pixel di dimensioni minori e quindi la dimensione dell’immagine sarà maggiore. Le dimensioni tipiche di una immagine radiologica possono andare da 64x64 (immagini SPECT) a 256x256 (MR cardiaca) fino a 512x512 (CT e MRI) o 1024x1024 (CT). La radiografia classica ha risoluzioni più alte, fino a 5000x5000 per la mammografia.

Nell’elaborazione delle immagini biomediche, può essere necessario cambiare la dimensione delle immagini, aumentando o diminuendo il numero di pixel componenti. Un tipico esempio è la visualizzazione dell’immagine su di uno schermo per la visualizzazione e refertazione. Tipicamente la dimensione di una immagine radiologica è minore (o comunque diversa) della dimensione del supporto digitale (stampa o schermo) su cui deve essere visualizzata. Un monitor tipico per refertazione radiologica ha risoluzione di 1600x1200 pixel, che è comunque una risoluzione usuale anche nei monitor di uso generale. È necessario quindi interpolare l’immagine in modo da permetterne la visualizzazione in maniera corretta adattando le dimensioni delle immagini a quelle dello schermo (o della porzione di schermo) in cui vanno visualizzate.

Un altro esempio, come sarà visto nel seguito, è il cambio di risoluzione spaziale nelle immagini 3D per ottenere dei voxel cubici e quindi delle immagini isotrope.

Come già visto in precedenza, l’immagine biomedica può essere vista come una serie di campioni di un processo fisico acquisiti in uno spazio discreto, la cui risoluzione è uguale alla dimensione del pixel. In pratica l’immagine ci fornisce i campioni in NxM locazioni spaziali poste al centro dei pixel dell’immagine (seguendo la convenzione DICOM).

Consideriamo l’immagine in figura con FOV 340x340 mm e dimensioni 256x256 e il suo ingrandimento a destra. Il pixel size è 340/256 = 1.32 mm, e questo è anche l’intervallo di campionamento spaziale.

Ogni pixel corrisponde ad un punto della griglia. Se vogliamo aumentare il numero di righe e colonne dell’immagine dobbiamo aumentare la risoluzione della griglia. Supponiamo di raddoppiare la risoluzione interpolando l’immagine a 512x512.

In base alla nuova griglia dovremo definire dei nuovi pixel nella posizione corrispondente ai nuovi punti della griglia. A seconda della procedura usata per definire dei nuovi valori avremo vari metodi di interpolazione. Tipicamente il nuovo valore dei pixel verrà dedotto da quello dei pixel vicini.

### Nearest Neighbor

Un primo approccio (NN, nearest neighbor) assegna al nuovo pixel il valore del pixel più vicino. Il metodo è molto veloce e ha il vantaggio di non introdurre nuovi livelli di grigio, cosa che può essere rilevante da un punto di vista diagnostico. D’altra parte, il metodo NN tende a creare dei gruppi di pixel con lo stesso valore, dando l’impressione visiva di una immagine “pixellata”.

### Interpolazione bilineare

Un metodo più perfezionato è la cosiddetta interpolazione bilineare, nella quale il nuovo valore del pixel viene stimato come la media pesata dei valori dei 4 (o 8 in 3D) pixel più vicini.

L’interpolazione bilineare offre solitamente una qualità visiva migliore dell’interpolazione NN.

L’interpolazione bilineare può essere vista come una stima del valore atteso del segnale modellando la transizione del segnale tra pixel come una retta. L’idea può essere estesa utilizzando modelli non lineari. Un esempio è l’interpolazione bicubica che utilizza polinomi di terzo grado e una griglia di 16 pixel.

Esistono poi metodi ancora più perfezionati (spline) in cui l’intera distribuzione dei pixel sull’immagine viene modellata come una superficie.

Nella visualizzazione di immagini mediche, oltre alla qualità visiva percepita dell’interpolazione è importante anche il tempo di elaborazione.

### Interpolazione 3D e reslicing

Il processo di interpolazione è di particolare rilevanza nell’elaborazione di immagini 3D, nelle quali tipicamente la distanza inter-fetta è maggiore della risoluzione del pixel sulla singola fetta. Per ottenere un volume isotropo è necessario eseguire una operazione di interpolazione 3D detta reslicing.

Il reslicing viene anche utilizzato per ricostruire piani di vista diversi da quelli acquisiti.

Si noti che gli algoritmi standard di interpolazione presuppongono pixel quadrati e voxel cubici, ipotesi non sempre valida nelle immagini biomediche.

---

## Filtraggio dell’immagine biomedica

Gli algoritmi di filtraggio consentono l’elaborazione dell’immagine biomedica in modo da modificarne le caratteristiche. Seguendo l’approccio della computer vision, possiamo definire filtraggio una operazione che cambia il contenuto informativo dell’immagine senza estrarre informazioni topologiche.

Il filtraggio ha due possibili scopi principali: migliorare la visualizzazione dell’immagine e predisporre l’immagine per elaborazioni successive.

### Operazioni di filtraggio

Le operazioni di filtraggio si classificano in:

- Puntuali
- Locali
- Globali

### Operazioni puntuali

Le operazioni puntuali trasformano i livelli di grigio secondo una funzione u = t(v). Tali operazioni possono essere implementate tramite lookup table.

Esempi di trasformazioni puntuali sono:

- inversione dei livelli di grigio
- windowing
- operazioni di tipo gamma
- binarizzazione

### Windowing

Le immagini biomediche sono in generale codificate su 16 bit, mentre i dispositivi di visualizzazione supportano solo 256 livelli di grigio. Il processo di adattamento prende il nome di windowing.

Il formato DICOM contiene i parametri necessari a pilotare il windowing, tra cui Window Centre e Window Width.

---

## Operazioni locali e filtraggio spaziale

L’operazione di base nel filtraggio locale è la convoluzione spaziale. Dato un kernel, il valore del pixel filtrato è una combinazione pesata dei pixel vicini.

I filtri di smoothing riducono il rumore ma introducono sfocamento dei contorni.

I filtri derivativi (Sobel, Prewitt, Laplaciano) permettono di evidenziare i bordi dell’immagine.

### Filtraggio adattivo

I filtri adattivi modulano il filtraggio in base al contenuto locale dell’immagine. Un esempio è il filtro di Wiener, che preserva i contorni riducendo il rumore nelle regioni omogenee.

---

## Equalizzazione dell’istogramma

L’equalizzazione dell’istogramma è un esempio di filtraggio globale che mira a rendere uniforme la distribuzione dei livelli di grigio, aumentando il contrasto percepito.

---

## Compressione dell’immagine biomedica

La compressione riduce la quantità di dati necessaria per memorizzare o trasmettere immagini biomediche. Si distingue tra compressione lossless e lossy.

Il formato DICOM supporta diversi algoritmi di compressione, tra cui JPEG, JPEG Lossless e JPEG 2000.

---

## Super-Resolution

Le tecniche di super-resolution combinano più immagini a bassa risoluzione della stessa struttura anatomica per stimare una immagine ad alta risoluzione.

Il problema è un tipico problema inverso, risolto tramite algoritmi iterativi come l’iterated back-projection, spesso con l’introduzione di termini di regolarizzazione.

