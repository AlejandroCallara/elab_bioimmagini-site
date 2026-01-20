# Capitolo 3: Segmentazione dell’immagine Biomedica

Seguendo la classificazione della Computer Vision, la segmentazione dell’immagine biomedica è una operazione di medio livello, intermedia tra le operazioni a basso livello (interpolazione e filtraggio) e quelle ad alto livello (classificazione e registrazione). La segmentazione, o pattern recognition, realizza l’estrazione dall’immagine di regioni di interesse. Se consideriamo l’immagine biomedica in figura, la segmentazione riconosce i sei pattern fondamentali (le quattro ossa, i tessuti molli, lo sfondo) assegnandogli una label (i numeri in questo caso). Il riconoscimento dei pattern (quindi l’associazione label/nome anatomico è invece tipica di operazioni al livello superiore.

METTERE FIGURA QUI 
*Atlante anatomico da http://www.info-radiologie.ch*

Come introdotto nel capitolo 1, l'immagine biomedica può essere modellata come una immagine “ideale”, formata da una serie di pattern omogenei, corrotta da vari processi quali l'effetto volume parziale, l'attenuazione, il rumore.  

METTERE FORMULA QUI

lo scopo della segmentazione è estrarre dall'immagine reale I(x,y), che è quella a noi nota, l'immagine ideale I0(x,y) sulla base della conoscenza del processo di formazione dell'immagine e identificare i pattern che compongono I0(x,y) associando ogni pattern una etichetta o label. 
Dall’osservazione dell’equazione che descrive il modello generale di immagine biomedica è facile capire come il problema sia fortemente indeterminato, in quanto i fattori che corrompono l’immagine reale sono solo parzialmente noti.
E' importante notare come i pattern in cui vogliamo raggruppare i pixel/voxel dell'immagine non siano “oggettivi”, ma dipendano dal quesito clinico a cui vogliamo rispondere ed in base al quale viene acquisita l'immagine. Infine, non è detto che sia di interesse estrarre tutti i pattern costituenti l'immagine, anzi in generale il processo di segmentazione sarà mirato ad estrarre un numero limitato di strutture di interesse. Possiamo quindi definire il processo di segmentazione di una immagine biomedica come:

**Definiamo segmentazione di una immagine biomedica un processo che consente di associare ad un certo numero di strutture anatomiche che ci interessa individuare una lista univoca di pixel/voxel afferenti a dette strutture.**

Per segmentazione si può intendere quindi una operazione che associ ad alcuni pixel/voxel dell’immagine una etichetta (label) che individui a quale oggetto appartiene quel determinato pixel/voxel. I pixel/voxel rimanenti possono essere assegnati ad una label “indeterminata” che raggruppa pixel/voxel afferenti a strutture anatomiche diverse ma che non ci interessa riconoscere.
In questo approccio il risultato della segmentazione sono delle maschere (mask), cioè delle immagini binarie di dimensioni uguali a quella su cui viene applicata la segmentazione, che assumono valore non nullo se un pixel appartiene ad un certo oggetto e zero altrove. Per quanto detto prima un processo di segmentazione produrrà K+1 maschere, dove K è il numero di oggetti segmentati. Un metodo alternativo è creare una immagine dove ogni pixel assume un valore V(k) con k=1…K+1. Nella segmentazione di immagini 3D le maschere diverranno anch’esse degli array 3D.      

Alternativamente, i K oggetti segmentati possono essere rappresentati da K contorni, dove ogni contorno è una lista ordinata e chiusa di punti definiti sullo spazio dell’immagine. Il numero di punti del contorno definirà la precisione con cui il contorno è definito. In 3D, il contorno diviene una nuvola di punti in uno spazio 3D che definisce una superficie chiusa. Essendo l’ordinamento di una nuvola di punti in 3D non banalmente definibile, la superficie viene tipicamente definita attraverso un particolare ordinamento dato da un processo di triangolarizzazione che produce una mesh, come quella utilizzata negli algoritmi di visualizzazione 3D.   

Le rappresentazioni a maschera e a contorni sono sostanzialmente equivalenti, nel senso che è possibile passare dall’una all’altra in modo semplice. Avendo una maschera, per ottenere il contorno corrispondente si può ad esempio calcolare la distance transform che sarà introdotta nel seguito e considerare solo i livelli di griglio pari ad uno. Oppure si può calcolare il gradiente che sarà diverso da zero solo sui bordi della maschera ed adottare l’algoritmo di Canny. Per passare da un contorno ad una maschera si può utilizzare il “fence algorithm” o “Even-odd rule”. In uno spazio discreto il contorno è definito come un poligono a N lati che approssima una curva. Per ogni pixel dell’immagine si definisce una semiretta che parte dal punto ed esce fuori dall’immagine stessa, la direzione della semiretta è irrilevante. 
     
 

Se la semiretta interseca il contorno un numero dispari di volte il punto è all’interno del contorno, se la semiretta interseca il contorno un numero pari di volte (0 si considera pari) il punto è all’esterno. 
L’implementazione è abbastanza semplice, se si considera una semiretta lungo l’asse x orientata verso lo 0:












test if the pixel (x,y) is inside the contour
function flag = pointInsidePoly(x,y,poly)
 
N = length(poly);    % number of contour points
j=N;
flag=0 ; % number of crossings
for i=1:N
if ( ((poly(i,1) > y) ~= (poly(j,1) > y)) & (x < ((poly(j,2) - poly(i,2))*     (y-poly(i,1))/(poly(j,1) - poly(i,1)) + poly(i,2) )) )
        flag = not(flag);
    end
    j=i;
end

((poly(i,1) > y) ~= (poly(j,1) > y)) controlla che la coordinata y sia tra la minima e massima coordinata y del segmento.  x < ((poly(j,2) - poly(i,2))*     (y-poly(i,1))/(poly(j,1) - poly(i,1)) + poly(i,2) )) controlla che coordinata x sia a destra del segmento. Usando la funzione MATLAB pointInsidePoly : 

% fence algorithm
mask = zeros(dim);
for x=1:dim       % explore all pixels
    for y=1:dim
        % test if the pixel (x,y) is inside the contour
        c = pointInsidePoly(x,y,pos);
        mask(x,y) = c;
    end
end

In Matlab la funzione  poly2mask implementa un algoritmo simile.

Il numero di algoritmi di segmentazione sviluppati in letteratura ed utilizzati nella pratica clinica è virtualmente infinito. Nel seguito verrà introdotta una classificazione generale e verranno forniti alcuni esempi.



 
MACHINE LEARNING

Dal punto di vista del “machine learning”, un algoritmo di segmentazione è un classificatore che assegna una classe a ciascun pixel dell’immagine. Gli algoritmi di machine learning si possono suddividere in unsupervised e supervised. 


 

Si parlerà di unsupervised learning se nel processo non viene utilizzato un set di dati validato, per cui l’algoritmo è basato su di un modello che deve includere una descrizione “accurata” dei dati da classificare. La base di conoscenza è quindi racchiusa nel modello, per cui l’approccio unsupervised è anche detto model-driven. 
Invece, nel supervised learning la base di conoscenza è data da un insieme di dati labellati, ad esempio coppie di immagini con la relativa segmentazione. Il modello qui è molto più “generale” rispetto la caso unsupervised ed “impara” la sua configurazione ottimale dall’esempio fornito attraverso i dati labellati. L’approccio supervised è quindi anche detto data-driven.



 
 

Procedure di ottimizzazione

Prima di introdurre in modo formale il concetto di segmentazione di immagini, è opportuno definire il concetto di ottimizzazione, cioè della ricerca della combinazione di parametri che massimizza (o minimizza) una certa funzione. Utilizziamo come esempio il cosiddetto problema TSP o Problema del Commesso Viaggiatore (Travelling Salesman Problem, da cui la sigla TSP). La definizione del problema è abbastanza semplice: un commesso viaggiatore deve visitare un certo numero di clienti prima di tornare a casa. Conosce la posizione dei clienti e il tempo necessario a spostarsi dall'uno all'altro. Vuole ovviamente visitare tutti i clienti una sola volta nel tempo più breve possibile. In termini più formali, il problema consiste nel costruire un grafo i cui nodi rappresentano i clienti, mentre gli archi rappresentano i percorsi fra i nodi, e di trovare su di esso un ciclo che tocchi tutti i nodi una ed una sola volta e abbia la durata complessiva minima. Il problema, semplice da descrivere, è però complesso da risolvere. Il numero delle sue soluzioni, infatti, cresce molto rapidamente con il numero dei nodi. Consideriamo l’esempio in figura:

 
Nel grafo per il problema TSP il nodo 1 rappresenta il punto di partenza, i nodi 2:5 i clienti da visitare. I numeri sugli archi che connettono i nodi la lunghezza del percorso. Il tutto può essere illustrato anche in forma tabellare:
 
Notiamo che la tabella in questo caso è simmetrica, cioè la distanza tra due nodi non dipende dall’ordine. Questo può non essere vero in alcune applicazioni. Nel caso illustrato in figura le soluzioni possibili sono 4! = 24, infatti il primo nodo (1) e l’ultimo nodo (1) sono fissi ed abbiamo quindi tutte le possibili combinazioni di 4 elementi. Nel dettaglio i percorsi possibili sono: 1-2-3-4-5-1 (di durata 36), 1-2-3-5-4-1 (di durata 38), 1-2-4-3-5-1 (di durata 42), 1-2-4-5-3-1 (di durata 40), 1-2-5-3-4-1 (di durata 40), 1-2-5-4-3-1 (di durata 44), 1-3-2-4-5-1 (di durata 48), 1-3-2-5-4-1 (di durata 46), 1-3-4-2-5-1 (di durata 48), 1-3-4-5-2-1 (di durata 44), 1-3-5-2-4-1 (di durata 44), 1-3-5-4-2-1 (di durata 40), 1-4-3-2-5-1 (di durata 38), 1-4-3-5-2-1 (di durata 40), 1-4-2-3-5-1 (di durata 44), 1-4-2-5-3-1 (di durata 44), 1-4-5-3-2-1 (di durata 38), 1-4-5-2-3-1 (di durata 46), 1-5-3-4-2-1 (di durata 42), 1-5-3-2-4-1 (di durata 44), 1-5-4-3-2-1 (di durata 36), 1-5-4-2-3-1 (di durata 48), 1-5-2-3-4-1 (di durata 38), 1-5-2-4-3-1 (di durata 48).
Le soluzioni migliori quindi sono 1-2-3-4-5-1 e 1-5-4-3-2-1, entrambe di durata 36, com'era ovvio,
dato che si è considerato un grafo simmetrico.
In questo caso abbiamo risolto il problema calcolando tutte le possibili soluzioni e scegliendo la soluzione ottima. Questo approccio è detto a ricerca esaustiva, o brute force. Se il dominio di input è finito, tali algoritmi trovano sempre la soluzione corretta. 
In generale, per n nodi il numero di soluzioni possibili sarà (n-1)!, che cresce molto rapidamente con n. Per comprendere la difficoltà di gestione di un problema di tale complessità, consideriamo un calcolatore capace di compiere 4 109 operazioni al secondo (circa 4 Gflops, dell’ordine della potenza di calcolo di un PC a 4 GHz di uso comune). Ammettiamo di riuscire a computare la lunghezza di un percorso con n operazioni (in realtà ce ne vorranno certamente di più, bisognerebbe contare anche gli accessi in memoria in lettura o scrittura). Allora il computo di tutte le soluzioni richiederà un numero di operazioni pari a:

