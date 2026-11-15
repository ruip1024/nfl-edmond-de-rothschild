> Version Rothschild, basée sur le dépôt public https://github.com/maximeleherle/nfl-dashboard-cbs

# Suivi d'équipe · No Finish Line Monaco 2026

Tableau de bord en direct d'une équipe à la [No Finish Line Monaco](https://nfl.childrenandfuture.com/no-finish-line/resultats/) (14 → 22 novembre 2026, boucle de 1,054 km).
Site statique sur GitHub Pages, sans compte : tout le monde ouvre la même adresse.

## Ce qu'on y trouve

- **Équipe** : total de km et de tours, classement parmi toutes les équipes avec l'écart en tours avec l'équipe devant et derrière, km du jour, marcheurs en piste, courbe des km cumulés, rythme par heure ou par jour, podium et liste des marcheurs (recherche, filtres).
- **Fiche marcheur** (`#coureur-<dossard>`) : km, tours, classements général / catégorie / sexe, place dans l'équipe, progression et km par jour.
- **Comparer** (`#comparer-<dossard>-<dossard>`) : deux marcheurs de l'équipe face à face.
- **Suivre** : l'étoile garde des marcheurs en haut de page (mémorisé dans le navigateur de chacun).

## Comment ça marche

```
GitHub Actions (toutes les 5 min)
  └─ scraper/scrape.mjs ──► API publique Sportips
       ├─ data/roster.json   marcheurs de l'équipe (recherchés 1 fois par jour)
       ├─ data/history.json  un relevé par ligne : équipe + marcheurs ayant marché
       └─ data/latest.json   dernier passage, erreurs, stats de tous les marcheurs
index.html + app.js  ──► lisent config.json et data/, se rafraîchissent toutes les 5 min
```

Le robot (`.github/workflows/scrape.yml`) ne compte pas sur le déclenchement planifié de GitHub, qui a souvent des heures de retard : chaque exécution fait elle-même un relevé toutes les 5 min pendant 5h30, pousse les données après chaque relevé, puis relance l'exécution suivante. Un déclenchement toutes les 30 min sert seulement de filet de sécurité si la chaîne s'arrête. Pour (re)démarrer à la main : *Actions → Relevé Sportips → Run workflow*. La boucle s'arrête seule 6 h après la fin de l'événement.

Sur le site, un marcheur est **en piste** si un nouveau tour est apparu dans les 4 derniers relevés (20 min), sinon **en pause**.

API utilisée (publique, sans token) : `https://inscription.sportips.fr/api/chronometrie/endpoints/public`

| Appel | Usage |
|---|---|
| `GET /event/get.php?base=NFL26` | identifiant de l'événement |
| `POST /races/get.php` `{event_id}` | courses de l'événement |
| `POST /results/get.php` `{id_race, search_terms, order_by, page, limit≤100}` | tours, km, classements d'un marcheur |
| `GET /customResults/list.php?event_id=…` puis `/customResults/get.php?id=…&page=…&limit=100` | classement des équipes |

Le relevé fait **un appel par dossard** toutes les 5 min, plus 5 appels pour le classement des équipes. La recherche des membres de l'équipe (≈ 130 appels) n'a lieu qu'une fois par jour.
L'API ne publie pas l'heure de chaque tour : le « dernier tour » et les courbes viennent des relevés, à 5 min près.

## Réglages (`config.json`)

- `team.organization` : le nom de l'équipe **exactement comme chez Sportips** (colonne « Équipe »).
- `team.color` / `colorDark` : couleur de l'équipe ; `compareColor` : couleur du 2e marcheur dans « Comparer ».
- `extraRunners` : dossards à ajouter même s'ils ne sont pas inscrits sous l'équipe, ex. `[1234, 5678]`.
- `excludeBibs` : dossards à retirer.
- `rosterRefreshHours` : fréquence de la recherche des membres (24 h par défaut).

Pour forcer une nouvelle recherche des membres : onglet **Actions** → « Relevé Sportips » → **Run workflow** → cocher « Reconstruire la liste des marcheurs ».

## En local

```bash
npm run scrape            # un relevé (Node 20+, aucune dépendance)
REFRESH_ROSTER=1 npm run scrape   # en refaisant la recherche des membres
npm run serve             # puis http://localhost:8000
```

Le relevé s'arrête tout seul 6 h après la fin de l'événement (`FORCE=1` pour passer outre).
