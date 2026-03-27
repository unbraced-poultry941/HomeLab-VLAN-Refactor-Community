# Homelab VLAN Refactor — Community Edition

> **Inspiré de la série YouTube iMot3k** :  
> *"Je change TOUT mon réseau, et j'ai encore tout pété. 🤷‍♂️"*  
> Ce dépôt en est la transcription structurée, reproductible et documentée pour la communauté.

---

## Table des matières

1. [Résumé exécutif](#1-résumé-exécutif)
2. [Pourquoi ce projet existe](#2-pourquoi-ce-projet-existe)
3. [Architecture cible](#3-architecture-cible)
4. [Plan d'adressage IP (VLAN)](#4-plan-dadressage-ip-vlan)
5. [Matériel actuel vs cible](#5-matériel-actuel-vs-cible-budget-1000-)
6. [Structure du dépôt](#6-structure-du-dépôt)
7. [Comment démarrer](#7-comment-démarrer-)
8. [Ordre de migration recommandé](#8-ordre-de-migration-recommandé)
9. [Risques et rollback](#9-risques-et-rollback)
10. [Contribuer](#10-contribuer)
11. [Licence](#11-licence)

---

## 1. Résumé exécutif

Ce dépôt propose un guide complet de reconstruction "from scratch" d'une infrastructure réseau
homelab, directement inspiré de la démarche documentée par la chaîne YouTube **iMot3k**.

L'objectif central est d'abandonner définitivement le réseau plat historique (tout sur VLAN 1,
le fameux "VLAN Merdouille") pour une architecture segmentée, sécurisée, performante
et surtout **maintenable dans le temps**.

Ce projet couvre de bout en bout : la création d'un agrégat de liens LACP entre le pare-feu et
le switch cœur, le déploiement d'un plan IP cohérent sur 6 VLANs fonctionnels, la centralisation
du DHCP et du DNS sur Windows Server 2022, le Wi-Fi multi-profil avec PPSK (sans multiplier
les SSID), la gestion du mDNS/multicast inter-VLAN, et la sécurisation des sauvegardes NAS via
rsync automatisé.

Ce "**Golden Path**" communautaire privilégie des configurations simples, lisibles, commentées
et reproductibles — même si certains choix techniques peuvent sembler sous-optimaux pour un
environnement de production d'entreprise. L'objectif est l'apprentissage et la montée en
compétences, pas la perfection opérationnelle.

---

## 2. Pourquoi ce projet existe

Un homelab typique démarre avec une box FAI, un switch non manageable et un Wi-Fi basique.
Tout est sur le même réseau, tout se parle, les imprimantes voient les serveurs, les caméras
voient les PC, et les appareils IoT douteux ont accès à tout. C'est pratique au début.
C'est une bombe à retardement ensuite.

Les problèmes classiques que ce projet cherche à résoudre sont les suivants :

- **Sécurité nulle** : un appareil IoT compromis a accès à l'ensemble du réseau.
- **QoS impossible** : la VoIP et les flux vidéo se battent avec les téléchargements.
- **Dépannage cauchemardesque** : impossible de savoir quel appareil fait quoi sur un réseau plat.
- **Sauvegardes non fiables** : pas de plan de backup structuré ni automatisé.
- **Wi-Fi ingérable** : un SSID par usage = prolifération incontrôlable de réseaux sans fil.

Ce dépôt fournit **la réponse structurée** à chacun de ces problèmes, avec des runbooks
pas-à-pas, des fichiers de configuration commentés et des scripts d'automatisation prêts à l'emploi.

---

## 3. Architecture cible

```text
Internet (FAI)
     │
     ▼
┌─────────────────────────┐
│   FortiGate 60F         │  ← Routage inter-VLAN, NAT, DHCP Relay,
│   (Pare-feu / Routeur)  │    Filtrage applicatif, mDNS Proxy
└────────────┬────────────┘
             │ LAG LACP (2x 1Gb/s → port-channel)
             ▼
┌─────────────────────────┐
│  Cisco Catalyst 2960X   │  ← Switch L2 cœur, trunk multi-VLAN,
│  (Switch cœur 48 ports) │    VLAN natif blackhole (999), PoE
└──────┬──────────────────┘
       │
  ─────┴──────────────────────────────────
  │              │            │          │
VLAN 10       VLAN 50      VLAN 20    VLAN 30
ADMIN         PC           IoT        VoIP
  │
  ├── Windows Server 2022 (DHCP/DNS/AD-DS)
  ├── Proxmox VE (hyperviseur)
  └── UniFi Network Controller (VM)
       │
       └── 4x UniFi U7 Lite (Wi-Fi PPSK → VLAN 20/50/200)
```

---

## 4. Plan d'adressage IP (VLAN)

| VLAN ID | Nom          | Sous-réseau       | Passerelle      | Usage principal                        |
| :-----: | :----------- | :---------------- | :-------------- | :------------------------------------- |
| 10      | ADMIN        | 10.20.10.0/24     | 10.20.10.254    | Serveurs, hyperviseurs, équipements réseau |
| 20      | IOT          | 10.20.20.0/24     | 10.20.20.254    | Appareils connectés, domotique         |
| 30      | VOIP         | 10.20.30.0/24     | 10.20.30.254    | Téléphonie IP, softphones              |
| 50      | PC           | 10.20.50.0/24     | 10.20.50.254    | Postes de travail, laptops             |
| 100     | CAMERAS      | 10.20.100.0/24    | 10.20.100.254   | Caméras IP (isolées, accès NVR only)   |
| 200     | INVITES      | 10.20.200.0/24    | 10.20.200.254   | Invités Wi-Fi (Internet only, isolé)   |
| 999     | NATIVE-BH    | *(non routé)*     | *(aucune)*      | VLAN natif blackhole (sécurité trunk)  |
| 1000    | MERDOUILLE   | 10.20.1000.0/24   | 10.20.1000.254  | VLAN de transition — à vider progressivement |

---

## 5. Matériel actuel vs cible (Budget ~1000 €)

| Composant | Matériel d'origine | Cible recommandée | Prix estimé | Justification |
| :--- | :--- | :--- | :---: | :--- |
| **Pare-feu / Routeur** | USG / Box FAI | Fortinet FortiGate 60F | ~350 € | Routage inter-VLAN, relais DHCP, filtrage applicatif, proxy mDNS natif. |
| **Switch cœur L2** | Zyxel GS1900-10HP V2 | Cisco Catalyst 2960X-48LPD-L | ~60 € | 48 ports PoE+, LACP robuste, VLAN natif configurable, prix d'occasion imbattable. |
| **Switch 10 Gb/s** | *(aucun)* | MikroTik CRS305-1G-4S+IN | ~120 € | Backbone 10 Gb/s abordable entre NAS, hyperviseur et PC principal. |
| **Carte HBA** | PCIe x1 (goulot d'étranglement) | LSI SAS 9300-8i (mode IT) | ~80 € | Passe en PCIe x8 pour approcher les débits 10 Gb/s réels sur Proxmox/TrueNAS. |
| **Points d'accès Wi-Fi** | Zyxel (PPSK payant) | 4x Ubiquiti UniFi U7 Lite | ~400 € | PPSK gratuit avec contrôleur local (VM), couverture Wi-Fi 6E, moins de SSID. |
| **Total estimé** | | | **~1 010 €** | *Sources d'occasion : Leboncoin, eBay, BackMarket.* |

---

## 6. Structure du dépôt

```text
homelab-vlan-refactor-community/
│
│   README.md                        ← Ce fichier
│
├───docs/
│   ├───01-architecture/
│   │       overview.md              ← Vue d'ensemble de l'architecture cible
│   │       vlan-table.md            ← Plan IP détaillé et règles inter-VLAN
│   │
│   ├───02-runbooks/
│   │       RB-01-preflight.md       ← Vérifications avant toute intervention
│   │       RB-02-lag-lacp.md        ← Création du LAG LACP FortiGate ↔ Cisco
│   │       RB-03-vlan-merdouille.md ← Migration du VLAN 1 vers les VLANs cibles
│   │       RB-04-dhcp-dns-winsrv2022.md ← DHCP centralisé sur Windows Server 2022
│   │       RB-05-unifi-ppsk.md      ← Wi-Fi PPSK avec mapping VLAN dynamique
│   │       RB-06-mdns-multicast.md  ← mDNS/Bonjour inter-VLAN (Chromecast, AirPlay…)
│   │       RB-07-backup-rsync-qnap.md ← Rsync automatisé TrueNAS → QNAP + extinction
│   │
│   ├───03-risk/
│   │       risk-register.md         ← Registre des risques et plans de mitigation
│   │
│   ├───04-validation/
│   │       checklist.md             ← Checklist de validation post-migration
│   │
│   └───05-roadmap/
│           roadmap.md               ← Évolutions futures et améliorations planifiées
│
├───configs/
│   └───cli/
│           cisco-lag-trunk.ios      ← Config Cisco 2960X : LAG LACP + trunk VLAN
│           fortigate-lag-vlan.conf  ← Config FortiGate 60F : interfaces LAG + VLAN
│           fortigate-dhcp-relay.conf ← Config FortiGate 60F : relais DHCP par VLAN
│
└───scripts/
    ├───bash/
    │       qnap-shutdown-after-rsync.sh ← Extinction QNAP après succès du rsync
    │
    └───powershell/
            dhcp-scopes-bootstrap.ps1    ← Création des scopes DHCP sur WS2022
```

---

## 7. Comment démarrer ?

**Étape 1 — Comprendre l'architecture**  
Lisez [`docs/01-architecture/overview.md`](docs/01-architecture/overview.md) pour visualiser
la topologie cible et comprendre les choix techniques retenus.

**Étape 2 — Étudier le plan IP**  
Consultez [`docs/01-architecture/vlan-table.md`](docs/01-architecture/vlan-table.md) pour
connaître le plan d'adressage complet, les règles inter-VLAN et les restrictions d'accès par segment.

**Étape 3 — Lire le preflight**  
Exécutez mentalement (puis physiquement) le
[`RB-01-preflight.md`](docs/02-runbooks/RB-01-preflight.md) **avant de toucher à quoi que ce soit**.
Ce runbook liste toutes les vérifications à effectuer, les sauvegardes à réaliser et les
"portes de sortie" à préparer en cas de problème.

**Étape 4 — Suivre les runbooks dans l'ordre**  
Chaque runbook est conçu pour être exécuté séquentiellement. Ne sautez pas d'étapes.

---

## 8. Ordre de migration recommandé

```text
RB-01  →  RB-02  →  RB-04  →  RB-03  →  RB-05  →  RB-06  →  RB-07
Preflight   LAG       DHCP      VLANs     Wi-Fi     mDNS      Backup
```

Chaque étape est réversible via les procédures de rollback documentées dans
[`docs/03-risk/risk-register.md`](docs/03-risk/risk-register.md).

---

## 9. Risques et rollback

Toute migration réseau comporte des risques. Les principaux scénarios de panne identifiés
(lockout FortiGate, boucle réseau LACP, VLAN natif mal configuré, DHCP injoignable) sont
documentés dans le registre des risques avec leur probabilité, leur impact et leur procédure
de rollback associée.

➡️ Voir [`docs/03-risk/risk-register.md`](docs/03-risk/risk-register.md)

**Règle d'or** : préparez toujours un PC portable avec une IP statique configurée dans le
sous-réseau d'administration (10.20.10.0/24) branché directement sur un port du FortiGate
avant de commencer toute intervention.

---

## 10. Contribuer

Les contributions sont les bienvenues ! Si vous avez testé ces runbooks sur un matériel
différent, corrigé une erreur ou ajouté le support d'un équipement non couvert, ouvrez
une *Pull Request* avec une description claire des modifications apportées.

Pour les questions ou discussions, utilisez les **GitHub Discussions** du dépôt.

---

## 11. Licence

Ce projet est distribué sous licence **MIT**.  
Vous êtes libre de l'utiliser, le modifier et le redistribuer, à condition de conserver
la mention d'attribution originale.

---

*Dépôt maintenu par [@valorisa](https://github.com/valorisa)*  
*Inspiré du travail de la chaîne YouTube [iMot3k](https://www.youtube.com/@imot3k)*
```

---

Ce `README.md` est maintenant complet avec une table des matières navigable, le diagramme ASCII de l'architecture, le plan IP intégré, la structure du dépôt annotée et l'ordre de migration clair.