NO = n*(n-1)! = n! 

Per n=20 abbiamo N! = 2.4 1018 operazioni. 
Il tempo necessario con il calcolatore ipotizzato prima sarà (consideriamo 3 107 sec in un anno):

T = 2.4 1018/ 4 109 = 6 108 sec ~ 20 Anni

Non avendo tanta pazienza usiamo il calcolatore più veloce esistente nel 2016 (Sunway TaihuLight, China, 93000 Tera Flops) composto da 10 milioni di processori (www.top500.org). Per curiosità la potenza necessaria al funzionamento del calcolatore è 15 MW pari a quella erogata da un piccola centrale elettrica. Con questo calcolatore sarebbero necessari:

T = 2.4 1018/ 93 1015 = 26 sec 

Purtroppo già per n=25 la soluzione cinese non funziona

 T = 1.55 1025/93 1015 = 1.6 108  sec ~ 5 Anni

Il grafico successivo mostra in scala logaritmica il tempo di elaborazione stimato in anni per un computer a 93000 Teraflops. Come si osserva, problemi di tipo TSP con complessità superiore a 25 sono di fatto incomputabili. 
 
L’esempio dimostra come al crescere della dimensione del problema non sia possibile risolvere lo stesso in modo esaustivo. Il problema TSP e un esempio della famiglia  di problemi NP-completi, cioè problemi di complessità che cresce in modo non lineare con la dimensione del problema. Tali problemi non possono essere risolti in modo esaustivo quando le dimensioni dei dati di input crescono sopra una certa dimensione. Anche se esistono approcci che consentono in alcuni casi di risolvere in modo esatto un particolare problema NP-completo, in generale quello che si può fare è trovare degli algoritmi che ottengano delle soluzioni approssimate con una complessità accettabile. Tali algoritmi non potranno comunque fornire mai una soluzione sicuramente ottima, proprio perché le soluzioni possibili non sono note e quindi è impossibile determinare se una soluzione sia la migliore o meno. 
L’esempio più semplice di soluzione non esaustiva è la ricerca casuale o random. Invece di fare una ricerca esaustiva in modo sistematico utilizzando sempre lo stesso ordine di ricerca, possiamo scegliere un percorso di ricerca variabile in modo random volta per volta. Se viene esplorato tutto il dominino di input, tali algoritmi sono un caso particolare di un algoritmo esaustivo. Se viene esplorata solo una parte del dominio di input, gli algoritmi random hanno una certa probabilità di trovare la soluzione ottima pari al rapporto tra numero di percorsi esplorati e numero totale di percorsi. L’algoritmo di ricerca casuale non ha utilità pratica, ma serve come confronto per gli altri algoritmi di ottimizzazione, nel senso che qualsiasi algoritmo di ottimizzazione ragionevole deve funzionare meglio della ricerca casuale.
Un esempio di soluzione approssimata del problema TSP si può ottenere con un approccio di tipo Greedy. Un esempio di programmazione Greedy sono gli algoritmi del tipo best-first-search, dove ci si muove in un grafo che descrive un problema scegliendo via via i nodi che sembrano migliori per risolvere il problema. Riconsideriamo il problema TSP visto in precedenza: 
Partendo dal nodo 1, il nodo alla minore distanza è il nodo 2 (distanza 8). Ci muoviamo quindi nel nodo 2. Dal nodo 2 il nodo più vicino è il nodo 3 (distanza 3). Dal nodo 3 andremo nel 4 (distanza 6) e dal nodo 4 dovremo andare nel 5 (distanza 9) e poi nell’1 (distanza 10). Il percorso (1-2-3-4-5-1) ha lunghezza totale 8+3+6+9+10=36, che come si era visto è la distanza minima. Con un algoritmo semplice abbiamo quindi trovato la soluzione ottima. In particolare il numero di passi dell’algoritmo Greedy è (N-1)+(N-2)+(N-3)+…..+1, infatti al primo passo devo fare N-1 confronti, al secondo N-2, etc. L’ordine di grandezza del numero di passi è N2 molto minore del numero di passi dell’algoritmo esaustivo. In generale non è detto che in questo modo si trovi la soluzione migliore, consideriamo ad esempio il grafo:

 
L’algoritmo best-first-search ritrova il percorso ottimo visto prima 1-2-3-4-5-1=36, ma esiste un percorso migliore 1-5-2-3-4-1=35 che non viene “visto”. Questo conferma quanto si era detto a proposito del problema TSP, cioè che esistono algoritmi di limitata complessità computazionale che
danno una soluzione approssimata, ma non necessariamente la migliore.
La soluzione prodotta dall’algoritmo best-first-search dipende dal nodo iniziale che si sceglie per la partenza dell’algoritmo. La figura mostra i risultati di un algoritmo best-first-search applicato al problema TSP Berlin52 variando il nodo di partenza dell’algoritmo. 
 Si osserva come la soluzione dipenda dal nodo, la soluzione migliore è 8182. Per confronto la migliore soluzione nota del problema Berlin52 è 7542
Per concludere, dato un certo problema NP-completo da risolvere per via algoritmica, esisterà una soluzione esaustiva che risolve il problema in maniere ottima, ma sarà applicabile solo per dimensioni del problema molto piccole e non interessanti da un punto di vista pratico. Per risolvere il problema sarà necessario utilizzare un algoritmo di ottimizzazione più veloce dell’algoritmo esaustivo. Tale algoritmo non potrà in generale assicurare la soluzione ottima, ma solo una soluzione “ragionevolmente” buona. In generale la soluzione trovata da un algoritmo non esaustivo dipenderà dalle condizioni iniziali, e quindi varierà in funzione delle condizioni iniziali stesse.   

In generale un processo di ottimizzazione sarà definito da tre caratteristiche:
    Spazio di ricerca (Search-space). Lo spazio di ricerca è l’insieme dei valori che possono assumere le possibili soluzioni. Ad esempio, nel problema TSP tutte le possibili liste dei nodi senza ripetizioni.
    Metrica: La quantità da massimizzare o minimizzare, nel caso del TSP la lunghezza di un percorso.
    Il processo di ottimizzazione: L’algoritmo utilizzato per trovare all’interno del search-space la soluzione che massimizza o minimizza la metrica. Nel caso del TSP il metodo greedy.
 
Gli algoritmi di ottimizzazione possono essere classificati come:
    Ottimizzatori locali. Un ottimizzatore locale trova una soluzione partendo da un punto del search-space (condizioni iniziali). La soluzione trovata dipenderà quindi dalle condizioni iniziali. Nel problema TSP, l’algoritmo greedy applicato ad un singolo nodo iniziale è un ottimizzatore locale e la soluzione dipende dal nodo selezionato.
    Ottimizzatori globali. Un ottimizzatore globale trova la soluzione migliore all’interno del search-space indipendentemente dai parametri di ingresso. Come detto in precedenza, l’unico vero ottimizzatore globale è la ricerca esaustiva. Nel caso del TSP un algoritmo greedy iterato su tutti i nodi si può considerare una approssimazione di un ottimizzatore globale, in quanto il risultato non dipende dalle condizioni iniziali.





 
Segmentazione a Soglia

Nella segmentazione a soglia ogni pixel viene associato ad un tipo di tessuto mediante l’intensità di segnale. La segmentazione a soglia corrisponde quindi a scegliere una o più soglie nell’istogramma dell’immagine e a classificare i pixel mediante tale informazione. Consideriamo il fantoccio MR in figura:

       
Il fantoccio è costituito da 6 pattern (in realtà c'è anche una parte esterna indistinguibile dallo sfondo data dallo zero padding) come in tabella:

    Pattern	Tessuto	Soglie
1	Sfondo	Aria (sfondo)	<90
2	Cilindro acqua esterno	Acqua	>90  <500
3	Parete cilindro ext	Vetro (sfondo)	<90
4	Cilindro olio	Olio	>500
5	Parete cilindro int	Vetro (sfondo)	<90
6	Cilindro acqua interno	Acqua	>90  <500

I sei pattern sono associati a quattro diversi “tessuti” (aria, vetro, olio, acqua). L’aria e il vetro in MR sono due oggetti del tutto equivalenti, in quanto non forniscono alcun segnale MR. Sono quindi impossibili da distinguere con una segmentazione a soglia e li possiamo raggruppare in una classe “sfondo” (background), ottenendo quindi tre classi. Analogamente le due regioni contenenti acqua hanno lo stesso segnale. Consideriamo l'istogramma dell'immagine. Appaiono i tre picchi che descrivono la distribuzione del segnale, con ampiezza proporzionale al numero di pixel. Per dividere le classi occorrono due soglie (il numero di soglie è N-1 dove N è il numero di classi). Fissiamo le soglie a T1=90 (sfondo-acqua) e T2=500 (acqua-olio). Estraiamo dall'immagine i pixel con valori s tali che s<T1, T1≤s<T2, s≥T2 e poniamo ad un valore diverso da 0 tali pixel e a zero gli altri. Otteniamo quindi tre maschere (mask) che descrivono la distribuzione delle tre classi sull'immagine.       

     

Le tre maschere rappresentano (colore bianco) la distribuzione dei pixel appartenenti alle classi sfondo, acqua e olio rispettivamente.
Notiamo che alcuni pixel sono stati erroneamente assegnati alla maschera “sfondo” anche se sono acqua. Questo è dovuto alla imperfetta separazione dei picchi sfondo-acqua nell'istogramma.   
Dall'esempio individuiamo i limiti fondamentali della segmentazione a soglia:

    La segmentazione a soglia non è in grado di distinguere oggetti topologicamente diversi ma ai quali è associato lo stesso livello di segnale. Nel nostro caso i pattern 1-3-5 e 2-6 vengono rappresentati in una sola maschera. Questo rappresenta un problema in molte applicazioni cliniche (es: ventricolo destro e sinistro nel cuore, grasso viscerale e grasso subcutaneo, etc).
    Possono apparire pixel spuri dovuti alle variazioni di segnale indotte dal rumore
    Le soglie vengono definite manualmente tramite esame visivo dell'istogramma.

Il punto 1) è un limite intrinseco della segmentazione a soglia. Come si vedrà nel seguito può essere superato facendo seguire alla segmentazione a soglia l’applicazione di algoritmi di tipo topologico applicati alle maschere prodotte dalla segmentazione a soglia, come il labeling. 
Il punto 2) implica che la qualità della segmentazione ottenibile con un algoritmo a soglia dipende sostanzialmente dal CNR tra i due tessuti da separare. Anche in questo caso gli errori di segmentazione possono essere corretti, almeno in parte, applicando algoritmi di filtraggio al risultato della segmentazione. 
Per quanto riguarda il punto 3) esistono vari approcci automatici che consentono di trovare la soglia “ottima” che divide i pixel in classi attraverso soglie opportune. Tali algoritmi adottano un approccio “unsupervised learning”.
 
