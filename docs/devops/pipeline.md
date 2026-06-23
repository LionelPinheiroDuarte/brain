# Pipeline CI/CD

## CI — Continuous Integration

Continuous integration (CI) is the practice of automatically running a series of checks on every push. The goal is to catch regressions as early as possible, before they reach production.

A typical CI pipeline follows this order:

```
push → lint → unit tests → build → E2E tests → deploy
```

Each step blocks the next. If any step fails, the pipeline stops and the deployment never runs.

---

### E2E Tests

Unit and integration tests verify that the code works. E2E tests verify that the user can actually do what they're supposed to do — they test the delivered output, not the source code.

Problems only E2E tests catch:

- A link points to the wrong URL after a permalink change
- A page returns a directory listing instead of HTML
- The language switcher breaks after a refactor
- An element is hidden by CSS

**Place in the pipeline:** E2E tests run after the build, against the real deployed artifact. They are few in number and focused on critical user flows — navigation, key content, and core interactions.

```
        /\
       /E2E\        ← few, slow, critical paths only
      /------\
     /  Integr.\
    /------------\
   /  Unit tests  \
  /________________\
```

---

## GitHub Actions — Setup pratique

### Structure minimale

```yaml
name: CI

on:
  push:
    branches: [dev]
  pull_request:
    branches: [dev, main]

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm run lint
      - run: npm run typecheck
      - run: npm test -- --passWithNoTests
```

Déclenché sur push vers `dev` **et** sur les PRs ciblant `dev` ou `main` — ça couvre les deux points d'entrée sans doublons.

`--passWithNoTests` évite que le pipeline échoue si aucun test n'existe encore sur une nouvelle branche. À retirer si on veut imposer qu'au moins un test existe.

### Ce que lint + typecheck testent — et ce qu'ils ne testent pas

C'est une distinction importante à avoir en tête :

| Outil | Ce qu'il vérifie | Ce qu'il ne voit pas |
|-------|-----------------|----------------------|
| **ESLint** | Style, règles de codage, imports inutilisés, hooks mal utilisés | Comportement à l'exécution |
| **TypeScript (`tsc --noEmit`)** | Cohérence des types, API correctement appelées | Logique métier, valeurs incorrectes |
| **Tests unitaires** | Que le code fait ce qu'il est censé faire | L'intégration avec le device, l'UI réelle |

TypeScript vérifie qu'on appelle une fonction avec les bons types. Il ne vérifie pas que la fonction retourne le bon résultat pour un cas limite donné. C'est le rôle des tests.

---

## Tests unitaires dans un contexte React Native / Expo

### La contrainte fondamentale

React Native dépend de modules natifs (SQLite, camera, file system) qui ne s'exécutent pas dans Node.js. Jest tourne dans Node — pas dans un simulateur, pas sur un device. Conséquence : **on ne peut pas tester tout le code en Jest**.

Ce qu'on peut tester :

- **Logique pure** : transformations, parsers, extracteurs d'URL, machines d'états
- **Hooks** : transitions d'état, annulation de requêtes, comportement sur erreur — avec `@testing-library/react-native` et des mocks du service sous-jacent
- **Stores** : logique métier qui ne touche pas directement SQLite (en mockant la couche DB)

Ce qu'on ne peut pas tester en Jest sans hacks fragiles :

- Appels SQLite natifs
- `expo-file-system`, `expo-image-picker`
- Rendu visuel réel, gestes

### Quoi tester en priorité

La règle pratique : un bug coûte cher quand il est **silencieux** (pas de crash, pas de log visible) ET **irréversible** (perte de données, corruption de l'état persisté).

```
Priorité haute   →  écriture en base, machines d'états, transformations sans exception
Priorité faible  →  rendu conditionnel, styles, cas déjà couverts par TypeScript
```

Exemples de cibles à fort ROI :
- Les transitions d'état d'un hook de recherche (idle → loading → success/error/not_found)
- La déduplication avant écriture (comparaison case-insensitive)
- Un extracteur d'URL avec plusieurs formats d'entrée possibles

### Stack — jest-expo

`jest-expo` est le preset officiel d'Expo. Il configure Babel, les mocks des modules natifs, et les `transformIgnorePatterns` pour que les packages ES modules du monorepo Expo soient transpilés correctement.

Dépendances requises (SDK 56 / React 19) :

```
jest-expo@~56.0.5
jest@29                          ← jest-expo est câblé pour Jest 29, pas 30
@testing-library/react-native@14
@react-native/jest-preset        ← peer dep de jest-expo
test-renderer                    ← peer dep de RTLN v14 (distinct de react-test-renderer)
react-test-renderer              ← gardé pour compatibilité
@types/jest                      ← pour les globals (describe, it, expect...)
```

`tsconfig.json` doit inclure `"types": ["jest"]` pour que TypeScript reconnaisse les globals Jest sans erreur.

### Colocalisés vs dossier `__tests__`

Les tests colocalisés (`foo.ts` → `foo.test.ts`) sont préférables à un dossier `__tests__` séparé : le fichier de test vit à côté du fichier testé, les imports sont relatifs et courts, et on voit immédiatement si un module n'a pas de test.

### Silencer les `console.error` attendus

Quand un hook loggue intentionnellement une erreur (`console.error('[hook]', error)`), le test qui déclenche ce cas va polluer la sortie CI. La bonne pratique : spy + mock dans le test concerné, restore après.

```ts
it('logs and sets error state on network failure', async () => {
  const spy = jest.spyOn(console, 'error').mockImplementation(() => {});
  // ... test ...
  spy.mockRestore();
});
```

---

## Stratégie de release — pas un build à chaque push

Un pipeline CI qui build et soumet à chaque push sur `dev` est trop coûteux et génère trop de bruit dans l'outil de distribution. La séparation correcte :

```
push dev    →  CI léger  (lint + typecheck + tests)         ~3 min, gratuit
tag beta-*  →  Build EAS  (Android/iOS) + submit internal   ~15-20 min, consomme des crédits EAS
tag v*      →  Build EAS  + submit production
```

Le tag est l'acte délibéré du développeur : "je déclare que ce commit mérite un build distribué." Ça évite de gaspiller des builds EAS sur des commits de travail en cours.

### Semantic Versioning avec tags annotés

```bash
# Sur main après merge de dev
git tag -a v2.1.0 -m "feat: image par définition + CI pipeline"
git push origin main --tags
```

| Incrément | Critère |
|-----------|---------|
| MAJOR `X.0.0` | Changement de schéma DB, rupture de flux existants |
| MINOR `0.X.0` | Feature additive, flux existants inchangés |
| PATCH `0.0.X` | Correctif, polish, docs |

---

## Production safety — exclure le code de dev du bundle

Les bundlers comme Metro (React Native) incluent par défaut tout ce qui est importé statiquement. Si un fichier de seed de données de test est importé en haut d'un fichier source, il finit dans le bundle de production.

La solution : **dynamic require à l'intérieur d'un guard `__DEV__`**.

```ts
// ❌ import statique — Metro inclut seed.ts dans le bundle prod
import { seedDatabase } from '../src/db/seed';

// ✅ require dynamique dans __DEV__ — Metro l'élimine en prod (dead-code elimination)
if (__DEV__) {
  globalThis.__testSeed = () => {
    (require('../src/db/seed') as typeof import('../src/db/seed')).seedDatabase();
  };
}
```

Metro évalue `__DEV__` au moment du bundle : en production il devient `false`, le bloc `if(false){...}` est éliminé, et le `require` ne résout jamais. Le module seed n'apparaît pas dans le bundle final.
