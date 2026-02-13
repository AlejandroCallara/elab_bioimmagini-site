# Capitolo 6: Machine Learning per l'Analisi delle Bioimmagini 

La classificazione di immagini nell’ambito della Computer Vision è considerata un’operazione ad alto livello. Nella classificazione di immagini, a un’immagine viene associata un’etichetta (**label**) che specifica a quale classe appartiene l’immagine. Si tratta quindi di un'operazione che associa ad un'immagine un concetto (visual class), in generale semplice per un essere umano ma non banale per un algoritmo. In campo biomedico le label hanno tipicamente un significato diagnostico, legato alla patologia del paziente. Un sistema di classificazione delle bioimmagini si configura quindi come un sistema di diagnosi computerizzata.  
L’approccio classico alla risoluzione del problema della classificazione di immagini prevede un approccio **model-based**: \
•	L’estrazione dell’oggetto dalla scena (segmentazione)\
•	L’estrazione della struttura dell’oggetto (ad esempio skelethonization)\
•	Il confronto della struttura con una libreria di strutture note. Questo passo comporta una procedura di registrazione in quanto il punto di vista può cambiare.

Il problema fondamentale di questo approccio non è la procedura di segmentazione che pure è non banale, ma la creazione della libreria di strutture di riferimento.
Si pensi al problema banale di riconoscere un'immagine che rappresenta una sedia (Figura 6.1). Innanzitutto, per riconoscere una sedia dovremmo definire in modo rigoroso il concetto stesso di sedia, cosa niente affatto banale.

<img src="./images/Figura_6_1.png" alt="Esempio di Immagini da Classificare (sedia)" style="width:100%;">

*Figura 6.1. Esempio di Immagini da Classificare (sedia) [^c6_1].*

