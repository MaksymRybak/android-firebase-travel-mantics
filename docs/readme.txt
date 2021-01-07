link al corso https://app.pluralsight.com/library/courses/android-app-android-studio-firebase/table-of-contents
repo git android-firebase-travel-mantics

viene costruita l'app che permette di fare la login, in base il ruolo dell'utente fa vedere una lista di offerte di viaggi
se utente amministratore, puo' inserire o modificare una offerta del viaggio
i dati sono salvati nel firebase 

Creazione progetto firebase 
	url per accedere https://console.firebase.google.com/
	creiamo il progetto dedicato (TravelMantics)
	scegliamo il database da usare, abbiamo due opzioni
		- Cloud firestore
		- Realtime database
		entrambi sono NoSQL db
		scegliamo il secondo, Realtime database
	sezione Authentication serve per configurare l'autenticazione nella nostra app
		useremo email e pwd
		useremo opzione gmail
	sezione Storage
		li dove andiamo a salvare i file (foto delle nostre offerte di viaggio)
Scrittura nel Realtime database
	abbiamo scelto Realtime database, e' un DB NoSQL
		non abbiamo uno schema preconfigurato, sta a noi gestire la consistenza dei dati
		denormalizzazione dei dati, abbiamo dati ripetuti, in questo caso quello che conta e' la velocita di lettura
		(non abbiamo join)
		per passare i dati usiamo il formato JSON
		NOTA: NoSQL da usare quando non dobbiamo avere la consistenza dei dati a livello di DB
		abbiamo la possibilita' di avere 32 livelli di profondita nel JSON ma e' meglio tenere la struttura di dati piu' piatta possibile 
	creiamo il realtime database dalla console di firebase
		NOTA: la navigazione tra una collection e altra avviene tramite i path 
	creiamo il progetto vuoto di android
		firebase richiede hash del debug key store
		Debug keystore
			e' un gruppo di caratteri hash che possano essere ottenuti usando il comando keytool
			questi caratteri identificano il pc, quindi da tenerli in un posto sicuro
			usiamo il tool della nostra JDK per generare hash
		eseguiamo il comando 
			keytool.exe -exportcert -list -v -alias androiddebugkey -keystore %USERPROFILE%\.android\debug.keystore
			(alla richiesta di password digitiamo android)
		copiamo la chiave SHA1
			es: 9D:6F:C9:C0:36:05:07:0B:BC:B2:B1:DC:80:59:1A:A3:D3:85:39:E5
		nella console web di firebase creiamo l'app nel nostro progetto 
			inseriamo il package id 
			incolliamo SHA1 nel campo dedicato 
			clichiamo register
			scarichiamo il file google-services.json e lo mettiamo sotto la cartella app del nostro progetto android
		aggiungiamo firebase sdk nella dipendenza di gradle 
			google services ha il proprio plugin gradle che legge il file google-services.json
			qui seguiamo la guida che ci fa vedere la console di firebase dopo che abbiamo cliccato Next dopo aver scaricato google-services.json
		ogni servizio di firebase ha una dipendenza dedicata da aggiungere al modulo del progetto
			vedi il link https://firebase.google.com/docs/android/setup
			in fondo di sono le dipendenze con le ultime versioni
			aggiungiamo com.google.firebase:firebase-database:19.4.0
		lanciamo app per verificare che tutti e' ok
			dobbiamo vedere la schermata di Hello World
			se qualcosa e' andato storto, per esempio errore di NOT FOUND, assicurarsi di avere google repository aggiornato sotto Android SDK Manager
			(per info vedi il link https://stackoverflow.com/questions/37310188/failed-to-resolve-com-google-firebasefirebase-core9-0-0)
	creazione activity per scrivere i dati nel nostro db
		aggiungiamo all'activity tre campi per inserire Titolo, Prezzo e Descrizione della nostra offerta del viaggio
		aggiungiamo il menu alla nostra app
			tasto dx sulla cartella res -> impostiamo resource type = menu -> create
			vediamo nuova cartella menu sotto res 
			tasto dx -> creiamo nuovo file di risorse dedicato al menu 
			modifichiamo il file vuoto appena creato aggiungendo l'item del menu relativo al salvataggio (vedi il codice)
			su AS vediamo anche la preview del nostro menu
		sovrascriviamo il metodo onCreateOptionsMenu nella nostra activity per caricare il menu dal file xml di risorse 
	scrittura dei dati nel db di firebase
		per scrivere nel db abbiamo disogno di un paio di oggetti
			FirebaseDatabase - entrypoint per il nostro DB
			DatabaseReference - il punto all'interno del nostro DB dove andiamo a scrivere i dati (aka la collection)
			firebaseDatabase = FirebaseDatabase.getInstance() x creare l'istanza al DB
			firebaseDatabase.getReference().child("traveldeals") x ottenere il riferimento al path traveldeals
			(vedi il codice)
			per gestire l'evento di click sul menu facciamo override del metodo onOptionsItemSelected
		se la prima scrittura e' KO, assicuriamoci che le regole di sicurezza nella console firebase sono:
			{
			  "rules": {
				".read": true,
				".write": true
			  }
			}
			NOTA: solo per lo sviluppo, accesso pubblico alle scritture e letture sul DB
		dopo l'inserimento, vediamo una chiave di tipo -MH0cQM1col4Szmh09ep relativa all'item inserito 
			e' cosi detto Push ID - ID univoco generato da Firebase nel momento di utilizzo del metodo push()
Lettura da Realtime database 
	normalmente, quando abbiamo bisogno dei dati, facciamo una richiesta al DB per leggere i dati
	on realtime db e' diverso, la nostra app rimane in ascolto sul cambiamento dei dati, e riceve dati aggiornati una volta che sono stati cambiati 
		viene usato oggetto ChildEventListener, rimane in ascolto sul cambiamento di children in un database reference
		nel caso della nostra app il child e' la collection traveldeals
		gli eventi che possiamo ascoltare sono:
			- onChildAdded (scatta per ogni inserimento nella location del listener)
			- onChildChanged (dati sono cambiati)
			- onChildRemoved (item figlio e' stato rimosso)
			- onChildMoved (la location di item figlio e' cambiata)
			- onCancelled (listener e' fallito a livello di servizio o quando ci sono problemi di sicurezza)
	aggiungiamo una nuova activity per stampare i dati letti dal realtime database 
		creiamo la connessione al db e oggetto ChildEventListener
		il metodo onChildAdded di ChildEventListener viene invocato per ogni item presente nel databaseReference (aka per ogni item nella collection)
		usiamo oggetto DataSnapshot (oggetto contenente i dati letti dal listener) per ottenere l'istanza di TravelDeal
		aggiungiamo l'istanza di ChildEventListener al databaseReference
		NOTA: il modello del nostro item (TravelDeal) deve avere un costruttore vuoto, serve per far funzionare correttamente la deserializzazione di dati ricevuti da realtime db
	refactoring del codice
		aggiungiamo utility class FirebaseUtil
		(vedi il codice)
		la classe serve per centralizzare la parte relativa a firebase (connessione, recupero di riferimento alla collection)
	utilizzo di RecyclerView con Firebase
		RecyclerView e' una view flessibile che fornisce una finestra limitata per grande mole di dati 
		e' una lista scrollabile di dati 
		e' ottimizzata per occupare la memoria in modo efficiente (parola recycler significa questo)
		puo' essere visualizzata per mostrare migliaia di righe ma crea gli item al suo interno solo per il numero che deve essere mostrato a video
		creiamo il layout dedicato alla singola riga di RecyclerView -> nuovo xml di layout (vedi il codice)
			aggiungiamo a questo layout una TextView per stampare il Title del nostro deal
		RecyclerView ha bisogno di un adapter per caricare i dati
			adapter recupera i dati da firebase 
			adapter invia i dati a RecyclerView tramite oggetto ViewHolder 
			ViewHolder descrive un item di RecyclerView, nel nostro caso e' il singolo TravelDeal
			ViewHolder contiene l'informazione del posizionamento dell'item all'interno di RecyclerView
		Adapter e ViewHolder lavorano insieme
			Adapter e' sottoclasse di ViewHolder
			carica i dati e fa il binding di dati alla view
			ViewHolder usa la cache per rendere lo scrolling piu' fluido
		usiamo oggetto LayoutManager per controllare in che modo i dati sono visualizzati all'interno di RecyclerView
	new layout per singolo item che rappresenta un offerta 
		usiamo ConstraintLayout
		modifichiamo DealAdapter per fare il bind dei dati nuovi (descrizione e il prezzo)
		creiamo il menu dedicato alla ListActivity con la voce New Travel Deal che permette di passare all'activity di inserimento nuova offerta 
			sovracriviamo onCreateOptionsMenu inserendo il layout del nostro menu (vedi il codice)
			sovracriviamo onOptionsItemSelected per gestire il click sulla unica voce - apertura di DealActivity usata per inserire nuova offerta viaggio
	possibilita' di modificare e eliminare un offerta dalla lista di offerte 
		abilitiamo evento click per la nostra RecyclerView
		(vedi il codice)
		e' sufficiente implementare View.OnClickListener da parte di DealViewHoler
		implementiamo il metodo onClick che ci permette a leggere l'indice dell'item cliccato nella nostra RecyclerView
		recuperiamo il deal selezionato dalla lista deals e lo passiamo come extra nell'activity TravelDeal
			NOTA: qui la nostra classe modello TravelDeal implementa semplicemente l'interfaccia Serializable per poter essere passata usando extra
				  e' per semplicita' di esempio e della classe stessa (ha poche proprieta')
				  e' consigliato cmq implementare interfaccia Parcelable come visto gia' durante il corso di introduzione ad android 
		modifichiamo DealActivity che puo' ricevere il Deal tramite extra (vedi codice)
		aggiungiamo la possibilita' di modificare e eliminare il deal 
			lato firebase abbiamo questi metodi:
				- ref.push().setValue(deal)					// creazione nuovo item
				- ref.child(deal.getId()).setValue(deal)	// modifica item esistente
				- ref.child(deal.getId()).removeValue()		// eliinazione item
			modifichiamo DealActivity opportunamente (vedi codice)
Gestione dell'autenticazione
	demandiamo la gestione di autenticazione e autorizzazione a firebase
	(spesso la parte di autenticazione e autorizzazione di una app e' molto noiosa, dobbiamo gestire la registrazione degli utenti, cambio pwd, recupero pwd, login, 
		aggiornamento della pwd quando scade)
	useremo FirebaseUI per gestire la login 
	configurazione regola di sicurezza di Realtime Database 
		{
		  "rules": {
			".read": "auth != null",
			".write": "auth != null"
		  }
		}
		possiamo configurare le regole come
			.read - se e quando i dati possano essere letti dall'utente
			.write - se e quando i dati possano essere scritti dall'utente 
			.validate - per specificare il formato di dati, il tipo, attributi figli
			.indexOn - supporto per ordinamento e querying 
	(se proviamo a lanciare la nostra app riceveremo errore "DatabaseError: Permission denied")
	Utilizzo di Firebase UI
		abilitiamo l'autenticazione dal menu Authentication
		in questa area possiamo specificare in che modo utenti possano accedere alla nostra app, per es. email e pwd, google, fb, twitter, github, telefono etc.
		dal menu Authentication abilitiamo le opzioni Email/Password e Google
		qui https://firebase.google.com/docs/auth possiamo vedere SDK di Firebase che facilita l'integrazione di autenticazione nella nostra app
			useremo FirebaseUI Auth
			e' una libreria open source che gestisce il processo di sign-in
			abbiamo la parte di UI gia' pronta 
			possiamo scegliere il provider di autenticazione che vogliamo usare
			FirebaseUI fornisce la parte di UI per tutti i provider gestiti 
		aggiungiamo le dipendenze necessarie specificate qui https://firebase.google.com/docs/auth/android/firebaseui
		recap di cosa vogliamo fare:
			quando apriamo l'app vogliamo controllare se l'utente e' loggato o no
			per fare cio' abbiamo l'oggetto AuthStateListener
				viene richiamato quando ci sono modifiche nello stato di autenticazione (es. sign-in, sign-out)
				agganciamo il listener nel metodo onResume()
				sganciamo il listener nel metodo onPause()
			se utente non e' loggato, dobbiamo abilitare il flusso di sign-in
			dopo il login utente e' abilitato a vedere le offerte di viaggio
			dobbiamo permettere all'utente di eseguire anche il log-out
			dopo il logout facciamo il detach da AuthStateListener e svuotiamo la nostra RecyclerView
	(vedi codice per la parte di login, usiamo FirebaseUI e AuthStateListener. NOTA: adapter esegue la richiesta a Realtime DB, spostiamo questa parte 
		nel metodo onResume() per rifare la richiesta quando il controllo torna alla nostra activity dopo la fase di login)
	Logout dell'utente
		aggiungiamo una nuova voce nel nostro menu per il Logout
		gestiamo il click di logout nel metodo onOptionsItemSelected della nostra activity
		il codice di logout e' recuperabile dal sito di firebase (https://firebase.google.com/docs/auth/android/firebaseui)
		NOTA: per far funzionare la login con account google dobbiamo autorizzare la nostra app a fare le richieste di autenticazione su Firebase
			-> registriamo la chiave SHA1 generata da gradle (signingReport) su Android Studio nelle impostazioni del nostro progetto sulla console di firebase 
			(aggiungiamo il fingerprint)
	Differenziare l'utilizzo dell'app in base al ruolo dell'utente
		predediamo nella nostra app due tipologie di utenti
			- user: utente normale, puo' solo consultare il contenuto senza privilegi di modifica
			- admin: utente amministratore, puo' anche modificare il contenuto 
		reminder: ricordiamo la differenza tra l'autenticazione e autorizzazione, autenticazione serve per identificare l'utente, autorizzazione per controllare 
			i permessi e risorse alle quali puo' accedere
		configurazione di regole in firebase databse
			// configurazione a livello globale, per tutti i nodi del nostro DB 
			{
			  "rules": {
				".read": "auth != null",
				".write": "auth != null"
			  }
			}
			// configurazione custom per diversi nodi
			{
			  "rules": {
				"traveldeals": {
					".read": true		// possano leggere tutti
				},
				"offers": {
					".read": "auth != null"		// possano leggere solo utenti autenticati, oggetto auth rappresenta l'autenticazione dell'utente
				}
			  }
			}
			NOTA: se non specifichiamo niente per una regola (per es. x ".write"), il permesso e' negato
			oggetto auth ha la prop uid contenente ID univoco dell'utente
			(ci sono vari modi per configurare le regole di autorizzazione in FirebaseDatabase, vedi le slide per gli esempi)
	demo: configurazione piu' utenti con ruoli diversi
		creiamo un nuovo nodo (collection) nel Realtime DB, chiamato administrators
		questo nodo contiene utenti amministratori della nostra app
		modifichiamo la regola ".write" del nostro db facendo un check se utente autenticazione e' nella collection di amministratori
		modifichiamo l'app facendo il check dell'UID loggato, se e' un amministratore o no
			interroghiamo la collection administrators su Realtime DB
			rimaniamo in ascolto sull'inderimento di nuovi utenti amministratori 
			usiamo un flag nella nostra classe FirebaseUtil per capire se un utente e' amministratore o no, se si, abilitiamo i menu di eliminazione e salvataggio offerta viaggio 
			un utente normale puo' solo vedere il contenuto di una offerta, il testo e' disabilitato non modificabile
Utilizzo di Firebase Storage
	(NOTA: questo capito viene ascoltato e basta, senza implementare il codice della demo)
	Firebase Storage e' la possibilita' di salvare i dati in cloud, dedicato piu' ai file, documenti, immagini, video, audio
	NOTA: Realtime DB invece e' per dati in formato JSON 
	PROS di Firebase Storage:
		- alla perdita di connessione il download/upload non si interrompe ma va in pausa, continuando il processo al ripristino della rete
		- sicurezza, integrazione di Firebase Authentication per proteggere i file
		- scalabile (google cloud service)
	app: verra' aggiunta la possibilita' x amministratori di caricare una foto per un offerta di viaggio
	sulla console di firebase viene creato un nuovo storage (bucket) con le impostazioni di sicurezza di default (massimi permessi per utenti autenticati)
		creiamo nuova cartella (deals_pictures)
		NOTA: possiamo impostare i permessi a livello di tutto lo storage o singole cartelle
	modifichiamo app per accedere allo storage
		aggiungiamo dipendenza a firebase-storage
		aggiungiamo il Button e ImageView all'activity di dettaglio offerta 
		il pulsante crea un nuovo intent per activity di selezione risorsa locale al dispositivo (es. immagine)
		modifichiamo FirebaseUtil aggiungendo gli oggetti di FirebaseStorage
		gestiamo il risultato dell'intent di scelta foto caricando l'immagine scelta su FirebaseStorage e salvando il link dell'immagine caricata nel deal relativo 
		per mostrare immagini all'interno dell'app viene usata la libreria Picasso
			carica img partendo da un URL in una ImageView
		Picasso Library
			lightweight
			handling ImageView recycling
			complex image trasformation with minimal memory use
			automatic memory and disk caching
			https://square.github.io/picasso/
		aggiungiamo la dipendenza alla libreria di picasso
		l'immagine viene anche ridimensionata per farcela stare nello schermo (recuperiamo width dello schermo dalle risorse e impostiamo il height 2/3 di larghezza, vedi demo)
		mostriamo un thumbnail dell'immagine nella lista di offerte, per visualizzare thumbnail usiamo sempre picasso ridimensionando la nostra immagine originale 
	eliminazione immagini dallo storage
		otteniamo il riferimento all'immagine e la eliminiamo
		vedi demo per dettagli
	limitare gli accessi ai file
		FirebaseStorage ha le regole di sicurezza (purtroppo la sintassi e' diversa da quella di RealtimeDB)
		usando le parole chiavi come match e allow possiamo configurare le regole di sicurezza in modo molto flessibile
		match per specificare il path
		allow per specificare la regola di sicurezza
		abbiamo due oggetti request (rappresenta la richiesta che arriva allo storage) e resource (rappresenta una risorsa nello storage)
		
		