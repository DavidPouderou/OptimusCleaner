# Journal des modifications

## 1.7.0 — 13/10/2026 — mise à jour automatique

- À chaque lancement de l'interface, l'application vérifie en arrière-plan si une version plus
  récente est publiée sur le dépôt public (`DavidPouderou/OptimusCleaner`, page Releases). Si oui,
  elle la télécharge, contrôle l'empreinte SHA-256 annoncée par le manifeste, la signature
  Authenticode (qui doit être celle du certificat de l'auteur, compilé dans l'application) et la
  version inscrite dans le fichier, puis se remplace et se relance dès que l'accueil est libre.
  Pendant une opération, la nouvelle version attend en `.new` et est mise en place à la fermeture
  ou au lancement suivant. Jamais de version plus ancienne.
- Hors ligne, dépôt injoignable ou dossier en lecture seule : rien ne change, l'application démarre
  normalement. Le mode sans interface ne se met jamais à jour de lui-même.
- `--update` : vérifie et installe la dernière version sans interface. `--no-update` : désactive la
  recherche. Journal `mises-a-jour.log` dans le dossier des journaux et sur le support amovible.
- `release.ps1` : compile, signe, pose le tag, met à jour le dépôt public et crée la Release GitHub
  avec les exécutables, `SHA256SUMS.txt` et le manifeste `latest.json`.
- Description de l'exécutable dans ses propriétés Windows : « Maintenance Windows : … ».
- Le dépôt privé du code s'appelle désormais `OptimusCleaner-src` ; `OptimusCleaner` est le dépôt
  public des versions.


## 1.6.3 — 11/10/2026 — propriété, licence et signature

- **Licence** : `LICENCE.txt` (logiciel propriétaire, tous droits réservés). La mention
  « © 2026 David Pouderou. Tous droits réservés. » figure dans les propriétés de l'exécutable, le
  pied de page de l'accueil, `--version` et `--help` (source unique : `Directory.Build.props`).
- **Signature Authenticode automatique** : `publish.ps1` signe chaque exécutable avec le certificat de
  l'auteur (auto-signé, `CN=David Pouderou`, partie publique versionnée dans
  `assets/certificat-signature.cer`) et l'horodate chez DigiCert : preuve d'intégrité, d'auteur et de
  date. Publication refusée si la clé privée manque sur le poste ou si l'horodatage échoue ;
  `-NoSign` publie un exécutable non officiel, `-Thumbprint` désigne un autre certificat.
- Le projet est versionné sur GitHub (dépôt privé) ; les commits sont signés (GPG) et vérifiés.
  Les clés privées (`*.pfx`, `*.key`…) sont exclues du dépôt.


## 1.6.2 — 11/10/2026 — scans Windows Defender en arrière-plan

- Deux petits boutons sur la ligne « DÉSINFECTION » de l'accueil : **Scan rapide** et **Scan complet**
  Windows Defender. Le scan est lancé par `MpCmdRun.exe -Scan`, en processus détaché : il continue
  même si Optimus Cleaner est fermé ou la clé retirée. Le scan complet est fait pour être lancé en
  partant ; une confirmation rappelle sa durée.
- L'état du scan s'affiche sur la même ligne (en cours avec son chronomètre, puis terminé : aucune
  menace, ou menaces détectées — cliquable, ouvre Sécurité Windows). Un scan déjà en cours au
  lancement de l'application est repris à l'écran. Le poste ne s'endort pas pendant le scan tant que
  l'application est ouverte.
- **Preuve indépendante** : à la fin, l'application relit l'heure de fin que Defender a enregistrée
  pour ce type de scan (`Get-MpComputerStatus`) et l'affiche (« confirmé par Defender (00:38:57) »).
  Sécurité Windows n'affiche pas en direct un scan lancé en dehors de son propre bouton, et sa page
  « Dernière analyse » ne se rafraîchit qu'à la réouverture : sans cette confirmation, le technicien
  pouvait croire que rien ne s'était passé. Le journal Windows « Microsoft-Windows-Windows
  Defender/Operational » (événements 1000 et 1001) garde aussi chaque scan.
- Boutons grisés pendant une intervention ou quand un outil de désinfection est ouvert ; message
  clair si Defender est inactif (autre antivirus installé).


## 1.6.1 — 10/10/2026 — nettoyage : jamais de déconnexion, jamais d'installation cassée

Question de terrain : « le nettoyage ne doit effacer ni les sessions ouvertes ni rien d'important ».
Vérification dans le code puis sur les sources Microsoft et éditeurs : aucune session (navigateurs,
compte Microsoft, Office, Teams, OneDrive, Windows Hello) ne vit dans les dossiers nettoyés. Trois
risques rares restaient ; ils sont fermés.

