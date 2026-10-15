# Optimus Cleaner

Un seul exécutable pour remettre un PC Windows d'aplomb : désinfection, nettoyage, optimisation
des disques, mise à jour des applications, réparation du système et fiche technique du matériel.
Chaque opération part d'un clic, se déroule sous vos yeux et se termine par un rapport. Windows 10
et 11. Un seul fichier `.exe`, sans installation.

## Téléchargement

Dernière version : [page Releases](https://github.com/DavidPouderou/OptimusCleaner/releases/latest).

| Fichier | Pour |
|---|---|
| `OptimusCleaner.exe` | Windows 10 (version 1809 ou plus récente) et Windows 11, 64 bits |
| `OptimusCleaner-arm64.exe` | Windows sur processeur ARM64 |

Aucune installation : enregistrer le fichier où l'on veut (clé USB, bureau) et le lancer.
L'application demande les droits administrateur au lancement ; la fiche technique, l'aide et la
version fonctionnent sans.

Au premier lancement d'un fichier téléchargé, Windows peut afficher « Windows a protégé votre
ordinateur » : cliquer sur « Informations complémentaires », puis « Exécuter quand même ». Le
certificat de signature est auto-signé (voir plus bas) et Windows ne le connaît pas.

## Mise à jour automatique

À chaque lancement, l'application vérifie si une version plus récente est publiée ici. Si c'est
le cas, elle la télécharge en arrière-plan, contrôle son empreinte, sa signature et sa version,
puis se remplace et se relance d'elle-même dès que l'accueil est libre. Sans connexion, elle
démarre normalement. `--no-update` désactive cette vérification ; `--update` l'exécute sans
interface.

## Fonctionnalités

- **Désinfection** : AdwCleaner (Malwarebytes) et ZHPCleaner (Nicolas Coolman), téléchargés
  depuis les sites de leurs éditeurs après vérification de leur signature, lancés à la demande
  avec leur propre interface ; leurs rapports sont lus et résumés. Scan rapide ou complet de
  Windows Defender en arrière-plan.
- **Nettoyage** : fichiers temporaires de tous les profils, rapports d'erreurs Windows, vidages
  de plantage, journaux CBS/DISM, cache de téléchargement de Windows Update, cache d'optimisation
  de la distribution, cache DNS.
- **Optimisation des disques** : TRIM sur SSD et NVMe ; sur disque dur, analyse de la
  fragmentation puis défragmentation seulement au-delà de 10 %, limitée à 30 minutes.
- **Mise à jour des applications** : `winget upgrade --all`.
- **Réparation de Windows** : DISM `RestoreHealth`, `sfc /scannow`, contrôle de l'état
  d'intégrité, réparation approfondie si une corruption persiste, `chkdsk` en ligne, verdict
  final, redémarrage automatique en option.
- **Réparation de Windows Update** : point de restauration, vérifications préalables, services,
  heure système, Windows Update Medic, file BITS, renommage de `SoftwareDistribution` et
  `catroot2`, nouvelle recherche de mises à jour.
- **Fiche technique** : matériel (SMBIOS, SMART, EDID, batterie, TPM), réseau, Windows, comptes,
  imprimantes, démarrage automatique, logiciels installés.
- **Rapports** : un rapport HTML par opération, journaux texte et JSON.
- **Ligne de commande** : chaque opération est disponible sans interface, avec codes de sortie.

## Utilisation

L'accueil présente les outils de désinfection et les boutons Nettoyer et optimiser, Mettre à jour
les applications, Réparer Windows, Réparer Windows Update et Fiche technique. Aucune opération ne
démarre sans un clic. Chaque opération produit un rapport HTML, ouvert par le bouton en fin
d'opération ; les journaux sont dans `%ProgramData%\OptimusCleaner\Logs` et, si l'exécutable est
lancé depuis un support amovible, dans un dossier `Logs` à côté de lui.

Sans interface :

```bat
OptimusCleaner.exe --unattended --profile clean
OptimusCleaner.exe --unattended --profile update
OptimusCleaner.exe --unattended --profile repair --reboot
OptimusCleaner.exe --unattended --profile wu-reset --report C:\rapport.json
OptimusCleaner.exe --fiche fiche.txt
OptimusCleaner.exe --update
```

Codes de sortie : `0` succès · `3010` succès, redémarrage requis · `1` erreurs ·
`2` arguments invalides · `3` pré-requis non satisfaits · `4` annulé · `5` déjà en cours.
`--help` liste toutes les options.

## Sécurité des opérations

1. Les navigateurs ne sont pas touchés : ni cache, ni cookies, ni sessions, ni mots de passe, ni
   historique.
2. Le nettoyage se limite à une liste de dossiers temporaires connus. Prefetch,
   `C:\Windows\Installer`, la corbeille et `Windows.old` ne sont pas concernés.
3. Les jonctions ne sont pas suivies.
4. Les fichiers temporaires de moins de 7 jours sont conservés ; vidages de plantage, rapports
   d'erreurs et journaux CBS/DISM sont conservés 30 jours.
5. Les dossiers de profil de connexion et d'exécution d'une application ouverte, dans `Temp`, ne
   sont pas vidés.
6. Quand un redémarrage est en attente, les dossiers `Temp` et le cache Windows Update ne sont
   pas vidés.
7. Les outils tiers sont téléchargés depuis les sites officiels uniquement, avec contrôle de
   l'hôte final, de la signature Authenticode et de l'identité du binaire.
8. Aucun redémarrage n'est déclenché sans l'avoir demandé.

## Vérifier un téléchargement

Chaque exécutable est signé (Authenticode, `CN=David Pouderou`) et horodaté par DigiCert. Le
certificat public est dans ce dépôt : `certificat-signature.cer`, empreinte SHA-1
`8F2D1E7BADDEE288B503D464997FD73DCAD6764A`. Les empreintes SHA-256 des fichiers sont dans
`SHA256SUMS.txt`, joint à chaque version.

```powershell
Get-AuthenticodeSignature .\OptimusCleaner.exe | Format-List Status, SignerCertificate, TimeStamperCertificate
Get-FileHash .\OptimusCleaner.exe -Algorithm SHA256
```

`UnknownError` signifie que la chaîne n'est pas approuvée (certificat auto-signé) et que le fichier
est intact ; `HashMismatch` signifie que le fichier a été modifié. L'application applique les mêmes
contrôles, plus stricts encore, avant chaque mise à jour automatique.

## Signaler un problème

Par les [Issues](https://github.com/DavidPouderou/OptimusCleaner/issues) de ce dépôt, avec la
version (`--version`), le rapport HTML de l'opération et, si possible, le journal texte du dossier
`Logs`. Les rapports contiennent des noms de machines et des numéros de série : les relire avant
de les joindre.

## Licence

Logiciel propriétaire, © 2026 David Pouderou, tous droits réservés : voir [LICENCE.txt](LICENCE.txt).
Le code source n'est pas distribué. Historique des versions : [CHANGELOG.md](CHANGELOG.md).
