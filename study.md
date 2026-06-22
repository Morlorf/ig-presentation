# Luna Park: Documentazione Tecnica e Architetturale

Questa documentazione costituisce un'analisi dettagliata, chiara ed esaustiva dell'architettura e delle scelte ingegneristiche alla base del progetto "Luna Park", realizzato con Three.js. L'obiettivo di questo documento è fornire un quadro completo di ogni elemento presente nella scena 3D, affrontando prima la sua descrizione concettuale e le sfide di sviluppo ad esso legate, per poi esaminare passo dopo passo il codice che ne permette il funzionamento matematico.

---

## 1. Asset Pipeline, Rendering PBR e Materiali

### Descrizione Generale
Prima ancora di animare il parco, l'applicazione deve costruire il mondo caricando le geometrie e i materiali. In un progetto grafico moderno, la fedeltà visiva è garantita dall'uso del Physically Based Rendering (PBR), un paradigma in cui i materiali reagiscono alla luce imitando fedelmente le superfici del mondo reale. Il problema principale affrontato in questa fase iniziale del progetto riguarda la discrepanza tra come i software di modellazione esportano i file (i `.glb` o gli `.obj`) e come Three.js deve intepretarli per garantire il calcolo corretto delle ombre, la trasparenza delle chiome degli alberi e lo spazio cromatico delle texture. Si rende quindi necessario un sistema che carichi e "igienizzi" (sanitize) tutti i modelli prima di immetterli nella scena.

### Spiegazione del Codice e Caricamento
Osserviamo come il modulo dei caricamenti garantisce la coerenza cromatica e fisica dei materiali.

```javascript
// src/utils/loaders.js
export function loadLinearTexture(url, { repeat = [1, 1], anisotropy = 8 } = {}) {
  const tex = textureLoader.load(url);
  tex.colorSpace = THREE.LinearSRGBColorSpace;
  tex.wrapS = tex.wrapT = THREE.RepeatWrapping;
  tex.repeat.set(repeat[0], repeat[1]);
  tex.anisotropy = anisotropy;
  return tex;
}
```