### Ce qui change
- **Âge minimal des fichiers temporaires : 7 jours** (168 h) au lieu de 24 h, comme le Nettoyage de
  disque de Windows. Un document modifié directement depuis une archive zip (il atterrit dans Temp)
  n'est plus perdu le lendemain. `--temp-age` reste disponible.
- **Profils de connexion épargnés dans Temp** : les dossiers WebView2/Chromium d'un logiciel métier
  (`EBWebView`, `User Data`, `…UDF`, ou contenant `Cookies`, `Local State`, `Login Data`…) ne sont
  jamais vidés — l'utilisateur y resterait connecté. Les dossiers d'exécution d'une application ouverte
  (`.net`, `_MEI…`) non plus.
- **Dossiers Temp conservés quand un redémarrage est en attente**, comme le cache Windows Update :
  un installeur y laisse souvent ce dont il a besoin pour finir après le redémarrage, et l'âge des
  fichiers ne protège rien sur un poste qui n'a pas vraiment redémarré depuis des semaines
  (démarrage rapide). Le journal le dit et invite à redémarrer puis relancer.
- **Chemins attendus par une installation toujours épargnés** : la file `PendingFileRenameOperations`
  et les commandes `RunOnce`/`RunOnceEx` (machine, utilisateur courant, utilisateurs connectés) sont
  lues ; chaque fichier référencé est protégé avec le dossier qui l'entoure, même quand Windows ne
  signale aucun redémarrage.
- Le rapport compte les éléments protégés. Suite de tests : 364 tests.


## 1.6.0 — 09/10/2026 — fiche technique native, audit complet

Question de terrain : « sur certains PC j'ai une fiche technique ultra détaillée, et dès que j'en ai
vraiment besoin pour remplacer une pièce, l'application n'y arrive pas ». Cause : toute la fiche
dépendait d'un seul script PowerShell (150 s) ; lent, bloqué par stratégie ou planté sur un cmdlet,
tout partait avec lui. Puis un audit complet (interface, moteur, outils) : une cinquantaine de
corrections.

### Fiche technique
- **Collecte native, sans WMI ni PowerShell, en moins d'une seconde** : SMBIOS (barrettes avec
  référence exacte et fabricant, carte mère, BIOS, processeur et caches, châssis), disques
  (modèle, numéro de série décodé, bus, SSD/HDD, taille, température, **SMART ATA et journal NVMe** :
  usure, heures, démarrages, secteurs réalloués ou en attente, erreurs média, réserve), écrans
  (EDID : marque, modèle, diagonale, série, année), réseau (adaptateurs, IP, passerelle, DNS, DHCP),
  batterie (référence, capacité réelle, **cycles**), TPM, activation, clé OEM (table ACPI), Windows.
  Affichée immédiatement, sur tout poste.
- **PowerShell devient un complément en flux** : chaque section arrive sur sa propre ligne, les
  lentes en dernier. Un délai ou un blocage ne perd que les dernières sections, et la fiche dit
  lesquelles manquent. Une fiche vide n'est plus possible.
- Nouvelles alertes : secteurs défectueux, avertissement critique NVMe, réserve épuisée, erreurs
  média. Plus de fausse alerte « aucun antivirus » tant que le Centre de sécurité n'a pas répondu.

### Moteur
- Une étape bloquée (appel natif figé : VSS, disque mourant) est **abandonnée après 10 min de grâce**
  et l'intervention continue : l'interface ne reste plus jamais figée, le rapport est toujours écrit.
- Plafond d'étape distinct (8 h) : sur un poste lent, le contrôle final n'est plus coupé en plein DISM.
- DISM avec `/NoRestart` partout, entrée standard toujours fermée : plus de processus qui attend une
  touche. Observateur de sortie sérialisé et protégé. PowerShell bloqué = résultat vide, pas une
  exception.
- Réparation Windows Update : les services (cryptsvc, bits, wuauserv, DoSvc) sont **toujours
  redémarrés**, même si la réparation est interrompue ; annulation honorée après chaque utilitaire ;
  utilitaires courts stricts ; proxy WinHTTP lu par l'API (plus de texte de netsh à interpréter).
- Point de restauration : « liste illisible » (WMI cassé) n'est plus confondu avec « point absent ».
- SFC : « Repairing 0 components » n'est plus pris pour une réparation ; apostrophe typographique reconnue.
- Défragmentation : la fragmentation de l'espace libre ne déclenche plus rien ; média inconnu borné
  comme un disque dur ; **les disques USB/1394 ne sont jamais défragmentés** ; tout ignoré = avertissement.
