# Journal des versions

## 0.3.4 — 2026-10-20 — Écran « Opération en cours »

- Après « Exécuter », rien ne montrait que l'outil travaillait : la confirmation se fermait et seul un petit anneau,
  en haut de la fenêtre, signalait l'activité, alors qu'un point de restauration ou un redémarrage du spouleur peut
  prendre plusieurs minutes. Il était tentant de recliquer.
- Désormais, dès la confirmation, un écran **« Opération en cours »** recouvre la fenêtre : animation, « Étape 2 sur 3 »,
  temps écoulé, et la liste de ce qui se fait, cochée au fil de l'exécution (✔ réussi, ✘ en échec, – non exécuté).
  Avec « Lancer la réparation », les étapes cochées sont listées et l'écran reste ouvert de l'une à l'autre.
- L'écran se ferme de lui-même une fois le bilan mis à jour ; une notification rappelle ensuite le résultat (orange si
  une étape a échoué). Tant qu'il est ouvert, aucun bouton n'est cliquable dessous : impossible de relancer une action
  par erreur ; fermer PrintRescue est aussi refusé, car cela interromprait l'action en cours (spouleur laissé arrêté,
  pilote à moitié retiré).
- Le bilan est refait dès qu'une étape a touché au poste, même si la séquence s'est arrêtée ensuite, et même si un bilan
  était déjà en cours au moment de l'action.
- Correction : après la suppression d'une imprimante, le bilan annonçait à tort **« Redémarrage de Windows en
  attente »**. Windows efface au démarrage suivant le cache de l'imprimante supprimée (`spool\V4Dirs`), ce qui ne
  demande pas de redémarrer. Seuls comptent désormais les fichiers de pilotes d'impression et du spouleur en attente,
  en plus de Windows Update et des mises à jour de composants.
- Tests : 91 unitaires (8 nouveaux).

## 0.3.3 — 2026-10-20 — Icône visible dans l'Explorateur

- `PrintRescue.exe` pouvait s'afficher avec une **page blanche à la place de son icône** dans l'Explorateur Windows
  (constaté sur une clé USB), alors que l'icône est bien dans le fichier. Quand l'Explorateur échoue une fois à lire
  l'icône d'un fichier, il mémorise cette page blanche pour ce chemin et cette date de modification, puis la ressert à
  chaque recopie du même fichier ; F5 n'y change rien. Les petites tailles de l'icône (16 à 48 px, celles des vues
  Liste et Détails) étaient compressées en PNG, format que Microsoft réserve à la taille 256 px.
- Correction : icône régénérée au format classique (bitmap non compressé de 16 à 48 px, PNG de 64 à 256 px), comme
  Optimus Cleaner. Même dessin. Cette version ayant une nouvelle date, l'Explorateur relit son icône.
- Si une ancienne version reste en page blanche sur une clé : la remplacer par celle-ci suffit (à défaut, renommer le
  fichier).
- Aucun changement de comportement. Tests : 83 unitaires, inchangés.

## 0.3.2 — 2026-10-15 — Libellés d'interface

- Le bouton de la zone Journal s'appelle désormais **Rapport d'intervention** (au lieu de « Rapport pour le client ») ;
  l'invite demande « Nom du site ou de l'organisation (facultatif) » et l'en-tête du rapport HTML indique
  « Organisation » à la place de « Client ». Le format du journal JSONL ne change pas.
- Même vocabulaire dans les autres textes visibles (bilan : « cas typique sur le poste », « réseau de l'organisation » ;
  rapport HTML : pied de page et recommandation réseau), dans le guide d'utilisation et dans les commentaires du code.
- Aucun changement de comportement. Tests : 83 unitaires, inchangés.

## 0.3.1 — 2026-10-14 — Nom de l'auteur visible, comme Optimus Cleaner

- Le bandeau de la fenêtre affiche « v0.3.1 · © 2026 David Pouderou » (mention lue dans les propriétés de l'exécutable,
  source unique `Directory.Build.props`), comme le pied de page d'Optimus Cleaner. Le rapport HTML porte la même mention
  en en-tête et en pied de page.