Algoritmo di Otsu (adattato da Wikipedia Italia)

L'algoritmo di Otsu nella sua versione originale presume che nell'immagine da segmentare siano presenti due sole classi e quindi calcola la soglia ottima per separare queste due classi minimizzando la varianza intra classe. L'algoritmo può essere esteso a più classi (multi Otsu method). L'algoritmo di Otsu standard minimizza la quantità (varianza intra classe):
  

dove ω è la probabilità di una classe separata dall’altra dalla soglia t e σ è la SD della classe. Quindi l’algoritmo cerca di trovare la soglia che separa l’immagine in due sottoclassi che siano il più possibile omogenee al loro interno. Intuitivamente, immaginando un istogramma a due picchi il metodo di Otsu consiste nel trovare la soglia t intermedia tra i due picchi che li divida in modo ottimale.
Si dimostra che minimizzare la varianza intra-classe è equivalente a massimizzare la varianza inter-classe: 


dove  μ è il valor medio di una classe. Il metodo consiste nel provare in modo esaustivo tutti i possibili t (che sono in numero uguale alla profondità dell'immagine) e prendere il t che minimizza la varianza inter classe. Da un punto di vista algoritmico: 

    Si calcola l'istogramma h dell'immagine. La probabilità ω si ottiene normalizzando l'istogramma per il numero di pixel dell'immagine (da un punto di vista pratico la normalizzazione non è necessaria).

    Si calcola il valore di σ2b(t) per ogni t calcolando
                                         
    e  ω2(t)  e μ2(t) in modo analogo operando le somme da t+1 in avanti. 

    Si sceglie il t che massimizza σ2b(t)

In questo modo la soglia t viene definita in modo automatico. In MATLAB il metodo di Otsu è implementato nella funzione otsuthresh quando si utilizza l’istogramma estratto dall’immagine o  graythresh quando si opera direttamente sull’immagine stessa.
Il metodo di Otsu opera secondo un processo di ottimizzazione basato sulla ricerca esaustiva su tutti i possibili valori del parametro t da ottimizzare, ed è quindi non particolarmente efficiente da un punto di vista computazionale. Tuttavia, essendo il calcolo della quantità σ2b(t) da ottimizzare molto rapido il calcolo di t può essere effettuato in tempi ragionevoli. Notiamo che il tempo di calcolo è fortemente dipendente dal valore di profondità dell’immagine che determina il numero di prove da effettuare e la lunghezza delle somme da calcolare. Si noti infine che possono esistere più valori di t per cui σ2b(t) è massima, si pensi al caso di un istogramma con due picchi ben separati. In questo caso l’algoritmo ritornerà il valor medio tra i valori di t che massimizzano σ2b(t).


Clustering

Un altro metodo per la scelta dei valori di soglia corretti che minimizzino l’errore nella segmentazione è l’uso di algoritmi di clustering.  Il Clustering è una tecnica di analisi dei dati volta alla selezione e raggruppamento di elementi omogenei in un insieme di dati. 
L’approccio è del tipo unsupervised, nel senso che l’algoritmo di clustering riesce a dividere i dati in una serie di insiemi avendo come unico input il numero di insiemi da trovare. Gli algoritmi di Clustering si possono applicare a dati di diversa natura, e trovano applicazione in molteplici discipline come la bioinformatica, il marketing, la genetica, etc. 

    OBS 1	OBS 2	OBS 3			OBS K
E 1						
E 2						
E 3						
                        
E N						


In generale i dati di ingresso di un algoritmo di clustering sono composti da una tabella con N righe, che corrispondono agli elementi da analizzare, e da K colonne che corrispondono alle osservazioni disponibili sugli elementi stessi. Ad esempio le righe della tabella potrebbero rappresentare dei pazienti e le colonne dei dati clinici sui pazienti (analisi del sangue, valori di pressione, etc). Lo scopo del clustering è raggruppare gli elementi simili tra loro in gruppi (cluster) sulla base delle osservazioni.
In figura è esemplificato un problema classico di clustering: abbiamo un insieme di dati (caratterizzati da una coppia di valori) intuitivamente raggruppabili in quattro classi e vogliamo ottenere la soluzione a destra in cui i dati sono opportunamente raggruppati. In questo caso l’osservazione intuitiva che i dati si raggruppano in quattro cluster corrisponde all’utilizzo come distanza tra i dati della distanza geometrica sul piano. Punti tra loro “vicini” secondo la distanza scelta sono assegnati allo stesso insieme. 


 

Nel campo dell’analisi delle immagini biomediche, le righe della tabella rappresentano le locazioni spaziali su cui vengono acquisite le immagini (quindi i pixel/voxel dell’immagine), mentre le colonne rappresentano i valori di segnale acquisiti in corrispondenza delle locazioni spaziali. In definitiva, l’input dell’algoritmo di clustering sarà una matrice NxM, dove N è il numero di locazioni spaziali considerate (numero di pixel o voxel) e M il numero di acquisizioni disponibili.
Quindi per una immagine 2D che contiene Nx x Ny pixel, la matrice di clustering corrispondente sarà un matrice a Nx x Ny righe ed una singola colonna. Per una immagine 3D la matrice di clustering corrispondente sarà una matrice a Nx x Ny xNz righe ed una singola colonna. Una immagine a colori RGB corrisponderà ad una matrice a tre colonne (K=3), in quanto per ogni pixel sono disponibili tre misure corrispondenti alla relativa tripletta RGB. 

 

Una immagine 2D +T corrisponderà ad una matrice di clustering con N = Nx x Ny e K = nT, in quanto per ogni pixel sono disponibili nT osservazioni ad intervallo di tempo diversi. Analogamente una immagine 3D + nT corrisponderà a una matrice di clustering con N = Nx x Ny x Nz e K = nT. E’ importante notare che la corrispondenza ha senso se gli nT frame temporali sono allineati, cioè se i pixel nella serie temporale corrispondono alle stesse locazioni spaziali. Ad esempio una serie temporale in cui il paziente è fermo e viene iniettato un contrasto che cambia i livelli di grigio può essere interpretata in questo modo, mentre una serie temporale che segue il battito cardiaco no. Altri esempi in MRI sono immagini T1, T2, PD pesate dello stesso distretto anatomico. Oppure potremmo avere immagini multiecho acquisite a tempi di eco diversi.   

 

Gli algoritmi di clustering usati nella segmentazione di immagini si possono classificare in tre classi: 

    Clustering esclusivo (k-means)
    Clustering non esclusivo (Fuzzy C-mean)
    Clustering probabilistico

Nel clustering esclusivo, un dato deve appartenere ad uno ed un solo cluster. L’implementazione più nota di questo approccio è l’algoritmo K-MEANS.

Algoritmo K-means

L’algoritmo K-means (MacQueen, 1967) è l’algoritmo base ed uno dei primi ad essere stato realizzato per la classificazione di dati. Dato un insieme Y = (y1,…yn) di dati da classificare in C gruppi (cluster) e una distanza || || definita su Y, possiamo definire una funzione obiettivo da minimizzare. Questa funzione è: 
  
dove è la distanza tra un dato yj e un centroide vk. Per ogni dato vengono contate solo le distanze rispetto al centroide più vicino. Questa funzione obiettivo è un indicatore della distanza dei dati y dai centri dei rispettivi cluster. La distanza utilizzata può essere una qualsiasi metrica. Il problema di trovare l’assegnazione dei dati ai cluster che realizzi il minimo di J è NP-completo con complessità computazionale O(ndC+1 log n) dove d è la dimensionalità dei dati. Il problema per n non piccolissimo non è quindi risolubile in modo esaustivo e si utilizzano algoritmi in grado di trovare un minimo locale di J. 
La procedura più nota è abbastanza semplice e richiede la definizione di un numero di cluster (C) in cui dividere i dati. Si definiscono C centroidi, uno per ogni cluster da trovare, in modo casuale ma in modo che siano abbastanza lontani tra loro, in modo da migliorare la convergenza dell’algoritmo. A questo punto per ogni dato calcoliamo la distanza dai centroidi ed associamo il dato al centroide più vicino. Calcoliamo ora C nuovi centroidi, come centro di massa dei cluster ottenuti al passo precedente. Possiamo ora ricomputare la distanza di tutti i dati dai nuovi centroidi e creare così dei nuovi cluster. Iterando il procedimento, arriveremo ad un punto in cui i centroidi si stabilizzano senza cambiare più di posizione. Saremo quindi in una situazione di convergenza dell’algoritmo che ci dà il clustering desiderato. 

La procedura per trovare il valore minimo di J è la seguente:
    Assegnare un valore iniziale ai C centroidi vk 
    Iterare i due passi seguenti fino a quando i valori di vk si modificano:
    Assegnare ogni dato y ad uno ed un solo cluster sulla base della distanza di y dal centroide del cluster.
    Aggiornare i valori dei centroidi vk come media dei dati y appartenenti al centroide k 

La procedura descritta precedentemente minimizza la funzione obiettivo, ma non è ovviamente un procedimento esaustivo. È possibile quindi provare che l’algoritmo k-means converge sempre, ma non è detto che la configurazione di convergenza sia quella che trova un minimo assoluto della funzione obiettivo. L’algoritmo k-means è quindi un ottimizzatore locale. Poiché lo stato di convergenza dell’algoritmo dipende dalla definizione iniziale dei centroidi, è possibile ripetere il procedimento più volte assegnando in modo casuale il valore iniziale dei centroidi e tra le varie configurazioni stabili trovare quella dove la funzione obiettivo è minima. 
Si noti che dal punto di vista dell’algoritmo di clustering il numero di dimensioni che caratterizza i dati y e quindi i centroidi v è irrilevante. In generale y sarà un vettore a n dimensioni e v avrà n dimensioni come y. La distanza verrà calcolata nello spazio n-dimensionale dei dati y.  
In MATLAB l’algoritmo K-MEANS è implementato nella funzione kmeans (Statistics Toolbox).

L’algoritmo presenta alcune difficoltà:
    È necessario un algoritmo per inizializzare i centroidi. Un possibile metodo è far coincidere i C centroidi iniziali con C campioni scelti a caso dai dati. Oppure è possibile definire i valori iniziali dei centroidi sulla base della conoscenza a priori dei dati (opzione start di kmeans)
    Il risultato finale dipende dai centroidi iniziali. Questo può essere sfruttato facendo girare più volte l’algoritmo in dipendenza da condizioni iniziali diverse e prendendo come risultato ottimo quello che minimizza J. (opzione nreplicates di kmeans).
    Può accadere che il un cluster si svuoti, per cui non può essere aggiornato. In questo caso l’algoritmo si blocca (opzione EmptyAction).
    Il risultato dipende dalla metrica (opzione Distance). A volte è opportuno normalizzare le variabili rispetto alla deviazione standard (variabili standardizzate). 
    Il risultato dipende dal numero C di cluster che è predefinito. 
Riguardo l’ultimo punto, in generale non esiste un metodo per la stima del numero ottimo di cluster. Si utilizza un modello (ad esempio nella segmentazione di immagini possiamo conoscere a priori quanti tessuti vogliamo segmentare), oppure si fanno diversi tentativi con C diversi e si cerca di stimare la soluzione migliore utilizzando il valore di J.
Una variante del k-means è il metodo “Iterative Self-Organizing Data Analysis Technique” (ISODATA), che è sostanzialmente un algoritmo k-means in cui il numero di cluster non è un parametro di input non modificabile ma uno dei risultati dell’algoritmo che viene computato insieme ai cluster. Nell’approccio ISODATA due cluster possono essere combinati tra loro se la loro separazione è minore di una certa soglia mentre un singolo cluster può essere diviso in due se è troppo “disperso” nello spazio di ricerca. 
Quando applicato alla segmentazione di immagini, la metrica utilizzata è tipicamente la differenza assoluta o la media quadratica delle differenze dei livelli di grigio. Nel caso di immagini singole le due metriche sono chiaramente coincidenti. Nel caso di sequenze temporali, si possono usare metriche di tipo correlativo che descrivono ad esempio la correlazione temporale tra la variazione del livello di segnale dei pixel nel tempo.
Nel caso dell'immagine del fantoccio introdotta in precedenza, l’analisi visiva dell’istogramma ci suggerisce l’uso di C=3 cluster. Il problema è chiaramente monodimensionale (singola immagine). L'algoritmo kmeans con tre cluster ci fornisce tre centroidi (12.4, 174, 921) che rappresentano i valori tipici dei tre tessuti e le maschere dei tre tessuti sull'immagine:
        
  

Questa rappresentazione è alternativa rispetto alle tre maschere distinte che avevamo introdotto in precedenza ma ha l’identico significato. Il risultato è molto simile a quello ottenuto con la valutazione manuale delle soglie, con la differenza che qui le soglie stesse vengono trovate automaticamente. Questo tipo di maschere possono essere visualizzate a falsi colori dove ad ogni colore corrisponde un tessuto come nella rappresentazione di destra.   

Supponiamo ora di acquisire lo stesso fantoccio con due sequenze MR diverse, come in figura. Il livello di grigio associato ai tre tessuti sarà diverso tra le due immagini. Ogni dato sarà quindi caratterizzato da una coppia di valori. Il funzionamento dell’algoritmo di clustering è esattamente lo stesso con la differenza che ora la distanza tra due pixel non è più la differenza assoluta tra due valori ma la distanza (euclidea o di Manhattan) tra i due pixel in uno spazio bi-dimensionale. 
 
in questo caso i dati da fornire all'algoritmo kmeans saranno un vettore Nx2, dove N è il numero di pixel dell'immagine. L'algoritmo k-means ci restituirà il valore dei centroidi, che sarà rappresentato come un vettore 3x2, perché ogni centroide è definito in uno spazio bidimensionale, ed una maschera come in precedenza.    
 
il grafico rappresenta la distribuzione degli N pixel in dipendenza dal valore che il segnale dei pixel assume sulle due immagini. Il kmeans fornisce come coordinate dei centroidi:

   14.4741   16.2826			sfondo
  170.3552  391.8894			acqua
  923.4150  758.6847			olio

che rappresentano i valori tipici di segnale dei tessuti nelle due immagini, e la maschera:


Il metodo può essere esteso a qualunque numero di immagini.

È importante notare come applicare un clustering multidimensionale abbia senso solo se si hanno più misure su una stessa locazione spaziale. Le immagini 3D (in cui ogni voxel corrisponde ad una diversa locazione spaziale) vengono elaborate con un algoritmo di clustering monodimensionale esattamente come le immagini 2D, quindi come una lista di pixels (o voxel). 

     
 
Clustering non esclusivo (FCM)

Nel clustering non esclusivo, spesso definito fuzzy clustering, un dato può appartenere a più cluster con diversi livelli di appartenenza. La somma dei livelli di appartenenza su tutti i possibili cluster per un dato dovrà essere 1. L’algoritmo fuzzy c-means (FCM) quindi generalizza l’algoritmo K-means, consentendo una segmentazione più graduale basata sulla teoria del set di regole fuzzy. 
La fuzzy logic o logica sfumata è un'estensione della logica booleana, basata su un grado di verità di ciascuna proposizione. È fortemente legata alla teoria degli insiemi sfocati e, dopo essere già stata intuita da pensatori precedenti, venne concretizzata da Lotfi Zadeh. 
La teoria degli insiemi fuzzy costituisce un'estensione della teoria classica degli insiemi poiché per essa non valgono i principi aristotelici di non contraddizione e del terzo escluso (o del Tertium non datur). Il principio di non contraddizione stabilisce che, dati due insiemi A e !A (non-A), ogni elemento appartenente all'insieme A non può contemporaneamente appartenere anche a non-A; l’intersezione di A e !A è l’insieme vuoto. Secondo il principio del terzo escluso, se un qualunque elemento non appartiene all'insieme A, esso necessariamente deve appartenere al suo complemento non-A. L'unione di un insieme A e del suo complemento non-A costituisce il dominio completo di definizione degli elementi di A.
La modifica introdotta dalla logica Fuzzy è di rifiutare questo assunto. Quando parliamo di grado di verità o valore di appartenenza intenderemo che una proprietà può assumere oltre che i valori vera (valore 1) o falsa (valore 0) come nella logica classica, anche valori intermedi. In logica fuzzy si può ad esempio dire che un bambino appena nato è giovane di valore 1, un diciottenne è giovane 0,8, ed un sessantacinquenne è giovane di valore 0,15 (tale valore dipende dall’età del docente che tiene il corso). Solitamente il valore di appartenenza si indica con u. È importante notare che il concetto di appartenenza fuzzy non ha nulla a che vedere con il concetto di probabilità. Nella probabilità una affermazione o è vera o è falsa con una certa probabilità, mentre nella logica fuzzy è insieme vera e falsa.
L’algoritmo fuzzy c-means (FCM) ha quindi la particolarità di consentire ad un dato di appartenere a più cluster contemporaneamente. Il metodo è stato ideato da Dunn nel 1973 e migliorato da Bezdek nel 1981). Il metodo è basato sulla minimizzazione della funzione obiettivo:
                                                      
