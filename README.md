# Catalogue distant — ukulele_tabs

Dépôt GitHub utilisé par l’application **ukulele_tuner** pour distribuer de
nouvelles tablatures sans mise à jour de l’app.

**URL distante :** https://github.com/johndev38/ukulele_tabs

## Structure attendue

```text
ukulele_tabs/
├── catalog.json          # liste des fichiers JSON disponibles
├── ma-chanson.json
└── autre-morceau.json
```

### `catalog.json`

```json
{
  "songs": [
    "ode-to-joy.json",
    "frere-jacques.json"
  ]
}
```

Chaque fichier listé doit être un morceau JSON valide (schéma v1, identique
à `assets/songs/` dans le projet Flutter). Voir `schemas/song.schema.json`.

**Pour ChatGPT / agents IA :** lisez `AGENTS.md` et copiez le prompt dans
`templates/song.template.json` comme point de départ.

## Publier une nouvelle tablature

1. Créez ou copiez un fichier `{id}.json` (l’`id` dans le JSON doit être
   unique et stable, en kebab-case).
2. Validez-le depuis le projet principal :

   ```bash
   dart run tool/validate_song.dart ukulele_tabs/ma-chanson.json
   ```

3. Ajoutez le nom du fichier dans `catalog.json`.
4. Committez et poussez sur `main` :

   ```bash
   git add catalog.json ma-chanson.json
   git commit -m "Ajoute ma-chanson"
   git push origin main
   ```

## Comportement côté application

Au lancement (et via l’icône nuage dans l’onglet **Morceaux**), l’app :

1. Télécharge `catalog.json` depuis
   `https://raw.githubusercontent.com/johndev38/ukulele_tabs/main/`
2. Compare les `id` avec le catalogue local (intégré + importé + déjà sync)
3. Télécharge **uniquement les morceaux manquants**
4. Les enregistre dans `<documents>/songs/synced/`

Les morceaux déjà intégrés à l’app ne sont pas re-téléchargés.

## Premier déploiement du dépôt

Si le dépôt GitHub est vide, initialisez-le avec ce dossier :

```bash
cd ukulele_tabs
git init
git remote add origin https://github.com/johndev38/ukulele_tabs.git
git add .
git commit -m "Catalogue initial"
git branch -M main
git push -u origin main
```
