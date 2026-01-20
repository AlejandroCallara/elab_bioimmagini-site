# Esercitazione 2 - stima dell'effetto di diversi tipi di filtraggio su alcune misure di qualità dell'immagine

Scopo dell’esercitazione è realizzare un programma che consenta di valutare vari algoritmi di filtraggio 3D su di un fantoccio MR in termini di SNR e conservazione delle transizioni.
 
Il fantoccio che utilizziamo è un fantoccio MR costituito da tre cilindri concentrici disassati. L’intercapedine tra il secondo ed il terzo cilindro è riempita con un liquido ad alto contrasto, le altre due con acqua. E’ stata acquisita una serie di fette assiali rispetto al fantoccio contenute nella directory volume3D. 

Poiché vogliamo applicare un filtraggio 3D, il primo passo è quello di verificare se il volume di dati è isotropo leggendo gli opportuni campi DICOM e nel caso non lo sia operare una interpolazione per ottenere un volume isotropo. La procedura di interpolazione dovrà conservare il FOV del volume originale. In MATLAB è possibile utilizzare le funzioni `interp3` in combinazione con `meshgrid`,
oppure la funzione `imresize3` con le opportune opzioni di interpolazione.
Il volume ottenuto può essere visualizzato tramite `volumeViewer`
(disponibile a partire da MATLAB R2017a). In Python, l’interpolazione tridimensionale può essere effettuata utilizzando le funzioni
`RegularGridInterpolator` o `interpn` della libreria **SciPy**, oppure, nel caso di volumi
isotropi con fattore di scala uniforme, tramite funzioni di ridimensionamento come
`scipy.ndimage.zoom` o `skimage.transform.resize`.

Il volume risultante può essere visualizzato mediante viewer tridimensionali dedicati,
come **napari**, che consente l’esplorazione interattiva di dati volumetrici, oppure
tramite librerie di visualizzazione 3D come **PyVista** o **VTK**.


Una volta ottenuto il volume interpolato, vogliamo sperimentare vari tipi di filtraggi in 3D e valutare l’SNR e la conservazione delle transizioni sul cilindro a massimo segnale. Per semplicità calcoleremo l’SNR e l’acutezza su una singola fetta in 2D (quella centrale) attraverso la definizione di una opportuna ROI sul cilindro centrale (acqua) e di un profilo verticale posto al centro dell’immagine.

Gli algoritmi di filtraggio da sperimentare includono: 
    Un filtro a media mobile con kernel 7x7x7 
    Un filtro gaussiano con kernel 7x7x7 e valore di sigma ottimizzato per massimizzare l’SNR e conservare le transizioni. 
    Un filtro adattivo di Wiener con kernel 7x7x7 

I primi due filtri sono convolutivi e possono essere implementati con le funzioni fspecial3 (MATLAB>=R2018b) e imfilter, o con la funzione conv3 definendo gli opportuni kernel. Per il filtro Gaussiano è anche disponibile la funzione imgaussfilt3 (MATLAB>=R2018b) che opera direttamente il filtraggio senza bisogno di utilizzare fspecial. 
Il filtro di Wiener bidimensionale è implementato dalla funzione wiener2, purtroppo non esiste la versione 3D che va quindi implementata, tenendo conto della formulazione del filtro di Wiener:
I_W=I_MM+|I_VAR-σ^2 |/I_VAR (I_OR-I_MM)
Nella formula riconosciamo l’immagine 3D originale I_OR, l’immagine filtrata a media mobile I_MM che avremo calcolato al passo precedente, σ che è la deviazione standard del rumore associato all’immagine. I_VAR è la mappa della varianza sull’immagine.
In teoria, essendo I_VAR≥σ^2, il fattore di modulazione del filtro di Wiener |I_VAR-σ^2 |/I_VAR  dovrebbe variarare tra 0 e 1. In pratica tale valore diverge per valori di I_VAR nulli (regioni di zero padding) o per valori di I_VAR molto piccoli. Tali regioni vanno quindi escluse dal filtraggio se presenti oppure bisogna porre |I_VAR-σ^2 |/I_VAR =1 dove assume valori non definiti (isnan, isinf) o maggiori di uno. Inoltre, sul fondo dell’immagine MR I_VAR sarà minore di σ^2 per le proprietà del rumore riciano, per cui il fattore di modulazione del filtro |I_VAR-σ^2 |/I_VAR  non si annullerà riducendo il filtraggio, cosa comunque irrilevante visto che interessa filtrare le regioni dove è presente un segnale.

Per il calcolo di I_VAR si può usare la funzione stdfilt, tenendo conto che occorre definire in modo opportuno il parametro nhood per ottenere il filtraggio in 3D (non documentato), altrimenti il filtraggio avviene in 2D slice-by-slice.

Per il calcolo di 𝜎 vogliamo implementare un metodo automatico che non richieda il tracciamento di ROI o una conoscenza a priori della geometria del fantoccio, quindi applicheremo un metodo automatico basato sull’analisi dell’istogramma di I_VAR.
Per quanto riguarda la valutazione dell’acutezza, un profilo estratto dal centro dell’immagine del fantoccio non filtrato apparirà come in figura, infatti il segnale in corrispondenza della parete dei cilindri (plexiglass) in MR è nullo a meno del rumore. Per semplicità valuteremo il valore dell’acutezza alla transizione tra fondo e acqua (aria->cilindro esterno).
 
In conclusione, dato il volume MR fornito il programma da realizzare deve eseguire i tre tipi di filtraggio e fornire i valori di SNR e acutezza calcolati nella fetta centrale del fantoccio per il volume originale ed il volume filtrato con i filtri a media mobile, gaussiano e di Wiener. Il filtro Gaussiano va ottimizzato per migliorarne le prestazioni. Per visualizzare l’effetto dei vari algoritmi, è utile visualizzare la differenza tra l’immagine filtrata e quella originale.
IMMAGINE	SNR CLINDRO CENTRALE	ACUTEZZA TRANSIZIONE FONDO
 ->CILINDRO ESTERNO
ORIGINALE		
FILTRATA MEDIA MOBILE		
FILTRATA FILTRO GAUSSIANO		
FILTRATA FILTRO DI WIENER		

