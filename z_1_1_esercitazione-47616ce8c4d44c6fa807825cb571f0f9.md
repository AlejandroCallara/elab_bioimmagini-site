# Esercitazione 1
## Teoria 
La misura del rumore associato ad una immagine biomedica è un punto fondamentale in molte applicazioni, come l’analisi della qualità di immagine e la realizzazione di filtri adattivi. Per la misura del rumore sono stati sviluppati vari approcci, che possono essere divisi in due categorie: 
1. Basati sul tracciamento di ROI 
2. Basati sull’analisi dell’istogramma 

Nei metodi basati su ROI, una o più ROI vengono tracciate sui tessuti di interesse, definendo delle zone omogenee ed evitando che la misura sia influenzata dal PVE o dalla presenza di attenuazione. Il rumore sul tessuto viene stimato come la deviazione standard (SD) del segnale sulla ROI. Le ROI possono essere definite manualmente o automaticamente attraverso algoritmi di segmentazione che saranno illustrati nel seguito. La definizione automatica è molto semplificata nel caso di fantocci per la misura della qualità dell’immagine di cui è nota la forma e posizione nello scanner. 

Nella misura attraverso ROI il punto fondamentale è il compromesso tra grandezza della ROI e necessità di posizionare la ROI in una regione omogenea.

In Figura 1.30 osserviamo le misure di Media e SD del segnale su ROI di dimensioni crescenti (100 realizzazioni di rumore, ROI quadrate). Si osserva chiaramente come la misura della SD sia più critica rispetto a quella del valor medio e come per la SD siano necessarie ROI con qualche decina di pixel per avere una misura affidabile. Da queste considerazioni derivano gli approcci a sottrazione di immagini e misura del rumore sul fondo utilizzati nella pratica clinica con lo scopo di aumentare il numero di pixel disponibili per la stima della SD.

<img src="./images/image-30.png" alt="M_SD_ROI" style="width:100%;">

*Figura 1.30. Stima di Media e SD su ROI al variare della dimensione della ROI.*

Consideriamo ad esempio il caso in Figura 1.31 di immagini cardiache in LGE (Late Gadolinium Enhancement). Lo scopo di questo tipo di immagini con contrasto è verificare l’estensione dell’infarto miocardio cronico, dove si deposita il contrasto, che appare bianco, mentre il miocardio sano appare con segnale annullato (in realtà avremo sempre un segnale diverso da zero per la natura del rumore MR). Per valutare la validità clinica delle immagini bisogna valutare il CNR tra la zona di LGE (cicatrice da infarto, ROI 1) ed il miocardio sano (ROI 2) (Figura 1.32).

<img src="./images/image-31.png" alt="LGE" style="width:100%;">

*Figura 1.31. Immagini cardiache in LGE.*

<img src="./images/image-32.png" alt="ROI_LGE" style="width:100%;">

*Figura 1.32. Tracciamento di ROI su immagini cardiache.*

Come si vede le regioni in cui tracciare le ROI sono molto piccole, l’area LGE è di 60 mm^2 e l’area miocardio di 140 mm^2, corrispondenti (pixel size = 1.5 mm) a 26 e 60 pixel circa. La misura della SD sarà quindi inaffidabile. E’ molto più affidabile una misura fatta nello sfondo che permette di tracciare una ROI di 3350 mm^2 (1550 pixel circa). In questo caso essendo il miocardio annullato (segnale zero) e quindi con distribuzione di rumore simile al fondo il valore di CNR sarà:

$$
CNR=2(M_{LGE}-M_{MIO})/(SD_{BK}+1.526S_{BK})
$$

I metodi basati sull’analisi dell’istogramma cercano invece di caratterizzare il rumore dall’immagine nel suo complesso. In questo caso si utilizza tutta l’informazione contenuta nell’immagine per cui il problema visto nelle ROI non si pone. Consideriamo l'immagine ideale a sei pattern corrotta solo da rumore gaussiano ed il suo istogramma riportati in Figura 1.33.