- winget qui ne démarre pas = étape ignorée (pas « erreur inattendue ») ; PATH : répertoires absolus seulement.

### Outils de désinfection et lancement
- Téléchargement borné (en-têtes 30 s, 30 s de silence réarmé), transfert tronqué = nouvel essai :
  un Wi-Fi qui tombe ne fige plus la carte. Téléchargement en `.part` dans le dossier des outils,
  remplacement atomique. Proxy d'entreprise authentifié.
- Signature : chaîne complétée en ligne après téléchargement (racine absente d'un vieux Windows),
  jamais de révocation hors ligne ; fichier verrouillé par l'antivirus réessayé puis nommé ;
  quarantaine antivirus détectée.
- **La copie `%APPDATA%\ZHP` n'est lancée que si elle est identique au cache vérifié.**
- Script de relance en `-EncodedCommand` (plus de `.ps1` bloquable par GPO/AppLocker), limité à la
  session ; suivi d'un outil limité à la session.
- **Élévation** : `asInvoker` + relance `runas`. Invite UAC refusée → message clair, code 3.
  `--fiche`, `--help`, `--version` sans droits. Erreurs fatales et arguments invalides montrés en
  boîte de message. Seconde instance → fenêtre existante au premier plan. AppLocker/WDAC nommés.
- `longPathAware`, `--resume-tool` validé, code 10 documenté, `publish.ps1` refuse une publication
  qui ne tient pas en un seul fichier.

### Interface
- Fuite GDI à chaque redimensionnement corrigée ; journal sans limite (il se figeait après 32 Ko) ;
  presse-papiers terminé proprement.
- **Tient sur 1366×768 à 125 %** : hauteur minimale bornée par l'écran, accueil compressible (la
  description des outils et les puces cèdent avant les boutons). Fenêtre par défaut 900×720.
- Plus d'état « en cours » figé : tout le lancement est protégé, l'état ne s'éteint que sur le
  thread d'interface ; file d'événements non ré-entrante ; boîtes modales jamais ouvertes depuis
  la file ; travail de démarrage hors du thread d'interface.
- Clavier : **Échap** interrompt ou revient à l'accueil, **Entrée** revient à l'accueil depuis le résultat.
- Journal : défilement en lignes (on peut remonter lire sans être ramené en bas), sélection
  conservée pendant une intervention, troncature par moitié ; la fiche technique a son propre
  contrôle (le journal réapparaît intact).
- Fermeture pendant un redémarrage programmé : la question est posée. « Accueil » qui annule un
  redémarrage le dit. Boîte de la réparation aux libellés explicites (Oui / Non / Annuler).
- Barre basse simplifiée : plus de « Masquer le journal » ; « Ouvrir le rapport » ou « Dossier des
  rapports », « Ne pas fermer », « Accueil », « Redémarrer ». Alerte critique de la fiche avant
  « redémarrage en attente ». Bandeau sur une à trois lignes. Animations à 25 i/s uniquement quand
  quelque chose bouge. Glyphes cohérents, badges à la largeur du texte, bordure visible, thème
  sombre des barres de défilement.
- Suite de tests : 355 tests.


## 1.5.1 — 15/09/2026 — nettoyage plus prudent

Question de terrain : « 1,9 Go libérés, c'est énorme, ça supprime quoi ? ». Réponse : rien
d'important (temporaires, cache Windows Update, vidages, rapports d'erreurs, journaux), mais trois
réglages étaient plus agressifs que nécessaire pour un nettoyage qui doit rester invisible.

### Ce qui change

- **Âge minimal des temporaires : 24 h** au lieu de 1 h (`--temp-age` inchangé pour forcer).
  C'est le défaut de CCleaner ; Nettoyage de disque attend 7 jours. Une installation longue ou un
  fichier ouvert depuis le matin n'est plus touché.
- **Les pièces de diagnostic sont conservées 30 jours** : vidages de plantage (Minidump,
  LiveKernelReports, CrashDumps, MEMORY.DMP), rapports d'erreurs Windows (archive et file
  d'attente), journaux CBS et DISM. Un écran bleu récent reste analysable ; les archives CBS de
  plusieurs Go des postes négligés partent toujours.
- **Cache de téléchargement Windows Update conservé si TrustedInstaller travaille** (même sonde
  que la réparation Windows Update). Le vider pendant une installation faisait échouer la mise à
  jour, qui se retéléchargeait.
