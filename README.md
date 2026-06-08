# Italpluvio — Profil climatique de site (Italie)

Application web (PWA) pour la **récupération d'eaux pluviales** (RWH) en Italie. Déclinaison italienne de [Pluvio RWH](../pluvio/), avec interface **FR / DE / IT**.

Deux outils :

- **Onglet Pluie** : histogramme des **cumuls mensuels de précipitations** (moyenne
  climatologique longue durée) pour n'importe quel point, avec année sèche/humide.
- **Onglet Cuve** : **dimensionnement d'une cuve d'arrosage** (méthode de Rippl) à partir de
  la surface de toit, du type de couverture et des surfaces arrosées par culture.

➡️ **https://niqoz.github.io/italpluvio/**

## Données

- Source : **Copernicus Climate Change Service — réanalyse ERA5-Land mensuelle** (~9 km), Licence Copernicus.
- Pluie : variable `tp` (m, cumul journalier moyen) → mm/mois = `tp × 1000 × nb_jours`.
- ET0 calculée **FAO-56 Penman-Monteith** à partir des variables ERA5-Land (température, rayonnement `ssrd`, humidité, vent).
- 3 235 points couvrant l'ensemble du territoire italien (polygone régions, mer masquée).
- Normales calculées sur 2 fenêtres : **1995-2020** (référence) et **15 dernières années** (climat récent).
- Commune récupérée **au clic, en ligne**, via Nominatim (OpenStreetMap) — non embarquée dans le JSON.

## Fonctionnement

Architecture **données pré-calculées + app cliente légère** :

- `pipeline/` : scripts Python qui téléchargent ERA5-Land depuis le Copernicus CDS, agrègent les normales
  et produisent `normales_italie.json` (~3 235 points).
- `docs/` : la PWA (HTML/JS, Chart maison, carte Leaflet). Servie par GitHub Pages.
  Fonctionne **hors-ligne** une fois installée (service worker). 3 modes de localisation :
  GPS, sélecteur de villes, carte interactive.
  **Responsive** : mise en page 2 colonnes automatique sur navigateur bureau (≥ 768 px).

### Module cuve

Même méthode que Pluvio France (bilan mensuel de Rippl). Types de plantes : gazon froid,
**gazon chaud / kikuyu** (Méditerranée, défaut), potager, massifs, verger, oliviers/agrumes.

### Trilingue FR / DE / IT

Dictionnaire `I18N` intégré à l'app ; bascule par boutons en en-tête, choix mémorisé.
Le DE cible le Haut-Adige/Tyrol du Sud (Bolzano).

## Régénérer les données

```bash
# Pré-requis : venv avec cdsapi xarray netcdf4 numpy + compte Copernicus + ~/.cdsapirc
cd ../pluvio   # le pipeline est dans le repo pluvio
.venv/bin/python pipeline/build_italie_cds.py --selftest   # valide la formule ET0 FAO-56
.venv/bin/python pipeline/build_italie_cds.py --download    # télécharge le NetCDF ERA5-Land (~44 Mo)
.venv/bin/python pipeline/build_italie_cds.py               # agrège -> docs/normales_italie.json
# Incrémenter CACHE dans docs/sw.js, puis commit/push
```

## Licences / attribution

- Données : © Copernicus Climate Change Service (ERA5-Land, Licence Copernicus).
- Communes : © OpenStreetMap contributors (Nominatim).
- Polygone régions : openpolis/geojson-italy.
- Fond de carte : © OpenStreetMap contributors.
- Bibliothèque carte : Leaflet (BSD-2).
