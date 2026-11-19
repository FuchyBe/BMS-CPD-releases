# BMS Glass — versions publiées

F-16 CPD et F-15C Eagle PAD sur tablette, pour Falcon BMS 4.38 (ex-BMS-CPD).

- **Programme** : dossier `v<version>/`
  - `BMS-Glass-Setup-<version>.exe` : **installateur** (recommandé ; remplace BMS-CPD et reprend les réglages) ;
  - `BMS-Glass.exe` : **exe portable**, sans installation.
- **Serveur d'escadrille** : `BMS-Glass-Server-Setup-<version>.exe` (Windows, installateur) ; `bms-glass-server-linux-<version>.tar.gz` (Linux / VPS, avec `install.sh`) ; binaires `bms-glass-server-linux-amd64` / `-arm64`.
- **Manuels** : `manuel/Manuel-BMS-Glass.fr.pdf`, `manuel/Manuel-BMS-Glass-Server.fr.pdf` (autres langues : anciens manuels `Manuel-BMS-CPD.*.pdf` jusqu'à la version finale).
- **Mise à jour automatique** : BMS Glass lit `beta.json` dans ce dépôt et propose la nouvelle version dans sa page d'administration.
- **Format des couches tactiques** : `format/format-cpd.md`.

Ce dépôt ne contient que les fichiers publiés. Les versions 0.x (BMS-CPD) ont été retirées ; la première version publiée de BMS Glass est la 1.0.0-beta.2.