dove m è un numero strettamente maggiore di 1, ujk è il grado di appartenenza di yj rispetto al cluster k, yj è il j-esimo di N dati d-dimensionali appartenenti all’insieme Ω, vk è il centro d-dimensionale  del cluster k, e ||*|| è una distanza. C è il numero di cluster.  m è un parametro detto fuzzyness dell’algoritmo. Di solito si pone m=2. Si noti che se u è una matrice binaria, cioè può assumere solo i valori 0 e 1, la funzione J diviene uguale a quella definita per il K-means, che si può quindi considerare un caso particolare dell’algoritmo FCM. In particolare, come si vede dalla figura al crescere del valore di m il peso dei valori di appartenenza “piccoli” nel computo di J diviene sempre minore, fino ad annullarsi per valori di m molto grandi. L’algoritmo FCM va quindi a coincidere con il k-means per m che tende ad infinito.  

La complessità computazionale del problema è equivalente a quella del K-means, per cui anche il partizionamento fuzzy è ottenuto attraverso una ottimizzazione iterativa della funzione obiettivo, in modo simile a quanto visto per il K-means, attraverso l’aggiornamento della funzione di appartenenza uik e dei centri dei cluster vk:

u_(j,k)=1/(∑_(q=1)^C▒(‖y_j-v_k ‖/‖y_j-v_q ‖ )^(2/(m-1)) )        v_k=(∑_(j=1)^N▒〖u_(jk )^m y_j 〗)/(∑_(j=1)^N▒u_(jk )^m )  

Il processo iterativo si ferma quando la differenza tra il valore corrente di u ed il valore precedente è più piccola di una soglia. La procedura descritta converge ad un minimo locale della funzione obiettivo, JFCM. Si noti che a differenza del k-means dove la condizione di convergenza è univoca nel FCM la condizione di convergenza dipende dalla soglia utilizzata.
L’algoritmo FCM presenta gli stessi problemi associati all’algoritmo k-means, con la possibile eccezione dello svuotamento dei cluster che è meno probabile per la natura continua (e non discreta come nel k-means) del processo di ottimizzazione.

L’algoritmo di fuzzy clustering è implementato in matlab attraverso la funzione fcm (Fuzzy Logic Toolbox). 
Se applichiamo l'algoritmo allo stesso fantoccio utilizzato in precedenza otteniamo i centroidi:

   13.53   15.86
  170.76  392.26
  925.27  773.48

chiaramente simili a quelli precedenti. Otterremo invece tre maschere, una per ogni tessuto, dove i valori delle maschere non sono più binari ma sono distribuiti tra 0 e 1. Un valore 1 (bianco) indica la massima appartenenza del tessuto al cluster.

              
L’approccio FCM è di interesse nella segmentazione delle immagini mediche in quanto consente di tener conto del PVE. Infatti in una immagine biomedica il segnale di alcuni pixel/voxel sarà dato da due o più tessuti che si compenetrano nella stessa regione elementare di spazio nella quale viene misurato il segnale. L’approccio FCM consente di assegnare a tali pixel/voxel un valore di appartenenza distribuito tra i tessuti presenti, interpretando nel modo corretto il PVE. 

A titolo di esempio, consideriamo l’immagine in figura (imageNS) formata da sei pattern omogenei, di valore [20, 70, 150, 300, 550, 750], con l’aggiunta di rumore gaussiano e l’applicazione di un filtro a media mobile per simulare il PVE.  

 
Abbiamo evidentemente C = 6. Applichiamo all’immagini il FCM attraverso la funzione MATLAB fcm.  

[center,U,J] = fcm(imageNS(:),6)

Si noti che abbiamo trasformato l’immagine in un vettore monodimensionale attraverso l’operatore (:). La funzione fcm restituisce il valore dei centroidi (center), la funzione di appartenenza (U) e il valore J della funzione obiettivo durante le iterazioni. Il grafico della funzione J mostra come l’algoritmo minimizzi il valore di J e si fermi quando J non varia più in modo significativo.   

 

I valori dei centroidi risultanti sono [19.95,   69.78,  153.42,  300.54,  548.46,  748.53]  e rappresentano una stima del valore di segnale dei 6 pattern omogenei. 
U è un array 6xN dove N è il numero di pixel dell’immagine. Ognuna delle 6 componenti di U rappresenta la mappa di appartenenza per un certo cluster. La mappa di appartenenza del cluster di valore massimo (750) risulta come in figura a sinistra, con un valore massimo (1) in corrispondenza del pattern.   

   
A destra viene riportata la mappa di appartenenza del cluster con valore 550. Come si vede i pixel di bordo del pattern che assumono un valore intermedio tra 550 e lo sfondo hanno un grado di appartenenza minore di 1 (intorno a 0.5), tranne nell’interfaccia tra il pattern “550” e il pattern “300” dove l’appartenenza assume un valore più alto in quanto il segnale risultante dal PVE è più vicino al valore di segnale del pattern. Notiamo infine che ai pixel di bordo del pattern “750” viene assegnato un grado di appartenenza elevato al cluster “550”. Tali pixel infatti risultano dal PVE tra il pattern “750” e il pattern “70” e quindi possono assumere valori simili al pattern “550”. Questa errata attribuzione è tipica degli algoritmi di segmentazione a soglia quali l’algoritmo FCM, che non avendo connotazioni topologiche non è in grado di distinguere strutture topologicamente connesse o meno.   
Algoritmo EM (Expectation Maximization)