- **Caches de miniatures et d'icônes de l'Explorateur : plus touchés.** C'était le seul effet
  visible pour l'utilisateur (vignettes recalculées pendant quelques secondes) pour quelques dizaines
  de Mo. Le dossier `Windows\Explorer` sort de la liste blanche.
- Le journal indique les deux âges appliqués (temporaires / diagnostic).
- Toujours rien côté navigateurs : ni cache, ni cookies, ni sessions (règle inchangée, couverte
  par les tests).


## 1.5.0 — 14/09/2026 — un processus lent n'est plus coupé

Question de terrain : sur un vieux poste, DISM ou SFC peuvent dépasser leur délai sans être
plantés, et l'application les arrêtait. Pire pour winget : l'installeur en cours était tué avec
lui, avec une application à moitié installée à la clé.

### Ce qui change

- **Le délai d'une commande devient indicatif.** Passé ce délai, l'application vérifie toutes
  les 30 secondes que le processus donne signe de vie : une ligne de sortie, du temps processeur
  ou des entrées-sorties dans tout son arbre (objet travail Windows, donc winget *et* ses
  installeurs), ou le service qui travaille pour lui (TrustedInstaller pour DISM et SFC, Windows
  Installer pour winget). Tant qu'il travaille, on le laisse finir.
- **Un processus réellement figé est quand même arrêté** : après 30 minutes sans aucune activité
  (5 minutes pour une commande courte), ou au **plafond absolu de 4 heures**. La réparation
  lancée « puis on part » se conclut donc toujours, et l'application se ferme seule.
- **Poste lent, délais doublés d'office** : disque dur mécanique ×2, moins de 6 Go de mémoire
  ×1,5, deux cœurs ou moins ×1,5 (plafonné ×4), avec la raison dans le journal. Rien à régler
  sur place ; `--timeout-scale` reste disponible en plus.
- **Ce qui reste strict, volontairement** : la défragmentation d'un disque dur (budget de 30 min,
  Windows poursuit) et la collecte de la fiche technique (150 s, elle doit rendre la main).
- Le journal dit clairement ce qui s'est passé : « délai indicatif de 80 min dépassé : le
  processus travaille encore, on le laisse finir », puis la durée réelle ; ou, en cas d'arrêt,
  la raison exacte (« aucune activité depuis 30 min », « plafond de 4 h atteint »).
- Suite de tests : 305 tests, dont de vrais processus lents, figés et plafonnés.


## 1.4.0 — 14/09/2026 — nettoyage et winget séparés, rapports lisibles

Deux retours de terrain.

- **Deux boutons au lieu d'un.** Le nettoyage et l'optimisation disque prennent quelques minutes,
  winget peut en prendre trente : les enchaîner obligeait à attendre. La carte de gauche devient
  **« Nettoyage et optimisation »** (bouton principal : temporaires, rapports d'erreurs, caches,
  DNS, TRIM / défragmentation ciblée) avec un second bouton **« Mettre à jour les applications
  (winget) »**. En ligne de commande : `--profile clean` et `--profile update` (winget seul).
- **Rapport HTML lisible par intervention** (`OptimusCleaner_<date>_<profil>.html`, aussi sur la
  clé) : verdict en tête, étapes en tableau coloré avec durées et détails, journal complet replié
  en bas avec avertissements et erreurs surlignés, journal ouvert d'office s'il y a des erreurs.
  Aucune ressource externe, aucun script : un double-clic depuis la clé suffit, et il s'imprime.
  En fin d'intervention, le bouton **Ouvrir le rapport** l'affiche ; le bouton **Journaux**
  s'appelle désormais **Rapports** et ouvre le dossier de la clé quand il y en a une.
- La ligne « dernière exécution » de chaque carte indique de quel bouton elle parle
  (« Nettoyage 14/09 18:38 · … ») et retient la plus récente des deux.
