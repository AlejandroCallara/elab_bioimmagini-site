# Esercitazione 5.1: Registrazione di Immagini

## Introduzione

Nello sviluppo di algoritmi di elaborazione delle bioimmagini è comune l’uso di set di dati generati per via informatica, i cosiddetti artificial data o synthetic data.  Si tratta di immagini generate attraverso un software in modo da essere il più possibile simili ad immagini reali. I vantaggi dei synthetic data sono numerosi:

1.	Non sono necessari le autorizzazioni ed i consensi informati obbligatori per l’uso delle immagini reali sia da volontari che da pazienti.
2.	Può essere generato un numero di immagini grande a piacere.  Questo punto è particolarmente importante nello sviluppo di algoritmi data-driven dove sono necessarie grandi quantità di dati.
3.	Possono essere variati facilmente i parametri di acquisizione, come l’SNR e il CNR.
4.	Il reference per la validazione dell’algoritmo è perfettamente noto, e quindi abbiamo un vero "ground truth"

L’ultimo punto in particolare libera il processo di validazione dall’incertezza sul "reference" propria della validazione fatta da utenti esperti che è sempre affetta dalla variabilità inter- ed intra-osservatore. 

Un modo per produrre dati sintetici è quello di utilizzare il modello teorico dell’immagine biomedica. In questo caso si definirà un’immagine ideale che sarà tipicamente estratta da un atlante anatomico, come il 4D Extended Cardiac-Torso (XCAT) Phantom o l’atlante di Talairach. Partendo dall’immagine ideale che costituirà il “gold standard”, si aggiungono i fattori di corruzione dell’immagine ideale:

•	Rumore biologico sui tessuti non isotropi\
•	Effetto volume parziale (filtraggio gaussiano)\
•	Campo di attenuazione\
•	Rumore