Gli algoritmi di segmentazione prima visti non assumono alcuna ipotesi sulla distribuzione probabilistica dei dati. Questo in generale può non essere corretto. 
Consideriamo ad esempio la separazione tra aria e acqua in un fantoccio MR esaminandone l'istogramma:


Un algoritmo di clustering (o il metodo di Otsu) troverà una soglia che massimizza la separazione  tra i due picchi. Questa soglia però non terra conto del fatto che a causa dell'effetto volume parziale i pixel dell'acqua possono assumere comunque livelli di grigio inferiori alla soglia sui bordi. Inoltre la distribuzione dell'aria è non gaussiana (è di Rician) e quindi i pixel dell'aria hanno maggior probabilità di superare la soglia rispetto a quelli dell'acqua. L'approccio ti tipo EM consente di tener conto di tali peculiarità introducendo informazioni sulla distribuzione di probabilità dei vari cluster.   
L’approccio di tipo EM introduce quindi il clustering basato su modelli (model-based approach), dove la distribuzione dei dati sui singoli cluster viene modellata attraverso funzioni probabilistiche note, tra le quali la distribuzione di tipo gaussiano è la più utilizzata. L’approccio modellistico consente di tenere in conto eventuali ipotesi sulla generazione dei dati e sul rumore che li accompagna. In pratica, ogni cluster è rappresentato matematicamente da una distribuzione di tipo parametrico, come le distribuzioni Gaussiana (Continua) o di Poisson (Discreta). L’insieme dei dati è modellato come una combinazione di queste distribuzioni. Le singole distribuzioni sono chiamate distribuzioni componenti. 

Il caso più semplice nel clustering probabilistico è quello della combinazione di gaussiane (mixture of Gaussian). I cluster sono modellati come gaussiane centrate sui centroidi dei cluster. In figura i cerchi in grigio rappresentano la varianza delle distribuzioni. Possiamo pensare che i due assi rappresentino il livello di grigio dei pixel di due immagini ed i punti nel piano il livello di grigio del singolo pixel. Vogliamo raggruppare gli spot in due classi, supponiamo quindi di avere due cluster (k=2). Dato un punto del piano, il punto avrà una certa probabilità di essere stato generato da ognuno dei due cluster, probabilità che dipende dai parametri della gaussiana che descrive il cluster stesso. In particolare, una gaussiana con un centro “lontano” dal punto o con una varianza piccola avrà basse probabilità di aver generato il punto, e viceversa. Nel caso delle immagini, una immagine con basso SNR avrà più possibilità di generare un pixel lontano dal suo valor medio e viceversa. 
 

Se i parametri che descrivono le gaussiane sono noti, il problema si riduce ad utilizzare come distanza nell’algoritmo di clustering la probabilità che un certo dato sia stato generato da una certa gaussiana. Avremo quindi:

  

Il problema diviene più complesso quando i parametri caratterizzanti le gaussiane non sono noti. L’algoritmo di clustering dovrà quindi stimare oltre ai cluster anche i parametri delle distribuzioni componenti. Quello che vogliamo trovare sono quindi i parametri delle gaussiane che hanno la maggiore probabilità di aver generato i dati osservati. 

Il procedimento da utilizzare, detto EM (expectation maximization) è di tipo iterativo. L’algoritmo EM è molto generale e può essere utilizzato per le più diverse distribuzioni di probabilità. Facciamo un esempio semplice nell’ipotesi che le distribuzioni di probabilità siano Gaussiane.

Il problema si può formalizzare come segue:
Siano date K classi; nel caso della segmentazione le classi corrispondono ai pattern dell’immagine.

Ogni classe abbia una probabilità  “a priori” Pk; nel caso della segmentazione la probabilità a priori corrisponde al numero normalizzato dei pixel appartenenti alla classe (altezza del picco nell’istogramma).

Ogni classe abbia media Mk e SD k; nel caso della segmentazione media e SD descrivono la posizione e larghezza del picco dell’istogramma relativo alla classe.

K (numero delle classi) è noto mentre gli altri parametri devono essere determinati.

Se ciascuna classe obbedisce ad una legge di probabilità gaussiana si può definire per la distribuzione dei livelli di grigio dell’immagine (cioè l’istogramma dell’immagine): 

h(s)=∑_(k=1)^K▒〖P_k G(s|M_k,σ_k ) 〗

Si tratta di scegliere il vettore delle probabilità a priori P, delle medie M e delle SD σ in modo da minimizzare la differenza tra istogramma e somma di gaussiane. Il problema può essere affrontato tramite l’algoritmo EM che è composto da due passi che vengono iterati. Nel caso della gaussian mixture, l’algoritmo EM è abbastanza semplice ed è composto dai due passi seguenti:

Si definisce un valore iniziale dei valori P, M e σ. Essendo l’algoritmo EM un ottimizzatore locale il risultato finale dipenderà dai valori iniziali. Dati P, M e σ si calcolano le probabilità che un certo valore di segnale sia stato generato da un certa gaussiana : 

Passo E-step (expectation) calcolo della probabilità che il dato xj sia stato generato dalla gaussiana i. 

p_ij=P_i G(x_j |M_i,σ_i )

che è facilmente ricavabile dalla formula della gaussiana, essendo il valore della gaussiana stessa nel punto per la probabilità a priori che un pixel appartenga a quella gaussiana. 

Il secondo passo è detto M-step (maximization) e consiste nell’aggiornare i parametri delle gaussiane in base al passo precedente. Definiamo:

 

Il passo M consiste nelle operazioni:
  
 

In questo modo abbiamo definito delle nuove gaussiane e torniamo al passo E. L’algoritmo al solito si ferma quando i valori della media, della varianza e dei pesi si stabilizzano. A questo punto i parametri trovati descrivono i cluster. 
E’ possibile dimostrare che l’algoritmo EM aumenta il grado di verosimiglianza della combinazione di gaussiane ad ogni iterazione e converge sotto certe condizioni ad un massimo locale della verosimiglianza della distribuzione di gaussiane. La dimostrazione è estremamente complessa e non viene qui riportata.   L’algoritmo EM può essere utilizzato non sono nella risoluzione del problema della gaussian mixture, ma in molte altre applicazioni. 
I problemi fondamentali in cui può incorrere l’algoritmo EM applicato alle gaussian mixture sono il fatto che due gaussiane convergano alla stessa gaussiana, con la scomparsa di un cluster, e che una gaussiana assuma varianza infinita e quindi si trasformi in un valore costante. Al solito variare le condizioni iniziali partendo da una stima ragionevole dei parametri può ottimizzare il funzionamento dell’algoritmo EM.
L'algoritmo EM-GM è implementato in Matlab dalla funzione fitgmdist nello Statistics and Machine Learning Toolbox.

 Applicato all'immagine del fantoccio MRI, il metodo fornisce tre gaussiane, con valori medi   11.7349  170.7140 905.0376 simili ai centroidi dei cluster prima trovati, e  varianze 137, 858, 12000, probabilmente per l'effetto del volume parziale tra acqua e olio che costringe a “spalmare” le due gaussiane.

La somma delle gaussiane trovate rappresenta poi un approssimazione dell'istogramma dell'immagine, per cui l'algoritmo EM può anche essere considerato un metodo per modellare l'istogramma.    


Clustering Gerarchico

In generale, è possibile che un cluster di dati possa essere diviso a sua volta in sub-cluster più piccoli, che a loro volta possono essere divisi in sub-cluster ancora più piccoli e così via. Un esempio tipico è la classificazione degli organismi vegetali o animali, che vengono divisi in categorie sempre più specifiche (ordini, specie). Nell’imaging il clustering gerarchico può essere utilizzato per raggruppare immagini appartenenti ad esempio ad una serie temporale, come si vedrà nel capitolo dedicato agli algoritmi di registrazione. Il clustering gerarchico è una tecnica che permette di creare un albero gerarchico, in cui gli elementi dell’insieme su cui si opera il clustering sono le foglie dell’albero. Una riga orizzontale nell’albero individua una serie di cluster analoghi a quelli ottenuti nel clustering tradizionale.
Dato un insieme di N oggetti da raggruppare, ed una matrice NxN di distanze tra gli oggetti, il processo di clustering gerarchico può essere definito come (S.C. Johnson 1967): 
    Si definiscono N cluster, uno per ogni oggetto. Abbiamo quindi N cluster che contengono ognuno un solo oggetto. Le distanze tra i cluster saranno uguali alle distanze tra gli oggetti che in questo primo passo si identificano con i cluster stessi. 
    Si identificano i due cluster più vicini nel senso della distanza adottata, cioè più simili. Questi due cluster vengono raggruppati in un cluster unico, abbiamo così N-1 cluster, uno con due oggetti e gli altri con un solo oggetto. 
    Ricomputiamo la matrice delle distanze, che sarà ora una matrice N-1xN-1. 
    Si ripetono i passi 2 e 3 fino a quando non rimane un solo cluster che contiene N oggetti. 
Il passo 3 può essere eseguito in modi diversi, in base ai diversi approcci possibili riconosciamo tre categorie di clustering gerarchico: single-linkage (singolo collegamento), complete-linkage (collegamento completo)  e average-linkage (collegamento mediato).
Nel single-linkage clustering, la distanza tra due cluster è definita come la minima distanza tra tutti gli elementi di un cluster e tutti quelli dell’altro cluster. In pratica si calcola la matrice delle distanze tra gli elementi dei due cluster e si prende come distanza tra i due cluster il minimo sulla matrice. Se invece di una funzione distanza si utilizza una funzione di similarità, cioè una funzione che è grande quando i due oggetti sono simili, si considererà il massimo della matrice di similarità. 
Nel complete-linkage clustering (chiamato anche metodo del diametro o del massimo), la distanza tra due cluster sarà definita come il massimo sulla matrice delle distanze computata tra I dati sui due cluster. Nell’average-linkage clustering, la distanza tra due cluster sarà la media della matrice delle distanze computata sui dati appartenenti ai due cluster. Una variazione abbastanza usata del metodo precedente è il metodo che usa la mediana della matrice delle distanze invece che la media, che è meno sensibile a dati spuri. 
L’approccio descritto finora è di tipo agglomerativo, perche si basa sul ragruppamento progressivo di cluster sempre più grandi. Esiste anche un approccio inverso, dove si parte da un cluster che comprende tutti i dati e si procede per divisioni successive (divisive hierarchical clustering). Questo approccio è comunque molto meno diffuso. 