[^c6_1]:Il materiale di questo capitolo è in parte adattato dal corso “Convolutional Neural Networks for Visual Recognition” dell’università di Stanford (https://cs231n.github.io/).

Esistono innumerevoli strutture che corrispondono al concetto di sedia, in quanto implementano la stessa funzione. In definitiva, la nostra definizione di sedia dovrebbe essere qualcosa del tipo: Oggetto mobile abbastanza rigido che permette ad una singola persona di sedersi appoggiando la schiena ad una spalliera.
Sicuramente esistono dei controesempi di sedie che non rispettano questa definizione e di altri oggetti che la rispettano e non sono sedie, ma anche dando per buona la definizione alcune caratteristiche di una sedia come la rigidezza e la mobilità non sono ricavabili dall’immagine della sedia in nessun modo. Quindi l’approccio classico funziona solo per problemi semplici (ad esempio riconoscimento di testo).

Per problemi più complessi un approccio di tipo algoritmico è spesso inadeguato e si adotta un approccio **data-driven** (guidato dai dati) che utilizza un approccio machine-learning. Di fatto si imita il processo di apprendimento umano che è basato sull’imparare attraverso esempi (Figura 6.2).

<img src="./images/Figura_6_2.png" alt="Esempio di Training set di immagini" style="width:100%;">

*Figura 6.2. Esempio di Training set di immagini [^c6_2].*

[^c6_2]:https://cs231n.github.io/classification

Partiamo quindi da un certo numero di immagini già classificate (nell’esempio quattro classi). Il classificatore dovrà utilizzare queste immagini e le rispettive etichette (**label**) per imparare (**machine learning**) ad effettuare la classificazione. Questo insieme di immagini “labellate” rappresenterà il training set dell’algoritmo. Per testare il classificatore, sottoporremo al classificatore stesso un insieme di immagini analogo al precedente (**test set**) le cui label sono ignote al classificatore, e verificheremo che il classificatore sviluppato funzioni correttamente confrontando le label corrette (**Ground Truth**, **Reference**) e quelle identificate dal classificatore. 
Illustriamo con un semplice esempio la differenza tra l’approccio **model-based** e l’approccio **data-driven** che caratterizza gli algoritmi di machine-learning. 

Consideriamo un problema di regressione lineare (Figura 6.3). Abbiamo una serie di $N$ coppie di dati $(X,Y)$ e vogliamo spiegare la relazione tra $X$ e $Y$. Introduciamo un **modello a priori** (modello lineare) che introduce la relazione: $Y=aX+b$, dove $a$ e $b$ sono i parametri (**parameters**) del modello. Attraverso un procedura di fitting minimizziamo l’errore tra dati e modello (**loss function**) trovando il valore di $a$ e $b$ che minimizza la loss. La procedura di fitting può essere eseguita in vari modi (tipi di ottimizzazione, definizione del search space, etc). Nel linguaggio del Machine Learning i parametri che definiscono la procedura di ottimizzazione vengono detti iperparametri (**hyperparameters**). Nel linguaggio del machine learning abbiamo addestrato (**training**) il modello lineare sulla base dei dati $(X,Y)$ per trovare il valore ottimo dei parametri ($a$ e $b$).    

<img src="./images/Figura_6_3.png" alt="Esempio di Training set di immagini" style="width:100%;">

*Figura 6.3. Esempio di fitting dei parametri di un modello.*

Noto il modello addestrato $Y=aX+b$, se abbiamo nuovi dati $X_t$ la cui corrispondente $Y_t$ non è nota, possiamo predire (**prediction**) il valore di $Y_t$ corrispondente attraverso il modello addestrato (Figura 6.4).     

<img src="./images/Figura_6_4.png" alt="Esempio di Training set di immagini" style="width:100%;">

*Figura 6.4: Esempio di predizione da un modello addestrato.*

La predizione sarà soddisfacente se i dati di training $(X,Y)$ sono rappresentativi del problema da risolvere, cioè se $(X_t,Y_t)$ sono “simili” a $(X,Y)$.
Questo tipo di approccio è detto **model-based**  perché il modello è “stretto”, cioè contiene delle ipotesi restrittive sulla relazione tra i dati. Infatti, il numero dei parametri (2) è piccolo. Supponiamo che tra $X$ e $Y$ ci sia una relazione di tipo cubico ($Y=aX^3+b$). Se proviamo ad addestrare il modello lineare avremo un valore alto della funzione di loss e la spiegazione dei dati sarà insoddisfacente. Per spiegare i nuovi dati dobbiamo cambiare modello (Figura 6.5). 

<img src="./images/Figura_6_5.png" alt="Esempio di Training set di immagini" style="width:100%;">

*Figura 6.5. Esempio di generalizzabilità di un modello.*

In termini di machine learning (**ML**), un approccio model-based (pochi parametri) è fortemente dipendente dalle ipotesi a priori fatte definendo il modello (lineare o cubico), per cui un approccio model-based non è **generalizzabile**.
Se vogliamo un modello più generalizzabile, dobbiamo aumentare il numero di parametri; ad esempio, potremmo scegliere un modello polinomiale di ordine $N$ che ha $(N+1)$ parametri: $Y = a + bX + cX^2 + dX^3 + … + X^n$. Rispetto ai modelli precedenti, il modello è più generale e permette di spiegare ambedue i data set visti in precedenza. Essendoci molti più parametri, trovare il loro valore ottimo con un algoritmo di ottimizzazione sarà più difficile e quindi in generale serviranno più dati. 

Riassumendo:

•	**Model-driven** \
o	Conoscenza all’interno del modello \
o	Non generalizzabile \
o	Pochi parametri (facile da ottimizzare) \
o	Modello sbagliato implica risultati sbagliati

•	**Data-driven** (Machine Learning) \
o	Conoscenza nei dati \
o	Generalizzabile \
o	Molti parametri (difficile da ottimizzare) \
o	Dati non rappresentativi implicano risultati sbagliati

## Nearest Neighbor Classifier

L’esempio più semplice di classificatore è il **Nearest Neighbor Classifier (NNC)**. L’NNC possiede in memoria il training set. Quando deve classificare una immagine, computa la distanza tra l’immagine da classificare e tutte le immagini del training set, dopodiché individua l’immagine a distanza minima e classifica l’immagine sotto esame con la stessa label. La distanza tra due immagini può essere definita con varie metriche come l’errore quadratico medio, la mutua informazione, etc. Visto che la posizione di un oggetto nell’immagine sotto esame può essere diversa da quella immagazzinata nel training set, è ragionevole far precedere il computo della distanza dalla registrazione delle due immagini, in modo da compensare la differenza di distanza dovuta al disallineamento. Oppure in alternativa il training set deve contenere tutte le possibili trasformazioni geometriche per ogni immagine rappresentativa. L’approccio NNC tipicamente fornisce prestazioni basse, per la difficoltà di individuare una metrica efficiente, e presenta un’alta complessità computazionale. Infatti, per far funzionare bene il classificatore bisogna avere un training set molto grande, ma questo comporta un numero di confronti molto alto. Inoltre, è necessaria una grande quantità di memoria per immagazzinare il training set. Sia la complessità computazionale che la memoria necessaria sono proporzionali alla dimensione del training set e quindi alle prestazioni del classificatore.  

Il **k-Nearest Neighbor Classifier (k-NNC)** estende l’NNC considerando, invece dell’immagine a distanza minima, le $k$ immagini con distanza minore. Scelte tali immagini, si implementa una “votazione” (**consensus**) in cui ogni immagine vota la propria label e si sceglie la label che ottiene la maggioranza (relativa). Per $k=1$ il k-NCC corrisponde al NNC. La complessità computazionale dei due algoritmi è sostanzialmente equivalente se $k <<$ dimensioni del training set.

Supponiamo di voler utilizzare l’algoritmo k-NNC per classificare un insieme di immagini. Abbiamo bisogno di definire il valore ottimale di $k$ per ottenere la massima accuratezza di classificazione (tuning). Il parametro $k$ è detto iperparametro del classificatore. In generale i parametri che determinano il funzionamento della procedura di ricerca dei parametri del modello da ottimizzare sono detti iperparametri (**hyperparameters**). Per ottimizzare gli iperparametri andremo a provare varie combinazioni degli stessi cercando  di  ottenere  la  massima  accuratezza. 
Nel caso del k-NNC avendo un singolo parametro da ottimizzare potremmo provare vari valori di $K$ (ad esempio $k=1,...,10$) sul training set e prendere quello che ottimizza il funzionamento del classificatore. Fissato $K$ proveremo il classificatore così configurato sul test set.   
Un punto  importante  è  che  nella  fase  di hyperparameters tuning non è possibile utilizzare nel tuning dati appartenenti allo stesso test set che verrà usato alla fine per valutare le prestazioni del classificatore. In questo caso, infatti il test set verrebbe utilizzato di fatto come training set, creando il fenomeno dell’**overfitting** e quindi sopravvalutando le prestazioni del classificatore. Una regola generale dello sviluppo di algoritmi di machine learning è quindi: 

•	**Utilizzare il test set una ed una sola volta alla fine dello sviluppo**

Il modo corretto di procedere è quindi dividere il training set in due parti, il training set vero e proprio che contiene la maggior parte dei dati ed il **validation test** da utilizzare per l’ hyperparameters tuning. Per evitare la dipendenza del tuning dalla scelta del validation set, si può applicare la tecnica della **cross-validation**, che consiste nel creare diverse coppie training set/validation set ed effettuare il tuning sulla base della media delle prestazioni sulle diverse coppie.

Un esempio di implementazione dell'algoritmo NCC è nel notebook {doc}`n_6_1_NNC_example.ipynb`

## Classificatore Lineare
L’approccio k-NNC presenta due problemi fondamentali. Il primo è la necessità di mantenere in memoria tutto il training set, che può essere molto grande. Il secondo è la complessità computazionale, visto che per classificare un'immagine bisogna confrontarla con tutte le immagini del training set. Per questi motivi il k-NNC è un approccio poco efficiente.
Un approccio alternativo che è poi alla base delle reti neurali e delle CNN prevede di introdurre una funzione score che associa l’immagine ad un punteggio di appartenenza alle varie classi (simile all’appartenenza nel FCM) ed una funzione loss che quantifica l’abilità del classificatore nel riconoscere le label corrette.

Sia $D$ la dimensione dell’immagine espressa come numero di pixel e $K$ il numero di classi a cui può appartenere l’immagine. La funzione score sarà una funzione:

\begin{align}
f:R^D\rightarrow R^K
\end{align}

che mappa i $D$ pixel dell’immagine in un vettore a $R$ elementi che esprime l’appartenenza dell’immagine alle varie classi (Figura 6.6).

<img src="./images/Figura_6_6.png" alt="Concetto di classificatore di immagini" style="width:100%;">

*Figura 6.6. Concetto di classificatore di immagini.*

Essendo il numero di pixel $D$ dell’immagine molto maggiore di $K$ il classificatore opera un forte “sottocampionamento” dallo spazio dell’immagine allo spazio delle classi.  

Essendo il numero di pixel $D$ dell’immagine molto maggiore di $K$ il classificatore opera un forte “sottocampionamento” dallo spazio dell’immagine allo spazio delle classi.  
Il caso più semplice di funzione score è una funzione lineare (Figura 6.7), il cui uso definisce un classificatore lineare. In questo caso lo score associato ad un'immagine sarà:

\begin{align}
f(I,W,b) = WI+b
\end{align}

dove $I$ è l’immagine espressa come un vettore di $D$ elementi, $W$ è una matrice $K \times D$ e $b$ è un vettore di $K$ elementi. La funzione $f$ quindi mappa l’immagine $I$ con $D$ elementi in un vettore a $K$ elementi. La matrice $W$ rappresenta i pesi (**weights**) mentre $b$ rappresenta il **bias** vector. Ogni riga di $W$ rappresenta un classificatore con $D$ pesi associato ad una delle $K$ classi, quindi quando facciamo l’operazione $WI$ stiamo applicando a $I$ $K$ classificatori, uno per ogni classe. La coppia $(W,b)$ rappresenta i parametri del classificatore, che devono essere definiti nella fase di addestramento. Una volta definiti i parametri, la classificazione di un'immagine richiede solo una singola moltiplicazione di matrici più un'addizione, e quindi è molto veloce. Inoltre, il training set viene usato solo nella fase di addestramento ma non in quella di classificazione. Il classificatore ha quindi il vantaggio di ”spostare” la complessità computazionale dalla fase di classificazione vera e propria alla fase di training che viene effettuata una sola volta.

<img src="./images/Figura_6_7.png" alt="Concetto di classificatore lineare" style="width:100%;">

*Figura 6.7. Concetto di classificatore lineare.*

Il classificatore lineare include quindi le principali caratteristiche di un classificatore data-driven, e cioè: \
•	Struttura molto generale del modello del classificatore (linearità)\
•	Alto numero di parametri che rendono il classificatore generalizzabile\
•	Efficienza computazionale del classificatore

Analogamente agli algoritmi di clustering, possiamo pensare che le immagini rappresentino dei punti in uno spazio $D$-dimensionale. I parametri $W$ e $b$ rappresentano $K$ iperpiani che dividono lo spazio in due parti. Cambiando i parametri i piani si spostano e cambia la classificazione dei punti.
Una formulazione alternativa della classificazione lineare include il vettore di bias nella matrice $W$ per semplicità computazionale, aggiungendo a tutte le immagini un pixel fittizio di valore uno, come illustrato in Figura 6.8.

<img src="./images/Figura_6_8.png" alt="Concetto di classificatore lineare semplificato" style="width:100%;">

$Figura 6.8. Concetto di classificatore lineare semplificato.$

In questo modo è possibile utilizzare una forma più compatta:

\begin{align}
f(I,W,b) = WI
\end{align}

La **loss function** per il classificatore lineare può essere definita in vari modi.
Un tipo molto noto di classificatore lineare è il **Softmax** classifier, dove la funzione loss (**cross-entropy loss**) è definita per l’immagine i-esima come:

\begin{align}
L_i=-log(\frac{e^{f_{y_j}}}{\sum_j e^{f_j}})=-f_{y_i}+log\sum_je^{f_j}
\end{align}

$y_i$ è la label assegnata all’immagine i, $f_i$  è la score function assegnata all’immagine. 

la funzione $\frac{e^{f_{y_j}}}{\sum_j e^{f_j}}$  è definita **softmax function** (Figura 6.9), e mappa una serie di valori reali in una nuova serie nel range [0:1] e con somma 1. 

<img src="./images/Figura_6_9.png" alt="Concetto di classificatore lineare" style="width:100%;">

*Figura 6.9. Funzione Softmax.*

Il classificatore softmax può essere visto come un minimizzatore della cross-entropy tra le probabilità
$q$ che una un'immagine appartenga alle varie classi e la distribuzione “vera” $p$ in cui tutte le probabilità sono zero tranne quella relativa alla classe “giusta” che è 1. Infatti, la cross-entropy è definita come:

\begin{align}
H(p,q) = -\sum_j p_j log(q_j) = -log(\frac{e^{f_{y_j}}}{\sum_j e^{f_j}})
\end{align}

Demo : http://vision.stanford.edu/teaching/cs231n-demos/linear-classify/

La funzione loss definita in questo modo presenta un difetto, cioè il fatto che esistono infiniti valori dei parametri che minimizzano la loss. Infatti, se supponiamo di aver trovato un set di parametri $W$ che azzera la loss ($L=0$), ogni set di parametri $kW$ con $k>1$ darà anch’esso  $L=0$. 
Per risolvere questo problema, alla funzione loss viene aggiunta una componente di **regolarizzazione**, che privilegia i valori di $W$ più bassi. La componente di regolarizzazione è semplicemente la somma quadratica dei pesi $W$ (**norma L2**). Quindi la funzione loss complessiva diviene: 

\begin{align}
L = \frac{1}{N} \sum_i L_i + \lambda \sum_l \sum_k W_{l,k}^2
\end{align}

dove $N$ è la dimensione del training set.

Oltre a portare ad una soluzione “unica”, la regolarizzazione porta anche a trovare dei valori di W “distribuiti” lungo la dimensione del problema, in quanto a parità di somma dei pesi la norma $L2$ è minima se tutti i pesi sono uguali. Questo porta a un miglioramento della convergenza e a una riduzione dell’overfitting. Da un punto di vista pratico, l’iperparametro di interesse è sostanzialmente $\lambda$.
Una volta definiti gli score e la loss function, l’addestramento del classificatore è un problema di minimizzazione della funzione loss attraverso l’ottimizzazione degli iperparametri. 
Come visto in precedenza, un ottimizzatore locale parte da una condizione iniziale (cioè un certo valore dei parametri) e cerca nello spazio di definizione dei parametri (search space) il minimo locale della funzione (in questo caso L) variando via via il valore delle variabili partendo dalle condizioni iniziali. Consideriamo un esempio semplice in una singola dimensione. In questo caso abbiamo una funzione $y=f(x)$ e vogliamo trovare il minimo di $y$ partendo da un valore iniziale $x_0$. Il search space è monodimensionale ed è rappresentato dai possibili valori di $x$. Supponiamo di muoverci nel search space con passo $\Delta x$. Da $x_0$ possiamo andare in $x- \Delta x$ e $x+ \Delta x$, dove avremo i corrispondenti valori di $y$: $y(x- \Delta x)$ e $y(x+ \Delta x)$. Visto che stiamo minimizzando $y$, ci conviene muoverci verso valori di $y$ più bassi, quindi la regola per la minimizzazione sarà (ad esempio in Matlab):

``` matlab 
x(1) = x_0 % definiamo le condizioni iniziali for i=2:N. % N numero massimo di passi \
if (y(x(i-1)-$\Delta$x) > y(x(i-1)+$\Delta$x))\
		x(i)=x(i-1)+$\Delta$x;\
else\
		x(i)=x(i-1)-$\Delta$x;\
end
```
in realtà all’interno del ciclo for ci sarà un controllo che ferma il ciclo se siamo nel minimo locale. In pratica ci muoviamo nel search space andando sempre nella direzione che riduce il valore di $y$. La condizione espressa nell’if è equivalente a verificare se la derivato di $y$ rispetto a $x$ è negativa nel punto $x(i-1)$. 

infatti per il metodo delle differenze finite

\begin{align}
\frac{dy}{dx} = \frac{y(x+\Delta x)-y(x-\Delta x)}{2 \Delta x} 
\end{align}

Quindi il processo di minimizzazione di $y$ consiste nel seguire la direzione in cui la derivata di $y$ è negativa. Per problemi multidimensionali la derivata corrisponde al gradiente (vettore delle derivate parziali rispetto alle variabili) ed il metodo prende il nome di  **gradient descend (GD)**.

Per problemi semplici la derivata può essere calcolata analiticamente, in questo caso il calcolo è più veloce perché l’algoritmo iterativo ad ogni passo può calcolare facilmente il valore della derivata stessa. In generale, la formulazione analitica della derivata non è nota e deve essere calcolata ad ogni passo per via numerica con il metodo delle differenze finite. La precisione nel calcolo numerico della derivata dipende dal passo del GD ($\Delta x$ nel caso monodimensionale). Più è piccolo $\Delta x$ migliore è il calcolo della derivata, ma maggiore è il numero di iterazioni richieste per trovare il minimo.

Notiamo che applicando il Gradient Descend al problema della minimizzazione di $L$ ad ogni passo dobbiamo calcolare le derivate numeriche per tutte le $N$ immagini del training set. Se $N$ è molto grande, è opportuno limitare la somma ad un sottoinsieme delle $N$ immagini (**Mini-batch Gradient Descend (SGD)**). Nella pratica spesso si usa il termine SGD anche per indicare l’approccio MGD.
L’uso di un mini-batch ha due vantaggi fondamentali: riduce il tempo di calcolo e riduce il consumo di memoria sulla GPU che viene comunemente usata nel machine learning. Allo stesso tempo, chiaramente riduce la significanza statistica della valutazione della loss.  

A questo punto si consiglia di studiare attentamente il notebook {doc}`n_6_2_Linear_Classifier.ipynb` che mostra un esempio di classificazione di immagini con un classificatore lineare.

In questo esempio, essendo il task di classificazione molto semplice, avremo accuratezza uno sia sul training che sul test set. Il risultato ottenuto era atteso, in quanto, vista la natura dei dati nel test set, troviamo delle copie di elementi presenti nel training set, per cui il modello ovviamente generalizza correttamente. 
Complichiamo il problema cercando di classificare un nuovo set di immagini, le immagini rappresentano due croci di cui una leggermente modificata. Il set di immagini è costruito ruotando in modo casuale le due immagini di partenza (TEST2) (Figura 6.10).

<img src="./images/Figura_6_10.png" alt="Immagini usate nella procedura di classificazione" style="width:100%;">

*Figura 6.10. Immagini usate nella procedura di classificazione.*

In questo caso nel test set non troviamo repliche esatte dei dati di training. Ottenere la convergenza è più difficile, e probabilmente non riusciremo a replicare esattamente le prestazioni ottenute sul training set nel test set.
 
La classificazione delle immagini in TEST2 è effettuata nel notebook {doc}`Linear_Classifier_test2.ipynb` 

Come si vede nel training set abbiamo ancora accuratezza quasi uno, che è leggermente ridotta nel test set, dove comunque abbiamo una accuratezza elevata. Questo esempio conferma le potenzialità di un classificatore lineare e la necessità dell’uso di un test set “vergine” per una misura realistica delle performance di un classificatore.

Una prova interessante consiste nel modificare le dimensioni dell'immagine in ingresso al modello attraverso la funzione di pre-processing. Nel classificatore lineare il numero di parametri da ottimizzare scala linearmente con il numero di pixel dell'immagine, per cui anche per dimensioni ragionevoli (512x512, una normale immagine TAC) il processo di training diventa molto lento. Questo evidenzia i limiti del classificatore, che non è **scalabile** per cui non è utilizzabile per la classificazione di immagini non piccolissime.    

Nei due esempi viene calcolata la confusion matrix, che misura la performance del classificatore. La confusion matrix è una matrice $N \times N$ dove $N$ è il numero delle classi. Sulla diagonale ci sono le predizioni corrette, mentre il resto della matrice rappresenta le predizioni sbagliate. Nel caso di due classi, gli elementi della matrice corrispondono a  VP, VN, FP, VN da cui si possono calcolare la sensitività e la specificità del classificatore. 


## Reti Neurali
Una volta trovati i parametri $W$, un classificatore lineare computa gli score per un’immagine I rispetto  alle  possibili  classi  come $s=Wx$ dove $x$  è  il vettore colonna contenente i  pixel dell’immagine. $W$ è una matrice $K \times N$ dove $N$ è il numero di  pixel dell’immagine e $K$ il numero di classi. Una rete neurale invece computa $s$ come $s=W_2 FNL(W_1x)$, dove $W_2$ è una matrice $Q \times N$, $W_1$ è una matrice $K \times Q$. La funzione $FNL$ è una funzione non lineare (ad esempio $max(0, W_1x)$ che mette a zero i valori negativi). Notiamo che la non linearità caratterizza la rete neurale, in quanto, se non fosse presente, ci ricondurremmo al caso del classificatore lineare con $W=W_2W_1$. Il valore di $Q$ è arbitrario, come vedremo, indica il numero dei cosiddetti strati nascosti (**deep layers**) della rete. $W_1$ e $W_2$ sono i parametri della rete individuabili attraverso il GD. Analogamente possiamo definire una rete a tre strati come $s=W_3 FNL(W_2 FNL(W_1x))$ e così  via.

Storicamente, la struttura di una rete neurale è stata modellata come una rete di neuroni, ciascuno modellato sul funzionamento di un neurone biologico (Figura 6.11).

<img src="./images/Figura_6_11.png" alt="Modello Biologico di una Rete Neurale" style="width:100%;">

*Figura 6.11. Modello Biologico di una Rete Neurale.*

Il neurone prende in input una serie di segnali dai dendriti ed emette un singolo segnale attraverso l’unico assone. I segnali di ingresso vengono pesati sulla base del peso delle sinapsi. Se la somma pesata è superiore ad una certa soglia, il neurone si attiva, altrimenti no. Un neurone può quindi essere visto come un classificatore di tipo binario, che classifica gli input in due classi. Il funzionamento del classificatore è determinato dalla funzione $f$ (activation function). Se $f$ è la funzione softmax, abbiamo un classificatore binario softmax (lo standard classico nelle NN), altrimenti possiamo implementare un classificatore SVM o di altro tipo. È possibile implementare vari tipi di funzioni $f$. Molte NN utilizzano la funzione ReLU, che vale zero per l’input negativo e copia i valori positivi. Questa funzione dà in generale prestazioni migliori rispetto alla classica softmax o sigmoide (Figura 6.12).

<img src="./images/Figura_6_12.png" alt=" Funzione RELU" style="width:100%;">

*Figura 6.12. Funzione ReLU.*

Una rete NN si può vedere come una rete di unità interconnesse (neuroni) che si evolve nel tempo (Figura 6.13).

<img src="./images/Figura_6_13.png" alt=" Rete Neurale (NN)" style="width:100%;">

*Figura 6.13. Rete Neurale (NN).*

Una NN tipicamente è composta di strati (layer), comunemente i layer sono completamente connessi fra loro (fully connected layers), mentre i neuroni interni ad un layer non sono tra loro connessi. Il layer di input normalmente non viene contato, quindi la rete illustrata ha tre layer, due layer interni o nascosti ed un layer di output. La connessione tra due nodi $i$ e $j$ viene di solito espressa come peso (weight) ed indicata con $w_{i,j}$. Nel seguito supporremo che un nodo non interagisca con se stesso, e quindi che $w_{i,j}=0$ per ogni $i$.
Le reti neurali possono venire classificate in modi diversi. Le denominazioni tipiche sono quelle di rete ricorrente, dove sono presenti dei cicli all’interno della rete, e di rete feed-forward, dove non sono presenti cicli.
La computazione in una NN avviene negli strati interni, detti nascosti, da cui il termine **deep learning** utilizzato per definire questo approccio.

La ragione che giustifica una struttura a layer è la possibilità di far operare la rete utilizzando operazioni vettore/matrice. Nella rete in figura, l’input della rete è un vettore $x_{3 \times 1}$. Il primo layer interno ha i pesi memorizzabili in una matrice $W1_{4 \times 3}$, mentre i bias sono in un vettore $b1_{4 \times 1}$. Analogamente il secondo layer interno sarà caratterizzato da una matrice $W2_{4 \times 4}$ e un bias $b2_{4 \times 1}$ mentre il layer di output sarà caratterizzato da un vettore $W3_{1 \times 4}$. Con queste notazioni l’uscita della rete è computabile con le operazioni:

$h1 = f(W1x + b1)$.	(uscita primo layer) \
$h2 = f(W2h1 + b2)$.	(uscita secondo layer) \
$out = W3h2$	(uscita della rete)

nell’addestramento della rete bisognerà considerare come input un vettore $x$ di dimensioni $3 \times N$ dove $N$ è il numero di esempi a disposizione. È importante notare che la computazione feed-forward sulla rete è totalmente parallelizzabile in quanto le colonne di $x$ possono essere elaborate in parallelo.
Inoltre, una struttura a layer assicura la rappresentatività (**rappresentational power**) della rete. Infatti, è stato dimostrato che una rete con almeno un layer nascosto è un **approssimatore universale**, cioè è in grado di approssimare con un errore inferiore a un $\epsilon$ prefissato qualsiasi funzione continua. Questo concetto non esclude il fatto che reti con più di due layer possano essere più efficienti in termini di velocità di calcolo e garanzia di una corretta convergenza.
Il numero ottimale di layer dipende dal bilanciamento tra la capacità della rete di rappresentare funzioni complesse e la necessità di evitare l’overfitting, cioè il fatto che la rete “vede” il rumore dei dati invece che la struttura degli stessi se la rete è troppo complessa rispetto ai dati. Siamo in uno scenario simile a quello del fitting di una curva che perde di senso se il numero di parametri del modello eguaglia il numero di campioni disponibili. In generale, il problema dell’overfitting può essere controllato efficacemente con varie tecniche come la regolarizzazione che abbiamo visto nei classificatori lineari, per cui si preferisce utilizzare reti con un numero alto di layer.
Una NN quindi può essere vista come un classificatore che rappresenta una score function dipendente dalla struttura e dai pesi della rete. Prima di discutere della loss function che insieme alla score function caratterizza il classificatore introduciamo alcuni aspetti (come il pre-processing dei dati e inizializzazione dei pesi) che sono determinanti nel funzionamento della rete.

### Addestramento di una NN

#### Pre-processing
L’ingresso della rete è rappresentato da una matrice $x_{N \times D}$ dove $N$ è il numero di dati disponibili e $D$ è la dimensione dei dati. Ad esempio, per delle immagini $N$ sarà il numero delle immagini e $D$ il numero di pixel delle immagini stesse. Come per gli algoritmi di Clustering, l’idea di base del pre-processing è normalizzare i dati di input in modo che ricadano nello stesso range. Le operazioni di base sono la sottrazione della media e la divisione per la deviazione standard per ogni dato, in modo da avere la stessa distribuzione con media nulla e $SD=1$ (standardize) (Figura 6.14).
Operazioni più complesse sono la PCA e il whitening.

<img src="./images/Figura_6_14.png" alt=" Rete Neurale (NN)" style="width:100%;">

*Figura 6.14. Pre-processing dei dati di input di una NN.*

#### Inizializzazione
Se i dati di input sono normalizzati, si può prevedere che la media dei pesi della rete, una volta addestrata, sarà vicina a zero. Tuttavia, non ha senso inizializzare tutti i pesi a zero, perché questo condurrebbe la rete a funzionare in maniera “uniforme”, rallentando la convergenza. Quindi i pesi si inizializzano tipicamente come piccoli numeri casuali generati da una distribuzione a media nulla, anche se sono possibili altre strategie. Per le reti con neuroni ReLU è stata dimostrata come ottima la scelta $w = randn(N)* \sqrt{2/N}$ dove $N$ è il numero dei pesi. I bias sono normalmente settati a zero.

#### Regolarizzazione
Come già visto, l’operazione di regolarizzazione ha lo scopo di evitare l’overfitting. La strategia più comune, come nei classificatori lineari, è quella di minimizzare la norma $L2$ ($L2$ regularization), per cui alla funzione obiettivo si aggiunge la somma dei quadrati dei pesi moltiplicata per una costante (regularization strenght). 
Un altro approccio possibile è la $L1$ regularization (valori assoluti dei pesi). Esistono poi altri approcci come il dropout che vedremo in seguito.

#### Loss Function
La loss function misura la compatibilità tra una predizione dei dati (score function) fatta con determinati valori dei pesi e le “ground truth label” prese come riferimento. La data loss sarà la media sui dati della loss function. Un esempio classico di loss function è la cross-entropy vista in precedenza.
Alla funzione di loss va aggiunto il fattore di regolarizzazione sui pesi discusso prima.

#### Learning
Il processo di apprendimento della rete ha lo scopo di ottimizzare i parametri della rete in modo da minimizzare il valore della loss function. L’apprendimento è un processo iterativo che può essere monitorato attraverso la visualizzazione del valore della loss rispetto al numero di iterazioni.
Di solito il periodo di apprendimento viene diviso in blocchi  (**epoche**). 
Ogni epoca contiene il numero di iterazioni necessario per utilizzare tutti i dati di training e quindi dipende dal **Mini-batch size**. Il numero di iterazioni in un epoca è uguale al numero di esempi nel training set diviso il Mini-batch size. 
Per ottimizzare il processo di apprendimento è opportuno dividere i dati di training in due parti, i dati di training veri e propri e i dati di validazione (validation set). Quindi nell’addestramento avremo tre data set che si ottengono nel modo seguente: 
Divido i dati in Test set (ad esempio il 10% dei dati) che viene usato solo al termine dell’addestramento e training set (90%) dei dati che uso per l’addestramento.
Divido il training set in validation set (ad esempio il 10% del training set) e training set vero e proprio. 
Lo scopo del validation set e di essere usato come “prototipo” del test set. Alla fine di ogni epoca il modello viene testato sul validation set ed il valore della loss e dell’accuratezza sul validation set viene visualizzato. Questa procedura consente di monitorare l’evoluzione della rete interrompendo i processo di apprendimento se non si osserva un decremento del valore della loss (Figura 6.15).   

<img src="./images/Figura_6_15.png" alt=" Curve di apprendimento" style="width:100%;">

*Figura 6.15. Curve di apprendimento (Loss).*

In generale la loss decrescerà in modo non monotono, oscillando intorno ad un valor medio come in figura, in quanto il valore della loss per ogni iterazione dipende dal particolare batch utilizzato. Ingrandire i batch porta tipicamente ad una riduzione delle oscillazioni. Si potrebbe pensare che sia desiderabile avere un ritmo di apprendimento più alto possibile e quindi una loss che decresce il più rapidamente possibile, ma in realtà un learning rate troppo alto può portare alla convergenza in un minimo locale della rete o addirittura ad una divergenza del processo di apprendimento.
Un altro parametro utile è l’accuratezza ottenuta sul test set rispetto al validation set (Figura 6.16).

<img src="./images/Figura_6_16.png" alt=" Rete Neurale (NN)" style="width:100%;">

*Figura 6.16. Curve di apprendimento (Accuratezza).*

La progressione dell’accuratezza sul validation set dovrebbe essere vicina a quella sul training set, nel caso contrario siamo in presenza di overfitting in quanto la rete si ottimizza su caratteristiche dei dati non significative per la classificazione desiderata.

Un esempio di addestramento di una NN per il problema di classificazione visto prima per il classificatore lineare è nel Notebook: {doc}`n_6_4_simple_NN_test2.ipynb`

Rispetto al classificatore lineare la NN è generalmente più accurata, grazie alla maggiore capacità di generalizzare rispetto a dati non visti prima (**unseen**).

## Radiomics e Reti Convolutive (CNN)

Come visto in precedenza, teoricamente è possibile usare un classificatore utilizzando come input l’intera immagine biomedica. Questa strategia non è però praticamente applicabile perché il numero delle variabili di ingresso sarebbe non gestibile, soprattutto nel caso di immagini 3D o serie temporali. Prima di applicare il classificatore è quindi necessario ridurre le dimensioni dei dati. Avremo quindi uno schema del tipo in Figura 6.17.

<img src="./images/Figura_6_17.png" alt="Classificazione attraverso estrazione di features" style="width:100%;">

*Figura 6.17. Classificazione attraverso estrazione di features.*

Dall’immagine viene estratto un certo numero $F$ di **features** (caratteristiche significative dell’immagine) con $F<<N$, in modo da ridurre in modo significativo le dimensioni del problema. 
Le features poi vengono date come input al classificatore. Idealmente le features dovrebbero estrarre dall’immagine tutte le informazioni significative senza perdita di informazione, per cui la classificazione delle features  dovrebbe essere equivalente alla classificazione fatta sull’intera immagine. 
L’estrazione di features  può essere fatta con due approcci principali, la **radiomica** e le **reti neurali convolutive** (o **convoluzionali**) (**CNN**).
 
## Radiomica

La costruzione di un modello radiomico prevede diverse fasi a seconda del tipo di obiettivo e di strumenti adottati per l’analisi. In generale, un approccio classico prevede :

•	Segmentazione: un radiologo esperto identifica delle regioni di interesse (per esempio, un nodulo polmonare) sulle immagini di un dataset. La segmentazione può essere anche automatizzata con i vari algoritmi a disposizione. 

•	Estrazione delle features: si estrae un vasto insieme di caratteristiche o “features” dalle regioni precedentemente delimitate utilizzando algoritmi di analisi dell’immagine. Queste features spaziano da quelle più semplici (forma, dimensioni, margini), analoghe agli aspetti qualitativi valutati dal radiologo, a più complesse e astratte calcolate attraverso equazioni matematiche.

•	Creazione del database: Il risultato di questo processo è un insieme di dati numerici che possono essere espressi in forma tabulare (all’interno di una tabella) e che rappresentano il contenuto informativo depositato all’interno dell’immagine digitale.

•	Analisi: La tabella ottenuta può essere analizzata con metodi statistici standard per verificare quali features siano predittive rispetto ad una certa patologia. In alternativa, la tabella può essere utilizzata per addestrare un classificatore capace di individuare la patologia dalle feature. Sul classificatore possono essere implementate tecniche XAI (che verranno introdotte nel seguito) per definire l’importanza delle varie features nel processo di classificazione.  

Considerando come esempio il software pyradiomics (https://pyradiomics.readthedocs.io/), il software permette di estrarre numerose feature con diverse tecniche:

•	First Order Statistics (19 features). Esempi di queste features sono il valore minimo, massimo, medio, la mediana, la deviazione standard, e l’entropia dell’immagine.

•	Shape-based (3D) (16 features). Queste metriche si ottengono da una mesh che descrive il volume estratto con la ROI di segmentazione. Esempi sono il volume, l’area della superficie, il rapporto area/volume. 

•	Shape-based (2D) (10 features). Analoghe alla precedente in 2D (area, perimetro).

•	Gray Level Co-occurrence Matrix (24 features). Esempi: correlazione, entropia congiunta tra patch estratte sull’immagine. 

•	Gray Level Run Length Matrix (16 features). Verifica l’esistenza di serie di pixel simili tra loro, simile al Run Length Encoding (RLE) usato nella compressione zip. 

•	Gray Level Size Zone Matrix (16 features). Estrae delle regioni con un algoritmo simile al region growing e calcola degli indici rappresentativi. 

•	Neighbouring Gray Tone Difference Matrix (5 features). Simile al precedente.

•	Gray Level Dependence Matrix (14 features). Simile al precedente.

Pyradiomics include tutte le features proposte in letteratura scientifica. Altri software commerciali estraggono una collezione di features specifiche in base al problema clinico da affrontare. 

## Reti neurali convolutive (convolutional neural networks CNN)

Le reti CNN (o ConvNets) sono reti convolutive (o convoluzionali), simili nella struttura alle reti neurali prima descritte (Figura 6.18). La differenza rispetto alle classiche NN è che sono dedicate in modo esclusivo all’elaborazione delle immagini e quindi sono ottimizzate in questo senso. Vale la pena di notare che le CNN sono normalmente dedicate all’elaborazione di immagini a colori, quindi è necessario fare spesso degli adattamenti per l’utilizzo con immagini biomediche a singolo canale e a 16 bit.

<img src="./images/Figura_6_18.png" alt="Classificazione attraverso estrazione di features" style="width:100%;">

*Figura 6.18. Concetto di rete CNN.*

In una normale NN usata per l’elaborazione di immagini, ogni neurone del primo layer riceve in input un numero di connessioni pari al numero di osservazioni relative all’oggetto da analizzare, nel caso di un’immagine, il numero di pixel/voxels. Si osserva facilmente che per immagini anche di “normali” dimensioni il numero di connessioni e quindi di parametri cresce in modo non controllabile portando all’overfitting della rete. Per risolvere questo problema, un layer di una CNN è organizzato come un oggetto 3D (height, width, depth) che è connesso non a tutti i neuroni del layer precedente, ma solo ad una regione abbastanza piccola. Questo approccio “legge” il fatto che le immagini hanno una continuità spaziale, per cui ha senso che una parte della rete lavori su una parte dell’immagine trascurando le altre.

In una CNN sono presenti tre tipi fondamentali di layer, il **Convolutional Layer**, il **Pooling Layer** ed il **Fully-Connected Layer** che è tipicamente il layer finale della rete. Concettualmente una CNN funziona secondo la logica:

•	INPUT (immagine 2D di 128x128 pixel)

•	CONV layer. Prende in ingresso l’immagine e la filtra con un certo numero di filtri, ad esempio 12. Il layer avrà dimensioni 128x128x12.

•	RELU layer. Applica una trasformazione non lineare (max(0,x)) lasciando invariata la dimensione dell’output rispetto all’input (128x128x12).

•	POOL layer. Effettua un sottocampionamento nella dimensione spaziale, ottenendo un volume, ad esempio, 16x16x12.

•	FC layer. Computa le classi di appartenenza, se abbiamo 10 classi l’uscita della rete sarà un vettore con 10 elementi che contiene il valore di appartenenza dell’immagine ad ogni classe.

I layer CONV e FC dipendono dai parametri che vanno ottimizzati, mentre i layer POOL e RELU sono funzioni fisse. Il processo di apprendimento guidato dal GD troverà i parametri che minimizzano l’errore nei dati di training.

Il concetto che caratterizza una CNN è il convolution layer, che non è altro che il kernel di un filtro convolutivo (Figura 6.19). Gli elementi del kernel rappresentano i parametri del layer, quindi height e width del layer corrispondono alle dimensioni del kernel. Il depth rappresenta il numero di filtri convolutivi che applichiamo, ognuno avrà un diverso valore degli elementi del kernel per cui i parametri saranno height x width x depth, un numero molto minore rispetto alla dimensione dell’immagine. I valori di height, width e depth definiscono la struttura della rete e sono quindi iperparametri del modello. 

<img src="./images/Figura_6_19.png" alt="Convoluzione Spaziale in una CNN (1)" style="width:100%;">

*Figura 6.19. Convoluzione Spaziale in una CNN (1).*

Rispetto ai classici filtri convolutivi che preservano la dimensione dell’immagine, nel caso delle CNN il kernel opera strettamente all’interno dell’immagine, quindi le dimensioni dell’immagine di output sono $N-2[K/2]$, dove $K$ è la dimensione del kernel ($[*]$ indica la parte intera). Quindi in generale il filtraggio riduce le dimensioni delle immagini. Nel filtraggio convolutivo il kernel si muove di un pixel alla volta (stride=1). Nelle CNN può essere utile definire uno stride non unitario, in questo caso la dimensione dell’immagine di output si ridurrà ulteriormente. Di solito si utilizzano valori di stride 1 o 2. Anche I valori di stride sono iperparametri (Figura 6.20). 

<img src="./images/Figura_6_20.png" alt="Convoluzione Spaziale in una CNN (2)" style="width:100%;">

*Figura 6.20. Convoluzione Spaziale in una CNN (2).*

Come nel filtraggio convolutivo, se adottiamo uno **zero padding** sul contorno dell’immagine di input prima del filtraggio, è possibile preservare la dimensione dell’immagine stessa.
Quindi lo **stride** e il **padding** definiscono la dimensione dell’output del filtro convolutivo. Se applichiamo depth filtraggi (ovviamente con lo stesso stride e padding in modo da avere la stessa dimensione di uscita), otteniamo depth immagini di uscita che costituiscono la mappa di attivazione (**activation map**) (Figura 6.21).

<img src="./images/Figura_6_21.png" alt="Convoluzione Spaziale in una CNN (2)" style="width:100%;">

*Figura 6.21. Activation Map.*

In questo esempio abbiamo una immagine RGB (tre canali) di 32x32 pixel, alla quale viene applicato un banco di 10 filtri convolutivi 5x5x3 (lo stesso filtro viene applicato ai tre canali RGB dell’immagine e viene fatta la media). Con stride=1 e senza zero padding otteniamo 10 immagini 28x28 che costituiscono l’attivation map.
In generale, se l’immagine di ingresso ha dimensione $W$, $F$ è la dimensione del convolution layer, $P$ è il numero di pixel di padding e $S$ è lo stride, la dimensione dell’immagine di output è $([W- F+2P]/S)+1$. Se volgiamo che la dimensione dell’immagine in output sia uguale a quella di input dobbiamo porre $P=(F-1)/2$ con $S=1$.
Il secondo tipo di layer è il cosiddetto **pooling layer** (Figura 6.22), che in pratica effettua un sottocampionamento dell’immagine riducendone le dimensioni. Lo scopo del pooling layer è adattare le dimensioni dell’immagine (grandi) al numero di classi (piccolo) che è l’output della CNN. Un pooling layer può essere implementato come un interpolatore o come un filtro sul modello del filtro mediano, ad esempio prendendo il valore massimo sul kernel (max pooling). Il filtro in figura ha stride=2 e computa il massimo sul kernel riducendo le dimensioni dell’immagine alla metà.

<img src="./images/Figura_6_22.png" alt="Pooling Layer" style="width:100%;">

*Figura 6.22. Pooling Layer.*

Il **fully connected layer** è identico a quello delle NN. È possibile convertire un layer FC in un layer convolutivo e viceversa, questa possibilità consente un'ottimizzazione del tempo di apprendimento.
La struttura di base di una CNN può essere esemplificata come (modello VGG):

$$
\begin{aligned}
\text{INPUT} 
&\rightarrow \bigl[\, [\text{CONV} \rightarrow \text{RELU}]^{N}
\rightarrow \text{POOL} \,\bigr]^{M} \\
&\rightarrow [\text{FC} \rightarrow \text{RELU}]^{K}
\rightarrow \text{FC}
\end{aligned}
$$


Ci sono N coppie CONV->RELU seguite da un POOL. Questa struttura si ripete $M$ volte fino a quando non si arriva alla parte di classificazione dove abbiamo coppie FC-RELU che si ripetono $K$ volte. Alla fine un layer FC fornisce l’output (Figura 6.23).


<img src="./images/Figura_6_23.png" alt="Pooling Layer" style="width:100%;">

*Figura 6.23. Riduzione della dimensione del problema di classificazione in una CNN.*

Concettualmente, una CNN riduce le dimensione del problema fino a quando è possibile utilizzare una NN per effettuare la classificazione su un numero ragionevole di dati.  
I parametri $N$, $M$, $K$ definiscono la struttura della rete VGG, sono quindi iperparametri del modello che devono essere ottimizzati in fase di design per ottenere le massime prestazioni. In generale, in una rete VGG avremo quindi una serie di iperparametri che definiscono la struttura della rete:  

| Iperparametro | Dimensione |
| --- | --- |
| Dimensione del kernel dei layer convolutivi | N*M |
| Numero di layer convolutivi | 1 |
| Profondità dei layer convolutivi  |N*M |
| Tipo di attivazione (RELU, SoftMax,….)| M |
| Pooling size | M |
| Pooling type (Max, Min, Average) | M |
| Numero di dense (FC) layer | 1 |
| Numero di neuroni dei dense layer | N |

Come si osserva dalla tabella, anche per una rete semplice come una VGG il numero di iperparametri che definiscono la struttura della rete è alto, rendendo non praticabile una strategia di design “trial-and-error” che procede per tentativi successivi. E' invece preferibile partire da architetture dimostrate funzionanti e modificarle attraverso una procedura di “fine-tuning” degli iperparametri.  

**In practice: use whatever works best on ImageNet. If you’re feeling a bit of a fatigue in thinking about the architectural decisions, you’ll be pleased to know that in 90% or more of applications you should not have to worry about these. I like to summarize this point as “don’t be a hero”: Instead of rolling your own architecture for a problem, you should look at whatever architecture currently works best on ImageNet, download a pretrained model and finetune it on your data. You should rarely ever have to train a ConvNet from scratch or design one from scratch (https://cs231n.github.io/convolutional-networks/).**

Una caratteristica importante delle CNN grazie all’alto parallelismo degli algoritmi utilizzati è quella di sfruttare in modo ottimale il cosiddetto **GPU computing**, cioè la possibilità di usare una o più schede grafiche (GPU) per l’esecuzione di calcoli. Infatti, la potenza di calcolo delle GPU è cresciuta esponenzialmente negli ultimi anni, con la capacità di elaborare milioni di poligoni al secondo attraverso l’uso in parallelo di migliaia di unità di elaborazione dei dati.
L’uso delle GPU comporta il fatto che il limite computazionale è dettato dalla memoria della GPU, che è abbastanza limitata (8-32 Gb). Quindi gli algoritmi di apprendimento comprendono delle strategie per minimizzare la quantità di memoria utilizzata.

L’addestramento di una CNN è analogo a quello di una comune rete neurale. Dal data set di immagini labellate che abbiamo a disposizione se ne seleziona una parte (test set) che deve essere rappresentativa dell’intero dataset, quindi indistinguibile dal dataset dal punto di vista statistico. Ad esempio, se nel dataset ci sono il 40% di soggetti sani e il 60% di malati, la stessa proporzione si dovrà trovare nel test set. Il modo più semplice di procedere e di estrarre casualmente dal dataset le immagini componenti il test set. La dimensione del test set sarà la più piccola possibile compatibile con la rappresentatività. 

Fissato il test set le immagini rimanenti formano il training set. Per monitorare il possibile overfitting, il training test andrà diviso in training set vero e proprio e validation set, per la generazione del quale valgono le considerazioni viste per il test set. Un metodo molto utilizzato è la cosiddetta cross-validation, in cui si utilizzano diversi validation set (Figura 6.24).

<img src="./images/Figura_6_24.png" alt="K-Fold" style="width:100%;">

*Figura 6.24. K-Fold.*

Come si vede dalla figura, una volta estratto il test set che non deve essere coinvolto in nessun modo nel processo di addestramento, si divide il training set in $K$ parti (fold), da cui il nome di K-folds validation. Si fa il training $K$ volte usando $(K-1)$ fold per il training ed un fold per la validation. Alla fine si valutano le performance sui $K$ training che dovrebbero essere molto simili se la rete è stata addestrata in modo corretto. 
Durante il processo di training verranno eseguite un certo numero di passi per l’ottimizzazione degli iperparametri. In teoria dovremmo usare tutte le immagini del training set per ogni passo, ma in pratica se ne usa un sottoinsieme (batch) per ottimizzare il tempo di training. Detto $N$ il numero di immagini e $B$ la grandezza di un batch, dopo $I=N/B$ iterazioni avremo utilizzato tutte le immagini del training set. Il numero di iterazioni $I$ definisce un’epoca dell’addestramento. Tipicamente alla fine di un’epoca si calcola la funzione loss e l’accuratezza sul validation set. Quindi tipicamente definiremo il numero di epoche ed il batch size e di conseguenza $I$. Notiamo che il batch size definisce la memoria necessaria per ogni iterazione, quindi se usiamo GPU con poca memoria dobbiamo tenere basso il batch size.      

I risultati della procedura di addestramento dipendono da una serie di iperparametri che caratterizzano il processo, quali ad esempio:

•	Il tipo di ottimizzatore utilizzato

•	I parametri dell’ottimizzatore, quali il “learning rate” cioè il passo utilizzato per il calcolo del gradiente. Il learning rate può essere costante nel processo di apprendimento o adattivo, ad esempio potremmo usare un learning rate più alto all’inizio che decresce con il numero di epoche (learning rate decay).  

•	La dimensione del batch size

•	Il massimo numero di epoche dopo il quale l’addestramento si ferma. In assenza di “early stopping” il modello salvato sarà quello corrispondente all’ultima iterazione. 

•	L’”Early stopping patience”. E’ possibile definire un numero di epoche dopo il quale se non c’è stato un incremento di prestazioni (ad esempio un incremento dell’accuratezza) sul validation set il processo di apprendimento si ferma e viene salvato il modello con le migliori prestazioni.  
   

Anche questi parametri vengono  ottimizzati attraverso una procedura di “fine tuning”.

### Dropout
Per ridurre la possibilità di overfitting, è possibile inserire nella rete degli elementi detti “**dropout**”. Una caratteristica fondamentale di un dropout è che agisce solo nella fase di training per ridurre l’overfitting, mentre non ha nessun effetto sul modello addestrato. Uno strato di dropout “spegne” in modo random un certo numero di uscite di un layer di qualsiasi tipo (convolutivo, FC, etc.). Il numero di uscite è definito da una probabilità di dropout compresa tra 0 e 1 che definisce la probabilità che ogni uscita sia “spenta”. Quindi un layer dropout con $p=0.5$ spegnerà approssimativamente la metà delle uscite del layer precedente. Essendo le uscite spente diverse ad ogni iterazione, l’effetto del dropout è di ”sporcare” il processo di apprendimento, forzando il processo ad evitare i minimi locali in quanto il modello diventa robusto rispetto a variazioni casuali dell’output dei vari layer. Concettualmente è una funzione simile a quella svolta dalla mutazione casuale negli algoritmi genetici.     

### Image Pre-processing
Come nelle normali NN, è spesso opportuno eseguire un pre-processing delle immagini per ottimizzare l’apprendimento della rete. Alcuni esempi per l’analisi di immagini biomediche includono:

#### Cropping
Tipicamente la rete CNN viene utilizzata per studiare un singolo organo, quindi può essere opportuno effettuare un cropping dell’immagine per eliminare regioni che non contengono l’organo di interesse. Questo può comportare l’utilizzo di un algoritmo di segmentazione che individui l’organo sotto esame. 

#### Resizing
Una CNN richiede che le immagini abbiano tutte le stesse dimensioni, per cui se le immagini da elaborare non hanno le stesse dimensioni è necessario interpolarle ad una dimensione comune. Un altro uso del resizing è quello di sottocampionare le immagini per ridurre la complessità computazionale e l’occupazione di memoria.

#### Windowing 
Alcune architetture CNN assumono come ingresso immagini a 8 bit, quindi è necessario utilizzare una finestra di windowing per ridurre la dinamica dell’immagine.

#### Filtraggio
Può essere utile ridurre il rumore sull’immagine attraverso opportuni filtri nella fase di pre-processing.

#### Canali
Una CNN tipicamente è strutturata per immagini a colori a tre canali (RGB). Nel caso di immagini biomediche, il numero di canali sarà uno. In alcuni casi è possibile utilizzare i canali per elaborare immagini multi-frame (numero di canali uguale ai frame temporali), tenendo presente che tutti i frame verranno filtrati nello stesso modo.    

### Data Augmentation
La procedura di addestramento di una rete è tanto migliore quanto più rappresentativo è il dataset utilizzato. Un data set rappresentativo dovrebbe contenere un ragionevole numero di immagini per ogni classe possibile. Ad esempio, se sviluppiamo un’applicazione per l’analisi di immagini MR dovremmo avere immagini dai vari scanner/sequenze disponibili sul mercato, e/o immagini di pazienti con tutte le possibili patologie, etc. Ottenere un dataset di grandezza sufficiente può essere molto difficoltoso, a causa dello sforzo richiesto per collezionare e soprattutto etichettare i dati. In campo biomedico, le possibili difficoltà vanno dalla rarità di alcune patologie, alla privacy dei pazienti che limita l’accessibilità alle immagini, alla difficoltà di coinvolgere operatori esperti che etichettino le immagini. Anche quando le label sono disponibili, il formato in cui sono codificate è spesso proprietario, non esistendo uno standard per cui le label provenienti da applicazioni diverse saranno in generale codificate in un diverso formato. Oltre alla numerosità del dataset, anche lo sbilanciamento del dataset stesso costituisce un problema non trascurabile. A livello medicale, ciò è dovuto tipicamente o alla bassa prevalenza di alcune malattie, per cui il numero di casi non patologici risulta molto maggiore di quello dei casi patologici (e questo si ripercuote sulle performance dei modelli che tentano di identificare ad esempio la presenza o assenza di un tumore), oppure alla differente incidenza con cui si verificano tumori di diversa aggressività. Questo fenomeno incide notevolmente sulle performance di accuratezza della rete, poiché, se essa viene maggiormente allenata a riconoscere una certa classe di dati, tenderà ad associare la label che ha imparato meglio a tutti gli esempi su cui viene testata. Questo porta ad ottenere elevati livelli di accuratezza, ma che di fatto non sono reali, poiché riflettono semplicemente la distribuzione delle classi sottostante. Questo fenomeno prende il nome di paradosso dell’accuratezza.
Una possibile soluzione è costituita dal Data Augmentation. Con questo termine ci si riferisce ad un gruppo di tecniche che hanno l’obiettivo di combattere la limitata quantità di dati disponibili, al fine di migliorare le capacità di generalizzazione del modello. L’idea alla base consiste nell’aumentare le dimensioni del set di dati di allenamento in modo artificiale, generando varianti fittizie di ogni sua istanza, così da incrementare la quantità e la varietà di informazione fornita alla rete. In tal senso è possibile seguire due approcci: 

•	generare le immagini aumentate prima della fase di addestramento e salvarle in memoria. In questo modo, ad ogni allenamento, le dimensioni del dataset saranno incrementate di una quantità e tipologia di immagini note. Questo è il caso della **Static Data Augmentation**;

•	generare le immagini aumentate durante le epoche di allenamento con una certa probabilità, in modo che, ad ogni epoca, la costituzione del dataset cambi. Questo metodo prende il nome di **On The Fly Data Augmentation**.

Nella pratica per database di immagini non piccolissimi è opportuno utilizzare il data augmentation “on the fly”, per ridurre l’occupazione di memoria fisica. In questo caso il data augmentation avviene nella fase di pre-processing al momento del caricamento dell’immagine.  

Attualmente esistono numerose tecniche di data augmentation, ma in generale è possibile distinguerle in due grandi sottogruppi: tecniche di manipolazione standard dell’immagine in input e tecniche basate su approcci di tipo deep learning generativo che non esaminiamo.

Le tecniche di manipolazione standard possono essere classificate come:

•	Trasformazioni geometriche: utilizzano una trasformazione geometrica per modificare l’orientamento o la scala dell’immagine stessa. Come visto nel capitolo sulla registrazione delle bioimmagini, tipicamente vengono utilizzate trasformazioni affini che consentono di ruotare, traslare, ingrandire o ridurre le dimensioni dell’immagine. Alle trasformazioni affini si uniscono le operazioni di flip (capovolgimento) verticali ed orizzontali. Un’altra possibilità è effettuare un’operazione di cropping estraendo una parte dell’immagine. Queste operazioni possono essere effettuate anche in 3D.

•	Filtraggio: si possono generare nuove immagini filtrando le immagini stesse, con filtri puntuali, locali, o globali. Nel caso dei filtri puntuali vale la pena notare che le CNN assumono come input immagini RGB (tre canali) ad 8 bit, quindi l’immagine biomedica in fase di input deve sottostare ad una operazione di windowing. Variando la finestra di windowing utilizzata, si può realizzare il data augmentation. Il filtraggio locale può essere realizzato con filtri di smoothing (quali il filtro gaussiano con diversi valori di sigma) o un filtro di sharpening.

•	Aggiunta di rumore: si generano nuove immagini aggiungendo del rumore con diversa intensità.

Vale la pena di notare che le ultime due operazioni di data augmentation corrispondono nella sostanza a variare i parametri del modello dell’immagine biomedica.

### Transfer Learning
Il tuning di una grande CNN è difficile da realizzare in quanto richiede un database di dimensioni molto grandi che è raramente disponibile. Al contrario di quanto avviene nel caso delle immagini comuni, dove sono disponibili database di libero accesso molto grandi come ImageNet (https://www.image-net.org/), in campo biomedico grandi database di immagini sono più difficili da ottenere per i limiti imposti dalla privacy dei pazienti e per la difficoltà di ottenere la diagnosi associata alle immagini che costituisce la label delle immagini stesse. Esistono comunque alcuni database labellati che possono essere utilizzati (https://www.aylward.org/notes/open-access-medical- image-repositories). Un’alternativa è utilizzare immagini provenienti da simulatori.
L’approccio più comune è quindi partire da una rete già addestrata su dati simili (**pretrained CNN**) ed effettuare un **fine tuning** della rete adattandola al problema corrente.
In pratica si parte da una rete addestrata (quindi con i pesi definiti per compiere un certo compito) e si “congelano” in modo che non vengano cambiati nel processo di addestramento. Quindi si “scongela” una piccola parte della rete per adattarla al problema corrente e si fa l’addestramento, sperando che le capacità della rete vengano preservate.  

Un esempio di addestramento di una CNN per il problema di classificazione visto prima è nel Notebook: {doc}`n_6_6_VGG_CNN_test2.ipynb`

La rete è una TinyVGG modificata (https://tinyvgg.streamlit.app/).
Rispetto agli esempi precedenti abbiamo inserito un validation set che permette di monitorare l'addestramento del modello confrontando le performance sul training e validation set, che dovrebbero essere simili. Se c'è divergenza, conviene interrompere l'addestramento. Notiamo che i dati del validation set non partecipano all'addestramento e quindi da questo punto di vista sono dati "persi". Il loro scopo è di permettere l'uso della tecnica dell **Early Stopping** e di evitare di consumare tempo di calcolo in addestramenti non fruttiferi.     

Per immagini 3x28x28 e 10 hidden_units, essendo 3x3=9+1 (bias) i parametri del CONV, il primo blocco CONV  avrà 10*(9x3+1) = 280 parametri, mentre gli altri avranno 10+(10*9+1) = 910 parametri. L’ultimo layer, essendo 7x7 la dimensione delle immagini in uscita, avrà 10x7x7x2+2= 982 parametri, in totale 3992. Per un classificatore lineare avremmo avuto 3x28x28x2=4706 parametri, quindi il numero di parametri incogniti è confrontabile. 
Già per un’immagine 64x64 avremmo avuto 24578 parametri per il classificatore lineare e 8132 per la CNN, che diventa sempre più conveniente all’aumentare della dimensione delle immagini. Per immagini 512x512 abbiamo circa 300.000 parametri per la CNN e 1.500.000 per il classificatore lineare, ma il numero di parametri della CNN può essere ridotto aumentando gli strati convolutivi.     

Finora abbiamo supposto che all’interno di un’immagine sia presente un unico oggetto. Se sono presenti più oggetti (ad esempio più organi come avviene nell’imaging biomedico), prima della classificazione è necessario identificare i vari oggetti attraverso una procedura di object detection. Se gli oggetti sono pochi, è possibile effettuare un’operazione di cropping e quindi definire l’oggetto attraverso una regione rettangolare che lo contiene.

## Architetture CNN

Sono state proposte numerose architetture per la classificazione di immagini, poi adottate nel campo dell’imaging biomedico (Figura 6.25).   

<img src="./images/Figura_6_25.png" alt="Esempi di architetture CNN" style="width:100%;">

*Figura 6.25. Esempi di architetture CNN (https://towardsdatascience.com/5-most-well-known-cnn-architectures-visualized-af76f1f0065e).*

Le reti **VGG**, evolute da LeNet-5 e AlexNet, rappresentano lo stato dell’arte delle reti “single thread”, cioè senza ritorno all’indietro dei dati durante la classificazione.  
La rete **Inception** introduce una serie di innovazioni (Figura 6.26).

<img src="./images/Figura_6_26.png" alt="Architettura Inception" style="width:100%;">

*Figura 6.26. Architettura Inception.*

In fase di training sono utilizzati due classificatori che intercettano il flusso di dati a monte dell’ultimo classificatore. I valori di loss di questi classificatori vengono sommati alla loss complessiva per un 30%. In fase di predizione i classificatori ausiliari vengono ignorati.

Il modulo inception parallelizza localmente il flusso di dati con moduli convolutivi il cui output viene poi concatenato in uscita.
Il vantaggio della inception è il ridotto numero di iperparametri.

La **ResNet** introduce due blocchi che introducono un parallellismo nel flusso di dati a livello del singolo blocco, introducendo un segnale identità (identity) o convolutivo (Conv) in parallelo al flusso principale (Figura 6.27). 

<img src="./images/Figura_6_27.png" alt="Architettura Resnet" style="width:100%;">

*Figura 6.27: Architettura ResNet.*

Il design della ResNet limita il problema della degradazione dell’accuratezza che si verifica quando alla rete vengono aggiunti strati.  

Nell’ambito dell’imaging biomedico esistono numerosi modelli addestrati che possono essere utilizzati in task di transfer learning. In particolare, è disponibile il MONAI model ZOO che include numerosi modelli addestrati su database pubblici di immagini biomedicali (https://github.com/project-monai/model-zoo).

## Segmentazione di immagini attraverso le CNN

La CNN prima descritta è in grado di classificare immagini contenenti particolari oggetti. Si può vedere una CNN classica come un oggetto che riduce la dimensione del problema, in quanto un’immagine composta da migliaia di pixel viene ridotta all’appartenenza ad un numero piccolo di classi, al limite due. In campo biomedico una CNN di questo tipo potrà classificare, ad esempio, la patologia di un paziente sulla base delle immagini del paziente stesso.
Pensando di aumentare il numero di oggetti riconosciuti, si passa a un riconoscimento a livello del pixel, corrispondente all’operazione di segmentazione (**semantic segmentation**). Nel caso della semantic segmentation l’output della rete avrà le stesse dimensioni dell’input, in quanto quello che vogliamo in uscita è una mappa di appartenenza del pixel alle varie classi, in modo analogo all’output dell’algoritmo di Fuzzy Clustering.
L’idea di base è quella dell’autoencoder, una CNN composta da due parti, un encoder che riduce le dimensioni dei dati come una classica CNN e un decoder che la ripristina (Figura 6.28).

<img src="./images/Figura_6_28.png" alt="Autoencoder" style="width:100%;">

*Figura 6.28. Autoencoder.*

Idealmente l’output dovrebbe risultare uguale all’input, ma in realtà la compressione dei dati porta ad una perdita di definizione che rende la struttura dell’autoencoder inadatta alla segmentazione di immagini.
L’esempio classico di CNN che implementa la semantic segmentation è invece la U-Net, che è una CNN con una particolare architettura derivata dall’autoencoder (Figura 6.29).

<img src="./images/Figura_6_29.png" alt="Architettura Unet" style="width:100%;">

*Figura 6.29. Architettura Unet.*

La U-Net è composta di due parti, il contraction path (ENCODER) e l’expansion path (DECODER). L’encoder è simile ad una classica CNN in cui la dimensione dei dati si riduce percorrendo la rete. Il decoder invece funziona in modo inverso ripristinando le dimensioni dell’immagine originale, infatti l’uscita della rete deve essere una maschera con dimensioni uguali all’input che contiene le informazioni di segmentazione. L’altra caratteristica della rete sono le connessioni “orizzontali” tra l’encoder ed il decoder.
Come si osserva dallo schema della rete, l’encoder è costituito da coppie di blocchi CONV+RELU seguiti da un POOL che effettua il sottocampionamento. Questa struttura è ripetuta quattro volte. Arrivati alla fine dell’encoder abbiamo due altre coppie CONV+RELU che portano l’immagine alle dimensioni 28x28 dai 572x572 iniziali. A questo punto inizia il decoder che con dei blocchi UP-CONV aumenta le dimensioni riportandole a 388x388 nella configurazione originale. Naturalmente si può configurare la rete per avere una dimensione di uscita uguale all’ingresso. Tra i blocchi UP-CONV sono poste delle coppie CONV-RELU come nell’encoder. Il blocco UP-CONV effettua una convoluzione con kernel 2x2 e stride=2 sull’output raddoppiando le dimensioni (Figura 6.30).

<img src="./images/Figura_6_30.png" alt="Up-CONV" style="width:100%;">

*Figura 6.30. UP-CONV.*

L’uscita del blocco UP-CONV è combinata con i dati dell’encoder che vengono copiati dopo un’operazione di cropping essendo diverse le dimensioni delle mappe sui due rami. La funzione loss della U-Net è tipicamente l’indice di DICE tra le label (il reference della segmentazione) e l’uscita della rete. Se la segmentazione avviene su più oggetti si utilizza il multiobject DICE. 

La U-Net originale è disponibile al sito https://lmb.informatik.uni-freiburg.de/people/ronneber/u-net/.
Esistono numerose varianti , la più famosa delle quali è  il **Total Segmentator**  disponibile anche come plug-in per 3D slicer (https://github.com/wasserth/TotalSegmentator).  

Un esempio di uso di una U-net per risolvere il problema della segmentazione è nel Notebook: {doc}`n_6_7_simple_Unet_test3.ipynb`

## Uso clinico degli approcci machine learning

Gli algoritmi di machine learning forniscono prestazioni sempre migliori e iniziano ad avere un uso di tipo clinico nell’analisi dell’immagine biomedica. Il loro uso è particolarmente rilevante nelle operazioni di screening radiologico, nel quale è necessario analizzare grandi quantità di immagini, cosa possibile solo automatizzando il procedimento. Quando però le decisioni prese da sistemi di machine learning influenzano la vita umana (come nel caso della decisione clinica), si sviluppa la necessità di capire in che modo tali decisioni siano state concepite, cosa non banale per la natura stessa di tali sistemi.

È possibile definire i concetti di interpretabilità ed esplicabilità nell’ambito dell’AI. Questi due termini possono essere facilmente confusi tra loro, ma l’idea su cui si basano è piuttosto diversa. Un modello, si definisce **interpretabile** se è noto il legame causa-effetto che sta alla base del suo criterio decisionale. Un modello, invece, è **esplicabile** se i meccanismi interni che regolano il suo funzionamento possono essere espressi in termini umani. In generale, mentre l’interpretabilità si riferisce alla capacità di capire il meccanismo dell’algoritmo senza necessariamente capirne il funzionamento, l’esplicabilità consiste nella capacità di spiegare l’algoritmo decisionale.

Con lo sviluppo di algoritmi IA sempre più complessi, si sono raggiunti livelli di accuratezza estremamente elevati, ma contemporaneamente livelli di interpretabilità sempre più bassi. Per cui, sembra esistere una tensione intrinseca tra questi due aspetti: i modelli più interpretabili sono anche quelli meno accurati e viceversa, quelli che forniscono le migliori performance sono anche quelli più “opachi”. Questo aspetto è particolarmente evidente nelle reti neurali che, con le loro centinaia di strati e milioni di parametri, ad oggi, possono essere considerate alla stregua di modelli a scatola nera. Nella figura viene mostrato il trade-off tra accuratezza e interpretabilità dei principali modelli di machine learning.

Pur essendo le performance un aspetto cruciale per l’utilizzo di modelli decisionali, in ambito medico è fondamentale avere a disposizione spiegazioni che ne supportino l’output per migliorare l’accettazione del modello da parte dei clinici.
Questa necessità ha portato alla nascita della **XAI** (**Explainable Artificial Intelligence**), un programma sviluppato dall’agenzia governativa degli Stati Uniti DARPA, con l’obiettivo di creare un insieme di tecniche di machine learning che possano rendano i modelli spiegabili, mantenendo comunque alto il livello delle performance (Figura 6.31).

<img src="./images/Figura_6_31.png" alt="Principali modelli ML in funzione della loro interpretabilità e accuratezza" style="width:100%;">

*Figura 6.31. Principali modelli ML in funzione della loro interpretabilità e accuratezza. S. Rane, Towards Data Science: https://towardsdatascience.com/the-balance-accuracy-vs-interpretability-1b3861408062.*

La XAI è un settore caratterizzato da una forte multidisciplinarietà e le strategie di explainability sono sviluppate per perseguire obiettivi di varia natura.
Alcuni concetti fondamentali della XAI sono:

•	**Trustworthiness**: (Affidabilità): Aumenta la fiducia nel fatto che un modello agisca come previsto di fronte a un determinato problema.\
•	**Causality** (Causalità): Individuare relazioni e causalità tra dati e variabili, ad esempio convalidando i risultati forniti dalle tecniche di inferenza causale o fornendo una prima intuizione di possibili relazioni causali.\
•	**Confidence**: Fiducia = L’utente ha fiducia nel funzionamento del modello\
•	**Informativeness** (Informatività): Fornire informazioni sul problema affrontato per mettere in relazione le decisioni dell'utente con le soluzioni del modello, allo scopo di supportare il processo decisionale.\
•	**Transferability**: Trasferibilità = Chiarire i limiti di applicabilità al fine di utilizzare i modelli sviluppati anche per applicazioni diverse da quelle per il quale è stato sviluppato originariamente.\
•	**Fairness**, ovvero l'equità del modello. Un predittore potrebbe essere addestrato utilizzando dataset sbilanciati o che presentano una certa tipologia di bias. Questo potrebbe portare il modello ad assumere comportamenti discriminatori nei confronti di determinate categorie di soggetti enfatizzando disuguaglianze sociali (status socioeconomico, etnia, genere, ecc...). Es. in healthcare: Accesso a determinate tecnologie solo alla popolazione "ricca" → ottenimento di un dataset non rappresentativo per determinate categorie sociali → il modello addestrato funziona peggio per tali soggetti (underrepresented groups).

Molti dei concetti illustrati per la XAI sono simili a quelli utilizzati nella certificazione dei dispositivi medici. Si presuppone che l’industria dei dispositivi medici dovrà porre particolare attenzione sui temi di interpretabilità e spiegabilità, in quanto l’UE ha optato un sviluppare un framework di regolamentazione dell’AI differenziando i requisiti richiesti in base sul rischio connesso all’utilizzo del sistema. I dispositivi medici in quanto applicazioni rientrano in quelle che sono considerate come applicazioni ad alto rischio, e tra i vari aspetti è richiesto di prestare attenzione anche su questi aspetti.
Nell’elaborazione delle immagini le tecniche XAI hanno lo scopo di “spiegare” perché una immagine è stata classificata in un certo modo. Consideriamo un esempio classico (https://medium.com/ginoasuncion/understanding-machine-learning-predictions-with-lime-7892998c7484). 
In questo esempio il modello deve riconoscere le immagini di cani husky da immagini di lupi (Figura 6.32).   

<img src="./images/Figura_6_32.png" alt="Esempio di XAI" style="width:100%;">

*Figura 6.32. Esempio di XAI(1).*

Consideriamo un caso in cui il classificatore ha sbagliato (reference: husky, predicted: wolf). Utilizziamo una tecnica XAI detta LIME che evidenzia la parte di immagine utilizzata dal classificatore nella predizione (Figura 6.33). 

<img src="./images/Figura_6_33.png" alt="Esempio di XAI" style="width:100%;">

*Figura 6.33. Esempio di XAI(2).*

Come si osserva, la predizione non è basata sull’animale ma sullo sfondo, in particolare sulla presenza della neve. In effetti il classificatore era stato addestrato con un dataset nel quale le immagini dei lupi avevano in prevalenza uno sfondo innevato, mentre negli husky no. Quindi in realtà il modello funzionava come un detector di neve. In questo caso la XAI ha permesso di individuare un bias nel processo di training. In campo biomedico potremmo avere il caso analogo di un modello addestrato su un database di soggetti dove i malati sono stati acquisiti in prevalenza sulla macchina A ed i sani sulla macchina B, per cui il modello impara a riconoscere la macchina di acquisizione e non la patologia.

Le tecniche XAI si possono dividere in tecniche **Ante-hoc** e **Post-hoc**. 

### Tecniche XAI ante-hoc

Nelle tecniche Ante-hoc si mira a sviluppare modelli che siano di per se interpretabili, ovvero ad incorporare il requisito di interpretabilità durante il processo di design del modello. Si tratta quindi di sviluppare alternative ai modelli black-box senza sacrificarne le performance. Il primo principio delle tecniche Ante-hoc è quello di evitare l’utilizzo di modelli complessi se non necessario. Se la classificazione può essere realizzata in modo semplice (ad esempio con un classificatore lineare) è inutile usare una rete neurale che è intrinsecamente oscura. Quando l’uso di un modello complesso è necessario, si vanno ad  implementare modifiche ai modelli black box per renderli maggiormente interpretabili.

#### Concept Learning Models

Un esempio sono i **Concept Learning Models** (Figura 6.34), che vanno a suddividere il task di predizione su due livelli: l’architettura viene addestrata per predire feature cliniche a partire dalle immagini, tali features saranno poi utilizzate da un classificatore interpretabile per restituire la diagnosi finale. In parallelo, dei predittori vengono addestrati per predire singoli concetti clinici comprensibili dall’utente.

<img src="./images/Figura_6_34.png" alt="Concept Learning Models" style="width:100%;">

*Figura 6.34. Concept Learning Models.*

Consideriamo ad esempio il problema di caratterizzare un nodulo. Nella pratica clinica l’operatore osserverà varie caratteristiche del nodulo (ad esempio la forma dei margini, l’uniformità del tessuto nodulare, etc) ed effettuerà una diagnosi da confermare all’istologia. Per ogni nodulo il reference sarà costituito da una label (maligno/benigno) e dalle caratteristiche osservate (margini regolari/irregolari, texture uniforme/non uniforme, etc). Nel modello la parte low-level (feature learning) estrae le features caratterizzanti dall’oggetto anatomico (ad esempio un nodulo da classificare). Una NN (dense main) classifica il nodulo come maligno/benigno. In parallelo vengono addestrati altri decisori sulle caratteristiche cliniche come il margine del nodulo e la sua struttura interna. In questo modo insieme alla predizione “black-box” della malignità verrà fornita una “spiegazione” fornendo i concetti clinici a cui l’operatore è abituato. Notiamo che in realtà non c’è nessuna garanzia che il modello abbia utilizzato i concetti clinici per la classificazione, gli output provenienti dai blocchi dropout potrebbero essere ininfluenti sulla decisione in quanto le features estratte vanno direttamente al Dense main 1. 

#### Prototypical Part Networks

Nei **Case-based models** o **Prototypical Part Networks (Protopnet)** (Figura 6.35) il modello viene addestrato per imparare le cosiddette prototypical-part delle immagini, ovvero dei pattern presenti in regioni delle immagini del training set che fungano da prototipi rappresentativi per basare il task di classificazione.
L’idea è simile a quella del clustering, nel quale i centroidi sono gli elementi rappresentativi (prototipi) dei cluster. Un elemento appartiene ad un certo cluster se “somiglia” al relativo centroide. Il concetto di ”somiglia” viene quantizzato attraverso la distanza dell’elemento dal centroide.   

<img src="./images/Figura_6_35.png" alt="Prototypical Part Networks " style="width:100%;">

*Figura 6.35. Prototypical Part Networks.*

Durante l’addestramento al solito vengono estratte delle features significative dalle immagini attraverso una CNN. Il prototipi vengono definiti attraverso il clustering dello spazio delle features attraverso la definizione di un certo numero di centroidi che rappresenta un iperparametro del processo odi addestramento. Una caratteristica della Protopnet è utilizzare piccoli set di dati (support sets) durante la generazione dei prototipi (few-shot learning). Una volta generati i prototipi la rete classifica i dati sulla base delle distanze degli stessi dai prototipi nello spazio delle features (Figura 6.36).    

<img src="./images/Figura_6_36.png" alt="Prototypical Part Networks " style="width:100%;">

*Figura 6.36. Esempio di Prototypical Part Networks.*

Dopo la classificazione è possibile mostrare per ogni immagine classificata i prototipi più simili “spiegando” il processo decisionale della rete. Un problema delle Protopnet è la trasformazione dei prototipi che sono definiti nello spazio delle features a bassa risoluzione in immagini con la stessa risoluzione delle originali. 

### Tecniche XAI post-hoc

Gli approcci di interpretabilità post-hoc hanno l’obiettivo di sviluppare tecniche capaci di fornire spiegazioni alle predizioni restituite dal modello black-box. Tali tecniche vengono quindi utilizzate per studiare a posteriori un modello addestrato con l’obiettivo di ottenere informazioni sulle relazioni che sono state imparate da tale modello.

#### Visual Explanation

Una prima classe di tecniche, molto usata nel medical imaging, sono le tecniche di **visual explanations** (Figura 6.37), che generano una **heatmap** delle stesse dimensioni delle immagini di ingresso che evidenzia le regioni delle immagini ritenute maggiormente importanti per l’output restituito. Tale mappa dovrebbe indicare quali sono le regioni delle immagini che la CNN “osserva” per generare una singola predizione. In questo caso la tecnica fornisce una **local interpretability**, cioè spiega la singola predizione e non il comportamento globale del modello (**global interpretability**).

<img src="./images/Figura_6_37.png" alt="Tecniche di visual explanations" style="width:100%;">

*Figura 6.37. Tecniche di visual explanations.*

L’output prodotto da questi algoritmi è sempre lo stesso, ovvero una **heatmap** che evidenzia quelle che sono le regioni dell’immagine ritenute più importanti dal modello black-box per la generazione dell’output. Ciò che differenzia un metodo dall’altro è ovviamente il modo in cui queste mappe vengono prodotte. Tutte le tecniche sostanzialmente cambiano dei pixel sull’immagine e verificano come cambia lo score per la predizione della classe, i pixel che influenzano di più la predizione vengono evidenziati. 

##### Saliency Maps

Le Backpropagation **Saliency Maps** identificano i pixel dell'immagine più importanti per la classe predetta ad una certa immagine I0 calcolando il gradiente della funzione di score della classe in output yc rispetto ai pixel dell’immagine di input I valutata nel punto I0.

$$
H_{BP}^c = \left. \frac{\partial y_c}{\partial I} \right|_{I_0}
$$

L'entità della derivata indica quali pixel devono essere modificati di meno per influenzare maggiormente il punteggio della classe. La derivata può essere facilmente calcolata tramite un singolo passaggio dell'algoritmo di back-propagation. 

##### Grad-CAM

Le mappe **Grad-CAM** sono sempre basate sul calcolo di un gradiente, con la differenza che in questo caso il gradiente della score function della classe viene calcolato rispetto alla feature map dell’ultimo strato della rete.

$$
\alpha_k^c = \frac{1}{Z} \sum_{i} \sum_{j} \frac{\partial y^c}{\partial A_{ij}^k}
$$

Viene quindi calcolato il valore medio di ciascuna di queste mappe di gradiente producendo i coefficienti $alfa_k$, ciascuno dei quali è associato ad una feature map del layer convolutivo della rete.
Verrà quindi effettuata una media delle features maps pesata per i coefficienti alfa. La mappa prodotta verrà quindi rettificata per evidenziare unicamente i contributi positivi per la classe predetta e sovracampionata per arrivare alla risoluzione dell’immagine in ingresso.

$$
H_{GradCAM}^c = \text{ReLU} \left( \sum_{k} \alpha_k^c A^k \right)
$$

##### Occlusion Sensitivity

L’**occlusion sensitivity** map sfrutta un approccio diverso. Essa è basata sulla perturbazione dell’immagine in ingresso tramite un’occlusion mask (una specie di kernel) che viene fatta scorrere su tutta l’immagine di input e valuta come lo score associato alla classe cambia in funzione della regione perturbata. Le regioni nelle quali la perturbazione provoca una maggiore variazione dello score sono quelle ritenute maggiormente importanti. La risoluzione della heatmap dipenderà dalla grandezza della finestra di occlusione (Figura 6.38). 

<img src="./images/Figura_6_38.png" alt="Occlusion Map" style="width:100%;">

*Figura 6.38. Occlusion Map.*

##### LIME

Un altra tecnica di Visual Explanation è il **LIME** (Figura 6.39). Il LIME va ad approssimare il modello black-box utilizzando un modello lineare nell’intorno dell’input di interesse, l’explanation della predizione corrisponderà ai pesi di tale modello lineare. Nel caso delle immagini, il modello lineare non utilizza come features direttamente i pixel dell'immagine in input. Tali pixel vengono infatti prima raggruppati in superpixel mediante un algoritmo di segmentazione e ciascun superpixel corrisponde ad una features del modello lineare locale. I coefficienti del modello lineare ricavato associati ad ogni superpixel determinano l’importanza del superpixel per la predizione restituita: maggiore è il valore del coefficiente più il superpixel sarà rilevante per la predizione effettuata.

<img src="./images/Figura_6_39.png" alt="Occlusion Map" style="width:100%;">

*Figura 6.39. LIME.*

Nella Figura 6.40 vediamo un esempio di varie metriche di attivazione applicate al problema del riconoscimento di immagini cardiache MR. In questo caso la rete doveva riconoscere se una vista in asse corto era basale, media o apicale. La prima immagine è stata classificata non correttamente, le altre due correttamente. Come si osserva, abbiamo metriche a “grana” fine come le Saliency maps e a grana grossa come le LIME. Se osserviamo le mappe occlusion vediamo che la rete ha identificato delle aree molto specifiche (intersezione superiore parete destra e sinistra nella basale e ventricolo destro nella media) quando ha fatto la classificazione corretta, mentre la mappa è molto più “confusa” quando ha fatto la classificazione sbagliata.      
In generale le tecniche di visual explanation sono estremamente dipendenti dalla tecnica utilizzata e dai parametri utilizzati nei vari metodi, per cui allo stato attuale la loro utilità in un contesto clinico è perlomeno dubbia.  

<img src="./images/Figura_6_41.png" alt="metriche di attivazione applicate al problema del riconoscimento di immagini cardiache MR" style="width:100%;">

*Figura 6.40. metriche di attivazione applicate al problema del riconoscimento di immagini cardiache MR.*

#### Dimensionality Reduction (t-sne)

Gli approcci di interpretabilità post-hoc includono anche diverse tecniche di **dimensionality reduction**. Con queste tecniche è possibile esplorare le features prodotte dalla rete nei vari strati, operando una riduzione di dimensionalità mappando le feature su di uno spazio bidimensionale (**t-sne**) (Figura 6.41). Lo spazio delle features estratte da una CNN è costituito infatti da dati N-dimensionali, non è perciò possibile visualizzarlo ed interpretarlo per poter identificare le somiglianze intra ed inter classe che la rete va ad estrarre dai dati o altre informazioni eventualmente utili per capire quali sono le caratteristiche che vengono mano a mano estratte. Può quindi essere utile impiegare tecniche di riduzione della dimensione per visualizzare in uno spazio a bassa dimensione le caratteristiche dei dati estratti nel quale la struttura significativa dei dati viene preservata per fornire una rappresentazione/intuizione di quale sia la mappatura input-output appresa dalla rete.
Tali tecniche non vengono quindi utilizzate per visualizzare il singolo input ma tutto il dataset, si tratta quindi di tecniche di **global interpretability**. In particolare, il t-SNE crea un embedding a bassa dimensione che impone che le somiglianze tra i punti dati nello spazio originale siano simili alle somiglianze nello spazio a bassa dimensione. Le somiglianze nello spazio originale sono rappresentate da probabilità congiunte gaussiane e le somiglianze nello spazio incorporato sono rappresentate da distribuzioni t a coda lunga di Student. L'algoritmo minimizza la divergenza di Kullback-Leibler (KL) delle probabilità congiunte nello spazio originale e nello spazio incorporato con discesa di gradiente.

<img src="./images/Figura_6_40.png" alt="t-sne" style="width:100%;">

*Figura 6.41. t-sne.*

Come illustrato in figura, tutte le $N$ immagini vengono mandate in ingresso alla rete addestrata e vengono salvate le N mappe di attivazione nei vari strati di interesse. Le mappe vengono riportate attraverso il t-SNE in uno spazio bidimensionale dove ogni punto rappresenta una immagine. I punti possono essere evidenziati secondo la classe “vera” o predetta. Si nota come attraversando la rete le varie classi vengano separate, evidenziando la capacità della rete di estrarre le features significative per la classificazione. 

Un esempio di uso di tecniche XAI di visual activation si trova nel Notebook: {doc}`n_6_8_VGG_CNN_XAI_test2.ipynb`