- Suite de tests : 289 tests (dont deux qui verrouillent la règle « l'application ne ferme jamais un outil de désinfection »).


## 1.3.0 — 14/09/2026 — « État des lieux » devient « Fiche technique du PC », et s'affiche vraiment

Retour terrain sur un Windows 10 en intervention : le bouton n'affichait rien.

### Ce qui se passait

Le script de collecte écrivait `[null]` pour tout bloc de liste en échec (`@($null)` en
PowerShell) — par exemple `Get-BitLockerVolume`, absent de Windows 10 Famille. Le premier élément
nul faisait planter la mise en forme avant le premier caractère affiché, et l'écran restait vide.
Ce poste-ci (Windows 11 Pro) a tous les modules : le défaut ne s'y voyait jamais. Par ailleurs, le
bouton restait grisé tant que la collecte (jusqu'à 90 s sur un poste lent) n'était pas terminée.

### Ce qui change

- **Renommage** : « Fiche technique du PC » (bouton, titre, fichier `Fiche-technique_<poste>_<date>.txt`,
  option `--fiche`, alias `--inventory` conservé).
- **Plus jamais vide** : le script ne produit plus d'élément nul (`Get-List`), la lecture retire
  ceux qui resteraient, et chaque section est isolée : une donnée inattendue n'efface que sa
  section, avec la raison écrite à sa place. La fiche s'ouvre à tout moment et se complète toute
  seule quand la collecte se termine (délai porté à 150 s pour les postes lents).
- **Beaucoup plus d'informations**, pour commander une pièce ou reprendre le dossier sans revenir :
  - **réseau** : nom du poste, groupe de travail ou domaine, puis pour chaque adaptateur adresse
    IP et masque, IPv6 publique, passerelle, DNS et suffixe, DHCP (serveur, date du bail) ou
    adresse fixe, réseau connecté (SSID), profil public/privé, accès Internet ;
  - **comptes utilisateurs** : session ouverte à l'écran, comptes locaux (hors comptes techniques
    de Windows), administrateurs, comptes désactivés, dernière connexion, mot de passe non exigé ;
  - **imprimantes** réelles (les imprimantes virtuelles de Windows sont écartées), port, pilote ;
  - **démarrage automatique** : programmes lancés à l'ouverture de session et d'où ils viennent
    (une fois par programme, profils de service écartés) ;
  - adresse 169.254 reconnue comme « aucun serveur DHCP n'a répondu », passerelles lien-local masquées ;
  - **logiciels installés** avec version, éditeur et date, triés par nom ;
  - Windows : fuseau horaire, langue, dernières mises à jour installées (KB et date).
- La sortie PowerShell qui n'a rien produit est décrite (code, dernières lignes) au lieu d'un
  « aucun inventaire » muet.
- **Script de collecte transmis sur l'entrée standard de PowerShell** quand il dépasse la ligne
  de commande : enrichi, il atteignait 33 000 caractères encodés, au-delà des 32 767 que Windows
  accepte, et `CreateProcess` refusait de démarrer (« Nom de fichier ou extension trop long »).
  Aucun fichier temporaire sur le poste, et la stratégie d'exécution ne s'applique pas.
- Suite de tests : 280 tests.


## 1.2.0 — 14/09/2026 — retours du terrain : désinfection fiable, journaux sur la clé

Premier test en conditions réelles, en intervention : ZHPCleaner « se fermait tout seul » et
Optimus Cleaner disparaissait pendant AdwCleaner. Les deux ont la même cause, et ce n'est pas
un plantage.

### Ce qui se passait

| Constat | Cause |
|---|---|
| L'application se ferme pendant AdwCleaner | **Au moment du nettoyage, AdwCleaner ferme tous les programmes ouverts** (comportement documenté par Malwarebytes), Optimus Cleaner compris. Aucun journal de plantage : ce n'en est pas un. |
| ZHPCleaner se ferme avant la fin | Deux mécanismes. AdwCleaner ferme aussi ZHPCleaner s'il tourne en même temps. Et lancé depuis le cache, ZHPCleaner se recopie dans `%APPDATA%\ZHP`, affiche « Mise à jour » et repart de là : l'application prenait la fin du premier processus pour la fin de l'outil, et le bouton relançait la mauvaise copie. |
| Un outil pouvait être remplacé pendant qu'il tournait | Le téléchargement de fond renommait l'exécutable en cours d'exécution pour installer la nouvelle version, alors que le bouton « Ouvrir » était déjà actif. |
| Rien à relire après coup | Journaux, rapports et plantages restaient dans `C:\ProgramData` du poste ; la clé USB repartait vide. |

### Ce qui change

- **Un seul outil à la fois**, et aucune intervention tant qu'un outil est ouvert : les boutons se
  grisent, un bandeau dit pourquoi.
- **Relance automatique** : au lancement d'un outil, un script Windows PowerShell invisible attend
  sa fin ; si Optimus Cleaner a été fermé entre-temps, il le rouvre au premier plan avec le rapport
  de l'outil. Si l'application est toujours là, le script se termine sans rien faire. Un
  avertissement le dit avant de lancer AdwCleaner (enregistrer le travail en cours).
- **Rapport de l'outil** lu à sa fermeture *et* au démarrage de l'application (moins de 12 h) :
  résumé sur la carte (« 14:32 · Nettoyage : 3 éléments supprimés ») et bouton **Rapport** qui
  ouvre le fichier. Lecture par l'en-tête des journaux AdwCleaner (`# Mode`, `# Detected`,
  `# Cleaned`, `# Failed`) et par les comptes de section de ZHPCleaner : indépendant de la langue.