- Rappel : l'exécutable est signé Authenticode avec le certificat de l'auteur (`CN=David Pouderou`) et horodaté ; ses
  propriétés Windows indiquent « David Pouderou » et « © 2026 David Pouderou. Tous droits réservés. ». Inchangé.

## 0.3.0 — 2026-10-14 — Distribution par GitHub, signature et mise à jour automatique

- **Deux dépôts GitHub** (ADR-0006) : le code source reste privé (`PrintRescue-src`) ; le dépôt public
  `DavidPouderou/PrintRescue` publie l'exécutable signé, `SHA256SUMS.txt`, le manifeste `latest.json`, le guide
  d'utilisation, la licence et ce journal des versions.
- **Mise à jour automatique** : une fois le bilan affiché, l'outil vérifie en arrière-plan si une version plus récente
  est publiée. Si oui, il la télécharge à côté de lui, contrôle l'empreinte SHA-256 annoncée, la signature Authenticode
  (qui doit être celle du certificat de l'auteur, compilé dans l'outil) et la version inscrite dans le fichier. Si rien
  n'a encore été fait sur le poste, il se remplace et redémarre aussitôt ; si une intervention est en cours, la
  nouvelle version est installée à la fermeture et utilisée au lancement suivant. Jamais de version plus ancienne.
  Hors ligne, dépôt injoignable ou clé en lecture seule : rien ne change. `--no-update`, `"AutoUpdate": false` dans
  `PrintRescue.config.json`, `--update` (sans fenêtre). Journal `mises-a-jour.log` à côté de l'EXE.
- **Exécutable signé** (Authenticode, certificat de l'auteur `CN=David Pouderou`, horodatage DigiCert) ; identité de
  l'exécutable alignée (auteur, copyright) ; licence propriétaire `LICENCE.txt`.
- **Livraison** : `publish.ps1` (tests, un seul EXE, signature, empreinte) et `release.ps1` (manifeste, tag signé,
  dépôt public, Release GitHub) remplacent `tools\build.ps1` / `tools\sign.ps1`. Version unique dans
  `Directory.Build.props`. L'empreinte SHA-256 de chaque version est désormais dans `SHA256SUMS.txt` et `latest.json`
  joints à la Release.
- Tests : 82 unitaires (42 nouveaux) + 11 réels.

## 0.2.1 — 2026-10-08 — Premiers retours d'usage
- **Faux « Redémarrage de Windows en attente »** : la liste `PendingFileRenameOperations` est remplie en permanence par
  des composants sans rapport (services Xbox, antivirus) ; elle n'est désormais prise en compte que si elle concerne
  des fichiers d'impression ou de pilotes. Les deux indicateurs fiables (maintenance Windows, Windows Update) restent.
- **« Windows change l'imprimante par défaut tout seul »** : texte réécrit en français courant (ce que fait le réglage,
  le symptôme typique sur le poste, ce que fait le bouton « Corriger », comment revenir en arrière). Plus de jargon
  « valeur 1 » ni « origine probable » pour les réglages locaux.
- Tests : 40 unitaires (2 nouveaux) + 11 réels.
- Empreinte SHA-256 de `publish\win-x64\PrintRescue.exe` (build du 2026-10-08, 67,5 Mo) :
  A3F746732BA6513A22FFE369035520FC7CA10FFF2BA3E3143017282A74837A81.

## 0.2.0 — 2026-10-06 — Version simplifiée (ADR-0005)
- **Une seule fenêtre, trois zones** : Bilan (constats avec bouton de correction, liste des imprimantes), Réparer
  (cinq cases : spouleur, imprimantes, pilotes constructeur, réinstallation IPP, page de test, un bouton), Journal
  (entrées + rapport HTML). Plus de navigation, plus de page Outils ni Paramètres, aucun réglage à l'écran.
- **Réparer** : enchaînement des étapes cochées, chaque étape confirmée séparément ; un pilote encore utilisé devient
  cochable (et se coche tout seul) quand ses imprimantes sont cochées ; la page de test cible l'imprimante réinstallée.