<img src="./images/image-33.png" alt="imm_isto" style="width:100%;">

*Figura 1.33. Fantoccio a 6 pattern con rumore Gaussiano e relativo istogramma.*

Se tracciamo una ROI all’interno di un pattern la SD del segnale nella ROI sarà sempre la stessa dipendendo dal rumore sommato all’immagine. Se invece tracciamo una ROI a cavallo di due pattern misureremo un valore più alto dovuto alla presenza di due tessuti con segnali diversi all’interno della ROI.

<img src="./images/image-34.png" alt="sd_isto" style="width:100%;">

*Figura 1.34. Mappa di SD e relativo istogramma.*

Automatizzando la procedura, possiamo far scorrere sull’immagine un kernel (ad esempio 5x5) che rappresenta la ROI e valutare il valore della deviazione standard SD sulla regione di immagine coperta dal kernel. In MATLAB questa operazione è implementata dalla funzione stdfilt. Si otterranno quindi un numero di valori di SD uguale al numero di pixel dell’immagine. Nelle regioni all’interno dei pattern uniformi il valore della SD sarà pari alla deviazione standard del rumore σ, almeno nel caso di rumore gaussiano additivo. Nelle regioni a cavallo di due pattern il valore di SD sarà maggiore di σ. Come al solito la dimensione del kernel rappresenterà un compromesso tra l’esigenza di calcolare correttamente la SD e la necessità di avere la maggior parte delle misure eseguita su regioni omogenee. 
Come si vede dalla figura, l’istogramma della mappa SD avrà un picco in corrispondenza del valore di $\sigma$ ed una serie di valori più alti corrispondenti alle transizioni distribuiti in modo uniforme. 
Se l’immagine è composta prevalentemente da regioni omogenee (pochi pattern di forma regolare) si può supporre che il contributo delle regioni di transizione sia trascurabile e calcolare $\sigma$ come media della mappa: 

$$
\sigma = mean(M_{SD})
$$

In realtà è preferibile eliminare gli outliers dovuti ai bordi e computare la mediana della mappa SD invece che la media, cosa che consente di ottenere una stima più accurata. Infatti, al contrario della media, la mediana pesa meno gli alti valori di SD nella parte destra dell’istogramma. 

$$
\sigma = median(M_{SD})
$$

Se l’immagine presenta molte transizioni il metodo della mediana può dare risultati non corretti. In questo caso è possibile utilizzare l’istogramma calcolando il valore di σ dal picco principale (tipicamente il primo) dell’istogramma che come visto contiene l’informazione sulle regioni omogenee. Il valor medio di σ misurato sarà dato dalla posizione sull’asse $x$ del massimo valore dell’istogramma $h$. Cercheremo quindi il massimo dell’istogramma e porremo $\sigma$ uguale al valore di $x$ corrispondente a tale massimo: 

$$
\sigma = x:h(x)=max(h)
$$

In Figura 1.35 è riportata la stima della SD per i diversi metodi. 

<img src="./images/image-35.png" alt="stima_sd" style="width:100%;">

*Figura 1.35. Valori stimati di SD per diversi metodi.*