- **ZHPCleaner démarre depuis sa copie installée** : la copie vérifiée est posée dans
  `%APPDATA%\ZHP` avant le lancement (absente, plus ancienne ou non conforme). Plus de fenêtre
  « Mise à jour », plus de redémarrage de l'outil. La fin d'un outil est suivie par nom de
  processus, relances comprises.
- **Jamais de remplacement d'un exécutable en cours d'exécution** : la nouvelle version attend en
  `.new` et est appliquée à la fermeture de l'outil. Un fichier identique n'est pas réinstallé.
- **Outils vérifiés depuis moins de 12 h** : le réseau n'est pas sollicité de nouveau (relance sur
  le même poste instantanée) ; les deux outils s'acquièrent en parallèle.
- **Marqueur de session** : si la session précédente s'est terminée brutalement (fermée par un
  outil, processus tué, coupure), l'accueil le dit, avec ce qui était en cours, et un journal
  `session-interrompue` est écrit.
- **Journaux sur la clé USB** : quand l'application tourne depuis un support amovible (ou qu'un
  dossier `Logs` existe à côté de l'exe), journaux d'intervention, rapports JSON, états des lieux
  et journaux de plantage y sont recopiés, nommés avec le nom du poste. Le bouton **Journaux**
  ouvre la clé.
- **Fermeture pendant une intervention** : l'opération en cours est interrompue proprement, le
  rapport écrit, puis la fenêtre se ferme seule (un second clic force). Avant, DISM ou winget
  restaient orphelins et aucun rapport n'était écrit.
- **Veille** : le blocage de la mise en veille est relâché depuis le bon thread ; un poste sur
  lequel l'application restait ouverte après une réparation ne pouvait plus s'endormir.
- **chkdsk** : seul le code 3 planifie une correction au démarrage ; un code inattendu ne marque
  plus le volume « dirty ».
- Suite de tests : 250 tests (rapports d'outils, installation sous verrou, marqueur de session,
  relance, chkdsk, journaux portables).


## 1.1.0 — 23/08/2026 — point de restauration et état des lieux

### Point de restauration avant réparation

Nouvelle étape `restore-point`, placée **en tête** des profils « Réparation Windows » et
« Réparer Windows Update ». Elle ne fait jamais échouer une intervention : si le point est
impossible, elle avertit et la réparation continue. `--no-restore-point` la retire.

Quatre comportements de Windows sont traités, et c'est là que les outils du marché se trompent :

| Piège | Traitement |
|---|---|
| Un point créé moins de 24 h après le précédent est **ignoré en silence** | `SystemRestorePointCreationFrequency` mise à 0 le temps de la création, valeur d'origine restaurée dans un `finally` |
| Protection système désactivée (erreur 1058), fréquent sur les postes OEM | `Enable-ComputerRestore` puis nouvelle tentative ; la protection reste activée ensuite |
| VSS ou `swprv` désactivés par un « optimiseur » | Démarrage manuel rétabli avant l'essai |
| `SRSetRestorePointW` répond « réussi » sans rien créer | Vérification par relecture de la liste des points, et non par le code de retour |

Deux constats vérifiés sur Windows 11 25H2, contraires à la documentation : l'API renvoie un
**numéro de séquence nul** alors que le point est bien créé, et le point **n'apparaît pas
immédiatement** dans la liste. La vérification se fait donc par description, avec attente active.

### État des lieux du poste

Nouveau bouton **État des lieux** sur l'accueil, et option `--inventory [fichier]` pour un relevé
sans interface. Rapport texte copiable et enregistrable, en douze sections.

- **Alertes en tête** : disque en fin de vie (SMART), volume système presque plein, SSD usé,
  batterie usée, Windows 10 hors support, absence d'antivirus, signatures périmées, BitLocker
  actif, système sur disque dur mécanique, poste sur batterie. Une alerte critique remonte en
  bandeau rouge sur l'accueil.
- **Références pour commande** : modèle et numéro de série du châssis, carte mère, processeur et
  socket, chaque barrette (référence, type, format, fréquence, emplacements libres), chaque disque
  (modèle, série, interface, format M.2 / 2,5" / 3,5"), batterie (référence constructeur et usure),
  cartes graphiques, écrans (marque EDID traduite, diagonale ramenée à une taille du commerce).
