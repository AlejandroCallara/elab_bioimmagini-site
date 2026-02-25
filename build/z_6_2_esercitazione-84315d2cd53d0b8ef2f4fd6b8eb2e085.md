# Esercitazione 6.2: CNN (segmentazione)

Lo scopo dell’esercitazione è utilizzare una convolutional neural network (CNN) per la segmentazione di immagini biomediche. In particolare verrà utilizzata una U-Net 2D per segmentare la materia cerebrale.


# Data Set

I dati provengono da un simulatore MR, in particolare il simulatore MR cerebrale brainweb (https://brainweb.bic.mni.mcgill.ca/) (Figura 6.e2.1). Il simulatore parte da un atlante anatomico (corrispondente all'immagine ideale del modello dell'immagine biomedica) e permette di definire il tipo di sequenza (T1, T2, PD), il thickness della fetta, il rumore e la disomogeneità del campo RF. Per un problema di segmentazione, l'atlante rappresenta la maschera di riferimento in cui il segnale di un pixel è la label del tessuto, mentre le immagini simulate sono le immagini da segmentare. 
Nel nostro caso utilizziamo i dati tratti da un database di 10 cervelli normali (https://brainweb.bic.mni.mcgill.ca/anatomic_normal_20.html)
Come visto in precedenza, i dati brainweb sono disponibili in formato MINC. In questa esercitazione sono stati convertiti in formato nii.gz che è un formato più usato nelle applicazioni di deep learning in quanto permette di lavorare con dati compressi. 

<img src="./images/Figura_ese_6_5.png" alt="Immagini MR brainweb" style="width:100%;">

*Figura 6.e2.1. Immagini MR brainweb.*

I dati ottenuti sono contenuti nella cartella BrainWebData. I dati “crisp_v” contengono l’atlante anatomico, i dati “t1w_p4” l’immagine MR simulata. I dati crisp di ogni soggetto consistono in un volume 3D 434x362x362. Per ogni volume abbiamo 362 slice, ognuna di dimensioni 362x434. Il volume è isotropo con dimensioni del voxel 0.5x0.5x0.5 mm. La simulazione è stata realizzata partendo da un set di dati fuzzy a 12 classi, quindi il volume crisp è caratterizzato da 12 possibili valori (0=Background, 1=CSF, 2=Gray Matter, 3=White Matter, 4=Fat, 5=Muscle, 6=Muscle/Skin, 7=Skull, 8=vessels, 9=around fat, 10=dura matter, 11=bone marrow) che definiscono le 12 mappe di appartenenza dei vari tessuti. Il set di dati “t1w_p4” comprende per ogni soggetto un'immagine T1 simulata 3D di dimensioni 256x256x181, volume isotropo con dimensione del pixel di 1 mm. 

In Figura 6.e2.2 sono riportati per una slice centrale del volume un esempio della mappa di appartenenza, della regione del cranio (valore della mappa diverso da zero), della materia bianca (valore della mappa =3) e l'immagine T1 corrispondente. 

<img src="./images/Figura_ese_6_6.png" alt="Maschere binarie estratte dall'atlante brainweb e immagine T1 corrispondente." style="width:100%;">

*Figura 6.e2.2. Maschere binarie estratte dall'atlante brainweb e immagine T1 corrispondente.*

Come si osserva, il FOV della mappa e dell’immagine generata non sono uguali, per cui è necessaria una registrazione per avere l’allineamento ed un’interpolazione per avere la stessa risoluzione spaziale sulle immagini e sulle maschere. Questo task può essere realizzato con una registrazione “DICOM-based”, nella quale si calcolano le coordinate dei voxel delle due immagini rispetto allo spazio di riferimento del paziente e si esegue un'interpolazione.

Essendo le immagini e le maschere in formato nii.gz, dobbiamo estrarre da questo formato i dati di interesse, utilizzando la libreria **nibabel** che consente di caricare numerosi formati utilizzati per la codifica di immagini biomediche. 

In particolare, con `nibabel.load` possiamo caricare l’immagine nii.gz. Otteniamo una struttura con vari campi che contiene un subset delle informazioni contenute nel DICOM. In particolare sono di interesse:

affine:
[[   0.      0.      .5   -127.75] \
 [   0.      . 5     0.   -145.75] \
 [   .5      0.      0.    -72.25] \
 [   0.      0.      0.      1.  ]]

contiene l' "image position" del volume rispetto al riferimento pazient (l'ultima colonna) e i versori “patient orientation” (vx, vy, vz). Differentemente dal DICOM i versori non sono normalizzati, quindi includono il voxel size. Nel nostro caso le immagini sono assiali, quindi una sola componente dei versori è diversa da zero. Quindi possiamo semplicemente scorrere il volume sugli assi principali.

header.get_data_shape() ritorna le dimensioni del volume (slices, righe, colonne)\
header.get_zooms() ritorna il pixel size (è comunque quello già presente in affine)\
get_fdata() ritorna l'immagine 3D come un array

Definiti i dati necessari, possiamo effettuare la registrazione dicom-based computando le coordinate del voxel dei due volumi rispetto allo spazio paziente ed eseguendo un'interpolazione 3D (ad esempio con scipy.interpolate.interpn). In pratica conviene calcolare le coordinate di tutti i voxel del volume crisp e del volume MR ed interpolare il volume crisp sulle stesse coordinate del volume MR. Per non corrompere la maschera bisogna usare un interpolatore NN. In questo modo si ottiene una maschera 256x256x181.

Ottenuti i nuovi volumi, li possiamo salvare in nii-gz con nibabel o come array numerici con numpy save

## Creazione Dati di Addestramento

L’approccio più efficiente per ottenere una buona segmentazione in presenza di volumi isotropi come quelli in oggetto sarebbe utilizzare una U-Net 3D per preservare le informazioni spaziali tridimensionali (Figura 6.e2.3). Questo approccio non è applicabile in questo caso per il numero limitato di soggetti e per motivi computazionali (l’uso della U-Net 3D richiede GPU dedicate). Utilizziamo quindi un approccio 2D segmentando le slice in modo indipendente le une dalle altre, anche se questo approccio dà in generale risultati peggiori. L’approccio 2D può essere facilmente esteso all’approccio 2.5 D, dove dal volume vengono estratte le slice nelle tre dimensioni (assiale, coronale e sagittale) che vengono processate in parallelo da tre modelli e si fa un’operazione di consensus sulle tre maschere risultanti (votazione a maggioranza) o si uniscono le maschere con un OR logico. Noi useremo solo le slice assiali. 

<img src="./images/Figura_ese_6_7.png" alt="Approcci 3D e 2.5D per la segmentazione di immagini volumetriche" style="width:100%;">

*Figura 6.e2.3. Approcci 3D e 2.5D per la segmentazione di immagini volumetriche.*

I nostri dati saranno quindi 10x181=1810 slices con le rispettive maschere. Vogliamo condurre due esperimenti di segmentazione, uno “facile” sul cranio e uno “difficile” sulla materia bianca. Avremo bisogno quindi di due classi di maschere,  una con le maschere di tutto il cranio, cioè i pixel di classe diversa da 0 (Background), e una con le corrispondenti maschere della materia bianca, cioè i pixel di classe 3 (White Matter).  

Dobbiamo dividere i dati in training, validation e test set. Un punto fondamentale è assicurare che slice dello stesso soggetto non siano presenti contemporaneamente in training e validation o in training e test. In questo caso avremmo infatti il fenomeno del “**data leakage**”, nel quale in fase di validazione o di test il modello addestrato “riconosce” un’immagine molto simile a quella usata nell’addestramento (nel nostro caso una slice contigua). In linea generale, validation e test set devono rappresentare dati “**unseen**” (mai visti) dal modello addestrato. La divisione dei dati va quindi fatta a livello paziente (“**patient level**”).

Avendo 10 pazienti, possiamo selezionare casualmente 8 pazienti nel training set, 1 nel validation set, e 1 nel test set. In questo caso la lettura on-the-fly non è banale, quindi rigeneriamo le immagini da usare nell’addestramento. 

Per ogni paziente dobbiamo estrarre le slice e salvarle su disco in un formato a scelta (nii.gz, numpy, etc) in delle sottocartelle in modo che il dataloader le legga correttamente. Dovremmo ottenere una gerarchia di cartelle del tipo in Figura 6.e2.4, con 181 immagini per cartella nel validation e test e 1448 nel training.

<img src="./images/Figura_ese_6_8.png" alt="Struttura dell'archivio dati" style="width:50%;">

*Figura 6.e2.4. Struttura dell'archivio dati.*

## Addestramento

Costruito il database (che chiaramente viene generato una singola volta e poi riutilizzato) possiamo addestrare l’algoritmo di segmentazione come nel notebook  {doc}:`simple_Unet_test3`.  In questo caso, avendo già definito training, validation e test a livello di paziente, non dividiamo casualmente i dati, ma carichiamo separatamente il training, validation e test nei loro dataLoader. In fase di training dovremo inserire uno shuffle per “rimescolare” i dati che ora sono ordinati per paziente.

Rispetto al notebook di esempio, gli iperparametri della U-Net e quelli di addestramento andranno opportunamente ottimizzati. Su un computer senza GPU compatibile può essere opportuno diminuire la dimensione delle immagini/maschere on-the-fly.

Per la segmentazione del cranio si dovrebbero ottenere risultati ottimali (Dice > 0.95) come in Figura 6.e2.5.

<img src="./images/Figura_ese_6_10.png" alt="Esempio di segmentazione del cranio" style="width:100%;">

*Figura 6.e2.5. Esempio di segmentazione del cranio.*

Per la segmentazione della materia bianca, la qualità della segmentazione dipenderà fortemente dal numero di iterazioni che, a sua volta dipende dalla capacità computazionale.  Si dovrebbero comunque ottenere valori di DICE intorno a 0.9. In figura è riportato un esempio di segmentazione con Dice=0.89, come si vede anche con questo valore di DICE il risultato non è di altissima qualità (Figura 6.e2.6).      

<img src="./images/Figura_ese_6_11.png" alt="Esempio di segmentazione della materia bianca" style="width:100%;">

*Figura 6.e2.6. Esempio di segmentazione della materia bianca.*

























 
        










