# 🔍 Forensique de disque avec Autopsy : analyse du NIST « Hacking Case »

![Domaine](https://img.shields.io/badge/Domaine-Forensique%20num%C3%A9rique-blue)
![Outil](https://img.shields.io/badge/Outil-Autopsy%204.19.3-orange)
![Format](https://img.shields.io/badge/Image-E01%20(EnCase)-lightgrey)
![Jeu de donn%C3%A9es](https://img.shields.io/badge/Dataset-NIST%20CFReDS-green)
![Plateforme](https://img.shields.io/badge/Plateforme-Windows-0078D6)

> Analyse forensique complète d'une image disque Windows XP issue du jeu de données public **NIST CFReDS – Hacking Case**, réalisée avec **Autopsy** : création du cas, ingestion, exploration des artefacts, identification du propriétaire du système, chronologie des événements et génération d'un rapport.

<!-- 📸 IMAGE : bannière ou capture d'ensemble de l'interface Autopsy avec le cas NIST ouvert -->
![Vue d'ensemble du projet](images/00-vue-ensemble.png)

---

## 📑 Sommaire

1. [Présentation du projet](#1--présentation-du-projet)
2. [Objectifs](#2--objectifs)
3. [Contexte du scénario](#3--contexte-du-scénario)
4. [Environnement et outils](#4--environnement-et-outils)
5. [Notions de forensique de disque](#5--notions-de-forensique-de-disque)
6. [Méthodologie](#6--méthodologie)
7. [Analyse détaillée des artefacts](#7--analyse-détaillée-des-artefacts)
8. [Reconstitution de l'activité (timeline)](#8--reconstitution-de-lactivité-timeline)
9. [Rapport et exports](#9--rapport-et-exports)
10. [Synthèse des résultats](#10--synthèse-des-résultats)
11. [Compétences démontrées](#11--compétences-démontrées)
12. [Bonnes pratiques et précautions](#12--bonnes-pratiques-et-précautions)
13. [Structure du dépôt](#13--structure-du-dépôt)
14. [Ressources et crédits](#14--ressources-et-crédits)

---

## 1. 📌 Présentation du projet

Ce laboratoire s'inscrit dans mon parcours de montée en compétences **Blue Team / DFIR** (*Digital Forensics and Incident Response*). Il consiste à analyser, avec **Autopsy**, l'image disque d'un ordinateur portable retrouvé abandonné, afin de déterminer **qui en était le propriétaire** et **à quoi il servait**.

Le projet couvre l'ensemble de la chaîne d'analyse :

- compréhension de ce qu'est une **image forensique** et de son mode d'acquisition ;
- **création et configuration d'un cas** dans Autopsy ;
- **ingestion** de l'image avec les modules d'analyse ;
- exploration des **fichiers**, des **artefacts de données** et des **résultats d'analyse** ;
- **annotation** des éléments intéressants (tags et commentaires) ;
- **reconstitution chronologique** de l'activité ;
- **génération d'un rapport** exploitable.

---

## 2. 🎯 Objectifs

| # | Objectif |
|---|----------|
| 1 | Comprendre le principe de la forensique de disque et son rôle dans une investigation |
| 2 | Installer et prendre en main Autopsy et FTK Imager |
| 3 | Créer un cas et y importer une image disque au format E01 |
| 4 | Identifier les **artefacts** pertinents : fichiers supprimés, historique web, cookies, programmes installés, comptes, e-mails |
| 5 | Identifier le **propriétaire** du système et ses outils |
| 6 | Reconstituer l'**activité** du système grâce à la timeline |
| 7 | Produire un **rapport** documentant les conclusions |

---

## 3. 🕵️ Contexte du scénario

Le jeu de données **Hacking Case** est fourni par le **NIST** (National Institute of Standards and Technology) via le portail **CFReDS** (*Computer Forensic Reference Data Sets*), conçu pour la formation, le test d'outils forensiques et la mise en pratique.

**Scénario :** le 20 septembre 2004, un ordinateur portable Dell CPi est retrouvé abandonné, accompagné d'une carte PCMCIA sans fil et d'une antenne 802.11b artisanale externe. La mission consiste à analyser le disque pour comprendre l'usage qui était fait de cette machine, et à identifier son propriétaire.

<!-- 📸 IMAGE : capture de la page du portail NIST CFReDS présentant le Hacking Case -->
![Portail NIST CFReDS](images/01-portail-nist-cfreds.png)
<img width="1365" height="668" alt="Capture d&#39;écran 2026-11-25 000910" src="https://github.com/user-attachments/assets/bb4ed33f-be07-4555-94fc-af1915021a05" />

---

## 4. 🛠️ Environnement et outils

| Élément | Détail |
|---|---|
| Système d'analyse | Windows 64 bits (virtualisé) |
| Outil principal | **Autopsy 4.19.3** |
| Acquisition | **FTK Imager** (démonstration de la création d'une image) |
| Source de données | Image **E01** du NIST Hacking Case (~600 Mo) |
| Système analysé | Windows XP (image datant de 2004) |
| Visualisation d'exports | Notepad++, navigateur web |

> 💡 **Pourquoi Windows ?** Autopsy fonctionne aussi sur Linux (avec The Sleuth Kit comme dépendance) et macOS, mais l'installation sous Windows est plus simple et offre de meilleures performances. Autopsy peut analyser une image issue de n'importe quel système d'exploitation.

<!-- 📸 IMAGE : capture de la page de téléchargement d'Autopsy (version 4.19.3) -->
![Téléchargement d'Autopsy](images/02-telechargement-autopsy.png)
<img width="1361" height="641" alt="Capture d&#39;écran 2026-11-25 001051" src="https://github.com/user-attachments/assets/a6d39bb5-d4ef-4b3d-9bcf-e8eee64136e5" />


<!-- 📸 IMAGE : capture de l'installation / des outils installés (Autopsy et FTK Imager) -->
![Outils installés](images/03-outils-installes.png)

---

## 5. 📚 Notions de forensique de disque

### 5.1 Qu'est-ce que la forensique de disque ?

C'est l'extraction d'informations forensiques depuis des supports de stockage : disques durs, clés USB, firmware, etc. Elle complète la **forensique mémoire** : la mémoire révèle les processus en cours et les commandes exécutées, tandis que le disque révèle les fichiers, l'historique, les programmes installés et les traces laissées dans le temps.

### 5.2 Image forensique

Une **image forensique** est une copie exacte, bit à bit, d'un disque et des données qu'il contient. L'acquisition ne doit **modifier aucune donnée** de l'original. Les formats courants sont :

- **RAW / DD** : copie brute ;
- **E01 (EnCase)** : standard de l'industrie pour stocker les images forensiques ;
- **AFF**, VMDK, etc.

### 5.3 Acquisition avec FTK Imager

Le processus d'acquisition présenté et pratiqué :

1. `File` → `Create Disk Image` ;
2. choix de la source : **disque physique** (cas d'un système compromis) ;
3. sélection du disque à copier ;
4. choix de la **destination** (obligatoirement un autre support, par exemple un disque externe) ;
5. choix du format (**E01** ou RAW) ;
6. renseignement des **métadonnées de preuve** : numéro de cas, numéro de preuve, description, nom de l'examinateur, notes ;
7. définition de la taille de fragmentation et du chiffrement éventuel.

<!-- 📸 IMAGE : FTK Imager, fenêtre « Create Disk Image » avec le choix du type de source -->
![FTK Imager - source](images/04-ftk-imager-source.png)

<!-- 📸 IMAGE : FTK Imager, fenêtre de saisie des informations de preuve (cas, preuve, examinateur) -->
![FTK Imager - informations de preuve](images/05-ftk-imager-preuve.png)

---

## 6. 🧭 Méthodologie

### 6.1 Création du cas dans Autopsy

1. `Create New Case` ;
2. saisie du **nom du cas**, du **répertoire de base** et du type de cas (*Single User*) ;
3. saisie du **numéro de cas** et des informations sur l'**examinateur** ;
4. ajout de la **source de données** : type *Disk Image or VM File* ;
5. sélection de l'image **E01** du NIST ;
6. configuration du **fuseau horaire** ;
7. sélection des **modules d'ingestion**.

<!-- 📸 IMAGE : écran « New Case Information » (nom du cas, répertoire de base, type de cas) -->
![Création du cas](images/06-creation-cas.png)

<!-- 📸 IMAGE : écran de sélection du type de source de données (Disk Image or VM File) -->
![Type de source de données](images/07-type-source.png)

<!-- 📸 IMAGE : écran de sélection du fichier image E01 et du fuseau horaire -->
![Sélection de l'image E01](images/08-selection-image-e01.png)

### 6.2 Modules d'ingestion

Les modules d'ingestion traitent l'image pour en extraire les informations exploitables. Pour un apprentissage complet, les modules ont été activés sur **tous les fichiers, répertoires et l'espace non alloué**. Principaux modules :

| Module | Rôle |
|---|---|
| **Recent Activity** | Extraction de l'activité récente de l'utilisateur (navigation web, etc.) |
| **Hash Lookup** | Identification de fichiers par leur empreinte (hash) |
| **Encryption Detection** | Détection de fichiers à forte entropie (chiffrement suspecté) |
| **Interesting Files Finder** | Repérage d'éléments d'intérêt (outils de confidentialité, chiffrement, portefeuilles crypto, stockage cloud) |
| **PhotoRec Carver** | Récupération de fichiers supprimés depuis l'espace non alloué |
| **Extension Mismatch Detector** | Détection d'extensions incohérentes avec le type réel |
| **Keyword Search** | Recherche de mots-clés et d'expressions régulières (e-mails, etc.) |
| **Virtual Machine Extractor** | Extraction de machines virtuelles |

> ⏱️ Le module **Plaso** a été identifié comme très long à exécuter : il peut être désactivé lors d'une première analyse.

<!-- 📸 IMAGE : écran de configuration des modules d'ingestion -->
![Modules d'ingestion](images/09-modules-ingestion.png)

### 6.3 Ouverture du cas et lecture de l'interface

Une fois l'ingestion terminée, Autopsy présente une arborescence structurée : **Data Sources**, **File Views**, **Data Artifacts**, **Analysis Results**, **OS Accounts**, **Tags**, **Reports**.

<!-- 📸 IMAGE : interface Autopsy avec l'arborescence complète à gauche -->
![Interface Autopsy](images/10-interface-autopsy.png)

---

## 7. 🔬 Analyse détaillée des artefacts

### 7.1 Structure du disque (Data Sources)

L'image contient **trois volumes** : un premier volume **non alloué**, puis un volume **NTFS** où Windows est installé. L'exploration de ce volume permet d'observer l'arborescence système et les dates de dernière modification, cohérentes avec le moment où l'ordinateur a été abandonné.

Autopsy fournit pour chaque fichier : **score** (élément notable ou non), **commentaire**, **nombre d'occurrences**, dates (modification, changement, accès, création), taille et emplacement. Le panneau inférieur permet de voir le contenu en **Hex**, **Texte**, **Strings**, **Application** et **Métadonnées**.

<!-- 📸 IMAGE : arborescence des volumes (non alloué + NTFS) de l'image -->
![Volumes de l'image](images/11-volumes-disque.png)

<!-- 📸 IMAGE : listing des fichiers avec le panneau inférieur (Hex / Text / File Metadata) -->
![Listing et panneau de détails](images/12-listing-fichiers.png)

### 7.2 Vues de fichiers (File Views)

Les fichiers sont classés **par extension** et **par type MIME** : images, vidéos, audio, archives, documents, exécutables.

- **Images, vidéos et audio** : exploration des médias présents, avec des fichiers encodés non lisibles directement.
- **Fichiers « carved »** : ces fichiers, dont le chemin contient la mention *carved*, sont des fichiers **supprimés** récupérés dans l'espace non alloué par le module PhotoRec. Ils n'ont plus de dates de modification, de changement ou d'accès, ce qui les distingue des fichiers normaux.
- **Archives** : présence d'outils typiques d'un profil « sécurité offensive » (voir [§10](#10--synthèse-des-résultats)).
- **Documents** (HTML, Office, PDF, texte) : certains documents supprimés contiennent de la documentation d'outils réseau et de sécurité.
- **Exécutables et scripts** : tri par colonne « notable » pour faire ressortir les binaires suspects.
- **Fichiers supprimés** : vue dédiée aux fichiers effacés du système de fichiers.

<!-- 📸 IMAGE : vue « File Views » → « By Extension » (images, archives, documents, exécutables) -->
![Vues par extension](images/13-file-views-extension.png)

<!-- 📸 IMAGE : exemple de fichier « carved » avec ses métadonnées (type carved, dates vides) -->
![Fichier carved](images/14-fichier-carved.png)

<!-- 📸 IMAGE : liste des archives contenant des outils (netcat, John the Ripper, nmap...) -->
![Archives et outils](images/15-archives-outils.png)

<!-- 📸 IMAGE : documents HTML supprimés (documentation d'outils réseau) -->
![Documents HTML supprimés](images/16-documents-html.png)

<!-- 📸 IMAGE : exécutables notables triés par colonne de score -->
![Exécutables notables](images/17-executables-notables.png)

<!-- 📸 IMAGE : vue des fichiers supprimés du système de fichiers -->
![Fichiers supprimés](images/18-fichiers-supprimes.png)

### 7.3 Extraction d'un fichier pour analyse

Autopsy permet d'**extraire** un fichier (clic droit → *Extract File(s)*) pour l'examiner avec d'autres outils. L'extraction d'une vidéo AVI et d'une image PNG a été testée.

> ⚠️ **Précaution :** extraire un fichier issu d'un système infecté sur la machine d'analyse peut l'infecter. À réaliser uniquement dans un environnement isolé.

<!-- 📸 IMAGE : menu clic droit « Extract File(s) » sur un fichier -->
![Extraction d'un fichier](images/19-extraction-fichier.png)

### 7.4 Artefacts de données (Data Artifacts)

#### 📧 Comptes de communication et e-mails
Des e-mails ont été retrouvés. L'un d'eux provient d'un utilisateur qui informe le développeur d'un outil de ping qu'il l'a porté vers VMS (MultiNet TCP/IP), ce qui montre que le propriétaire détenait des outils et de la documentation de développeurs.

<!-- 📸 IMAGE : artefact « E-Mail Messages » avec le contenu de l'e-mail (onglet Text) -->
![E-mails](images/20-emails.png)

#### 💽 Programmes installés
La liste des programmes installés révèle les outils présents à la date de l'image, notamment des analyseurs et scanners réseau (WinPcap, Ethereal, un scanner réseau) et un **service d'anonymisation**.

<!-- 📸 IMAGE : artefact « Installed Programs » -->
![Programmes installés](images/21-programmes-installes.png)

#### 🖥️ Informations sur le système d'exploitation
Les informations extraites du **registre Windows** (`system32\config`) indiquent la version du système (**Windows XP**), la date d'installation, l'identifiant du produit et surtout le **propriétaire déclaré de la licence : Greg Schardt**. C'est un élément central de l'identification, recherché systématiquement par les enquêteurs.

<!-- 📸 IMAGE : artefact « Operating System Information » avec le nom du propriétaire -->
![Informations OS](images/22-informations-os.png)

> 🏷️ Cet élément a été **annoté** avec un tag (*owner information*) et un commentaire, afin de le retrouver facilement et de le faire apparaître dans le rapport.

<!-- 📸 IMAGE : fenêtre d'ajout de tag et commentaire sur l'artefact du propriétaire -->
![Ajout d'un tag](images/23-tag-commentaire.png)

#### 📂 Documents récents, programmes exécutés, USB
- **Recent Documents** : documents récemment ouverts ;
- **Run Programs** : programmes lancés, dont un **anonymiseur** et un client **IRC** ;
- **Shell Bags** : traces de navigation dans les dossiers ;
- **USB Device Attached** : un périphérique USB a été connecté, avec son identifiant.

<!-- 📸 IMAGE : artefacts « Recent Documents », « Run Programs » et « USB Device Attached » -->
![Activité utilisateur](images/24-activite-utilisateur.png)

#### 🌐 Cookies, historique et recherches web
- **Web Cookies** : cookies d'Internet Explorer révélant plusieurs adresses e-mail associées au pseudonyme **Mr. Evil** ;
- **Web History** : historique de navigation de l'utilisateur ;
- **Web Search** : recherches Google de type « who am I / what's my IP », cohérentes avec une préoccupation d'anonymat.

<!-- 📸 IMAGE : artefact « Web Cookies » -->
![Cookies web](images/25-web-cookies.png)

<!-- 📸 IMAGE : artefacts « Web History » et « Web Search » -->
![Historique et recherches web](images/26-historique-recherches.png)

### 7.5 Résultats d'analyse (Analysis Results)

| Catégorie | Constat |
|---|---|
| **Encryption Suspected** | Fichiers à forte entropie signalés ; une forte entropie n'implique pas nécessairement du chiffrement (par exemple un fichier `.bin`) |
| **Extension Mismatch Detected** | Fichiers dont l'extension ne correspond pas au type MIME réel (par exemple un type Office sous une autre extension), technique courante pour éviter la détection |
| **Interesting Files** | Présence d'un élément identifié comme *possible zip bomb* |
| **Keyword Hits** | Adresses e-mail détectées par expression régulière |
| **OS Accounts** | Compte **Mr. Evil** (propriétaire), compte **Administrator** (SID se terminant par 500) et un compte de support |

<!-- 📸 IMAGE : « Encryption Suspected » avec la justification (forte entropie) -->
![Chiffrement suspecté](images/27-encryption-suspected.png)

<!-- 📸 IMAGE : « Extension Mismatch Detected » -->
![Extensions incohérentes](images/28-extension-mismatch.png)

<!-- 📸 IMAGE : « Keyword Hits » → « Email Addresses » -->
![Keyword hits](images/29-keyword-hits.png)

<!-- 📸 IMAGE : « OS Accounts » avec les comptes Mr. Evil, Administrator et support -->
![Comptes du système](images/30-os-accounts.png)

### 7.6 Tags et commentaires

Autopsy permet d'annoter les éléments (*follow-up*, *bookmark*, *notable*…). Ces annotations structurent l'enquête et alimentent directement le rapport final.

<!-- 📸 IMAGE : section « Tags » avec l'élément annoté « owner information » -->
![Tags](images/31-tags.png)

---

## 8. ⏳ Reconstitution de l'activité (timeline)

L'onglet **Timeline** fournit une vue chronologique détaillée de l'usage du système, sous trois formes : **Counts**, **Details** et **List**. En réduisant la plage temporelle (par exemple de 2003 à 2004), on observe les événements typés (*file modified*, *file accessed*…) à l'approche de la date de l'incident.

L'onglet **Discovery** permet de retrouver des fichiers selon des filtres (type, taille, source de données, occurrences passées, éléments intéressants, dossier parent, domaines). Les vues **Images/Videos**, **Communications** et **Geolocation** complètent l'analyse (aucune géolocalisation exploitable en 2004).

<!-- 📸 IMAGE : onglet Timeline, vue Counts / Details -->
![Timeline](images/32-timeline.png)

<!-- 📸 IMAGE : timeline en vue List sur une période restreinte -->
![Timeline en liste](images/33-timeline-liste.png)

<!-- 📸 IMAGE : onglet Discovery avec filtres de documents -->
![Discovery](images/34-discovery.png)

<!-- 📸 IMAGE : vue Communications -->
![Communications](images/35-communications.png)

---

## 9. 📄 Rapport et exports

### 9.1 Export de données au format CSV
Les tableaux de résultats peuvent être exportés (*Save table as CSV*). Les adresses e-mail identifiées ont été exportées en `emails.csv`, puis ouvertes dans Notepad++.

<!-- 📸 IMAGE : export « Save table as CSV » des adresses e-mail -->
![Export CSV](images/36-export-csv.png)

<!-- 📸 IMAGE : fichier emails.csv ouvert dans Notepad++ -->
![CSV dans Notepad++](images/37-csv-notepad.png)

### 9.2 Génération du rapport HTML
Un rapport **HTML** a été généré via `Generate Report` : nom *Hacking Case 2004*, pied de page *NIST Hacking Case 2004*, source de données sélectionnée, tous les résultats inclus (ou uniquement les résultats taggés dans un cas réel). Le rapport contient le numéro de cas, l'examinateur, les informations sur l'image, le résumé du cas, les résultats taggés, les fichiers intéressants et les artefacts extraits.

<!-- 📸 IMAGE : assistant « Generate Report » (nom du rapport, sélection du format HTML) -->
![Génération du rapport](images/38-generation-rapport.png)

<!-- 📸 IMAGE : page d'accueil du rapport HTML ouvert dans le navigateur -->
![Rapport HTML](images/39-rapport-html.png)

<!-- 📸 IMAGE : résumé du cas dans le rapport HTML (résultats taggés) -->
![Résumé du cas](images/40-resume-cas.png)

---

## 10. ✅ Synthèse des résultats

| Question | Réponse issue de l'analyse |
|---|---|
| **Quel système ?** | Windows XP, disque NTFS, image de 2004 |
| **Qui est le propriétaire ?** | **Greg Schardt**, propriétaire déclaré de la licence Windows XP ; compte utilisateur **Mr. Evil** |
| **Quels outils étaient présents ?** | Netcat 1.10, John the Ripper, Nmap, Pandora, client VNC, Cerberus Internet Scanner, LSADump, Ethereal et WinPcap, outils de reconnaissance et d'attaque |
| **Quelles traces d'anonymat ?** | Programme d'anonymisation, client IRC, recherches « who am I / what's my IP » |
| **Quelles traces de suppression ?** | Fichiers carved et documents HTML supprimés (documentation d'outils de sécurité) |
| **Quelles ruses de dissimulation ?** | Extensions modifiées (extension mismatch), fichiers à forte entropie |
| **Comptes** | Mr. Evil, Administrator (RID 500), compte de support |
| **Matériel externe** | Un périphérique USB connecté |

**Conclusion :** l'analyse montre un ordinateur utilisé par une personne disposant d'un **arsenal d'outils de reconnaissance, de cassage de mots de passe et de capture réseau**, avec une volonté d'**anonymat**, ce qui est cohérent avec le nom du cas, *Hacking Case*.

---

## 11. 🎓 Compétences démontrées

- **Forensique de disque** : notions d'image forensique, formats E01/RAW/DD, acquisition avec FTK Imager
- **Maîtrise d'Autopsy** : création de cas, ingestion, navigation, tags, timeline, rapports
- **Analyse d'artefacts Windows** : registre, comptes, programmes installés, historique web, cookies, USB, shell bags
- **Récupération de fichiers supprimés** (file carving)
- **Détection d'anomalies** : entropie élevée, extensions incohérentes, fichiers suspects
- **Corrélation et reconstitution chronologique** d'événements
- **Documentation et reporting** : exports CSV, rapport HTML, traçabilité des preuves
- **Rigueur forensique** : préservation de l'intégrité des preuves et précautions d'extraction

---

## 12. 🛡️ Bonnes pratiques et précautions

- Toujours travailler sur une **image forensique**, jamais sur le disque original ;
- Conserver l'image sur un **support distinct** de la source ;
- Documenter chaque preuve (numéro de cas, numéro de preuve, examinateur) ;
- **Extraire des fichiers suspects uniquement dans un environnement isolé** ;
- Utiliser les **tags et commentaires** pour garder la traçabilité de l'enquête ;
- Ne pas se limiter aux résultats automatiques : certaines analyses doivent être faites **manuellement** ;
- Désactiver les modules très coûteux (Plaso) lors d'une première passe pour gagner du temps.

---

## 13. 🗂️ Structure du dépôt

```text
autopsy-nist-hacking-case/
├── README.md
├── images/
│   ├── 00-vue-ensemble.png
│   ├── 01-portail-nist-cfreds.png
│   ├── ...
│   └── 40-resume-cas.png
└── rapport/
    └── hacking-case-2004.html   (optionnel : rapport généré)
```

---

## 14. 📎 Ressources et crédits

- **Autopsy** : [autopsy.com](https://www.autopsy.com)
- **FTK Imager** : [exterro.com](https://www.exterro.com)
- **NIST CFReDS** – *Hacking Case* : [cfreds.nist.gov](https://cfreds.nist.gov)
- **Série Blue Team Training** (HackerSploit, avec Linode) : démonstration suivie pour ce laboratoire, reproduite et documentée ici

---

### 👤 Auteur

**Dannon Rock** : Réseaux, systèmes et cybersécurité
🔗 [LinkedIn](https://www.linkedin.com/in/rock-dannon)

> ℹ️ *Ce dépôt est un projet d'apprentissage réalisé à partir d'un jeu de données public et pédagogique. Aucune donnée réelle n'est exploitée.*