- **Windows** : édition, activation et canal, clé OEM gravée dans l'UEFI, Product ID, TPM,
  BitLocker par volume, antivirus, compatibilité Windows 11 motivée.
- Ligne de synthèse matérielle sur l'accueil dès l'ouverture, et identité du poste en tête de
  chaque journal d'intervention.

Collecte en deux temps : registre et API Win32 (quelques millisecondes, affichés immédiatement),
puis un **unique** script PowerShell en arrière-plan (5 à 15 s) qui rend du JSON — `System.Management`
n'étant pas compatible Native AOT. Un poste où PowerShell est bloqué donne un inventaire partiel,
jamais une erreur.

### Corrections

- Les tailles s'expriment en unités du commerce : un SSD de 2 To ne s'affiche plus « 1862 Go », et
  32 Go de RAM ne s'affichent plus « 31,1 Go ». Les espaces libres restent en unités de
  l'explorateur Windows.
- `PowerShellRunner` partagé : script transmis en `-EncodedCommand`, et suppression des barres de
  progression, qui polluaient la sortie de fragments CLIXML et cassaient la lecture des résultats.
- Test de délai dépassé rendu déterministe (l'étape simulée durait exactement le plancher de
  `ScaleTimeout` : une chance sur deux d'échouer).


## 1.0.1 — 23/08/2026 — faux « redémarrage en attente »

Le bandeau orange « Redémarrage en attente » s'allumait sur un poste parfaitement sain.

`PendingFileRenameOperations` était traité comme un redémarrage de maintenance. Cette valeur est
remplie par `MoveFileEx` dès qu'un programme demande le renommage ou la suppression d'un fichier
verrouillé au prochain démarrage : un installeur, un antivirus, une mise à jour du Microsoft Store
ou une application Electron qui déballe un module natif dans `%TEMP%` la remplissent en
permanence. Aucun rapport avec une transaction de servicing.

La détection distingue maintenant les deux natures :

| Source | Effet |
|---|---|
| CBS `RebootPending`, `PackagesPending`, Windows Update `RebootRequired`, `PostRebootReporting` | Bandeau orange, `StartComponentCleanup` reporté, cache Windows Update conservé, réinitialisation WU refusée |
| `PendingFileRenameOperations` | Ligne d'information dans le journal, rien d'autre |

Conséquence : sur un poste dont seuls des fichiers sont en file d'attente, plus de bandeau, et le
nettoyage des composants comme la purge du cache Windows Update s'exécutent normalement au lieu
d'être sautés sans raison.


## 1.0.0 — 23/08/2026 — fusion de PCUpdater et de Nettoyage Optimus

Une seule application, un seul exécutable (~7,7 Mo, Native AOT), quatre actions claires.

### Ce qui change par rapport aux deux applications d'origine

| Avant | Maintenant |
|---|---|
| Nettoyage Optimus lançait AdwCleaner **automatiquement** au démarrage | **Plus aucun lancement automatique** : les deux outils sont installés en arrière-plan, le technicien les ouvre quand il veut |
| 6 puis 3 sections (Outils, Diagnostic, Rapport) | 3 blocs : **Désinfection**, **Mise à jour**, **Réparation Windows** (+ bouton Réparer Windows Update) |
| PCUpdater démarrait la maintenance dès l'ouverture | **Accueil** : rien ne se déclenche sans clic |
| PCUpdater installait les mises à jour Windows (interop COM WUA) | **Supprimé** — Windows s'en charge seul. Reste un bouton **Réparer Windows Update**, manuel |
| Réparation en plusieurs boutons, verdicts contradictoires | **Un bouton**, séquence complète avec escalade automatique et **verdict unique** |
| Le technicien devait rester devant | La réparation **ferme l'application toute seule** à la fin, redémarrage automatique optionnel |
| Défragmentation systématique (jusqu'à 4 h sur un vieux HDD) | Analyse préalable, seuil de 10 %, budget de 30 min |
| .NET Framework 4.7.2 WinForms (168 Ko) + .NET 10 AOT (4,5 Mo) | Un seul binaire .NET 10 Native AOT |

### Désinfection
- **AdwCleaner** et **ZHPCleaner** uniquement, téléchargés en arrière-plan au lancement depuis les
  sites officiels, hôte final contrôlé après redirections, signature Authenticode et identité du
  binaire vérifiées avant installation. Aucun outil n'est embarqué dans l'exécutable.
- Ouverture manuelle, avec l'interface de l'outil. À la fermeture, le rapport de l'outil est lu et
  résumé sur sa carte (`C:\AdwCleaner\Logs`, `%APPDATA%\ZHP`).
- ZHPCleaner : avertissement préalable (il ferme les navigateurs, crée un point de restauration) ;
  sa signature auto-signée est contrôlée sans exiger de chaîne de confiance.
- Retirés : Safety Scanner, KVRT, Malwarebytes résident, Autoruns, Process Explorer.

### Mise à jour et optimisation (automatique)
- winget : `--source winget` (msstore exclu : compte Microsoft requis, échecs silencieux),
  `--no-progress` avec repli pour les versions anciennes, `--include-unknown` retiré (il
  réinstallait en boucle), épinglages de l'utilisateur respectés et listés.
- Nettoyage étendu : tous les profils, WER, vidages de plantage, `MEMORY.DMP`, journaux CBS/DISM,
  cache d'optimisation de la distribution (API officielle), caches de miniatures (par motif),
  cache DNS. Services Windows Update arrêtés puis relancés le temps de vider leur cache.
- Disques : TRIM sur SSD ; sur HDD, `defrag /A` d'abord, défragmentation seulement au-delà du
  seuil et bornée dans le temps (interruption sans risque, Windows poursuit).

### Réparation Windows (un bouton, fermeture automatique)
- `DISM /RestoreHealth` → `SFC /scannow` → contrôle final → `chkdsk /scan`.
- Escalade automatique si une corruption réelle subsiste : `StartComponentCleanup` →
  `RestoreHealth` → `SFC` → nouveau contrôle.
- Verdict unique et lisible, écrit dans le rapport et réaffiché à l'accueil au lancement suivant.
- `chkdsk` en ligne ; si une réparation hors ligne est nécessaire, elle est **planifiée**
  (`fsutil dirty set`) au lieu d'imposer un `chkdsk /f /r` de plusieurs heures.
- Veille bloquée pendant l'opération ; redémarrage automatique optionnel (120 s, annulable).

### Réparer Windows Update (manuel)
Nouveau bouton, séquence sûre alignée sur le dépanneur officiel de Microsoft : pré-vol complet
(redémarrage en attente, installation CBS en cours, espace, réseau, poste géré, erreurs des
7 derniers jours), services désactivés rétablis, `w32tm /resync`, **Windows Update Medic**
(COM `Microsoft.WaaSMedic.1`), purge BITS, **renommage** de `SoftwareDistribution` et `catroot2`,
cache d'optimisation de la distribution, proxy WinHTTP conditionnel, redémarrage des services,
nouvelle détection par l'API COM documentée.
Jamais : suppression de dossiers, `catroot`, `pending.xml`, `PendingFileRenameOperations`,
désactivation de `WaaSMedicSvc`, stratégies WU d'un poste géré, `winsock reset`, `regsvr32`.

### Interface
Refonte complète : Fluent 2 (Windows 11), accent système, cartes arrondies anti-aliasées,
glyphes Segoe Fluent Icons, barre de progression et anneau d'activité dessinés, journal
monospace repliable, DPI par moniteur, largeurs de boutons calculées sur le texte réel.

### Corrections notables trouvées pendant le développement
- `activeCodePage=UTF-8` dans le manifeste faisait renvoyer 65001 à `GetOEMCP()` : toute la sortie
  de `defrag`, `DISM` et `chkdsk` était illisible (`r?optimisation`). Manifeste corrigé + repli
  registre `Nls\CodePage\OEMCP`.
- Le module DISM de PowerShell échoue sous PowerShell 7 (« Classe non enregistrée ») :
  l'application appelle explicitement `System32\WindowsPowerShell\v1.0\powershell.exe`, et un
  repli `dism.exe /ScanHealth /English` couvre le cas où le module est cassé.
- `--only` en mode graphique perdait ses étapes (écrasées par le profil) : 0 étape exécutée.
- Boutons tronqués (« Masquer le jo… ») : largeur désormais mesurée sur le texte.
- Cartes d'outils : la puce d'état chevauchait le bouton ; hauteur et espacement corrigés.

### Vérifié sur machine réelle (Windows 11 Pro 25H2, build 26200.9168)
TRIM SSD en 6 s · téléchargement + vérification de signature d'AdwCleaner 8.8.1.639 et de
ZHPCleaner 2026.8.15.26 · `IMAGESTATE=Healthy` via Windows PowerShell 5.1 · détection
WaaSMedic (`UsoCleanupPlugin`) · `dism /ScanHealth /English` (« No component store corruption
detected ») · nettoyage à blanc (15 emplacements, cache d'optimisation et DNS vidés) ·
125 tests unitaires.
