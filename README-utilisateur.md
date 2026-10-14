# PrintRescue — guide d'utilisation (v0.3)

Outil portable de dépannage d'impression pour Windows 10 et 11 (éditions Famille comprises). Un seul fichier :
`PrintRescue.exe`, téléchargé depuis la page [Releases](https://github.com/DavidPouderou/PrintRescue/releases/latest).
Aucune installation, aucune télémétrie ; la seule connexion Internet est la recherche de mise à jour (voir plus bas).
Le journal est écrit à côté de l'EXE (ou dans `%LOCALAPPDATA%\PrintRescue` si la clé USB est en lecture seule).

## Démarrage

1. Double-cliquer sur `PrintRescue.exe` et accepter la demande de droits administrateur (UAC). L'outil les exige :
   sans eux, il ne démarre pas. Au premier lancement d'un fichier téléchargé, Windows peut afficher « Windows a protégé
   votre ordinateur » : « Informations complémentaires », puis « Exécuter quand même » (certificat auto-signé de l'auteur).
2. Le **bilan** se lance tout seul (2 à 5 secondes). Tout tient dans une seule fenêtre, à lire de haut en bas.

## Mise à jour automatique

Une fois le bilan affiché, l'outil vérifie en arrière-plan si une version plus récente est publiée sur la page
Releases. Si oui, un bandeau l'indique en haut de la fenêtre pendant le téléchargement (vous pouvez continuer à
travailler), puis :

- si **rien n'a encore été fait sur le poste**, l'outil se remplace et redémarre aussitôt avec la nouvelle version
  (bandeau « PrintRescue a été mis à jour ») ;
- si une intervention est en cours (une action ou une sauvegarde a déjà été faite), la nouvelle version est installée
  **à la fermeture** de l'outil et utilisée au lancement suivant : rien n'est interrompu, le journal reste entier.

Avant de remplacer l'exécutable, l'outil contrôle l'empreinte du fichier, sa signature (qui doit être celle de l'auteur)
et sa version ; une version plus ancienne n'est jamais installée. Sans connexion, dépôt injoignable ou clé en lecture
seule : rien ne change, l'outil démarre normalement. Pour désactiver : lancer avec `--no-update` (une fois) ou mettre
`"AutoUpdate": false` dans `PrintRescue.config.json` (définitif). `PrintRescue.exe --update` vérifie et installe la
dernière version sans ouvrir la fenêtre. Le détail est consigné dans `mises-a-jour.log`, à côté de l'EXE.

## La fenêtre

### 1 · Bilan

- Une ligne d'en-tête : nombre de problèmes et d'avertissements.
- La liste des **constats**, du plus grave au moins grave. Quand une correction existe, un bouton est à droite
  (« Réinitialiser le spouleur », « Remettre en ligne », « Désactiver l'état SNMP », « Corriger », « Réinstaller en IPP »…).
  Le bouton affiche d'abord ce qui va être fait, puis attend votre confirmation.
- La liste des **imprimantes** : état (Prête, Hors connexion, En pause, Erreur), pilote, type de port et adresse.
  « Page de test » imprime la page de test Windows. « Actions » ouvre un menu adapté à l'imprimante : par défaut,
  remettre en ligne, reprendre, purger, convertir le port WSD en TCP/IP, ouvrir la file, page web, supprimer.

### 2 · Réparer — la routine en cinq cases

1. **Réinitialiser le spouleur** : arrêt, purge des travaux bloqués, démarrage automatique, redémarrage.
2. **Supprimer des imprimantes** : cocher celles à retirer. File purgée, port orphelin et périphérique fantôme nettoyés.
3. **Supprimer les pilotes constructeur** : cocher ceux à retirer (pilote + package). Les pilotes Windows ne sont jamais
   proposés. Un pilote encore utilisé devient cochable dès que ses imprimantes sont cochées à l'étape 2 ; il se coche alors
   tout seul (décochable).
4. **Réinstaller l'imprimante réseau en IPP** avec le pilote Microsoft intégré : adresse IP pré-remplie, bouton
   « Trouver sur le réseau » si vous ne la connaissez pas.
5. **Page de test** sur l'imprimante choisie (ou sur celle qui vient d'être réinstallée).