Questo approccio è simile ad un altro metodo utilizzato per la stima del rumore nel quale l’immagine viene divisa in $N$ quadrati di lato $k$ non sovrapposti di cui viene calcolata la SD, ed il valore di $\sigma$ viene stimato come media degli m campioni con valore di SD più basso (al limite un singolo campione). Il metodo basato sull’istogramma è comunque in generale più accurato. 
Come si osserva dal grafico il metodo Mean sovrastima il valore del rumore, mentre gli altri due metodi danno una stima sostanzialmente corretta. 
Tutti questi metodi si basano sull’assunzione di rumore gaussiano additivo. Se come avviene tipicamente nelle immagini biomediche questa assunzione non è verificata, bisogna operare sulla base della conoscenza del processo di acquisizione. Consideriamo ad esempio l’immagine MR di un phantom cilindrico, composto da tre cilindri concentrici, due riempiti con acqua e quello intermedio con olio (valore di segnale più alto). Essendo il fantoccio riempito con liquido perfettamente omogeneo il rumore biologico è trascurabile. Le dimensioni del fantoccio sono piccole rispetto al bore della macchina MR (15 cm circa) e quindi si può considerare nulla l’attenuazione. Come si osserva nell’immagine a destra ottenuta con una opportuna finestra di windowing, alcuni pixel dell’immagine non sono stati ricostruiti dal K-spazio ma rappresentano un “riempimento” per ottenere una immagine quadrata (zero-padding). A tali pixel in MR viene assegnato il valore convenzionale 0 e devono essere ignorati nell’elaborazione (Figura 1.36).
                                
<img src="./images/image-36.png" alt="sd_e_zero_padding" style="width:100%;">

*Figura 1.36. Zone di zero-padding evidenziate.*

Come sappiamo il valore di $\sigma$ può essere stimato tracciando una ROI su un tessuto omogeneo oppure sul fondo introducendo un opportuno fattore di correzione che è noto trattandosi di una immagine MR. 
Procedendo in questo modo otteniamo i valori di $\sigma$ per acqua e olio (uguali a meno dell’errore sperimentale) e del fondo che come ci aspettavamo è più basso. Dal valore stimato sul fondo possiamo ottenere la stima corretta applicando il fattore di conversione 1.526 (Figura 1.37).

<img src="./images/image-37.png" alt="stima_sd_ROI" style="width:100%;">

*Figura 1.37. Stima di SD basato su ROI.*

Se vogliamo applicare i metodi automatici basati sull’istogramma dobbiamo calcolare la mappa SD ed il relativo istogramma, dove abbiamo eliminato dal computo i pixel a valore nullo (zero padding) (Figura 1.38).

<img src="./images/image-38.png" alt="stima_SD_automatica" style="width:100%;">

*Figura 1.38. Stima della SD automatica.*

Come vediamo dal grafico abbiamo un singolo picco, che è però la combinazione dei valori di SD del fondo e di quelli dei due tessuti ad alto SNR. Il calcolo della media della mappa SD sovrastima fortemente $\sigma$, in quanto gli alti valori di SD sui bordi alzano la media in modo significativo. Il valore della mediana è abbastanza simile alla SD del rumore di fondo, che rappresenta una parte rilevante dell’immagine. Considerando il massimo dell’istogramma si ha un valore addirittura minore alla SD del fondo. Tali valori dipendono dal rapporto tra numero di pixel del fondo e numero di pixel ad alto SNR e non sono generalizzabili. I valori di SD per i tre metodi sono disponibili nella seguente tabella:

SD mean | SD median | max hist
---|---|---
22.3108|10.0968|7.8155
 
In questo caso sarà necessario identificare in qualche modo le due classi di pixel attraverso un opportuno algoritmo di segmentazione. Banalmente, se consideriamo solo i pixel con livello di grigio superiore a 100 (quindi solo acqua e olio, vedremo in seguito come ottenere tale soglia) abbiamo come istogramma della SD quello di Figura 1.39.


<img src="./images/image-39.png" alt="esempio_SD" style="width:100%;">

*Figura 1.39. Mappa di deviazione standard e relativo istogramma.*

SD mean | SD median | max hist
---|---|---
69.5255|15.7743|12.3800

Come si osserva la stima con i metodi median e max hist migliora in modo significativo. In definitiva per la valutazione automatica del rumore nell’immagine biomedica occorre una scelta oculata dell’algoritmo sulla base delle caratteristiche dell’immagine.

