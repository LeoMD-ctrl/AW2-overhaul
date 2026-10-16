---
type: index
---
# Accueil

## Projets
- [[AW2 Overhaul]]

## Idées en réserve (tous projets)
```dataview
TABLE projet, theme
FROM ""
WHERE type = "idee" AND statut = "reserve"
SORT file.mtime DESC
```

## Patterns réutilisables
```dataview
LIST
FROM "30 Patterns"
SORT file.name ASC
```

## Moteurs évalués
```dataview
TABLE verdict, langage
FROM "20 Moteurs"
```
