# poeme-genere

> **Drifting buoys = drifting boys**
> Les bouées dérivantes suivent le courant océanique.
> Les garçons dérivants suivent le courant social.
> Les deux mesurent des flux, des isolements, des directions.

---

## Concept

Un poème généré à l'intersection de deux sources de données en temps réel :

| Bouées maritimes (AIS/DBCP) | Garçons sur Reddit (r/IncelExit…) |
|---|---|
| Identifiant WMO | Identifiant auteur (MMSI ↔ username) |
| Position géographique | Localisation sociale/émotionnelle |
| Trajectoire dérivante | Historique des posts |
| Hors position charté | Hors norme sociale |
| Transmission météo | Transmission de détresse |
| Vitesse et cap | Tendance du discours |

> *Lancer une bouée à la mer = lancer un appel à l'aide*

---

## Outils

### 1. `reddit-community.html` — Données communauté Reddit

Récupère les posts d'un subreddit filtrés par mots-clés et produit des **cartes d'identité** d'auteurs.

#### Utilisation

Ouvrir le fichier dans un navigateur. Aucune installation requise.

#### Contrôles

| Champ | Description |
|---|---|
| Subreddit | Nom de la communauté (ex : `IncelExit`, `askmec`) |
| Mots-clés | Liste séparée par virgules (ex : `alone, drift, storm, isolated, boy`) |
| Tri | `nouveau` · `populaire` · `top` · `montant` |
| Limite | Nombre de posts à récupérer (1–100) |

#### Vues

- **Liste** — posts filtrés dans l'ordre de la requête, avec mise en évidence des mots-clés
- **Par auteur** — regroupement par auteur trié par nombre de posts, avec :
  - Avatar (chargé depuis le profil Reddit)
  - Karma total, ancienneté du compte, badge premium
  - Dépliage/repliage de chaque groupe

#### Export JSON

Le bouton **Exporter JSON** produit :
- En vue *Liste* : tableau de posts bruts
- En vue *Par auteur* : objet `{ "username": [posts…] }`

#### Carte d'identité d'un auteur (modèle conceptuel)

```
Drifter ID Card          Transmitting

  Boy id:          (MMSI Reddit = username)
  Boy name:        (author)
  Boy type:        (flair / communauté)
  Deployment on:   (created_utc → date locale)
  Propelled to shore: (score = ups)
  Data type:       (title du post)
  Transmitters:    (num_comments)
```

---

### 2. `maritime-beacons.html` — Balises maritimes en temps réel

Carte interactive des balises AIS (aides à la navigation) et des navires, avec trajectoires pour les objets mobiles.

#### Prérequis

Une clé API gratuite **aisstream.io** :
1. S'inscrire sur [aisstream.io](https://aisstream.io)
2. Copier la clé API générée
3. La coller dans le champ *Clé API* — elle est sauvegardée dans `localStorage`

#### Utilisation

Ouvrir dans un navigateur, entrer la clé, sélectionner une zone, cliquer **Connecter**.

#### Contrôles

| Élément | Description |
|---|---|
| Clé API | Clé aisstream.io (stockée localement) |
| Zone | Preset géographique (voir ci-dessous) |
| Vue carte | Utilise les limites visibles de la carte comme zone de réception |
| ⚓ Balises | Active/désactive les balises AIS (type AtoN 21) |
| ⛵ Navires | Active/désactive les navires (rapport de position) |
| Connecter | Ouvre / ferme la connexion WebSocket |

#### Zones prédéfinies

| Preset | Couverture |
|---|---|
| Manche & Mer du Nord | 48–62°N, 5°O–10°E |
| Côtes françaises | 43–51°N, 5°O–9°E |
| Europe | 35–72°N, 15°O–45°E |
| Méditerranée | 30–47°N, 6°O–36°E |
| Atlantique Nord | 30–65°N, 80–0°O |
| Monde | Globe entier |

#### Données affichées

**Balises (AtoN type 21)**

- ⚓ cercle **jaune** = fixe
- ⚓ cercle **bleu** = mobile (dérive détectée)
- ⚓ cercle **rouge** = hors position charté (`OffPosition = true`)
- Types couverts : bouées cardinales N/E/S/O, bouées bâbord/tribord, balises de danger isolé, eaux sûres, navires-feux, RACON…

**Navires**

- ▲ triangle orienté selon le cap réel (`TrueHeading`)

**Trajectoires**

Polyligne colorée (couleur unique par MMSI) tracée dès qu'un objet s'est déplacé de plus de ~50 m. Les 120 dernières positions sont conservées par objet.

#### Panneau latéral

- Compteurs : balises · navires · mobiles · messages reçus
- Liste triée : hors-position d'abord → mobiles → ordre alphabétique
- Champ de recherche par nom ou MMSI
- Clic → popup détaillé + centrage carte
- Âge des données mis à jour toutes les 10 s

#### Popup

| Champ | Source AIS |
|---|---|
| MMSI | Identifiant unique de l'objet |
| Type | `TypeOfAidToNavigation` (balise) ou « Navire » |
| Position | `Latitude`, `Longitude` |
| Vitesse | `SpeedOverGround` (navires) |
| Cap | `TrueHeading` / `CourseOverGround` |
| Hors position | `OffPosition` |
| Mobilité | Calculé sur l'historique local |
| Dernière MAJ | Horodatage `MetaData.time_utc` |

---

## Architecture technique

```
poeme-genere/
├── reddit-community.html    # scraping Reddit + vue par auteur
├── maritime-beacons.html    # carte AIS temps réel
└── README.md
```

### Dépendances externes (CDN, aucune installation)

| Bibliothèque | Usage | Version |
|---|---|---|
| [Leaflet.js](https://leafletjs.com) | Carte interactive | 1.9.4 |
| OpenStreetMap tiles | Fond de carte | — |
| OpenSeaMap tiles | Symboles nautiques | — |
| [aisstream.io](https://aisstream.io) | Flux WebSocket AIS | — |
| Reddit JSON API | Posts subreddit | `reddit.com/*.json` |

### Flux de données

```
Reddit .json (REST)          aisstream.io (WebSocket)
        │                            │
        ▼                            ▼
  Filtre mots-clés           Messages AIS (type 21 + pos.)
        │                            │
        ▼                            ▼
  Groupe par auteur          Mise à jour markers + polylines
        │                            │
        ▼                            ▼
  Profil auteur (/about.json) Panneau + popup
```

---

## Références

- Carte bouées dérivantes OceanOPS : [ocean-ops.org](https://www.ocean-ops.org/maps/static/?t=OceanOPS&displayedMap=DBCP_ALL)
- Bouées françaises SURFMAR : [esurfmar.meteo.fr](https://esurfmar.meteo.fr/cgi-bin/blackpos_surfmar.cgi)
- Marine Traffic : [marinetraffic.com](https://www.marinetraffic.com/en/ais/home/centerx:-12.0/centery:25.0/zoom:4)
- Communauté Reddit : [r/IncelExit](https://www.reddit.com/r/IncelExit/)
- Norme AIS IEC 62287 — message type 21 (Aid-to-Navigation)