Questo frammento di codice definisce la funzione per caricare le texture che portano dati matematici (come le Normal Map per i rilievi o le Roughness Map per l'opacità). A differenza delle texture che definiscono il colore base visibile dall'occhio umano (che vengono caricate in spazio colore SRGB, quindi alterato per adattarsi ai monitor), le texture matematiche devono essere lette nello spazio colore lineare, indicato da `THREE.LinearSRGBColorSpace`. Questo è cruciale: se una mappa delle normali subisse la correzione Gamma, i vettori tridimensionali in essa scritti come colori si deformerebbero, causando riflessi di luce completamente errati. 
Successivamente, il codice applica l'anisotropia impostandola a `8`. Il filtro anisotropico risolve un problema noto del texturing tridimensionale: quando guardiamo una texture estesa come l'erba o il suolo da un'angolazione rasente al suolo, il normale filtraggio MipMap impasterebbe e sfoccherebbe l'immagine. Impostando l'anisotropia a un valore alto, la scheda video migliora la nitidezza dei dettagli in lontananza calcolando il colore dei pixel proiettati su forme trapezoidali anziché quadrate, restituendo un suolo nitido fino all'orizzonte.

---

## 2. Le Attrazioni del Parco

### La Ruota Panoramica (Ferris Wheel)

**Descrizione Generale e Ruolo nel Progetto:**
La Ruota Panoramica funge da enorme faro visivo al centro del parco. Essendo visibile da ogni angolazione, richiede un colpo d'occhio maestoso ma, allo stesso tempo, deve rispettare la gravità in ogni momento. La sfida principale legata a questo tipo di attrazione consiste nel rapporto gerarchico degli oggetti 3D (Scene Graph): se le cabine dei passeggeri venissero semplicemente collegate alla ruota centrale gigante, man mano che la ruota gira porterebbe inevitabilmente i passeggeri a ritrovarsi a testa in giù nel punto più alto dell'orbita. Bisogna quindi disaccoppiare attivamente l'orientamento delle cabine. A questo si aggiunge la necessità di simulare in maniera organica il vento e l'inerzia, per evitare che le cabine sembrino robotizzate.

**Implementazione e Spiegazione del Codice:**

```javascript
// src/rides/FerrisWheel.js
this.angle += speedMult * MAX_SPEED * delta;
this.wheelSpin.rotation.x = this.angle;

for (let i = 0; i < this.gondolaMounts.length; i++) {
  const mount = this.gondolaMounts[i];
  
  let tilt = -this.angle;
  tilt += Math.sin(t * 1.5 + i) * 0.08; 
  
  mount.gondolaMesh.rotation.x = tilt;
}
```

Il blocco di codice analizzato gestisce questo esatto problema frame per frame. Innanzitutto, l'intero anello della ruota avanza di rotazione calcolando la propria inclinazione moltiplicata per il tempo intercorso (delta). 
Entrando nel ciclo `for`, il motore scorre ogni singola cabina installata. La variabile `tilt` viene inizializzata col valore `-this.angle`. Questo è il cuore della soluzione al capovolgimento: applichiamo sulla mesh della cabina una contro-rotazione esatta, di segno opposto rispetto a quella che sta subendo dal suo oggetto "genitore" (la grande ruota). Il risultato globale netto di questa compensazione è zero gradi, il che mantiene l'asse verticale costantemente allineato alla gravità terrestre.
Ma una cabina perfettamente dritta sembra rigida e finta. La riga successiva aggiunge quindi alla contro-rotazione un disturbo organico usando una curva del `Math.sin`. Calcolando il seno in base allo scorrere del tempo `t` moltiplicato per una certa velocità d'oscillazione, la cabina ottiene un dondolio passivo. La genialità risiede nel sommare l'indice della cabina (`+ i`) all'interno dell'argomento del seno: sfalsando matematicamente la fase dell'onda per ogni cabina, si fa in modo che il vento "investa" le navicelle in istanti temporali sfalsati fra loro, disarmonizzandole e portando vero caos controllato alla scena.


### Le Montagne Russe (Tangled Twister Roller Coaster)

**Descrizione Generale e Ruolo nel Progetto:**
Il Roller Coaster è la simulazione fisica più impegnativa dell'intero parco. Il tracciato si estende tramite una complessa serpentina geometrica tridimensionale. A differenza di una giostra classica che ruota su perni definiti, le montagne russe devono guidare i propri convogli attraverso direzioni che cambiano continuamente nello spazio. Se non curato propriamente, inseguire rigidamente il binario in grafica 3D causa il cosiddetto "Parasitic Twist" (Torsione Parassita): nel tentativo di seguire curve repentine, l'asse "Alto" della telecamera e del vagone collassa lateralmente ruotando bruscamente e senza controllo su se stesso, finendo in un groviglio. Per mantenere i passeggeri saldi all'interno dei seggiolini ed impedire gravi attacchi di chinetosi (Motion Sickness) ai giocatori in visuale Prima Persona, si utilizza un rigoroso sistema di "Trasporto Parallelo".

**Implementazione e Spiegazione del Codice:**

```javascript
// src/rides/Coaster.js 
for (let r = 1; r < n; r++) {
  const prevT = rawPts[r].clone().sub(rawPts[(r - 2 + n) % n]).normalize();
  const currT = rawPts[(r + 1) % n].clone().sub(rawPts[r - 1]).normalize();
  
  const qTrans = new THREE.Quaternion().setFromUnitVectors(prevT, currT);
  const projectedPrevU = rawUps[r - 1].clone().applyQuaternion(qTrans);
}
```

Questo snippet di codice si occupa di precostruire le rotaie al caricamento del progetto. Il suo scopo primario è delineare per ogni pezzettino di binario un vettore che dica al treno qual è il verso dell'"Alto" da rispettare rigorosamente per non capovolgersi di lato.
Nella prima e seconda linea, il programma determina in che direzione viaggiava il treno un istante fa (`prevT`) e in che direzione viaggia adesso (`currT`). Per farlo, sottrae matematicamente i vertici tra loro. Poiché i binari costituiscono un anello chiuso e unito, non possiamo rischiare di calcolare i vertici "precedenti" mandando l'array in difetto e bloccando l'applicazione: la curiosa formula `(r - 2 + n) % n` è un espediente algebrico che sfrutta il numero di elementi totali del binario (`n`) e l'operatore "Modulo" (`%`) per permettere al calcolatore di leggere un percorso continuo. Quando arriva al termine della lista riparte da zero, e quando cerca indietro rispetto allo zero finisce in fondo alla lista senza interruzioni.

Successivamente, il motore calcola il quaternione `qTrans`. La funzione `setFromUnitVectors` rileva in via assoluta quale sia il tasso esatto di curvatura intercorso per passare dalla direzione del vecchio binario a quella del nuovo. 
Trovata la curvatura spaziale, la applica (`applyQuaternion`) all'orientamento Alto originario del vecchio binario. Questa catena di operazioni, ripetuta per tutta la rotaia segmento per segmento, si assicura che il cielo rimanga sempre "sopra" e il carrello ruoti il minimo indispensabile per assecondare la curva in totale fluidità.


### Turbo Tagada

**Descrizione Generale e Ruolo nel Progetto:**
Il Tagada rappresenta un classico divertimento da luna park ad alta intensità. Consiste di un enorme catino circolare in grado di ospitare la folla e di centrifugare, ballare ed inclinarsi asimmetricamente a diverse angolazioni sfidando i limiti dell'equilibrio dei passeggeri. In programmazione, la criticità risiede nel combinare forze rotatorie indipendenti fra loro: una rotazione veloce del disco su sé stesso, il braccio meccanico sotto che si solleva (inclinazione), i sussulti dovuti al finto circuito idraulico sotto sforzo, e l'ondeggiamento rotazionale laterale. L'incastro imperfetto di matrici di rotazione lungo assi interconnessi porta matematicamente al "Gimbal Lock", o blocco cardanico, un errore di simulazione per il quale la giostra perde l'accesso a una dimensione e s'incarta ruotando in direzioni disastrose.

**Implementazione e Spiegazione del Codice:**

```javascript
// src/rides/Tagada.js
const currentPitch = BASE_PITCH + Math.sin(this.pitchAngle) * PITCH_AMP * ease;
const currentRoll = Math.cos(this.rollAngle) * ROLL_AMP * ease;
const currentBump = Math.sin(this.bumpAngle) * BUMP_AMP * ease;

this.armPivot.rotation.set(currentPitch + currentBump, this.armYawAngle, currentRoll, 'YXZ');
```

Qui il programma decostruisce il moto della piattaforma in frazioni sinusoidali che vengono sommate in tempo reale. Le prime due righe orchestrano l'inclinazione in avanti (Pitch) e lateralmente (Roll) del catino metallico. Sfruttando contemporaneamente il seno e il coseno per i due assi si genera un movimento ellittico armonioso, una "Nutazione" lenta e passiva tipica del macchinario in fase d'avvio. 
A questo moto ondulatorio morbido, il codice somma `currentBump`. Usando un'onda a cortissima frequenza (`bumpAngle`), introduce sobbalzi aggressivi e veloci. È un "colpo secco" artificiale imposto alla base meccanica.

Nel momento cruciale in cui tutti questi moti combinati vengono applicati all'oggetto fisico 3D, interviene la risoluzione ai problemi del Gimbal Lock descritti. Nel comando `rotation.set`, la stringa finale `'YXZ'` stabilisce in maniera ferrea per il motore grafico di non interpretare gli angoli nello standard di fabbrica (che solitamente è XYZ). Questo ordine esplicito forza il motore ad elaborare dapprima lo Spin primario impazzito del disco sull'asse verticale (Y), garantendo che i beccheggi avvengano con indipendenza rispetto a quest'ultimo, portando a casa una simulazione estremamente fluida e indistruttibile.


### Le Mongolfiere e il Vento Procedurale (Hot Air Balloons)

**Descrizione Generale e Ruolo nel Progetto:**
Le mongolfiere forniscono volume ai cieli del parco e donano una sensazione di libertà aerea rispetto alle attrazioni ancorate in terra ferma. A differenza del treno sui binari, non seguono percorsi prefissati, ma essendo svincolate da trazioni meccaniche rigide, devono simulare in via approssimativa la spinta dei flussi d'aria calda. Se le loro coordinate venissero pilotate per mezzo di generatori matematici randomici generici (come `Math.random()`), scatterebbero confusamente per i cieli come mosche impazzite ad ogni fotogramma per via del rumore bianco dell'informazione disorganizzata. Occorre invece una deriva controllata, che le faccia spostare morbidamente di lato come trascinate da una lenta carezza atmosferica. Questo si risolve usando algoritmi derivati dal Noise Mapping.

**Implementazione e Spiegazione del Codice:**

```javascript
// src/rides/Balloon.js
function noise2D(x, y) {
  const ix = Math.floor(x), iy = Math.floor(y);
  const fx = x - ix, fy = y - iy;
  const a = hash2D(ix, iy); 
  const b = hash2D(ix + 1, iy);
  // ... si misurano i restanti vertici ...
  return (a * (1 - u) + b * u) * (1 - v) + (c * (1 - u) + d * u) * v;
}
```

La funzione qui proposta incarna il concetto del **Value Noise** bidimensionale, vitale per generare correnti d'aria credibili. Nel momento in cui la mongolfiera attraversa il cielo, il codice rileva la sua esatta locazione planimetrica dividendola tra la sua componente Intera (`ix`, `iy`) e Frazionaria (`fx`, `fy`).
Il cielo viene virtualmente diviso in quadrati perfetti. Sulla base delle coordinate intere (gli spigoli di questi quadrati), l'algoritmo calcola quattro distinti venti casuali e deterministici richiamando un hasher logico (`a, b, c, d`). 
Tuttavia, siccome la mongolfiera sosta quasi sempre *in mezzo* al quadrato virtuale fluttuante (indicato dalle costanti frazionarie che fungono da tracciante puntuale), il blocco calcola una interpolazione bilineare algebrica (nell'istruzione `return` finale). In poche parole, l'algoritmo fa una complessa e ponderata media d'attrazione tra i quattro estremi sfumati. Il pallone subisce così vettori di spostamento transizionali puliti e coerenti per centinaia di fotogrammi senza sussulti o difetti.

---

## 3. L'Acqua, Gli Shaders Ambientali e i Particellari

### Acqua Fluviale Procedurale (`Water.js`)

**Descrizione Generale e Ruolo nel Progetto:**
L'enorme superficie acquatica che taglia a metà la mappa di gioco ha il compito di sembrare dinamica, reattiva alla luce, trasparente per far apparire i riflessi sottostanti e realistica nel suo lento defluire attraverso il fossato. La maggior parte degli sviluppatori in contesti generici applica un video 2D sull'oggetto e lo fa scorrere alterandone le coordinate texture. In questo progetto, l'acqua non è un dipinto: consiste di poligoni solidi ed esiste in uno stato liquido programmato in Shader Language. Sono equazioni fisiche eseguite direttamente all'interno della memoria video (GPU) che innalzano attivamente il fiume per creare increspature di onde. 

**Implementazione e Spiegazione del Codice:**

```glsl
// src/environment/Water.js - Vertex Shader 
vec3 sinWave(vec3 pos, vec2 dir, float wavelength, float amp, float speed, float t) {
  float k = 6.2831853 / wavelength;               
  float phase = k * dot(dir, pos.xz) - speed * t; 
  float h = amp * sin(phase);                     
  float dh = amp * cos(phase) * k;                
  return vec3(h, dh * dir.x, dh * dir.y);
}
```

Questo vertiginoso frammento in linguaggio nativo GLSL si occupa di calcolare l'impatto di un'onda sul singolo micron di fiume esaminato.
Inizialmente, stabilisce una costante fisica `k` dividendo Due Pi Greco per la lunghezza attesa dell'onda (una misura fondamentale per decifrarne la fittezza nello spazio planimetrico). Successivamente stabilisce la `phase`: con l'utilizzo di un prodotto puntuale tra la direzione dell'increspatura e la posizione attuale sulla piantina orizzontale, scopre a che punto del "viaggio" si colloca l'onda in questo istante dettato dal parametro del tempo universale `t`.
Calcolando banalmente l'ampiezza per il Seno matematico della fase ottiene l'innalzamento desiderato `h`. L'acqua perciò si muove tridimensionalmente seguendo una corrente.

Ma sollevare un poligono non basta. Se una luce (come il Sole) lo colpisce, essendo la geometria alterata programmaticamente e non in base alla modellazione in file originaria, il raggio di luce crederebbe di scontrarsi contro un lago interamente piatto, oscurandone le creste liquide. 
Questo viene corretto magnificamente con l'espressione `dh`. Al posto di calcolare approssimativamente l'inclinazione dell'onda comparandola ai poligoni vicini in modo laborioso per l'hardware, lo sviluppatore sfrutta il puro concetto di Derivata Spaziale (Regola della Catena): sapendo in partenza l'equazione del Seno, tira fuori il suo corrispettivo integrato del Coseno assieme a `k`. Passando poi questo dato di ripidità dell'onda alle normali del Fragment Shader, si illude perfettamente la luce di riflettersi su un vero flusso cristallino a scacchiera liquida con zero spreco computazionale.

### Fuochi d'Artificio CPU-less (`Fireworks.js`)

**Descrizione Generale e Ruolo nel Progetto:**
Al giungere del buio e all'attivazione via console dei razzi illuminanti, enormi fuochi d'artificio colorati brillano proiettandosi verso la luna, esplodendo in corone gigantesche e decadendo in scintille libere sul prato del parco giochi. Simulazioni simili con migliaia di particelle richiederebbero di creare migliaia di mesh JavaScript individuali nel DOM. Ne deriverebbe un surriscaldamento spietato del thread del browser e del Garbage Collector. Anche qui la soluzione ingegneristica è delegare del tutto l'animazione particellare alle unità di Shading parallele della scheda madre: viene istanziata un'unica, enorme scatola geometrica virtuale su cui esplode la miscela pirotecnica calcolando velocità iniziale, gravità terrestre e fardello aerodinamico per un singolo minuscolo vertice indipendente, a una spesa computazionale nulla.

**Implementazione e Spiegazione del Codice:**

```glsl
// Vertex Shader - Fisica Integrale per le Polveri Pirotecniche
float drag = 1.2;
vec3 vel = aVelocity;
pos += vel / drag * (1.0 - exp(-drag * age));

pos.y -= 4.9 * age * age;
```

L'algoritmo non potrebbe essere una resa più elegante dell'equazione del Moto Viscoso tratta direttamente dai manuali di meccanica classica.
Una normale particella, sottoposta alla sola istruzione propulsiva di detonazione, si spaccherebbe schizzando via alle coordinate dell'universo infinito spinta da una velocità inesorabile. Quello di cui abbiamo bisogno sono le forze che modellano il fuoco di un razzo: una spinta rapidissima seguita da una stasi frenata bruscamente in aria mentre la polvere incandescente brucia restando sospesa in forma sferica per gli spettatori.
Questo fenomeno si traduce con il calcolo che include la variabile di frizione atmosferica `drag` (resistenza dell'aria). Attraverso la formula dell'Esponenziale di Nepero incrociata all'età vitale della scintilla (`age`), si innesca la magia: all'istante dell'esplosione, l'esponenziale restituisce un fattore neutro scaraventando la polvere al massimo potenziale; subito dopo la grandezza esponenziale collassa in caduta libera. La parentesi crolla smorzando drasticamente la spinta espansiva forzandola contro il tetto invalicabile imposto matematicamente dalla porzione `vel/drag`. La scintilla frena di schianto simulando di aver consumato il colpo nel vuoto del cielo.
Ad arricchire l'estetica interviene l'ultima equazione differenziale di gravità: decrementa uniformemente l'asse verticale basandosi sullo scorrere del tempo moltiplicato al quadrato in rapporto al `4.9` (esattamente la metà arrotondata della costante gravitazionale terrestre 9.8). La particella piomba romanticamente a parabola formando i tipici "salici piangenti" pirotecnici di chiusura esplosiva, prima di essere annullata per sempre dallo schermo per risparmiare rendering.

---

## 4. IA, Navigazione Spaziale e Cinematiche Inverse (Gli NPC)

### Navigazione, Collisioni e Rasterizzazione (La NavGrid)

**Descrizione Generale e Ruolo nel Progetto:**
Gli avatar non umani che affollano il parco godono del dono del movimento autonomo. Sono i frequentatori della fiera (Visitors). Essi sanno perfettamente aggirare le mura, attraversare il ponte sull'acqua, esaminare le giostre in blocco, ed evitare contatti frontali l'un l'altro sfalsandosi come una vera calca ai lati. Per donargli questo livello di consapevolezza, il sistema necessita in primis di una mappa, perché il mondo tridimensionale (se calcolato usando Raycasting fisici o mesh collisori 3D reali su decine di soggetti in loop continuo) spegnerebbe immediatamente il motore JavaScript per i lag. La mappa del parco deve essere smontata "offline", prima della fase di avvio, rasterizzata sotto forma di piantina su una matrice bidimensionale a pesi, che codifica aree impossibili, aree lente e aree consigliate all'attraversamento algoritmico intelligente `A*`.

**Implementazione e Spiegazione del Codice:**

```javascript
// src/utils/NavGrid.js
const dx = x - cx; 
const dz = z - cz;
if (dx * dx + dz * dz < (cr + VIS_CLEAR) * (cr + VIS_CLEAR)) {
    blocked = true;
}
```

La funzione qui presente è adibita al "Censimento" e alla chiusura dei colli di bottiglia statici: le mura dei recinti, gli alberi massicci e il corso d'acqua in base ai raggi di ingombro circolari `cr` prelevati dal file costante. 
La logica per testare le distanze di collisione (e definire un pezzo di cella come inagibile precludendolo all'algoritmo) solitamente impiega la formula della Radice Quadrata tra asse e ordinata (il celebre Teorema di Pitagora). Siccome la radice quadrata in informatica è uno dei calcoli più pesanti del processore aritmetico, l'autore ottimizza in maniera radicale.
Se vogliamo sapere se il quadrato in questione si trova *dentro* al raggio dell'ostacolo `cx, cz`, innalziamo le variabili di calcolo e distanza di ambo i lati della disequazione al quadrato in una forma algebrica. Confrontando la semplice somma `dx*dx + dz*dz` con il margine di rischio totale e raggio per sé stesso, eseguiamo comparazioni velocissime escludendo chirurgicamente i pixel della via senza intoppare il frame zero all'avvio. Da questo schema solido, i pedoni applicano successivi ricalcoli morbidi sulle maglie della matrice (`String-Pulling`) per rendere il cammino fluido, sensato ed in armonia con le passerelle del parco cittadino curvo e asimmetrico.

### L'Appoggio Plantare e l'IA Cinematica dei Pedoni

**Descrizione Generale e Ruolo nel Progetto:**
Di enorme interesse ingegneristico è l'assenza assoluta di animazioni pre-cotte nella vita digitale degli avatar che si aggirano sulla griglia analizzata prima. Nessun file esterno definisce i cicli di camminata. Questo per garantire che i piedi non slittino mai a vuoto sopra il prato come se stessero sfregando sul ghiaccio nel momento in cui l'avatar decide di mutare direzione, voltarsi improvvisamente o rallentare per guardare i chioschi. Il motore di animazione utilizza in maniera pionieristica l'Inverse Kinematics (IK, Cinematica Inversa) a 2 Ossa. È un procedimento in cui si ragiona "al contrario": non si animano spalla, braccio e mano ma, indicando al sistema che si vuole far posare la mano precisamente sulla maniglia, le ossa soprastanti si calcoleranno le pieghe, le flessioni del gomito e della spalla da sole per raggiungere matematicamente e nel modo più comodo possibile lo scopo della maniglia nello spazio 3D circostante.

**Implementazione e Spiegazione del Codice:**

```javascript
// src/people/Visitors.js
const d = clamp(_vV.length(), Math.abs(L1 - L2) + 1e-3, (L1 + L2) * 0.999);
const a = Math.acos(clamp((L1 * L1 + d * d - L2 * L2) / (2 * L1 * d), -1, 1));

_vPole.copy(FWD).addScaledVector(_vV, -_vV.dot(FWD));
```

Questo capolavoro di trigonometria vettoriale viene applicato individualmente alla gamba di ogni pedone a schermo centinaia di volte al secondo.
Viene in primo luogo calcolata rigidamente la lunghezza in scala `d` che sussiste fra il bacino umano e il pavimento 3D prescelto nel mondo in coordinate YXZ a cui mira l'allungo della gamba per atterrare col tallone ed equilibrare l'organismo. Usando i blocchi `clamp` congiuntamente alle dimensioni fisse di osso della coscia `L1` ed osso tibiale `L2`, il programma limita bruscamente tentativi disumani da parte del codice: se l'NPC avesse un target per qualsiasi errore a trenta metri dal busto, la coscia eviterebbe lo stretch brutale per cercare invano di ricoprire il divario strappandosi. 
Nel passaggio cardine della meccanica calcolatrice, si ricorre a nient'altro che una rigorosa decostruzione del celebre Teorema del Coseno. Trattando l'apparato anca-ginocchio-caviglia alla stregua di un triangolo scaleno con i tre lati precedentemente acquisiti in modo chiaro, si invoca la formula inserendola nella funzione `Math.acos` (Arcocoseno). Si spreme così in radianti palesi la spaccatura angolare precisa e indissolubile con cui flettere il ginocchio prima di posarlo ad accompagnamento per fare peso a terra con la gravità.

Infine, ad operare in sinergia collaterale c'è l'utilizzo del processo di estrazione di Gram-Schmidt vettoriale (nella variabile finale). Utilizzando dot product e manipolazioni del vettore in avanti (`FWD`), si direziona l'angolo e la base del muscolo ginocchio forzandolo ad assistere il tronco del corpo seguendo le correnti dell'inclinazione e spingendo logicamente la falcata dell'umanoide con un realismo biomeccanico impressionante e mai appoggiato da simulazioni false (come gli Export da Blender/Maya).