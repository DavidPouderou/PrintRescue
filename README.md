# PrintRescue

Une imprimante qui « bugue », et toujours la même routine à refaire à la main : purger le spouleur,
supprimer la file, retirer les pilotes, réinstaller proprement, vérifier. PrintRescue la joue en
cinq cases cochées, sans jamais rien casser : il dresse d'abord le bilan du poste, répare ce que
vous cochez après vous avoir montré son plan, puis consigne tout dans un journal et un rapport
d'intervention. Windows 10 et 11, éditions Famille comprises. Un seul fichier `.exe`, sans
installation.

## Téléchargement

Dernière version : [page Releases](https://github.com/DavidPouderou/PrintRescue/releases/latest).

| Fichier | Pour |
|---|---|
| `PrintRescue.exe` | Windows 10 et Windows 11, 64 bits (un seul fichier, environ 70 Mo) |

Aucune installation : enregistrer le fichier où l'on veut (clé USB, bureau) et le lancer.
L'outil demande les droits administrateur au lancement ; sans eux, il ne démarre pas.

Au premier lancement d'un fichier téléchargé, Windows peut afficher « Windows a protégé votre
ordinateur » : cliquer sur « Informations complémentaires », puis « Exécuter quand même ». Le
certificat de signature est auto-signé (voir plus bas) et Windows ne le connaît pas.

Le guide d'utilisation est dans [`README-utilisateur.md`](README-utilisateur.md).

## Mise à jour automatique

À chaque lancement, une fois le bilan affiché, l'outil vérifie en arrière-plan si une version
plus récente est publiée ici. Si c'est le cas, il la télécharge, contrôle son empreinte, sa
signature et sa version, puis :

- si rien n'a encore été fait sur le poste, il se remplace et redémarre aussitôt ;
- si une intervention est en cours, la nouvelle version est mise en place à la fermeture de
  l'outil et utilisée au lancement suivant : rien n'est interrompu.

Sans connexion, il démarre normalement. `--no-update` désactive la vérification pour un
lancement ; `"AutoUpdate": false` dans `PrintRescue.config.json` la désactive définitivement ;
`--update` l'exécute sans ouvrir la fenêtre. C'est la seule adresse Internet que l'outil
contacte ; il ne contacte jamais autre chose que les imprimantes du réseau local.

## Ce que fait l'outil

Une seule fenêtre, trois zones lues de haut en bas :

1. **Bilan** : constats classés du plus grave au moins grave (spouleur, imprimantes, pilotes
   constructeur, réglages qui bloquent l'impression, faux « hors connexion »…), chacun avec son
   bouton de correction ; liste des imprimantes avec état, page de test et menu d'actions.
2. **Réparer** : cinq cases — réinitialiser le spouleur, supprimer des imprimantes, supprimer
   les pilotes constructeur, réinstaller l'imprimante réseau en IPP avec le pilote Microsoft
   intégré, page de test — et un bouton. Chaque étape affiche son plan et attend confirmation.
3. **Journal** : tout ce qui a été constaté et fait ; rapport HTML d'intervention.

## Sécurité des opérations

1. Aucune action ne modifie le poste sans un plan affiché et confirmé ; les suppressions de
   pilotes et les modifications de registre exigent en plus la case « J'ai compris ».
2. Les pilotes Microsoft et intégrés à Windows ne sont jamais proposés à la suppression.
3. Avant toute suppression : point de restauration (si la Protection système le permet), export
   du registre d'impression en `.reg` et inventaire.
4. Aucune écriture de registre hors d'un catalogue fermé de réglages d'impression.
5. Les fichiers de spool ne sont jamais copiés ni ouverts ; les noms de documents ne sont jamais
   affichés ni écrits dans le journal.
6. Portable : rien n'est installé, aucune tâche planifiée, aucun service ; journal écrit à côté
   de l'exécutable.

## Vérifier un téléchargement

L'exécutable est signé (Authenticode, `CN=David Pouderou`) et horodaté par DigiCert. Le
certificat public est dans ce dépôt : `certificat-signature.cer`, empreinte SHA-1
`8F2D1E7BADDEE288B503D464997FD73DCAD6764A`. L'empreinte SHA-256 du fichier est dans
`SHA256SUMS.txt`, joint à chaque version.

```powershell
Get-AuthenticodeSignature .\PrintRescue.exe | Format-List Status, SignerCertificate, TimeStamperCertificate
Get-FileHash .\PrintRescue.exe -Algorithm SHA256
```

`UnknownError` signifie que la chaîne n'est pas approuvée (certificat auto-signé) et que le fichier
est intact ; `HashMismatch` signifie que le fichier a été modifié. L'outil applique les mêmes
contrôles, plus stricts encore, avant chaque mise à jour automatique.

## Signaler un problème

Par les [Issues](https://github.com/DavidPouderou/PrintRescue/issues) de ce dépôt, avec la
version (affichée dans le bandeau de la fenêtre), le dossier `Journal\<date>_<poste>\` de
l'intervention et, si possible, `PrintRescue.debug.log` (après avoir mis `"DebugLog": true` dans
`PrintRescue.config.json`). Le journal contient des noms de postes et d'imprimantes, jamais de
noms de documents : le relire avant de le joindre.

## Licence

Logiciel propriétaire, © 2026 David Pouderou, tous droits réservés : voir [LICENCE.txt](LICENCE.txt).
Le code source n'est pas distribué. Historique des versions : [CHANGELOG.md](CHANGELOG.md).
