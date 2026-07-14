# Instructions pour agents IA (ChatGPT, Claude, Cursor…)

Ce dépôt alimente le catalogue distant de l’application **ukulele_tuner**.
Les utilisateurs reçoivent automatiquement les nouvelles tablatures au lancement
de l’app (synchronisation « missing only »).

**Dépôt :** https://github.com/johndev38/ukulele_tabs  
**Branche :** `main`  
**URL raw (lecture app) :** `https://raw.githubusercontent.com/johndev38/ukulele_tabs/main/`

---

## Rôle de l’agent

Tu es un contributeur technique de ce dépôt. Quand on te demande d’ajouter une
tablature, tu dois :

1. Lire l’état actuel du dépôt (`catalog.json` + fichiers `*.json` existants).
2. Produire un morceau JSON **valide** (schéma v1).
3. Créer le fichier `{id}.json` à la racine du dépôt.
4. Mettre à jour `catalog.json` (ajouter le nom de fichier, sans doublon).
5. Committer et pousser sur `main` (ou proposer un patch / une PR).

Ne modifie jamais un morceau existant sans demande explicite. Ne supprime pas
d’entrées du catalogue sans demande explicite.

---

## Structure du dépôt

```text
ukulele_tabs/
├── AGENTS.md              ← ce fichier (instructions IA)
├── README.md              ← documentation humaine
├── catalog.json           ← index obligatoire
├── templates/
│   └── song.template.json ← squelette à copier
├── ode-to-joy.json
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

- Liste **tous** les fichiers `*.json` de morceaux (pas `catalog.json` lui-même).
- Un fichier = une entrée dans `songs`.
- Garder la liste triée par ordre alphabétique.
- Le nom de fichier recommandé : `{id}.json` où `id` est le champ `id` du JSON.

---

## Contrat JSON (schéma v1)

Chaque morceau est un objet JSON avec ces champs **obligatoires** :

| Champ | Règle |
|-------|--------|
| `schemaVersion` | Toujours `1` |
| `id` | Identifiant unique, stable, kebab-case (`frere-jacques`, `ode-to-joy`) |
| `title` | Titre affiché dans l’app |
| `artist` | Auteur ou « Traditionnel » |
| `description` | 1–2 phrases |
| `difficulty` | `beginner`, `intermediate` ou `advanced` |
| `rights` | `{ "type": "...", "source": "..." }` |
| `instrument` | `{ "type": "ukulele", "tuning": "high-g-gcea", "maximumFret": 12 }` |
| `timing` | `bpm` (30–240), `beatsPerBar`, `beatUnit`, `ticksPerBeat`: **480**, `countInBars`: 0–4 |
| `display` | `{ "showNoteNames": true, "showTablature": true }` |
| `events` | Tableau trié par `startTick`, au moins 1 événement |

### Droits (`rights.type`)

Pour ce catalogue public, privilégier :

- `public-domain` — compositions du domaine public
- `original` — exercices créés pour l’app

Éviter les œuvres sous copyright récent sans licence explicite.

### Accordage High G (GCEA)

| Corde | Nom | MIDI ouvert |
|------:|-----|------------:|
| 1 | A4 | 69 |
| 2 | E4 | 64 |
| 3 | C4 | 60 |
| 4 | G4 | 67 |

**Formule :** `midi = corde_ouverte + fret`

### Événements

**Note** (`type: "note"`) :

```json
{
  "id": "note-001",
  "type": "note",
  "startTick": 0,
  "durationTicks": 480,
  "string": 3,
  "fret": 0,
  "midi": 60,
  "noteName": "C4"
}
```

**Silence** (`type: "rest"`) :

```json
{
  "id": "rest-001",
  "type": "rest",
  "startTick": 960,
  "durationTicks": 480
}
```

**Accord** (`type: "chord"`) — plusieurs cordes grattées ensemble :

```json
{
  "id": "chord-001",
  "type": "chord",
  "startTick": 0,
  "durationTicks": 1920,
  "name": "C",
  "fingers": [
    { "string": 4, "fret": 0, "midi": 67, "noteName": "G4" },
    { "string": 3, "fret": 0, "midi": 60, "noteName": "C4" },
    { "string": 2, "fret": 0, "midi": 64, "noteName": "E4" },
    { "string": 1, "fret": 3, "midi": 72, "noteName": "C5" }
  ]
}
```

- `fret: -1` = corde étouffée (sans `midi` ni `noteName`).
- Voir `basic-chords.json` pour un morceau complet C – Am – F – G.

### Règles musicales strictes

- **Timeline** : une note **ou** un accord par créneau — pas de chevauchement entre événements.
- **Tri** : `events` ordonnés par `startTick` croissant.
- **IDs uniques** : chaque `id` d’événement est unique dans le morceau.
- **Cohérence MIDI** : `midi` = corde ouverte + `fret` ; `noteName` correspond au MIDI (notation internationale, dièses uniquement : `C4`, `C#4`, `A4`…).
- **Cases** : `fret` entre 0 et `maximumFret` (12 par défaut).
- **Préférence** : cases basses, peu de sauts de main (débutant).

### Durées en ticks (`ticksPerBeat = 480`)

| Figure | Ticks |
|--------|------:|
| Double croche | 120 |
| Croche | 240 |
| Noire | 480 |
| Noire pointée | 720 |
| Blanche | 960 |
| Ronde | 1920 |
| Mesure 4/4 | 1920 |

---

## Procédure : ajouter une tablature

