# IdentMap — Détection d'événements routiers sur trace GPX

Application web autonome (un seul fichier `index.html`) qui affiche ta trace
**GPX/KML** sur une carte interactive et détecte plusieurs **événements routiers**
le long du parcours. Chaque type est une **couche activable** (case à cocher) et
apparaît dans un **tableau de synthèse trié par kilomètre**.

## Événements détectés

| Événement | Source | Fiabilité |
|---|---|---|
| **Nombre de virages** | Géométrie du tracé | ✅ bonne |
| **Successions de virages** | Géométrie du tracé | ✅ bonne |
| **Ronds-points / giratoires / mini ronds-points** | OpenStreetMap | ✅ bonne |
| **Échangeurs** (`highway=motorway_junction`) | OpenStreetMap | ✅ bonne |
| **Intersection prise en virage** | OSM (intersections) + géométrie | 🟡 heuristique |
| **Intersection traversée en virage** | OSM + géométrie | 🟡 heuristique |
| **Route dégradée sans marquage** | OSM (`surface`, `lane_markings=no`) | 🟠 dépend d'OSM |

> Les deux dernières s'appuient sur des tags OSM souvent incomplets : elles sont
> exhaustives seulement là où la carte est bien renseignée.

## Utilisation

1. Ouvre `index.html` dans un navigateur (ou via un lien hébergé sur mobile).
2. **Charge un GPX** (bouton ou glisser-déposer). Formats : `.gpx`, `.kml`.
3. La trace s'affiche, puis l'analyse tourne **zone par zone** (barre de progression).
4. Coche/décoche les types d'événements pour les afficher sur la carte et dans le tableau.
5. Clique une ligne du tableau pour recentrer la carte sur l'événement.

## Comment ça marche

- **Géométrie** : le tracé est ré-échantillonné à pas constant ; on mesure la
  rotation cumulée pour isoler les virages, compter, et repérer les successions.
- **OpenStreetMap** : le parcours est découpé en zones ; une requête
  [Overpass](https://overpass-api.de/) par zone renvoie les routes et nœuds, dont
  on tire ronds-points, échangeurs, intersections (nœuds de degré ≥ 3) et
  revêtements dégradés. Filtrage par proximité au tracé.
- **Intersections en virage** : intersection OSM + forte courbure locale du tracé ;
  « prise » si le véhicule tourne, « traversée » sinon.

## Technique

- [Leaflet](https://leafletjs.com/) **intégré** dans `index.html` (aucun CDN).
- Fond de carte : tuiles OpenStreetMap. Données : API Overpass (miroirs multiples).
- Aucune dépendance à installer. Connexion internet requise (carte + Overpass).
