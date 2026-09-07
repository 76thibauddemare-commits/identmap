# IdentMap — Compteur de ronds-points sur trace GPX

Application web autonome (un seul fichier `index.html`) qui :

1. affiche ta trace **GPX** sur une carte interactive ;
2. détecte les **ronds-points** rencontrés le long du parcours grâce aux données **OpenStreetMap** ;
3. les **marque sur la carte** (pastilles numérotées) ;
4. affiche dans un volet latéral un **tableau de synthèse** : nombre total de ronds-points et **kilomètre** de chacun le long du tracé.

## Utilisation

1. Ouvre `index.html` dans un navigateur (double-clic, ou héberge le fichier).
2. Clique sur **Charger un GPX** (ou glisse-dépose ton fichier `.gpx` sur la page).
3. La trace s'affiche, puis les ronds-points sont recherchés automatiquement.
4. Clique sur une ligne du tableau pour centrer la carte sur le rond-point correspondant.

> Une connexion internet est nécessaire (fond de carte OpenStreetMap + recherche
> des ronds-points via l'API Overpass).

## Comment ça marche

- Le fichier GPX est lu côté navigateur (`trkpt`, sinon `rtept`/`wpt`).
- La distance cumulée est calculée point par point (formule de haversine).
- Une version allégée du tracé est envoyée à l'API **Overpass** avec un filtre
  `around` (25 m) pour ne récupérer que les objets proches du parcours :
  - `junction=roundabout` → **rond-point**
  - `junction=circular` → **giratoire**
  - `highway=mini_roundabout` → **mini rond-point** (pastille bleue)
- Pour chaque rond-point trouvé, on projette son centre sur le tracé afin de
  déterminer son **kilomètre** exact. Un filtre de sécurité (> 60 m du tracé)
  écarte les faux positifs.

## Technique

- [Leaflet](https://leafletjs.com/) pour la carte, tuiles OpenStreetMap.
- [API Overpass](https://overpass-api.de/) pour les données ronds-points
  (plusieurs miroirs sont essayés en cas d'indisponibilité).
- Aucune dépendance à installer, aucun serveur : tout est dans `index.html`.