## Esercitazione
Lo scopo dell’esercitazione è replicare le misure di qualità dell’immagine che vengono eseguite in un laboratorio MR in modo routinario. L’immagine a disposizione è quella di un fantoccio sferico utilizzato per la valutazione del rapporto segnale rumore e dell’uniformità di segnale (quindi una valutazione dell’eventuale presenza di attenuazione) (Figura 1.40). 
L’immagine è stata acquisita su una macchina MR Signa HDxt General Electric a 3 Tesla, come si può osservare dall’header DICOM. Il valore del campo PatientID = ‘geservice’ indica che le immagini sono state acquisite per un controllo di qualità. L’immagine è una immagine 2D Fast Spin Echo. Il fantoccio è una sfera con diametro 26 cm acquisita in modo da ottenere la massima sezione. Il FOV dovrebbe essere centrato sul centro della sfera, in realtà è abbastanza disassato ed ha il centro intorno a (233 253). 

<img src="./images/image-40.png" alt="fantoccio" style="width:100%;">

*Figura 1.40. Fantoccio.*

Per la definizione del protocollo di misura ci riferiamo al Protocollo definito dalla CONSIP per l’esecuzione dei controlli di qualità su scanner MR. La CONSIP è la centrale acquisti per la pubblica amministrazione italiana, e gestisce quindi anche gli acquisti nella sanità pubblica. In particolare, la CONSIP dovrebbe riuscire a migliorare la qualità degli acquisti e ridurre i costi grazie all’aggregazione della domanda sul tutto il territorio nazionale. Nella pratica con cadenza periodica (tipicamente due anni) la CONSIP bandisce una gara per l’acquisizione di apparecchiature mediche e valuta la qualità delle apparecchiature oltre che il costo proposto dai fornitori partecipanti. In base alla qualità rilevata e al costo viene selezionato un fornitore al quale nei due anni successivi si dovrà rivolgere preferibilmente ogni struttura sanitaria pubblica. Il protocollo, che è pubblico, è disponibile come materiale integrativo del corso. 
Per la misura dell’SNR il protocollo è tipicamente strutturato come: 
1. Definire sull’immagine una ROI (ROI75) posizionata al centro dell’oggetto test di dimensioni pari al 75% dell’oggetto. Determinare il valor medio del segnale nella ROI75. 
2. Definire una ROI (ROI10) in una zona priva di segnale (fondo) di dimensioni pari al 10% dell’oggetto. Determinare il valor medio del segnale nella ROI10 (valore di baseline). 
3. Determinare il segnale S come differenza dei valori di segnale tra ROI75 e ROI10. 
4. Valutare il rumore N sull’immagine come la deviazione standard del segnale all’interno della ROI75 
5. Calcolare il valore di SNR come SNR=S/N.

Il motivo per cui il protocollo prevede il calcolo della baseline (punto 2) è che il produttore della macchina potrebbe sommare arbitrariamente un valore costante all’immagine facendo salire S e quindi il valore di SNR senza modificare il contrasto. Quindi in realtà la procedura misura il valore di CNR tra il fantoccio ed il fondo.
Dovremo quindi realizzare un programma MATLAB che implementi in modo automatico la procedura del protocollo assicurando che vengano rispettate le indicazioni, in particolare quelle sulle dimensioni delle ROI e che fornisca in uscita il valore di SNR. 

Il MATLAB fornisce varie funzioni per il tracciamento di ROI. La più semplice è la funzione `getrect` che permette di tracciare una ROI rettangolare con il mouse su una immagine. Funzioni più complete sono quelle del toolbox “ROI-Based Processing” come `drawfreehand` che permette di tracciare una ROI a mano libera, `drawcircle` (cerchio), `drawellipse` (ellisse), `drawpolygon` (poligono generico), `drawrectangle` (rettangolo), etc. In Python, funzionalità analoghe sono fornite da librerie di visualizzazione e image processing.
In particolare, la libreria **matplotlib** permette di tracciare ROI rettangolari interattive
tramite widget dedicati, mentre la libreria **napari** offre strumenti più completi per il
disegno e la gestione di ROI interattive. Napari consente il tracciamento di ROI a mano libera, circolari, ellittiche, poligonali e
rettangolari, in modo analogo alle funzioni `drawfreehand`, `drawcircle`, `drawellipse`,
`drawpolygon` e `drawrectangle` del toolbox *ROI-Based Processing* di MATLAB.