### Étape 1 — Vérifier l’unicité

1. Lire `catalog.json`.
2. Pour chaque fichier listé, noter le champ `id` interne.
3. Choisir un `id` **nouveau** (kebab-case, pas déjà présent).

Ids déjà courants dans le catalogue (ne pas réutiliser) :

- `open-strings`, `c-major-exercise`, `repeated-notes-exercise`
- `ode-to-joy`, `frere-jacques`, `twinkle-twinkle`, `au-clair-de-la-lune`, `happy-birthday`

### Étape 2 — Générer le JSON

1. Copier `templates/song.template.json`.
2. Remplacer tous les champs métadonnées.
3. Construire la mélodie en respectant les règles ci-dessus.
4. Vérifier : premier événement à `startTick: 0`, pas de trou illogique sauf silences voulus.

### Étape 3 — Valider (si accès au projet Flutter)

Depuis le dépôt parent `ukulele_app` :

```bash
dart run tool/validate_song.dart ukulele_tabs/MON-ID.json
```

Code de sortie `0` = valide. En cas d’erreur, corriger et relancer.

Sans accès au validateur : relire manuellement chaque règle de la section « Contrat JSON ».

### Étape 4 — Publier dans ce dépôt

```bash
# Créer le fichier (exemple)
# ukulele_tabs/la-cucaracha.json

# Mettre à jour catalog.json — ajouter "la-cucaracha.json" dans "songs"

git add catalog.json la-cucaracha.json
git commit -m "Ajoute la-cucaracha"
git push origin main
```

### Étape 5 — Vérifier la diffusion

Après le push, le fichier doit être accessible :

`https://raw.githubusercontent.com/johndev38/ukulele_tabs/main/la-cucaracha.json`

L’app télécharge les morceaux dont l’`id` n’est pas déjà connu localement.

---

## Prompt utilisateur (à coller dans ChatGPT)

Copie ce bloc quand tu demandes une nouvelle tab. Remplace les champs `[…]`.

```text
Tu contribues au dépôt GitHub johndev38/ukulele_tabs (branche main).
Suis intégralement ukulele_tabs/AGENTS.md.

Tâche : ajouter une tablature ukulélé monophonique.

Morceau demandé : [TITRE / DESCRIPTION DE LA MÉLODIE]
Compositeur / origine : [ARTISTE]
Difficulté : [beginner | intermediate | advanced]
Tempo : [BPM] BPM, mesure [4/4 ou autre]
Nombre de mesures : [N]
Droits : [public-domain | original]

Étapes obligatoires :
1. Lire catalog.json et les id existants — ne pas créer de doublon.
2. Proposer un id kebab-case unique (ex. la-cucaracha).
3. Générer le JSON complet (schemaVersion 1, ticksPerBeat 480).
4. Créer le fichier {id}.json à la racine du dépôt.
5. Ajouter "{id}.json" dans catalog.json (liste triée).
6. Commit + push sur main.

Contraintes musicales :
- Ukulélé High G GCEA, cases 0–12, une note à la fois.
- midi = corde ouverte + fret ; noteName cohérent (C4, D4, E4…).
- Événements triés par startTick, ids uniques, pas de chevauchement.

Livrables :
- Contenu complet de {id}.json
- catalog.json mis à jour
- Message de commit proposé

Ne renvoie pas de Markdown autour du JSON des fichiers finaux.
```

---

## Connexion ChatGPT ↔ GitHub

Pour que ChatGPT modifie ce dépôt directement :

1. **ChatGPT** → Paramètres → Connecteurs → activer **GitHub** (ou utiliser une Custom GPT avec Action GitHub).
2. Donner accès au dépôt `johndev38/ukulele_tabs`.
3. Coller le prompt ci-dessus + référencer ce fichier : « Lis AGENTS.md dans le repo ».
4. Demander explicitement : « commit et push sur main ».

Alternative sans connecteur : ChatGPT produit les deux fichiers JSON ; l’humain copie-colle et pousse.

---

## Messages de commit

Format recommandé :

```text
Ajoute {id} — {titre court}
```

Exemples :

- `Ajoute la-cucaracha — comptine mexicaine`
- `Ajoute greensleeves — arrangement débutant`

---

## Erreurs fréquentes à éviter

| Erreur | Correction |
|--------|------------|
| `id` dupliqué | Choisir un nouvel identifiant kebab-case |
| `midi` incohérent avec corde/case | Recalculer : MIDI ouvert + fret |
| Notes simultanées | Décaler en `startTick` (monophonique) |
| `events` non triés | Trier par `startTick` |
| Oubli de `catalog.json` | L’app ne verra pas le morceau |
| Copyright récent sans licence | Utiliser domaine public ou original uniquement |
| `ticksPerBeat` ≠ 480 | Toujours 480 |

---

## Référence rapide — notes en première position

| Note | MIDI | Corde | Case |
|------|-----:|------:|-----:|
| C4 | 60 | 3 | 0 |
| D4 | 62 | 3 | 2 |
| E4 | 64 | 2 | 0 |
| F4 | 65 | 2 | 1 |
| G4 | 67 | 4 | 0 |
| A4 | 69 | 1 | 0 |
| B4 | 71 | 1 | 2 |
| C5 | 72 | 1 | 3 |

---

## Fichiers de référence dans ce dépôt

- `templates/song.template.json` — squelette vide
- `ode-to-joy.json` — exemple complet valide
- Schéma formel (projet parent) : `ukulele_app/schemas/song.schema.json`