Esempio di Single-Linkage Clustering
Consideriamo un approccio single-linkage e descriviamo come è impostato un algoritmo che lo implementi. L’idea di base è cancellare in modo progressivo righe e colonne della matrice delle distanze dei dati coinvolti nell’operazione conglobando coppie di righe e di colonne in una sola riga o colonna.
Sia D = [d(i,j)] la matrice NxN delle distanze. Vogliamo ottenere n cluster 0,1,......, (n-1) dove L(k) è il livello gerarchico del singolo cluster. Il cluster m sarà indicato con (m) mentre la distanza tra due cluster (r) e (s) sara indicata da d [(r),(s)]. 
L’algoritmo procederà nel modo seguente:

    Inizia con il cluster di livello L(0) = 0 e di posizione nella sequenza dei cluster m = 0.
    Trova i due cluster più simili, siano essi (r) e (s) 
d[(r),(s)] = min d[(i),(j)]
dove l’operazione di minimo è estesa a tutte le possibili coppie di cluster.
    Incrementa l’indice della sequenza dei cluster: m = m +1. Combina i cluster (r) and (s) in un singolo cluster di livello
L(m) = d[(r),(s)]
    Aggiorna la matrice delle distanze D, cancellando le righe e le colonne corrispondenti ai cluster (r) e (s) e aggiungendo una nuova riga ed una nuova colonna corrispondenti al nuovo cluster ottenuto al passo precedente. La distanza tra il nuovo cluster e un vecchi cluster k è definita come:
      d[(k), (r,s)] = min d[(k),(r)], d[(k),(s)]
    Se esiste un solo cluster, la procedura si ferma. Altrimenti vai al passo 2. 
 

E’ importante notare che il processo è dipendente dalla funzione distanza scelta. Scelte diverse della funzione distanza producono risultati anche completamente diversi. 
Metriche negli algoritmi di clustering
Negli algoritmi d clustering è importante la scelta della metrica, cioè della grandezza che misura la differenza (o distanza) tra due punti nello spazio da classificare.
La scelta più immediata è quella di distanze di tipo geometrico, scelte cioè nella famiglia delle metriche di Minkowski:

      

che comprendono la classica distanza euclidea come caso particolare per p=2. Per p=1 abbiamo la cosiddetta distanza Manhattan. 
La tabella mostra le principali metriche utilizzate negli algoritmi di clustering come misura di distanza tra gli elementi. Come si vede la maggior parte delle distanze sono derivate dalla distanza di Minkowski. Un esempio di applicazione della correlazione come distanza può essere la segmentazione di immagini 2D+T con iniezione di mezzo di contrasto, nella quale ci interessa raggruppare tra loro i pixel che rispondono in modo simile all’all’arrivo del contrasto. 
Infine è opportuno notare come il funzionamento di un algoritmo di clustering multidimensionale dipenda dalla scala su cui sono misurati i valori dei dati su cui fare il clustering. Se i dati non sono omogenei, i dati con valori maggiori peseranno di più nel computo della distanza falsando il comportamento dell’algoritmo. Si pensi ad esempio ad un clustering in cui si utilizzano immagini su 8 bit ed immagini a 16 bit. In questi casi è opportuno normalizzare i dati su distribuzioni di tipo standard (tipicamente distribuzioni gaussiane con media zero e deviazione standard 1), un processo detto standardizzazione delle variabili. L’operazione di standardizzazione in Matlab è implementato dalle funzioni zscore o normalize (dalla versione R2018).  
 


 
Algoritmi di Labeling

Come detto in precedenza, la segmentazione a soglia non è in grado di distinguere oggetti topologicamente diversi ma ai quali è associato lo stesso livello di segnale. Nell’immagine del fantoccio ad esempio abbiamo pattern topologicamente distinti che vengono estratti come un oggetto unico. Questo è un limite intrinseco della segmentazione a soglia, che può essere superato elaborando i dati della segmentazione stessa. 
Il metodo più diretto è scomporre la segmentazione usando un algoritmo detto “label region”.  L’algoritmo lavora su una immagine binaria (quindi una maschera) e funziona nel seguente modo:

Per ogni pixel dell’immagine:
    Se è il primo pixel, crea un gruppo (blob) e aggiungi il pixel al blob
    Per tutti i pixel dell’intorno:
    Controlla se sono già stati assegnati a un blob. Se si, vai a 2
    Controlla se appartengono allo stesso blob del pixel di partenza. Se si, aggiungili al blob e vai a 2
Completato il primo blob, si prende un pixel non appartenente al blob e si ritorna a 1), creando un nuovo blob. Si itera fino a quando tutti i pixel sono stati assegnati. 
I blob ottenuti possono essere classificati per grandezza e i blob più piccoli eventualmente eliminati (come nel filtro mediano). Per pixel dell'intorno si intendono i pixel contenuti in un kernel centrato sul pixel stesso di dimensioni 3x3 e può includere o meno le diagonali.

In MATLAB la funzione da utilizzare è bwlabel o bwconncomp (Vedi esempio ES-MAT-5). Se consideriamo il fantoccio MR (sinistra) e applichiamo una segmentazione a soglia con ad esempio l’algoritmo kmeans, ricaveremo una maschera dell’acqua come al centro della figura. L’algoritmo di labeling fornirà una mappa in cui le regioni non appartenenti alla maschera dell’acqua saranno settate a 0, mentre la maschera verrà divisa in una serie di regioni topologicamente connesse identificate da un indice univoco. 
  

Region Growing 

Una estensione della segmentazione a soglia è l'algoritmo “region growing”, dove partendo da un pixel all’interno dell’oggetto da segmentare si estende la segmentazione a tutta la regione di interesse. In questo caso la soglia è quindi definita in modo locale come la differenza di segnale tra un pixel ed i pixel vicini. Questo approccio sfrutta le informazioni spaziali e garantisce la formazione di regioni tra loro collegate. Di fatto è una implementazione ricorsiva di una segmentazione effettuata per pixel adiacenti. 
La routine prevede la definizione di un punto di partenza all’interno del pattern da riconoscere. Il valore di livello di grigio così individuato costituisce il punto di partenza (detto ‘seed pixel’, ‘pixel seme’) per la successiva elaborazione: si vanno  ad analizzare ricorsivamente i pixel adiacenti a quello selezionato inizialmente. Quelli che hanno una differenza di livello di grigio appartenente ad un intervallo prefissato vengono selezionati, gli altri scartati. 

L’approccio Region Growing prevede quindi i seguenti passi:

    Si sceglie un arbitrario ‘seed pixel’ che viene confrontato con quelli adiacenti.
    Dal seme si passa ad una regione che ‘cresce’ aggiungendo progressivamente quei pixel adiacenti che sono ‘simili’ a quello iniziale secondo criteri di somiglianza fissati. 
    Quando la crescita della regione si ferma il processo termina 

Il seed pixel può essere scelto in modo manuale o con algoritmi automatici. 
La funzione “paint” definita in tutti i programmi di image processing utilizza questo metodo. Questo tipo di segmentazione consente di individuare in modo ottimale i confini degli oggetti individuati per osservazione. 
Il punto cruciale della tecnica “region growing” è il criterio di similarità dei pixel, che determina se un pixel debba o meno essere aggiunto alla regione. Nel caso di immagini binarie, il criterio di simiglianza si identifica con l'uguaglianza del valore dei pixel. In questo caso l’algoritmo si comporta come l’algoritmo di labeling.
Nel caso di immagini a livello di grigio, la condizione di inclusione diviene ABS(s0-s1) < T, dove T è un valore di soglia e ABS è il valore assoluto. Si include quindi un pixel se il suo valore differisce da quello di un pixel adiacente appartenente alla regione per meno di T. Per il buon funzionamento dell'algoritmo, T deve essere abbastanza grande da includere tutti i pixel nella regione omogenea che vogliamo segmentare e abbastanza piccolo da impedire che la segmentazione “debordi” nei tessuti vicini.  

   
In figura da sinistra a destra un esempio di segmentazione region growing con T ottimale, con T troppo basso (alcuni pixel vengono persi) e con T troppo alto.  

T può essere definito esplicitamente come numero di livelli di grigio (es T=50). E' anche possibile definire T come percentuale rispetto al valore dei pixel, la condizione di inclusione diviene ABS(s0-s1)/MEAN(s0,s1) < T . In questo caso T sarà compreso tra 0 e 1 e un pixel verrà incluso se la sua differenza rispetto al pixel vicino è inferiore al 100*T %. 
Un approccio ragionevole può essere quello di definire T come proporzionale al rumore sull'immagine. Infatti, se consideriamo una zona omogenea dell'immagine, come ad esempio l'acqua o l'olio nel fantoccio, il segnale nella regione omogenea sarà S0+N(0,σ) nel caso di rumore gaussiano a media nulla. Se ad esempio poniamo T=1.96σ, otterremo di includere il 95% dei pixel della regione. Questo approccio è simile a quello visto nel design dei filtri adattivi.
L'algoritmo di region growing è  implementato in MATLAB dalla funzione grayconnected.
Segmentazione a contorni

Un approccio diverso al problema della segmentazione consiste nell'identificare sull'immagine i contorni dei pattern, che corrispondono alle zone di transizione tra due tessuti diversi. 
Consideriamo l'immagine del fantoccio in figura ed il relativo profilo dei livelli di grigio estratto dall'immagine stessa.   
 
I contorni visti sul profilo saranno caratterizzati da una transizione netta del segnale. Se calcoliamo la derivata del profilo, valori alti della derivata corrisponderanno ai contorni, come si vede dalla figura. Per trovare i contorni, possiamo definite una soglia sul valore assoluto della derivata, valori della derivata superiori alla soglia definiscono una transizione e quindi un contorno. 

     
Estendendo il concetto in 2D o 3D, un mappa di gradiente dell'immagine consente di identificare i contorni. La mappa di gradiente di una immagine può essere ottenuta in vari modi, tipicamente attraverso un filtro convolutivo. Ad esempio il filtro di Sobel calcola la derivate del gradiente per righe e per colonne utilizzando i due kernel convolutivi:



        -1	0	1		1	2	 1  
Kx	=-	2	0	2	Ky=	0	0	 0
        -1	0	1		-1	-2	-1
 
e il gradiente viene poi calcolato come G=SQRT(Gx^2+Gy^2) 
Un’altra possibilità è usare il filtro Laplaciano:

0	-1	0
-1	4	-1
0	-1	0 

la dimensione del kernel può essere aumentata per ottimizzare il calcolo. Le operazioni di filtraggio convolutivo possono essere implementate attraverso la funzione conv2 di MATLAB, tuttavia è più opportuno utilizzare delle funzioni specifiche. 
In particolare la funzione edge dell'Image Processing Toolbox trova i contorni dell'immagine attraverso una serie di filtri, che possono essere scelti tra quello di Sobel, di Roberts, Lapalciano, etc. Sull'immagine di gradiente viene applicata una soglia per tagliare i valori bassi di gradiente e evidenziare i contorni.

