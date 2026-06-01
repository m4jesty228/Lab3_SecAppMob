# Lab 20 – Interception de trafic Android avec Burp Suite
**Cours : Sécurité des Applications Mobiles ** &nbsp;|&nbsp; **Date : Mai 2026**

---

## 1. Introduction

Ce laboratoire met en place un proxy d'observation entre un Android Emulator et une cible autorisée (site de test de formation). L'objectif est de comprendre comment un proxy s'insère dans le chemin réseau, ce que Burp Suite capture, et comment documenter proprement les observations.

> ⚠️ **Périmètre strict** : trafic intercepté uniquement sur cible autorisée, aucun compte personnel, aucune donnée sensible. Tout certificat de labo installé est retiré en fin de séance.

---

## 2. Objectifs du lab

À l'issue de ce laboratoire :

1. Vérifier qu'un navigateur Android envoie son trafic via Burp Suite.
2. Identifier les éléments essentiels d'une requête HTTP : méthode, URL, headers, cookies, paramètres.
3. Expliquer la différence HTTP vs HTTPS et le rôle d'un certificat CA en contexte de labo.
4. Produire une trace d'audit structurée (preuves + contexte).

---

## 3. Environnement de travail

| Composant | Détail |
|---|---|
| Machine hôte | Windows/Linux avec Burp Suite Community |
| Émulateur | Android Emulator (Nexus/Pixel – Android Studio) |
| Proxy | Burp Suite Community Edition |
| Adresse hôte | `<IP_HOTE>` (relevée à l'étape 3) |
| Port proxy | `<PORT_PROXY>` (relevé à l'étape 2, ex. 8080) |
| Cible | Site de test autorisé / maquette de formation |
| Réseau | Environnement de labo isolé |

---

## 4. Architecture du lab

```
┌─────────────────────────┐        ┌──────────────────────┐        ┌──────────────┐
│   Android Emulator      │        │   Machine hôte        │        │   Cible      │
│                         │        │                        │        │   autorisée  │
│  Navigateur             │──────▶ │  Burp Suite Proxy     │──────▶ │  (test/lab)  │
│  (proxy: IP_HOTE:PORT)  │  HTTP  │  Listener: PORT_PROXY │  HTTP  │              │
│                         │◀────── │  HTTP History         │◀────── │              │
└─────────────────────────┘        └──────────────────────┘        └──────────────┘
         Trafic redirigé                 Observation passive
```

Le proxy s'insère de manière transparente : l'émulateur croit dialoguer directement avec la cible, mais chaque requête transite par Burp qui la consigne dans l'historique.

---

## 5. Étapes réalisées

### Étape 1 — Préparation de Burp Suite
Burp Suite est lancé en projet temporaire de labo. L'onglet **Proxy → Intercept** est ouvert avec l'interception **désactivée** (`Intercept is off`) — volontairement, pour ne pas bloquer le trafic durant la phase de validation de la configuration.

### Étape 2 — Vérification du Proxy Listener
Dans **Proxy → Proxy settings**, un listener est confirmé actif (`Running`) sur le port `<PORT_PROXY>`. L'adresse d'écoute est configurée sur **All interfaces** pour être accessible depuis l'émulateur.

### Étape 3 — Identification de l'adresse hôte
Sur la machine hôte, l'adresse IPv4 du réseau de labo est relevée (`<IP_HOTE>`). C'est l'adresse que l'émulateur utilisera pour joindre le proxy. Une adresse d'un réseau inactif ou VPN provoquerait un échec silencieux.

### Étape 4 — Configuration du proxy sur l'émulateur Android
Dans les paramètres Wi-Fi de l'émulateur → réseau actif → options avancées → **Proxy : Manual** :
- Proxy hostname : `<IP_HOTE>`
- Proxy port : `<PORT_PROXY>`

Sans cette configuration, le trafic du navigateur part directement vers Internet et n'apparaît pas dans Burp.

### Étape 5 — Validation de la capture HTTP
Le navigateur de l'émulateur accède à la cible autorisée. Dans **Proxy → HTTP history**, les requêtes apparaissent immédiatement : méthode, URL, code de réponse, taille. La chaîne proxy est validée.

### Étape 6 — Analyse d'une requête (mode observateur)
Une requête est sélectionnée dans l'historique. Les éléments analysés :

- **Onglet Raw** : vue brute complète (ligne de requête, headers, corps)
- **Onglet Inspector** : lecture structurée — Query parameters, Cookies, Headers

Éléments observés sur la cible de test :
- Méthode `GET` avec paramètres en clair dans l'URL
- Headers standards : `User-Agent`, `Accept`, `Host`
- Cookies présents, sans attributs `HttpOnly` / `Secure` côté client observable

### Étape 7 — Démonstration de l'interception active
L'interception est activée brièvement (`Intercept is on`). Une requête est déclenchée depuis le navigateur — elle apparaît **en attente** dans Burp, prouvant que le proxy est bien un point de passage actif et non un simple observateur passif. L'interception est immédiatement désactivée après observation.

### Étape 8 — HTTPS et certificat CA (principe)
En naviguant vers une cible HTTPS sans certificat CA installé, le navigateur affiche une erreur de certificat — comportement attendu et sain. L'écran d'installation de certificat dans l'émulateur identifie trois types : **CA certificate**, **VPN & app user certificate**, **Wi-Fi certificate**. Dans un contexte de labo, un certificat CA Burp peut être installé temporairement pour observer le trafic HTTPS — il doit être retiré en fin de séance.

---

## 6. Analyse des observations

| Élément observé | Risque potentiel | Recommandation défensive |
|---|---|---|
| Paramètres en clair dans l'URL | Exposition dans les logs serveur / proxy | Passer les paramètres sensibles en corps POST |
| Cookies sans attribut `Secure` | Transmissibles sur HTTP non chiffré | Forcer `Secure` + `HttpOnly` côté serveur |
| Cookies sans attribut `HttpOnly` | Accessibles depuis JavaScript | Ajouter `HttpOnly` systématiquement |
| Absence d'en-têtes de sécurité | Pas de politique CSP, pas de HSTS visible | Configurer les headers côté serveur |
| Token visible en URL | Loggué dans l'historique du proxy/navigateur | Utiliser Authorization header ou corps chiffré |

---

## 7. Checkpoints validés

- [x] Burp capture au moins une requête dans HTTP history
- [x] Proxy listener actif sur `<IP_HOTE>:<PORT_PROXY>`
- [x] Proxy Android configuré en mode Manual
- [x] Interception utilisée uniquement en démonstration, puis désactivée
- [x] Analyse d'une requête complète (headers + paramètres + cookies)
- [x] Principe HTTPS / certificat CA compris

---

## 8. Nettoyage de fin de séance

1. Proxy Android remis sur **None** dans les paramètres Wi-Fi de l'émulateur.
2. Certificat de labo retiré si installé (Paramètres → Sécurité → Certificats de confiance).
3. Projet Burp fermé — aucune donnée sensible archivée.

> Un lab non nettoyé laisse un émulateur avec un proxy actif qui échouera silencieusement lors des prochaines séances réseau.

---

## 9. Conclusion

Ce laboratoire démontre concrètement qu'un proxy d'interception comme Burp Suite s'insère dans le chemin réseau de manière transparente pour l'application cible. La compétence clé n'est pas la manipulation des requêtes, mais **l'analyse** : comprendre ce qui transite, identifier les informations exposées, et produire une documentation exploitable.

La différence entre HTTP et HTTPS est rendue tangible : sans certificat CA de labo, HTTPS résiste à l'observation — ce qui est précisément le comportement attendu d'une application correctement configurée.

---

## 👤 Auteur

**DOSSAH Landry**  
ENSA Marrakech | GCDSTE S4  
Module : Cybersécurité Mobile — Analyse réseau
