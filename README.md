# IdentMap — Détection d'événements routiers (ADAS) sur trace GPX

Application web autonome (un seul fichier `index.html`) pour la recherche ADAS
connectée : tu charges une trace **GPX/KML**, tu **choisis les événements à
rechercher**, tu lances l'analyse, puis tu **affiches/masques** chaque type sur
une carte interactive avec un **tableau de synthèse trié par kilomètre**.

## Déroulé

1. **Charger un GPX/KML** (bouton ou glisser-déposer).
2. **Onglet Configuration** : coche les événements à rechercher (par catégorie,
   avec « tout cocher »). On n'interroge OpenStreetMap que pour ce qui est coché
   → si tu ne coches que de la géométrie, l'analyse tourne 100 % hors-ligne.
3. **Lancer l'analyse** (zone par zone, avec progression).
4. **Onglet Résultats** : cases à cocher pour afficher chaque type sur la carte,
   compteurs, et tableau (clic = recentrage).
5. **Filtre environnement** : *Tout / En ville / Hors ville* — n'affiche que les
   événements en agglomération ou hors agglomération (carte + tableau + compteurs).
6. **Bouton Google Maps** : ouvre le parcours en navigation dans Google Maps
   (app sur mobile). L'URL est limitée par Google à ~9 étapes : le tracé est
   envoyé sous forme de départ + arrivée + ~8 points intermédiaires répartis,
   Google reconstruit l'itinéraire routier entre eux (approximation, pas au
   point près).

## Catalogue d'événements

**Géométrie du tracé** (fiable, hors-ligne)
- Virages (+ sévérité), Épingles à cheveux (angle + rayon serré)
- Successions de virages, Longues lignes droites
- Virages à rayon décroissant, Enchaînements en S

**Altitude / relief** (si la trace contient `<ele>`, détecté automatiquement)
- Fortes pentes (montée/descente), Virage après sommet de côte
- Longues descentes continues, Dénivelé +

**Topologie & régulation** (OpenStreetMap)
- Ronds-points / giratoires / mini ronds-points, Échangeurs, Bretelles
- Intersections (toutes) et **intersections en virage** (prise / traversée)
- Feux tricolores, STOP, Cédez-le-passage, Passages piétons, Passages à niveau
- Ralentisseurs, Chicanes, Radars, Péages, Barrières / bornes
- Traversées d'agglomération, Zones scolaires, Arrêts de bus, Gués,
  Créneaux de dépassement

**Attributs de la route** (OpenStreetMap)
- Route dégradée (sans marquage), Route étroite, Tunnels, Ponts
- Travaux / route en chantier, Sections à sens unique, Non éclairé,
  Pente signalée (OSM), Zone à basse vitesse (≤30)
- Changement de limitation ≈, de nombre de voies ≈, de catégorie ≈

> Fiabilité : la géométrie et les points OSM bien cartographiés sont fiables.
> Les intersections-en-virage, les routes dégradées et les changements
> d'attributs (≈) dépendent de la complétude d'OpenStreetMap et sont exhaustifs
> seulement là où la carte est bien renseignée.
>
> Hors périmètre (nécessite les capteurs/logs du véhicule) : vitesse réelle,
> distance inter-véhiculaire, autres usagers, météo, qualité GNSS, performance
> réelle des aides à la conduite.

## Comment ça marche

- **Géométrie** : tracé ré-échantillonné à pas constant, rotation cumulée par
  segment pour isoler virages, épingles (angle + rayon), successions, lignes droites.
- **Altitude** : profil ré-échantillonné (25 m) et lissé, pente sur base 50 m.
- **OpenStreetMap** : découpage en zones ; une requête [Overpass](https://overpass-api.de/)
  par zone (uniquement les objets cochés) → routes + nœuds. Intersections =
  nœuds de degré ≥ 3. Filtrage par proximité au tracé via un index spatial.
- **Changements d'attributs** : appariement léger des tronçons OSM au tracé.
- **Environnement (ville / hors ville)** : chaque événement est classé d'après le
  contexte routier OSM à sa position — *en ville* si vitesse ≤ 50, voie
  résidentielle/zone de rencontre, ou éclairage public ; *hors ville* sinon.
  Heuristique dépendant de la complétude d'OSM (maxspeed, lit…).
- **Ronds-points** : dédoublonnés géographiquement (un rond-point cartographié en
  plusieurs tronçons OSM, ou repassé, n'apparaît qu'une fois) ; leur zone est
  exclue des virages, épingles, rayons décroissants, enchaînements en S et
  intersections — un rond-point n'est jamais compté comme un virage ni une
  intersection (le « nombre de virages » l'exclut aussi).

## Technique

- [Leaflet](https://leafletjs.com/) **intégré** dans `index.html` (aucun CDN).
- Fond de carte : tuiles OpenStreetMap. Données : API Overpass (miroirs multiples).
- Aucune dépendance à installer. Connexion internet requise pour les événements OSM.