Una volta ottenuta la mappa dei contorni (edge map) con questo tipo di algoritmi, è necessario costruire un contorno continuo che unisca le aree ad alto valore di gradiente. Uno degli approcci più semplici per collegare i punti di discontinuità in un’immagine di gradiente è quello di analizzare le caratteristiche dei pixel in un intorno ristretto (3x3 o 5x5) del punto in esame. In altre parole si cercano delle proprietà comuni ai punti dell’intorno e successivamente si collegano i punti simili con qualche criterio. La creazione di un contorno continuo da una mappa di gradiente utilizza tipicamente due parametri (algoritmo di Canny): 

    Intensità della mappa di gradiente. Un punto nell’intorno è simile al punto considerato se la differenza del valore di modulo del gradiente nel punto è minore di una soglia T.
            

    Direzione del gradiente.  Un punto nell’intorno è simile al punto considerato se la direzione del gradiente nel punto (data dal rapporto tra le componenti x e y del gradiente) è minor e di una soglia A
 
Il processo può essere completato eliminando piccoli tratti di segmenti isolati e colmando piccoli intervalli tra i segmenti.

Active Contours (Snakes)

I metodi prima visti hanno il limite fondamentale di non includere al loro interno un modello dell’oggetto da segmentare. Nell’imaging biomedico, ad esempio, possiamo senz’altro supporre che gli organi siano oggetti con una superficie che non presenta spigoli o punti ad elevata curvatura. Per molti organi abbiamo inoltre una informazione a priori sulla forma, ad esempio un vaso sarà simile ad un cilindro, un ventricolo visto in asse lungo ad un ellissoide, etc. 
Esistono vari algoritmi che introducono queste informazioni ottimizzando la segmentazione. L’esempio più noto sono gli algoritmi a contorni attivi o snakes. 
L’algoritmo tradizionale, introdotto per la prima volta da Kass, consiste nell’inizializzare una certa curva (snake) e nel deformarla poi in modo da farla convergere in corrispondenza di un minimo locale di energia, cioè di un contorno dell’immagine.
Il primo problema da affrontare è quindi quello della definizione del contorno iniziale: essendo un metodo di analisi locale infatti i risultati dipenderanno dal modo in cui è stato inizializzato lo snake. Pertanto in dipendenza dell’applicazione si dovrà decidere se inizializzare automaticamente la curva oppure usare una procedura semiautomatica in cui le condizioni iniziali sono gestite direttamente da un utente esperto. 
La curva iniziale C è definita da un insieme di N punti ordinati caratterizzati dalle loro coordinate  (xi,yi) per i=1,..,N. La curva è chiusa quindi il punto N è connesso con il punto 1.

   

Possiamo anche definire C come una curva parametrica in s dove s indica la posizione del punto sulla curva:

C(s) = {(x(s), y(s)) : 0 ≤ s ≤ 1}

Ad uno snake può essere associata una energia, vista come somma di una componente interna e di una esterna o associata all’immagine: 

E(C)=∫_0^1▒〖E_int+E_imm 〗

Facendo evolvere lo snake in modo che l’energia dello snake venga minimizzata, si ottiene una curva che evolvendosi nel tempo va ad operare una segmentazione sulle immagini. Le forze interne tendono a conservare la forma dello snake e quindi determinano il modo in cui lo snake si evolve, quelle esterne sono correlate all’immagine e determinano le caratteristiche che vengono rilevate. 
L’energia interna può essere divisa in due componenti:

E_int (C(s))=α|C'(s)|^2+β|C''(s)|^2

La prima componente è legata alla derivata prima e rappresenta la tendenza dello snake a mantenere la sua forma opponendosi alla trazione. L’energia elastica è legata alla distanza dei punti dello snake, aumentando la distanza tra due punti aumenta anche l’energia elastica.  La seconda componente (rigidità) è legata alla derivata seconda e rappresenta la tendenza dello snake ad opporsi alle modifiche della sua curvatura. Impedisce allo snake di “ingarbugliarsi”.
Manipolando la tensione o elasticità e la rigidità dello snake è possibile modificare l’importanza relativa delle due componenti dell’energia interna. Intuitivamente si può pensare allo snake come ad un elastico, che se lasciato libero riassume la sua forma originale grazie alla sua elasticità.  
Il secondo contributo viene di solito chiamato energia dell’immagine o energia esterna in quanto viene ricavata dall’immagine stessa in modo da assumere valore minimo nelle zone di maggiore interesse come ad esempio i bordi. Tipiche e semplici funzioni di energia dell’immagine sono legate all'edge map dell’immagine:

 

dove  è una maschera gaussiana bidimensionale con una certa deviazione standard  (filtro gaussiano) aumentando la quale si rendono i contorni più sfocati, ma si aumenta il range di cattura di tali contorni. Introducendo dei pesi per le forze prima definite abbiamo per l’energia dello snake:  

E_Snake=∝∫▒〖|C'(s)|^2 ds+〗  β∫▒〖|C''(s)|^2 ds〗+kE_imm (s) 

Dove α è il peso dell'energia elastica, β il peso dell'energia di curvatura, k il peso delle forze esterne relative all’immagine.
L’idea è di lasciare evolvere lo snake sull’immagine fino quando non raggiunge un minimo dell’energia. Questo stato di minimo corrisponderà ad una situazione stabile in cui lo snake corrisponderà ai contorni definiti sull’immagine stessa a meno dei limiti di rigidità ed elasticità imposti. L’algoritmo per minimizzare l’energia è un processo iterativo che parte da un contorno iniziale e converge (se la differenza della posizione dello snake tra due iterazioni è abbastanza piccola) ad un minimo locale. Il processo iterativo è efficiente dal punto di vista computazionale (ogni passo è rappresentato dalla moltiplicazione di matrici numeriche) ed il suo risultato dipende dai valori ,  e . Da un punto di vista pratico l’utilizzo del metodo AC (active contours) richiede il tuning accurato dei parametri ,  e  e la possibilità di definire un contorno iniziale abbastanza vicino al risultato desiderato in modo da ottimizzare la convergenza dell’operatore locale.
Come si osserva dall’equazione dello snake, le forze esterne sono computate solo sullo snake stesso. Questo implica che lo snake nella sua evoluzione “vede” i contorni dell’immagine solo quando gli attraversa. Una modifica dell’algoritmo implica l’uso di informazioni interne ed esterne allo snake (region-based models) in modo da favorire la convergenza. L’implementazione più nota è l’algoritmo di Chan-Vese che troviamo implementato in MATLAB. 
Consideriamo una immagine con due pattern con livello di grigio I1 e I2, a meno del rumore e di altri fattori quali il PVE e l’attenuazione. Nell’approccio di Chan-Vese l’energia da minimizzare è valutata come:

F(C)=F_1 (C)+F_2 (C)=∫_(inside(C))▒〖|I-C1|^2+∫_(outside(C))▒|I-C2|^2 〗 

Dove I è l’immagine, C1 è la media dei pixel dell’immagine dentro il contorno C, C2 è la media dell’immagine fuori dal contorno C. Chiaramente F(C) si minimizza quando C corrisponde al bordo tra i due pattern e divide l’immagine in due regioni omogenee in cui F1 e F2 sono uguali alla potenza del rumore associato all’immagine. L’approccio realmente implementato è più complesso in quando si estende ad una immagine ad N pattern omogenei.
In MATLAB la funzione activecontour implementa le versioni proposte da Chan (Chan-Vese) e Caselles (edge).  La funzione dei parametri alfa e beta dello snake è svolta dai parametri ContractionBias (determina se il contorno si espande o si restringe, alfa) e SmoothFactor (determina la regolarità del contorno, beta).
Gli algoritmi a contorni attivi funzionano su immagini bidimensionali. In teoria è possibile estendere l’algoritmo ad immagini 3D, in questo caso il contorno diviene una superficie chiusa (balloon) definita da una mesh geometrica come negli algoritmi di visualizzazione 3D. Nella pratica nell’elaborazione di immagini 3D si utilizza un approccio diverso detto level set.   
 
Level set
L’algoritmo level set (o implicit Active Contour) definisce il contorno C come l’intersezione di una funzione (x,y) con il piano dell’immagine che corrisponde a (x,y)=0.   

 
Hoang Ngan Le T. et al. (2020) Active Contour Model in Deep Learning Era: A Revise and Review. In: Oliva D., Hinojosa S. (eds) Applications of Hybrid Metaheuristic Algorithms for Image Processing. Studies in Computational Intelligence, vol 890. Springer, Cham. https://doi.org/10.1007/978-3-030-40977-7_11

La figura mostra come il processo iterativo che modifica  nel tempo causi l’evoluzione del contorno C sul piano. Da un punto di vista formale si definisce:

C = {(x, y): φ(x, y) = 0},

l’evoluzione della curva è governata dall’equazione:
∂φ/∂t+F⌈∇φ⌉=0

Dove F è la “speed function” che caratterizza l’algoritmo. F deve essere zero sui contorni dell’immagine in modo da assicurare la convergenza. Ad esempio potremmo definire:

F(x,y)=1/(1+λ⌈∇I(x,y)⌉ )

Quindi per un valore del gradiente dell’immagine alto (transizione tra tessuti) F tenderà a 0, per un valore del gradiente molto basso (regione omogenea) F tenderà a 1 ed il contorno avanzerà velocemente. Il parametro  modula il peso del gradiente sul valore di F.  

Partendo da un valore iniziale di , e quindi di C, viene applicato un algoritmo iterativo che simula nel discreto l’evoluzione temporale ed il contorno si modifica seguendo l’evoluzione di  fino alla convergenza. Una potenzialità interessante dell’algoritmo level set è che il contorno definito da  si può dividere in due o più contorni durante la progressione dell’algoritmo (e viceversa) permettendo una maggiore flessibilità rispetto all’algoritmo a contorni attivi.

 
Da Wikipedia (Eng)



 
Watershed transform

Una importante famiglia di tecniche per la segmentazione di immagini si basa su concetto della ricerca delle componenti connesse (Connected component, CC). Il concetto alla base di queste tecniche è la cosiddetta watershed transform, (spartiacque) nella quale il valore di livello di grigio dell’immagine rappresenta la profondità di un pixel rispetto ad un valore di riferimento. Una immagine viene quindi vista come una struttura (ad esempio un lago) che viene via via riempita con acqua in una certa posizione. Man mano che si versa l’acqua, il pelo dell’acqua stessa si alza disegnando delle strutture (maschere) che individuano le regioni sotto una certa altezza. Quando si arriva ad un watershed, l’acqua trabocca in una regione vicina. L’algoritmo, concettualmente simile al region growing, individua le configurazioni “borderline” che precedono il travaso dell’acqua da una regione dove la superficie dell’acqua è “stabile” ad un’altra.    

  

Consideriamo il fantoccio MR in figura e il relativo profilo. L’asse Y del profilo esprime in questo caso la distanza dal fondo che ha riferimento 0. Immettiamo l’acqua da un seed posto sul fondo. L’acqua riempirà i pixel connessi al seed e più “bassi” (quindi con livello di grigio minore) del seed.  Questa funzione è implementata in Matlab da grayconnected. Man mano che sale l’acqua i pixel del fondo vengono sommersi e la maschera risultante cresce.

       
         Th=1		       Th=20	                     Th=30                              Th=70

A Th=1 viene coperto lo zero-padding, a Th=20 una parte del fondo. Ad un certo punto la maschera si stabilizza fino a quando il livello non supera lo spartiacque della parete del fantoccio, dopodiché ricomincia a crescere lentamente fino a stabilizzarsi di nuovo fino al riempimento dell’olio. 
    
