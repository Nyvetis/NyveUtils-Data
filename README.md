# nyveutils-data

Données distantes du mod **NyveUtils** (Fabric, Hypixel SkyBlock, Minecraft 26.1.2).
Le mod télécharge ces fichiers via `raw.githubusercontent.com` : modifier un fichier ici
et pousser sur `main` suffit, sans republier le mod.

Ce dépôt ne contient que des données. Une GitHub Action valide chaque fichier contre son
JSON Schema à chaque push et pull request : si la validation échoue, ne fusionnez pas.

```
names.json                    pseudos personnalisés affichés dans le chat
themes.json                   thèmes de couleurs du menu
schemas/names.schema.json     schéma de names.json
schemas/themes.schema.json    schéma de themes.json
.github/workflows/validate.yml
```

## URLs raw

Remplacez `<utilisateur>` par le compte ou l'organisation GitHub qui héberge le dépôt :

```
https://raw.githubusercontent.com/<utilisateur>/nyveutils-data/main/names.json
https://raw.githubusercontent.com/<utilisateur>/nyveutils-data/main/themes.json
```

Le CDN de GitHub met en cache les fichiers raw environ 5 minutes : une modification peut
mettre ce délai à apparaître en jeu.

## names.json

Objet dont chaque **clé est le pseudo Minecraft exact** (sensible à la casse, 3 à 16
caractères `A-Z a-z 0-9 _`).

| Champ     | Type                | Obligatoire | Description |
|-----------|---------------------|-------------|-------------|
| `display` | string              | oui         | Texte affiché à la place du pseudo. |
| `effect`  | `"solid"` \| `"gradient"` \| `"chroma"` | oui | Rendu : couleur unie, dégradé fixe, ou dégradé animé. |
| `colors`  | liste de `"#RRGGBB"` | oui        | Exactement 1 couleur pour `solid`, 2 ou plus pour `gradient` et `chroma`. |
| `speed`   | nombre > 0          | non         | Vitesse de l'animation. Autorisé uniquement avec `chroma`. |
| `bold`    | booléen             | non         | Gras (défaut : `false`). |
| `italic`  | booléen             | non         | Italique (défaut : `false`). |

Exemple :

```json
{
  "Nyvetis": {
    "display": "Nyvetis",
    "effect": "chroma",
    "colors": ["#B18CFF", "#FF7EC4", "#5C9DFF"],
    "speed": 1.0,
    "bold": true,
    "italic": false
  }
}
```

### Ajouter un pseudo

1. Ouvrez `names.json` et ajoutez une entrée dont la clé est le pseudo exact du joueur :
   ```json
   "Steve": {
     "display": "Steve",
     "effect": "gradient",
     "colors": ["#FF8A3D", "#FF4D6D"]
   }
   ```
2. N'oubliez pas la virgule entre deux entrées, et aucune virgule après la dernière.
3. Ouvrez une pull request (ou poussez) et vérifiez que l'action **Validate data** passe.

## themes.json

Objet dont chaque **clé est l'id du thème** (minuscules, chiffres, `_` ; ex. `dark`,
`skyblock`). L'id est ce que le mod enregistre dans la config : ne le renommez pas
une fois publié.

Toutes les couleurs sont au format **`#AARRGGBB`** (alpha en premier), c'est-à-dire
exactement la valeur `Int` ARGB du mod : `0xF0111116.toInt()` s'écrit `"#F0111116"`.
`FF` = opaque, `00` = transparent.

Tous les champs sont obligatoires et aucun champ inconnu n'est accepté.

| Champ          | Rôle |
|----------------|------|
| `name`         | Nom affiché dans le sélecteur de thème. |
| `overlay`      | Voile par-dessus le jeu flouté. |
| `panel`        | Fond du panneau (légèrement transparent : effet verre). |
| `header`       | Barre latérale. |
| `surface`      | Cartes, boutons au repos. |
| `surfaceHover` | Cartes, boutons au survol. |
| `border`       | Contours fins (souvent du blanc/noir très transparent). |
| `track`        | Rail de slider, interrupteur éteint. |
| `accent`       | Couleur d'accent principale. |
| `accentHover`  | Accent au survol. |
| `accent2`      | Seconde couleur d'accent : fin des dégradés (préfixe du chat). |
| `onAccent`     | Texte posé sur l'accent. |
| `text`         | Texte principal. |
| `textMuted`    | Texte secondaire. |

Thèmes fournis : `dark`, `skyblock`, `nyve`, `cyan` (Noir & Cyan), `ember` (Braise),
`emerald` (Émeraude).

### Ajouter un thème

1. Copiez un thème existant dans `themes.json`, donnez-lui un nouvel id unique et un `name`.
2. Modifiez les couleurs (`#AARRGGBB`). Gardez `panel` légèrement transparent (alpha
   autour de `F0`) pour l'effet verre, et `border` très transparent (alpha `1F` à `29`).
3. Vérifiez le contraste de `text` sur `panel` et de `onAccent` sur `accent`.
4. Ouvrez une pull request (ou poussez) et vérifiez que l'action **Validate data** passe.

## Lecture côté mod (Kotlin)

```kotlin
// "#AARRGGBB" -> Int ARGB
fun argb(hex: String): Int = hex.removePrefix("#").toLong(16).toInt()

// "#RRGGBB" -> Int ARGB opaque
fun rgb(hex: String): Int = (0xFF000000 or hex.removePrefix("#").toLong(16)).toInt()
```

Gardez les thèmes intégrés au mod comme repli si le téléchargement échoue ou si un thème
distant est invalide.

## Valider en local

Avec Python :

```
pip install check-jsonschema
check-jsonschema --schemafile schemas/names.schema.json names.json
check-jsonschema --schemafile schemas/themes.schema.json themes.json
```