Il calcolo dell’uniformità, e quindi della presenza di un campo di attenuazione, viene effettuato con la seguente procedura: 
1. Definire sull’immagine una ROI (ROI80) posizionata al centro dell’oggetto test di dimensioni pari al 80% dell’oggetto. 
2. Determinare il valor medio del segnale nella ROI80 (Sm) ed il numero di pixel contenuti nella ROI80 (N) 
3. Determinare la deviazione media assoluta AAD = 𝐴𝐴𝐷=Σ|𝑆𝑖−𝑆𝑚|𝑁𝑖 dove Si è il valore di segnale dei singoli pixel contenuti nella ROI80 
4. Calcolare UH = 1-ADD/Sm 

In assenza di rumore, se non c’è attenuazione ADD=0 e UH=1 (massima uniformità, nessuna attenuazione). Se c’è attenuazione, ci sarà una variazione di segnale e ADD sarà maggiore di zero abbassando il valore di UH. Per un’immagine reale ci sarà presenza di rumore e ADD sarà comunque diverso da zero in quanto il segnale varierà. Il presupposto della misura è che Sm sarà molto alto, in quanto il fantoccio sarà costruito a questo scopo, per cui in assenza di attenuazione UH sarà molto vicino ad uno. 
Anche in questo caso dovremo realizzare un programma MATLAB automatico che implementi la procedura del protocollo assicurando che vengano rispettate le indicazioni, in particolare quelle sulle dimensioni delle ROI, e che fornisca in uscita il valore di UH. 
Infine, andiamo a valutare la presenza di PVE sull’immagine 1 del fantoccio calcolando l’acutezza della transizione. A questo scopo occorre definire un profilo, cioè un segmento posto sull’immagine che intersechi una transizione, ed estrarre il grafico del valore di segnale sul profilo, come esemplificato in figura per il software ImageJ.
 
Dato il profilo, il valore di acutezza si ottiene utilizzando la formula:
A=1/(f(b)-f(a)) ∫_a^b▒〖[d/dx f(x)]^2 dx〗
Dove f è il profilo estratto e a e b sono l’inizio e la fine della transizione. Essendo l’immagine discreta, l’integrale viene approssimato come:
A=1/(f(b)-f(a)) ∑_(i=a)^b▒[(f(i+1)-f(i))/d]^2 
Dove d è la dimensione del pixel. I punti a e b (inizio e fine della transizione) possono essere definiti manualmente, ma è opportuno automatizzare la procedura per ridurre la variabilità dovuta all’osservatore. Da un punto di vista teorico, all’inizio del profilo il segnale avrà un valore dato dalla baseline (valor medio del fondo) più il rumore con SD sigma. Potremmo definire una soglia conservativa (come baseline+4sigma) al di sopra della quale è estremamente improbabile che il segnale misurato sul profilo sia originato da un pixel del fondo. Analogamente potremmo definire una soglia per la fine della transizione. Nella pratica si utilizza un approccio più semplice, ad esempio, possiamo convenire che a e b siano i punti in cui il segnale raggiunge il 10% ed il 90% del suo valore massimo teorico misurato in precedenza (Sm), rispettivamente. Andranno quindi fissate due soglie 0.1*Sm e 0.9*Sm in modo da trovare a e b. 

Dovremo realizzare quindi un programma MATLAB che implementi il calcolo dell’acutezza su di un profilo definibile dall’utente o calcolato in modo automatico.

Risultati attesi 
Il valore di SNR è circa uguale a 8. 
UH circa 0.90 
Acutezza circa 415
