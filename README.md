# backstage-module-watch-later

Module externe de [BACKSTAGE](https://github.com/MattiaPARRINELLO/backstage) :
la liste « À voir plus tard ». Il garde des liens (articles, vidéos, photos,
musiques), les classe par catégorie et peut les résumer.

Extrait du dépôt noyau pour la phase 7 de #84 (issue #208) : il sert de preuve
qu'un module déjà en place survit à son extraction, puis se réinstalle depuis un
dépôt git sans modification du noyau.

## Contrat

| Champ | Valeur |
| --- | --- |
| `id` | `watch-later` |
| `apiVersion` | `1` |
| `version` | `1.0.1` |
| `capabilities` | `filesystem: "module-data"` |
| Outil | `watch-later.add` (effet `write`) |
| Actions | `load`, `create`, `summarize`, `edit`, `remove`, `reorder` |
| Interface | page `/m/watch-later`, rail « À voir », panneau, carte d'accueil |

Le module n'importe que `@backstage/module-kit` et `zod`, tous deux fournis par
le noyau hôte. Aucun import de `lib/`, `app/` ou `components/`.

## Données

Fichier unique : `data/watch-later.json`, à la racine de `data/` (chemin
historique conservé via `legacyPath`, donc aucune migration).

```json
{
  "items": [
    {
      "id": "identifiant",
      "url": "https://exemple.fr/article",
      "title": "Titre lisible",
      "description": "optionnel",
      "thumbnail": "optionnel",
      "source": "exemple.fr",
      "category": "video | article | photo | music | other",
      "createdAt": "2026-11-23T09:00:00.000Z",
      "summary": "optionnel, produit par la capacité IA du noyau",
      "aiTags": ["optionnel"],
      "read": false
    }
  ]
}
```

Le schéma est déclaré dans `schema.ts` et validé par le store (`store.ts`) en
lecture comme en écriture.

## Capacités utilisées

- `ctx.net.isSafeUrl` / `ctx.net.fetchText` : accès réseau contrôlé par le noyau
  (anti-SSRF), pour lire la page avant résumé.
- `ctx.ai.complete` : complétion IA avec le modèle applicatif du noyau.
- `ctx.activity` : journal d'activité de l'application.

Les trois sont optionnelles dans le contrat : sans elles, le module refuse une
URL non vérifiée et n'écrit pas de résumé, sans faire échouer l'ajout du lien.

## Installation

Depuis BACKSTAGE, avec une **ref épinglée** (tag ou commit, jamais une branche) :

```bash
bun run module:add https://github.com/MattiaPARRINELLO/backstage-module-watch-later.git#v1.0.0
bun run build
```

`module:add` écrit exactement quatre choses : le dossier `modules/watch-later/`,
`modules.config.json`, `modules.lock.json` et `modules/index.ts`.

## Tests

Les tests de ce dépôt s'exécutent avec le lanceur du dépôt hôte, après copie du
module dans `modules/watch-later/` (ou directement depuis ce dossier avec
`vitest`). Ils ne dépendent que de `vitest` et du module lui-même :

```bash
bunx vitest run __tests__/
```

## Désinstallation

Retirer l'entrée de `modules.config.json`, supprimer `modules/watch-later`, son
entrée dans `modules.lock.json` et son import dans `modules/index.ts`, puis
rebuild. `bun run module:doctor` signale tout reste orphelin. Aucune commande
`module:remove` n'existe encore (issue #209).

## Licence

MIT.