Si ottiene così un'immagine realistica.
Metodi più perfezionati comportano la simulazione numerica del processo di acquisizione. In questo caso, partendo dall’immagine ideale, viene simulata la fisica del processo di acquisizione in dipendenza dal metodo di acquisizione utilizzato. Consideriamo ad esempio il simulatore MR cerebrale brainweb (https://brainweb.bic.mni.mcgill.ca/) (Figura 5.e1).


<img src="./images/Figura_ese_5_1.png" alt="Immagini MR Simulate (Brainweb)" style="width:100%;">

*Figura 5.e1. Immagini MR Simulate (Brainweb).*

Il simulatore parte da un atlante anatomico (a) e permette di definire il tipo di sequenza (T1, T2, PD), il thickness di fetta, il rumore e la disomogeneità del campo RF. Il simulatore è in grado di risolvere per via numerica le equazioni di Bloch e quindi simulare il processo di generazione delle immagini MRI.  
Esistono altri simulatori MR free, come JEMRIS (http://www.jemris.org/) che fornisce anche un'interfaccia MATLAB, coreMRI (https://www.coremri.org/) ed altri.

Per l’imaging tomografico CT e SPECT/PET è possibile una simulazione realistica del processo di imaging con metodi Monte Carlo (MC), che offrono un compromesso tra accuratezza/flessibilità della simulazione e tempo di calcolo. Il tool più usato è il GATE (http://www.opengatecollaboration.org).
Attualmente GATE supporta la simulazione di esperimenti di Tomografia a Emissione di Positroni (PET) e Tomografia Computerizzata a Emissione di Fotoni Singoli (SPECT), Tomografia Computerizzata (TC), Imaging Ottico (Bioluminescenza e Fluorescenza) e Radioterapia.
Il GATE è uno strumento potente e flessibile, in quanto consente di modellare diversi aspetti del process odi acquisizione, ad esempio per la medicina nucleare:

●	Geometria d’acquisizione e.g. Scanner, Phantom\
●	Sorgente radioattiva, e.g. Tipologia di radioisotopo, distribuzione, attività\
●	Meccanismi di interazione dei Raggi Gamma: Effetto fotoelettrico, Scattering Compton e Rayleigh, conversione dei raggi Gamma\
●	Soglia di tracking dei raggi gamma \
●	Processi di detezione: Singole detezioni, coincidenze, scattering, eventi random \
●	Fenomeni tempo-dipendenti e.g. Movimenti di sorgente or detettore, cinetica del decadimento radioattivo.

Con questi strumenti è possibile effettuare simulazioni molto dettagliate. Ad esempio, il 4D-MCAT phantom permette di simulare il battito cardiaco e la respirazione deformando in modo realistico l’anatomia del fantoccio software. Se l’anatomia ottenuta viene inserita in un simulatore di tipo GATE è possibile simulare l’acquisizione di una SPECT/PET gated, valutare l’effetto della respirazione sulla qualità delle immagini, etc (Figura 5.e2).   

<img src="./images/Figura_ese_5_2.png" alt="Immagini PET/CT Simulate (GATE)" style="width:100%;">

*Figura 5.e2. Immagini PET/CT Simulate (GATE).*

CTsim implementa la simulazione di immagini CT (http://www.ctsim.org/)

Sono disponibili anche immagini US simulate: https://team.inria.fr/epione/en/data/straus/

La disponibilità di synthetic data permette di fare una prima valutazione del funzionamento di un algoritmo di elaborazione variando le caratteristiche delle immagini in modo efficiente ed economico. 

## Esercitazione

Lo scopo dell’esercitazione è sviluppare una procedura di registrazione basata sugli algoritmi genetici che esegua la registrazione 2D di immagini MR. Le immagini da registrare sono quelle di un cosiddetto “synthetic phantom”, cioè un phantom generato via software. In particolare utilizziamo immagini 3D dal sito brainweb prima descritto:

•	t1_icbm_normal_1mm_pn3_rf0.mnc  (T1 pesata) \
•	pd_icbm_normal_1mm_pn3_rf0.mnc  (PD) 

Le immagini sono in formato MINC (McConnell Brain Imaging Center (McBIC) at the Montreal Neurological Institute (MNI)), un formato proprietario del centro che ha realizzato il simulatore brainweb. Possono essere importate in python attraverso la libreria **nibabel**, che fornisce un’interfaccia per l’import di numerosi formati di immagine con la funzione load (https://nipy.org/nibabel/).

Per ridurre i tempi di calcolo consideriamo una singola slice (Figura 5.e3). Quindi andremo ad estrarre dai due volumi una slice centrale, ottenendo due immagini 2D T1 e PD.

<img src="./images/Figura_ese_5_3.png" alt="Immagini 2D T1 e PD simulate (slice #62)" style="width:100%;">

*Figura 5.e3. Immagini 2D T1 e PD simulate (slice #62).*

Come si osserva, BrainWeb genera immagini con un FOV corrispondente all’area cerebrale. Visto che dobbiamo simulare un disallineamento, è opportuno effettuare uno zero padding che consenta di spostare le immagini senza perderne una parte e renda la matrice quadrata (ad esempio con `numpy.pad`) (Figura 5.e4).

<img src="./images/Figura_ese_5_4.png" alt="Immagini 2D T1 e PD simulate dopo lo zero-padding" style="width:100%;">

*Figura 5.e4: Immagini 2D T1 e PD simulate dopo lo zero-padding.*

Partendo da questa coppia di immagini, vogliamo realizzare un simulatore che simuli un disallineamento rigido noto (rototraslazione in 2D), esegua il riallineamento delle due immagini e misuri la differenza tra i parametri simulati e quelli trovati dall’algoritmo di registrazione. La flow chart dell’algoritmo risulta la seguente (Figura 5.e5):

<img src="./images/Figura_ese_5_5.png" alt="Flow-chart del simulatore da realizzare" style="width:100%;">

*Figura 5.e5. Flow-chart del simulatore da realizzare.*

All’inizio abbiamo le due immagini che saranno caratterizzate da una certa somiglianza, ad esempio il valore della mutua informazione MI start. Un'immagine (la PD nell’esempio) funge da immagine floating e viene sottoposta ad una rototraslazione casuale di cui vengono memorizzati i tre parametri Psim. L’immagine floating viene registrata con l’immagine T1 da un algoritmo di registrazione che produce l’immagine floating registrata e i parametri di correzione del disallineamento Preg. Inoltre potremo calcolare la MI tra le immagini registrate (MI end). 
Se la registrazione è corretta, dovremmo trovare Psim = - Preg ed MIstart = MIend.
La bontà dell’algoritmo di registrazione può quindi essere valutata dalle differenze (MI start – MI end) e (Psim-Preg).

Iterando il procedimento, ad ogni iterazione avremo un disallineamento diverso e quindi potremo verificare l’algoritmo in situazioni diverse facendo un’analisi di tipo statistico.

Questo tipo di approccio simula una procedura di test di un software (il blocco REGISTRAZIONE) che viene testato in modo automatico attraverso prove ripetute che simulano un caso reale di disallineamento dell’immagine.  

La procedura di registrazione al solito è caratterizzata da tre parametri:

•	Search space: la tripletta di valori che definisce la roto-traslazione (3 parametri). La massima traslazione indotta da disImage2D è +/- dim/10, dove dim è la dimensione dell’immagine, mentre il massimo range di rotazione è [-20°,+20°].\
•	Metrica: essendo variabile la distribuzione dei livelli di grigio, sarà opportuno usare la mutua informazione. Avendo due immagini e prendendo la prima immagine I1 come riferimento (fixed) la metrica per l’allineamento sarà MI(I1,I2_rt) dove I2_rt è l’immagine floating rototraslata. La rototraslazione può essere implementata usando come modello la funzione disImage2D. La MI può essere calcolata con una delle funzioni disponibili su varie librerie come scikit-learn  o implementata direttamente. Si ricorda che la MI deve essere calcolata sull’area comune delle due immagini. \   
•	Ottimizzatore: Algoritmo che massimizza la MI

L’algoritmo dovrà stimare correttamente i parametri di trasformazione P=[tx1 ty1 a1] che allineano correttamente l’immagine I2 su I1 e quindi massimizzano la MI tra I1 e I2 rototraslata usando i parametri P.

Come ottimizzatore vogliamo utilizzare un algoritmo genetico che massimizzi il valore di MI in dipendenza dai parametri di registrazione (tre parametri). In python è disponibile la libreria PyGad (https://pygad.readthedocs.io/). 
In pygad è necessario definire una fitness function, che prende in input tre parametri (gaclass,solution, solution_idx). gaclass è un’istanza della classe pygad.GA cioè la configurazione dell’ottimizzatore GA che stiamo usando. solution sono i parametri del problema (nel nostro caso i parametri di rototraslazione).   solution_idx è l’identificatore dell’individuo per cui l’algoritmo genetico sta calcolando la fitness. Si veda l’esempio test_pygad.py per un esempio di configurazione dell’ottimizzatore. 
E’ possibile configurare l’ottimizzatore con vari parametri per migliorare la convergenza (https://pygad.readthedocs.io/en/latest/pygad.html). Alcuni parametri di interesse sono:

•	init_range_low, init_range_high: definiscono indirettamente il search space. La popolazione iniziale viene generata nel range indicato. E possibile comunque che durante l’evoluzione vengano creati individui fuori dal range.\
•	num_generations: il numero di generazioni dopo il quale il processo di ottimizzazione si ferma.\
•	num_genes: il numero di parametri da trovare (nel nostro caso tre)\
•	sol_per_pop: numerosità della popolazione\
•	parent_selection_type: il tipo di selezione utiizzata. Ad esempio ‘rws’ indica la roulette wheel.\
•	keep_elitism: numero di individui che fanno parte dell’elite.\
•	crossover_type: tipo di crossover\
•	mutation_probability: probabilità di mutazione

Sarà quindi necessario definire i parametri ottimali della metrica e dell’ottimizzatore per ottenere la migliore registrazione possibile. 

Una volta messo a punto l’algoritmo, per valutarne le prestazioni possiamo quindi eseguirlo per un numero N di volte (ad esempio N=20) ed annotare i parametri imposti e quelli calcolati per le tre incognite rappresentate dai parametri di rototraslazione. I dati raccolti possono essere analizzati tracciando i relativi tre Bland-Altman plot. In python il BA plot è implementato nella libreria statsmodels ma ne esistono molte altre implementazioni disponibili in rete. 
Per quanto riguarda la MI il valore MIstart sarà sempre lo stesso, per cui potremo valutare media ed SD della differenza tra MIend e MIstart, che teoricamente dovrebbe essere nulla.

Un esempio di risultato è riportato in Figura 5.e6. Ovviamente ogni run dell’algoritmo darà un risultato diverso, essendo il disallineamento random.

<img src="./images/Figura_ese_5_6.png" alt="Esempio di analisi BA de irisulatati della simulazione" style="width:100%;">

*Figura 5.e6. Esempio di analisi BA dei risulatati della simulazione.*


