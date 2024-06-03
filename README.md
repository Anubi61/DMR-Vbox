# DMR-Vbox
Private Voice box (recorder and answering machine) for DMR / Semplice segreteria "radiofonica"

ITA Lo scopo del progetto è quello di avere un sistema che si connetta alla rete DMR e registri messaggi diretti al proprio ID, e che poi i messaggi possano essere ascoltati a scelta tramite l'invio in allegato ad una mail, listati su di una pagina php o direttamente attraverso la radio.

Rispetto al codice di [narspt](https://github.com/narspt/DMRVMsg), ho introdotto la lettura di un file di configurazione invece di dover inserire tutte le variabili su di una riga di comando, la lettura e scrittura da/per server mysql, l’invio dei messaggi come allegato di un’email ed infine il riascolto dei messaggi registrati da parte del destinatario usando unicamente la radio.

Il programma può essere compilato sia su raspberry pi che debian su pc. Scrivo Debian perche’ è la distribuzione che ho utilizzato per lo sviluppo. Altre distribuzioni possono essere idonee ma probabilmente il nome dei pacchetti o delle librerie necessarie può cambiare.

Prima di iniziare, occorre avere già funzionante, un server mysql/mariadb. Ovviamente se si vuole usare solo l’email, mysql non serve, bisogna pero’ modificare il sorgente in modo tale che non ci siano riferimenti a sql. 
Comunque mi limito alle spiegazioni per la versione “Full”.<br>
Su mysql creare un database ed una tabella come riportato qui sotto.<br>
Per la successiva compilazione di dmrvbox occorre avere installato la libreria libmariadb-dev

Ulteriore server, questa volta da compilare, e’ md280-emu che serve per convertire i frames da ambe (DMR) a pcm e viceversa e quindi scrivere e leggere i files .wav
Il sorgente lo trovate qui:
solito git clone, poi entrate dentro, digitate make e finita la compilazione ottenete il server che potete mettere dove vi pare.<br> Io ho optato di tenere ogni eseguibile all’interno di cartelle separate il tutto sotto /opt.<br>
La riga di comando per il server e’ ./md380-emu -s 4000<br>
Sul mio repository, trovate un esempio di file service da dare in pasto a systemd e cosi’ avere sia md380-emu che dmrvbox gestiti come servizio dal sistema operativo.
Attenzione che md380-emu va compilato su raspberry o eventualmente cross-compilato su pc, pertanto per farlo funzionare da pc va chiamato tramite l’emulatore arm di qemu.
Quindi se fate girare tutto su raspberry, compilate e siete a posto, diversamente su pc dopo aver compilato con il cross-compiler dovete installare qemu-user-static e qemu-system-arm

La libreria che gestisce l’email la trovate qui:    , scaricate lo zip e lo estraete dove fate lo sviluppo.
Per poterla compilare, dovete necessariamente installare la libcurl4-gnutls-dev
Poi basta entrare nella cartella libquickmail, digitare make e se tutto va bene make install.
E per quanto riguarda la libreria ci siamo.<br> E' necessario però che il file quickmail.h sia visibile successivamente al compilatore, <br>ho scelto dunque di copiarlo dalla cartella libquickmail in /usr/include/email/quickmail.h

Ci si trasferisce nella cartella clonata da github ( git clone https://github.com/Anubi61/DMR-Vbox )

La riga di comando per compilare il tutto e’ 
gcc -o dmrvbox dmrvbox.c ini.c $(mariadb_config --include --libs) -lquickmail

Nota: suggerisco di copiare l’eseguibile in una cartella sotto /opt.
L’importante che dentro questa cartella ci siano le cartelle messages e voices, il file eseguibile ed il file getids.sh (serve per scaricare il database degli ID DMR)

Ad esempio, il mio prototipo in test è qui:

/opt/DMRVbox-qtf<br>
-rw-r--r-- 1 root root     585 Jun  2 09:08 config.ini<br>
-rw-r--r-- 1 root root 5517119 May 31 20:36 DMRIds.dat<br>
-rwxr-xr-x 1 root root   68800 Jun  2 09:03 dmrvbox<br>
-rw-r--r-- 1 root root     111 May  7 09:09 getids.sh<br>
drwxr-xr-x 2 root root    4096 Jun  3 07:27 messages<br>
drwxr-xr-x 2 root root    4096 Jun  2 09:00 voices<br>

Andiamo avanti. Abbiamo l’eseguibile ed ora passiamo al config.ini (puo’ anche avere un altro nome)

Il file config.ini e’ comprensibile leggendolo, basta aprirlo e leggere.
Da tenere presente che il blocco [opts] attiva o disattiva le funzioni compilate.
Ovvero voglio avere solo l’accesso a sql e non mi interessa il discorso email o radio, allora setto:<br>
[opts]<br>
use_email = "false"<br>
use_sql = "true"<br>
use_radio = "false"<br>

Oppure, voglio solo l’ascolto dalla radio e non mi interessa l’invio per email:<br>
[opts]<br>
use_email = "false"<br>
use_sql = "true"   # ininfluente, use_radio setta autonomamente use_sql<br>
use_radio = "true"<br>

Nel blocco [dmr] prestate attenzione che la chiave dmrid non e’ il vostro ID ma l’id + due numeri per identificarlo sulla rete DMR come se fosse un hotspot. <br>Ad esempio il mio id e’:
2230026 ed il dmrid del voicebox e’ 223002604
La chiave dmrtg invece contiene il vostro ID a 7 cifre. 
Nota: Per usare il voicebox in modalita’ tg dmrtg = vostro ID, mentre in modalita’ cp (chiamata privata) dmrtg = 0

Infine nella cartella voices ci sono dei messaggi vocali generati online. Sono sia inglese sia in italiano.<br>
Per l’utente che ascolta i messaggi via radio<br>
err = messaggio di file inesistente  <br> 
n1 = un nuovo messaggio<br>
n2 = nuovi messaggi<br>
nx = fine messaggi<br>
nn = nessun nuovo messaggio<br>
<br>
Per il corrispondente che li registra.<br>
tx0 = benvenuto<br>
tx1 = messaggio registrato e 73<br>
<br>
Posto di avere tutto funzionante ecco come si usa:<br>

Utente Corrispondente imposta il nostro “tg” e preme il ptt.<br>
Se il tempo di trasmissione e’ inferiore a 3 secondi, viene inviato il file tx0, diversamente il file tx1 e la sua trasmissione e’ stata registrata nella cartella messages.<br>
Se abbiamo scelto l’uso di email e configurato i parametri nel file ini, i messaggi ci arrivano in allegato.<br>
Se invece abbiamo optato per la radio, impostiamo il nostro tg sulla nostra radio e premiamo il ptt (brevemente). Dopo alcuni secondi a seconda della condizione, il sistema puo’ trasmettere nn o n1 o n2.<br>
In caso di n1 o n2, per ascoltare il messaggio successivo, altra breve pressione di ptt per avanzare nell’ascolto fino a terminare con nx.<br>
A questo link, una dimostrazione pratica dell’utilizzo.<br>

Per il momento è tutto.<br>

73 de IU4QTF (ex IK4DRV)

<i>Riferimenti:<br>
https://github.com/narspt/DMRVMsg<br>
https://github.com/nostar/reflector_connectors<br>
https://github.com/juribeparada/MMDVM_CM<br>
https://github.com/narspt/md380tools<br>