- **Bilan** : constats reformulés en français de technicien ; faux « hors connexion » détecté sans SNMP (imprimante
  joignable + état SNMP activé sur le port TCP/IP → « Désactiver l'état SNMP ») ; imprimante injoignable signalée sans
  action côté PC ; imprimante par défaut proposée directement ; menu « Actions » par imprimante limité aux entrées
  pertinentes.
- **Élévation** : manifeste `requireAdministrator` (une invite UAC au lancement), plus de mode lecture seule.
- **Retirés** : SNMP, test direct IPP, Identify-Printer, impression directe et PDF minimal, page de test GDI (et la
  dépendance WinForms), inventaire des logiciels constructeur, vérification/réparation WMI, sfc, journal Operational,
  installation depuis INF, retrait manuel de périphérique PnP, mode simulation, thème, échelle, rapport Markdown.
- **Catalogue de réglages** réduit de 21 à 14 (ceux qui bloquent réellement l'impression sur un poste), avec des
  titres lisibles.
- **Tests** : 38 unitaires (10 nouveaux) + 11 réels, tous verts le 2026-10-06 sur la machine de développement.
- **Code** : 72 → 53 fichiers, 8 400 → 6 500 lignes ; moteur d'actions, adaptateurs Windows et actions inchangés.
- Publication : un seul EXE autonome x64 (67,5 Mo), non signé (voir docs/06). Bilan en 1,6 s sur la machine de développement.
- Empreinte SHA-256 de `publish\win-x64\PrintRescue.exe` (build final du 2026-10-06) :
  15ABDB30C85C4C9C99037315B8A7A2CB562A357D3162FE5C07A7519FE1128AD5 (à régénérer après chaque `tools\build.ps1`).

## 0.1.0 — 2026-09-27 — Première version complète (à éprouver sur le terrain)
- Bilan express : spouleur, imprimantes, pilotes (protection des pilotes Windows), ports (TCP/IP, WSD, IPP, USB),
  périphériques PnP, magasin de pilotes (packages tiers), stratégies bloquantes (catalogue de 21 réglages),
  logiciels constructeur, système (redémarrage en attente, profil réseau, WMI, Protection système, WPP),
  sondes réseau (ping, ports, IPP, SNMP), moteur de constats (20 fiches KB) avec actions.
- Réparer : sauvegarde préalable (point de restauration, exports .reg, inventaire), réinitialisation du spouleur,
  logiciels constructeur (lancement des désinstalleurs), suppression d'imprimantes (file, port orphelin, périphérique
  fantôme), suppression de pilotes et packages (jamais Microsoft), ajout IPP avec pilote Microsoft (repli TCP/IP),
  conversion WSD → TCP/IP, installation depuis INF, vérification (pages de test Windows et PrintRescue, suivi de la file,
  test direct IPP avec Identify-Printer et impression directe, imprimante par défaut pour l'utilisateur connecté,
  check-list), rapport HTML/Markdown.
- Outils : raccourcis Windows, découverte mDNS/WS-Discovery, sonde d'adresse, WMI (vérification/réparation), journal
  PrintService/Operational, sfc.
- Sécurité : moteur Plan → sauvegarde → confirmation (case « J'ai compris » pour les risques élevés) → exécution →
  vérification → journal ; mode simulation ; catalogue fermé des clés de registre modifiables ; aucune connexion Internet.
- Publication : un seul EXE autonome x64 (76 Mo), non signé (voir docs/06).
- Option avancée : retrait d'un moniteur de port ou d'un processeur d'impression constructeur (désactivée par défaut).
- Empreinte SHA-256 de `publish\win-x64\PrintRescue.exe` (build final du 2026-09-27, 76,1 Mo) :
  71816844B0AA45E007C882DAD5D30AFA91FDC990D00D50D982D8682938DADC13.

## 0.0.1 — 2026-09-27 — Phase 0 : cadrage
- Dossier de travail, journal de bord, brief v1.1, étude technique, architecture, spécification, questions/décisions,
  plan de tests, signature, base de connaissances, environnement, ADR 0001–0004, modèle de rapport.