Cochez, puis **Lancer la réparation**. Chaque étape affiche son plan et attend votre confirmation ; si vous annulez
une étape, la séquence s'arrête. Avant toute suppression (étapes 2 et 3), l'outil crée un point de restauration (si la
Protection système le permet), exporte le registre d'impression en `.reg` et enregistre un inventaire. Le bilan est
relancé à la fin.

### 3 · Journal

Tout ce qui a été constaté et fait, avec l'heure et le résultat. **Rapport pour le client** demande le nom du client
puis ouvre un rapport HTML (imprimable en PDF). **Ouvrir le dossier** montre les fichiers de l'intervention.

### Menu « Windows » (en haut à droite)

Raccourcis : Imprimantes et scanners, Périphériques et imprimantes, assistant « Ajouter une imprimante », propriétés
du serveur d'impression (pilotes), Gestionnaire de périphériques, Services, Applications installées (pour désinstaller
un logiciel constructeur).

## Règles de sécurité

- **Aucune action ne modifie le poste sans un plan affiché et confirmé.** Les suppressions de pilotes et les
  modifications de registre exigent en plus la case « J'ai compris ».
- **Les pilotes Microsoft et intégrés à Windows ne sont jamais proposés à la suppression.**
- **Réseau** : l'outil ne contacte que les imprimantes du réseau local et, pour la mise à jour, la page Releases du
  dépôt public GitHub (`github.com`). Rien d'autre, jamais.
- **Documents des clients** : les fichiers de spool ne sont jamais copiés ni ouverts ; les noms de documents ne sont
  jamais affichés ni écrits dans le journal.

## Ce que fait l'outil sur le poste (pour une mise en liste blanche antivirus)

- Lecture : API du spouleur (winspool), registre, SetupAPI, journaux d'événements, `pnputil /enum-drivers`,
  sondes réseau locales (ping, ports 9100/631/515/80/443, requête IPP).
- Sur confirmation : arrêt/démarrage du service Spouleur d'impression, terminaison de `spoolsv.exe`,
  `PrintIsolationHost.exe`, `splwow64.exe`, suppression des fichiers du dossier de spool, suppression d'imprimantes,
  de ports, de pilotes et de packages (API winspool, repli `pnputil`), écriture de valeurs de registre listées dans le
  catalogue de l'outil (14 réglages d'impression + ports TCP/IP + imprimante par défaut), création de point de
  restauration, `reg.exe export`, `powershell.exe Add-Printer -IppURL` (ajout d'une file IPP), `rundll32 printui.dll`
  (page de test, boîtes de dialogue Windows).
- Mise à jour : une requête HTTPS vers `github.com` (manifeste `latest.json`), puis, s'il existe une version plus récente,
  téléchargement de `PrintRescue.exe` dans le dossier de l'outil (`PrintRescue.exe.new`), renommage de l'ancien en
  `PrintRescue.exe.old` (effacé au lancement suivant). Rien hors de ce dossier.

## Fichiers produits

`Journal\<date_heure>_<poste>\` : `journal.jsonl` (toutes les entrées), `sauvegarde-registre\*.reg`,
`inventaire-avant.json`, `rapport-intervention.html` après export. `mises-a-jour.log` : historique des mises à jour de
l'outil (versions, dates, raisons), sans donnée client.

## Vérifier l'exécutable

`PrintRescue.exe` est signé par l'auteur (certificat `CN=David Pouderou`, auto-signé, horodaté par DigiCert). Dans
PowerShell : `Get-AuthenticodeSignature .\PrintRescue.exe` → `UnknownError` = signature intacte mais certificat non
reconnu par Windows (normal), `HashMismatch` = fichier modifié, `NotSigned` = pas une version officielle. L'empreinte
SHA-256 de chaque version est dans `SHA256SUMS.txt` sur la page Releases.

## Réglages (facultatif)

Aucun écran de réglages. Pour les cas particuliers, un fichier `PrintRescue.config.json` à côté de l'EXE est lu s'il
existe :

```json
{ "Technician": "Nom affiché dans le rapport", "DebugLog": false, "ProbeNetwork": true, "NetworkTimeoutMs": 1200, "JournalFolder": "", "AutoUpdate": true }
```

## En cas de problème avec l'outil lui-même

Mettre `"DebugLog": true` dans le fichier ci-dessus, reproduire, puis consulter `PrintRescue.debug.log`.
Un plantage est toujours consigné dans `PrintRescue.crash.log`.