Se tracciamo il grafico dell’area della maschera in funzione del livello otteniamo la situazione in figura, dove come si è detto periodi di area stabile indicano che sono state riconosciute le strutture principali del fantoccio. L’algoritmo watershed attraverso soglie opportune riconosce le transizioni tra i periodi di riempimento e produce le relative maschere. Un metodo semplice è calcolare il gradiente della curva (in rosso in figura) e scegliere le soglie tra due picchi del gradiente.  
 
Mappe per th=200, th=500, th=650, th=1900.
 
Mappe ottenute per sottrazione progressiva delle maschere. Notiamo che pur utilizzando un approccio a soglia, l’algoritmo riesce a distinguere regioni topologicamente non connesse.

Algoritmo MSERs

L’algoritmo prima descritto può essere generalizzato ottenendo l’algoritmo Maximally Stable Extremal Regions (MSERs). L’algoritmo MSERs ricerca le regioni estreme massimamente stabili all’interno dell’immagine. Le regioni estreme sono definite come le regioni connesse i cui livelli di grigio sono tutti al di sopra o al di sotto dei valori dei pixel che circondano la regione. Le connessioni tra un pixel ed i pixel circostanti come al solito possono essere definiti dai 4 (6) pixel vicini o dagli 8 (24) pixel vicini in 2D o 3D, rispettivamente. Come visto in precedenza, tra le regioni estreme quelle massimamente stabili sono quelle dove, definita una soglia Δ, la variazione del numero di pixel della regione variando il livello di grigio g da g-Δ a g+Δ è un minimo locale. Il valore di Δ determina il numero di regioni che vengono trovate, che è inversamente proporzionale a Δ.  
L’algoritmo MSERs nella sua forma originaria prevede i seguenti passi:

    I pixel dell’immagine vengono ordinati secondo il valore del livello di grigio

    Partendo dal valore più basso di livello di grigio, i pixel vengono inseriti nell’immagine (si considera di partire da una immagine vuota) e si individuano le componenti connesse con un opportuno algoritmo di labeling.

    Viene memorizzata per ogni livello di grigio la lista delle componenti connesse e la loro area. Se due componenti si uniscono, la più piccola viene inglobata nella più grande.

Si ottiene quindi una lista che per ogni livello di grigio contiene l’area delle componenti connesse.
Per ogni componente, si considera la curva area vs livello di grigio ed in corrispondenza dei minimi locali della curva si individuano le componenti massimamente stabili.
  
Dal punto di vista computazionale, il punto critico è il terzo. Bisogna per ogni passo ordinare le componenti per grandezza e riconoscere quali provengono da componenti esistenti al passo precedente. Per ottimizzare la procedura si memorizzano i dati con un approccio Component Tree, nel quale le componenti vengono memorizzate in un albero che consente di realizzare l’algoritmo in modo efficiente dal punto di vista computazionale. 
L’algoritmo è implementato dalla funzione Matlab detectMSERFeatures. In figura è illustrato il risultato dell’applicazione dell’algoritmo al fantoccio prima considerato. 

 

Component Tree

L’approccio Component Tree come detto in precedenza permette di memorizzate il contenuto di una immagine in un albero, permettendo di realizzare algoritmi di elaborazione dell’immagine in modo efficiente. Si tratta di un approccio molto usato e vale quindi la pena di esaminarlo in maggior dettaglio.
Consideriamo una immagine  a quattro livelli di grigio come quella in figura. L’immagine contiene 8 o 9 pattern a seconda del tipo di connessione considerata, per un kernel di connessione a 8 elementi contiamo 8 pattern distinti. Al passo uno dell’algoritmo viene selezionata la componente a minima intensità (1) c1, che va a costituire la radice dell’albero e rappresenta il background. Al passo due, alla componente uno vengono aggiunte due componenti topologicamente distinte c2 e c3, con livello di grigio 2, che diventano due rami dell’albero. Al passo tre vengono aggiunte tre componenti (c4, c5, c6), con livello di grigio 3, delle quali c6 è contigua a c3 e quindi va nel ramo corrispondente dell’albero, mentre le altre due nell’altro essendo contigue a c2. Al passo quattro l’albero si completa con le componenti c7 e c8 relative al livello gi grigio 4. In generale i livelli dell’albero saranno pari al numero di livelli di grigio presenti nell’immagine ed il numero di nodi sarà uguale al numero di pattern presenti nell’immagine.

  
 
In ogni nodo sono memorizzati i pixel appartenenti al nodo. Il peso delle connessioni (edge) tra nodi è dato dalla differenza di livello di grigio tra i livelli (in questo caso uno per tutte le connessioni). 
 
                             
Una volta effettuato il processo di conversione da immagine ad albero connesso, le operazioni di filtraggio e segmentazione possono essere effettuate sull’albero invece che sull’immagine. Ad esempio se selezioniamo i due rami principali dell’albero attraverso una funzione di ricerca depth-first search, otteniamo i due “macro-pattern” in cui è divisa l’immagine, come illustrato in figura.
Una segmentazione a soglia si ottiene immediatamente selezionando tutti i nodi sopra un certo livello. Il region-growing è equivalente a partire da un nodo e muoversi sull’albero con la condizione che il peso di una connessione deve essere minore uguale alla soglia di tolleranza dell’algoritmo. In generale molti algoritmi di filtraggio e segmentazione possono essere implementati utilizzando in modo efficace la rappresentazione ad albero connesso.
 
Skeletonization 
Adattato da HIPR (http://homepages.inf.ed.ac.uk/rbf/HIPR2/hipr_top.htm)
Nell’elaborazione di immagini biomediche, una volta definita la maschera che definisce una certa struttura anatomica, può essere utile caratterizzare la struttura da un punto di vista geometrico, estraendone i caratteri fondamentali, quali la connettività, la topologia, la lunghezza, la direzione e l'ampiezza. Questo può essere ottenuto attraverso l’estrazione del cosiddetto scheletro topologico (skeleton). Intuitivamente, lo scheletro topologico di una maschera si ottiene “assottigliando” la maschera stessa fino ad ottenerne una versione essenziale, tipicamente composta da linee. 

In figura è mostrato lo scheletro topologico dell’albero bronchiale estratto da una maschera dell’albero stesso ottenuta da immagini TAC.
In questo caso lo scheletro estratto può essere utile in varie applicazioni. Ad esempio lo scheletro, rappresentando la linea di equidistanza dalle pareti del bronco (center line) può essere usata come guida per applicazioni di endoscopia virtuale. Dallo scheletro è possibile individuare facilmente le biforcazioni dell’albero bronchiale, permettendo la costruzione di un albero binario che permette di confrontare il paziente in esami eseguiti a tempi diversi o pazienti diversi tra loro. 
L’operazione di skelethonization può essere eseguita in vari modi. I due approcci fondamentali sono il thinning (assottigliamento) morfologico e il calcolo della distance transform dell’immagine. 
L’operazione di thinning si basa sull’idea di eliminare i pixel di confine della maschera in modo progressivo (erosione) riducendone le dimensioni fino a quando non è più possibile assottigliare ulteriormente la maschera e si ottiene lo scheletro topologico.
Per formulare in termini algoritmici la funzione di thinning è utile introdurre la hit-and-miss transform, (HAM) che, come avviene nei filtri spaziali, è basata sulla convoluzione della maschera con un kernel (structuring element). Il kernel contiene la descrizione della struttura geometrica che si vuole individuare. La struttura del kernel (supponendo di voler individuare un angolo della maschera) sarà del tipo in figura, dove 0 indica un pixel appartenente allo sfondo della maschera, 1 un pixel appartenente alla maschera e le caselle bianche indicano locazioni non di interesse. In altri termini le caselle vuote vengono ignorate ed il loro scopo è ottenere un kernel quadrato più comodo da usare a fini computazionali. 
Analogamente alla convoluzione spaziale, il kernel viene fatto scorrere sulla maschera e se esiste una totale corrispondenza tra il kernel e la maschera sottostante il pixel della maschera risultante viene posto ad uno, altrimenti viene lasciato a zero.
Tipicamente vengono generate una serie di versioni del kernel che corrispondono alle possibili orientazioni della struttura che interessa determinare, ad esempio nel caso in oggetto potremmo avere i quattro kernel:
  

Che corrispondono alla ricerca di un angolo nella maschera nelle quattro orientazioni possibili. L’ OR logico delle quattro maschere risultanti permette di identificare gli angoli sulla maschera. 


Un esempio di uso della hit-and-miss transform è il rilevamento dei punti isolati di una maschera, che può essere utile nella correzione degli errori di segmentazione. In questo caso il kernel da utilizzare risulta:
[■(0&0&0@0&1&0@0&0&0)]
 
E si ottiene una maschera che definisce i punti isolati della maschera di input.
La trasformazione HAM è implementata in MATLAB dalla funzione bwhitmiss. 
Nel caso del thinning si può usare una coppia di kernel del tipo:

[■(0&0&0@&1&@1&1&1)]      [■(&0&0@1&1&0@&1&)]   

L’applicazione di questi due kernel equivale a conservare i pixel che sono al centro di un ottagono che è totalmente incluso nella maschera. I kernel vengono ruotati nelle quattro direzioni possibili ottenendo quindi 8 trasformazioni hit-and-miss. L’operazione di thinning si ottiene calcolando:

Mthin = M – MHAM

In pratica il risultato della trasformazione HAM viene sottratto alla maschera originale, che quindi si ”assottiglia”. Per ottenere lo skeleton l’operazione di thinning viene iterata fino a quando la maschera originale non si modifica ulteriormente.  
Se consideriamo la maschera originale a sinistra l’applicazione dell’operazione di thinning porta alla maschera al centro, l’effetto del thinning è visibile nell’immagine a destra che rappresenta la differenza tra le due maschere. 
 
Come si osserva la parte più esterna della maschera originale viene asportata. Applicando 10 volte il thinning si ottiene:

 
Ed infine per N grande (ad esempio N=100) la procedura iterativa conduce allo skeleton:
 

In MATLAB l’operazione prima descritta di skelethonization è implementata direttamente nella funzione bwmorph attraverso l’opzione 'skel'.

 

Il calcolo della distance transform dell’immagine prevede invece che per ogni punto della maschera di input venga calcolata la distanza minima dal contorno della maschera stessa (o dallo sfondo della maschera). Di solito si utilizza la distanza City-block o `chessboard' che vale il numero di pixel che bisogna attraversare per andare da un pixel all’altro dell’immagine (nella City-block non si possono usare le diagonali). Si ottiene quindi una immagine di dimensioni uguali all’originale e a cui a ciascun pixel corrisponde un livello di grigio pari alla minima distanza dai bordi. I pixel con valore più alto rappresentano lo skeleton in quando sono i più “centrali”. Utilizzando una soglia si può estrarre lo skeleton vero e proprio.   
In MATLAB la distance transform viene computata con la funzione bwdist che permette di definire il tipo di distanza da utilizzare. La funziona computa la distanza da un pixel non nullo, quindi per ottenere lo skeleton va invertita la maschera.

 




