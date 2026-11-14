# TP3 — Réponses

**monôme :** Mohamed Yessine Allani

## Partie 1 — Pyramide des tests

**Q1.** Pour chacun des dossiers, voici les informations :

| Dossier | Niveau de la pyramide | Outil utilisé | Stack Docker nécessaire ? |
|---|---|---|---|
| `tests/unit` | Tests unitaires | Vitest | Non |
| `tests/component` | Tests de composants | Vitest + React Testing Library | Non |
| `tests/integration` | Tests d'intégration | Vitest | Oui |
| `tests/e2e` | Tests de bout en bout | Playwright | Oui |

**Q2.**

La pyramide des tests est large à la base et étroite au sommet pour plusieurs raisons :

- **La durée d'exécution :** les tests unitaires sont rapides, tandis que les tests E2E sont plus longs.
- **Le coût :** les tests unitaires sont simples et peu coûteux, contrairement aux tests E2E qui nécessitent plusieurs services.
- **La complexité :** les tests unitaires vérifient une fonction isolée, alors que les tests E2E vérifient toute l'application.

## Partie 2 — Tests unitaires

**Q3.**

Lors de la première exécution, 17 tests ont été exécutés : 16 ont réussi et 1 a échoué.

Le message d'erreur était :

```text
Expected: "1 h"
Received: "1 h 0"
```

- `Expected` représente le résultat attendu par le test.
- `Received` représente le résultat réellement retourné par la fonction.

Le test a échoué parce que la fonction ajoutait 0 minute alors qu'il ne restait aucune minute.

**Q4.**

Le bug se trouvait dans la fonction `formatDuration`. Lorsque la durée était un multiple de 60, la fonction affichait également les minutes restantes, même si elles étaient égales à zéro.

La correction consiste à vérifier si `rest === 0`. Dans ce cas, la fonction retourne uniquement le nombre d'heures. Sinon, elle retourne les heures et les minutes.

```typescript
export function formatDuration(minutes: number): string {
  if (minutes < 60) {
    return `${minutes} min`;
  }

  const hours = Math.floor(minutes / 60);
  const rest = minutes % 60;

  return rest === 0 ? `${hours} h` : `${hours} h ${rest}`;
}
```

**Q5.**

La couverture initiale de `lib/cours.ts` avant l'ajout des deux tests supplémentaires n'a pas été conservée dans les résultats disponibles.

Il faut distinguer cette couverture initiale de la couverture obtenue après l'ajout des tests. Les valeurs exactes des lignes, des branches et des fonctions doivent être relevées dans le premier rapport HTML de couverture.

**Q6.**

Après l'ajout des deux tests, la couverture de `lib/cours.ts` est :

| Indicateur | Couverture |
|---|---:|
| Statements | 94,11 % |
| Branches | 100 % |
| Functions | 85,71 % |
| Lines | 100 % |

Les deux tests supplémentaires vérifient :
- Le retour d'une liste vide lorsqu'aucun cours ne correspond aux critères.
- La combinaison de la recherche par mot-clé et du filtrage par niveau.

La couverture des branches et des lignes atteint 100 %. Certaines instructions et fonctions restent cependant non couvertes.

**Q7.**

La couverture des lignes indique le pourcentage de lignes de code exécutées pendant les tests.

La couverture des branches indique le pourcentage des différentes possibilités d'exécution des conditions qui ont été testées.

Un code couvert à 100 % n'est pas forcément sans bug. Par exemple, dans notre TP, la fonction `formatDuration` retournait un résultat incorrect pour une durée de 60 minutes. La couverture seule ne garantit donc pas que les résultats sont corrects.

## Partie 3 — Tests de composant

**Q8.**

Le test `CoursCard.test.tsx` vérifie le comportement du composant `CoursCard`, notamment l'affichage du lien, de la durée, du niveau et des mots-clés.

Contrairement au test unitaire de `formatDuration`, il vérifie que les informations sont correctement affichées dans le composant React.

Les tests de composants utilisent React Testing Library et un environnement DOM simulé par JSDOM. La configuration de cet environnement est définie dans la configuration Vitest du projet.

**Q9.**

Les méthodes `getByRole` et `getByText` permettent de rechercher les éléments selon leur rôle ou leur contenu textuel, comme le ferait un utilisateur.

Elles rendent les tests plus fiables et plus proches de l'utilisation réelle de l'application. Les tests basés sur les classes CSS sont plus fragiles, car une modification du style peut les faire échouer même si le comportement reste correct.

**Q10.**

Il n'y a pas de test de composant spécifique pour `app/page.tsx`, car cette page dépend de l'application complète et de la récupération des données depuis Strapi.

Elle est principalement vérifiée par les tests E2E, qui contrôlent le comportement de la page comme un utilisateur réel.

## Partie 4 — Tests d'intégration

**Q11.**

Les tests d'intégration vérifient la communication réelle entre l'application et l'API Strapi.

Ils contrôlent notamment :
- La récupération des six cours.
- La présence des informations nécessaires dans les cours.
- Le tri des cours selon leur session.
- La récupération d'un cours par son identifiant.
- Le comportement lorsqu'un cours n'existe pas.

Ces interactions avec le véritable service Strapi ne sont pas vérifiées par les tests unitaires ou les tests de composants isolés.

**Q12.**

Lorsque Strapi est arrêté, les quatre tests d'intégration échouent à cause de l'indisponibilité de l'API et du délai d'attente dépassé.

Les tests unitaires continuent de réussir parce qu'ils ne dépendent pas de Strapi.

L'option `--no-deps` permet d'exécuter un service sans démarrer automatiquement ses services dépendants. Les tests sont donc exécutés avec les services déjà présents dans l'environnement.

**Q13.**

Non, un test d'intégration en échec ne signifie pas forcément qu'il existe un bug dans le code.

Deux autres causes possibles sont :

- Un service externe, comme Strapi, est arrêté ou inaccessible.
- Une mauvaise configuration du réseau, du port ou des variables d'environnement.

Une base de données indisponible ou des données de test manquantes peuvent également provoquer un échec.

## Partie 5 — Tests de bout en bout

**Q14.**

Le scénario qui échoue concerne l'affichage de la durée sur la page de détail d'un cours.

La première exécution a réussi pour trois tests sur quatre. Le problème venait de l'ancien conteneur frontend qui affichait encore `1 h 0` au lieu de `1 h`.

Ce problème est lié à la question Q3, car il concerne le même bug dans `formatDuration`.

Même après la correction du code, l'ancien conteneur Docker utilisait encore l'ancienne version de l'application. La reconstruction du frontend a permis de résoudre le problème.

**Q15.**

Le test unitaire permet de trouver la cause du bug plus rapidement, car il vérifie directement la fonction concernée.

Le test E2E permet de prouver que le problème touche réellement l'utilisateur dans l'application complète.

Les deux tests sont complémentaires.

**Q16.**

| Niveau | Nombre de tests | Durée d'exécution | Besoin de la stack ? |
|---|---:|---:|---|
| Unitaire + composant | 19 | 35,02 s | Non |
| Intégration | 4 | 8,88 s | Oui |
| E2E | 4 | 37,5 s | Oui |

**Q17.**

Un test flaky est un test instable qui peut réussir ou échouer de manière aléatoire, même si le code n'a pas changé.

Playwright réduit ce risque grâce à l'attente automatique des éléments et aux assertions comme `expect(...).toHaveCount()`.

Ces mécanismes attendent que la condition attendue soit satisfaite avant de considérer le test comme réussi ou échoué, dans la limite du délai configuré.

## Partie 6 — SonarQube

**Q18.**

Voici la configuration utilisée dans `sonar-project.properties` :

```properties
sonar.projectKey=tp3-cours
sonar.projectName=TP3 - Cours DevOps

sonar.sources=app,components,lib
sonar.tests=tests

sonar.javascript.lcov.reportPaths=coverage/lcov.info

sonar.exclusions=node_modules/**,.next/**,coverage/**,playwright-report/**,test-results/**

sonar.coverage.exclusions=app/**,lib/api.ts,components/CoursList.tsx,components/DevOpsLoop.tsx,*.config.*

sonar.sourceEncoding=UTF-8
sonar.qualitygate.wait=true
```

Explication :

- `sonar.projectKey` : identifiant unique du projet.
- `sonar.projectName` : nom du projet dans SonarQube.
- `sonar.sources` : dossiers contenant le code source.
- `sonar.tests` : dossier contenant les tests.
- `sonar.javascript.lcov.reportPaths` : emplacement du rapport de couverture.
- `sonar.exclusions` : fichiers générés ou dossiers inutiles exclus de l'analyse.
- `sonar.coverage.exclusions` : fichiers exclus du calcul de couverture.
- `sonar.sourceEncoding` : encodage UTF-8.
- `sonar.qualitygate.wait=true` : attend le résultat du Quality Gate.

On exclut `app/**` et `lib/api.ts` du calcul de couverture parce que ces parties dépendent de l'application complète ou de services externes et ne sont pas couvertes par les tests unitaires classiques.

**Q19.**

Lors de l'analyse initiale avec le Quality Gate TP3, le statut était **Failed**.

Les résultats obtenus étaient :

| Indicateur | Résultat |
|---|---:|
| Bugs | 0 |
| Vulnerabilities | 1 |
| Security Hotspots | 0 |
| Code Smells | 14 |
| Coverage | 97,8 % |
| Duplications | 5,6 % |

Le Quality Gate échouait à cause de la présence d'une vulnérabilité, alors que la couverture respectait le seuil de 90 %.

Après les corrections, une nouvelle analyse a donné le statut **PASSED**.

**Q20.**

- **Bug :** erreur dans le code qui peut provoquer un mauvais fonctionnement.
- **Vulnerability :** faiblesse de sécurité pouvant être exploitée par un attaquant. Exemple : un mot de passe écrit directement dans le code.
- **Security Hotspot :** partie du code qui nécessite une vérification manuelle pour déterminer si elle présente un risque.
- **Code Smell :** problème de lisibilité ou de maintenabilité du code, sans forcément provoquer une erreur.

Dans notre projet, SonarQube a notamment détecté une vulnérabilité liée à un mot de passe écrit directement dans le code.

**Q21.**

La vulnérabilité concernait un mot de passe écrit directement dans le fichier `frontend/lib/api.ts`.

Le risque est qu'un utilisateur ayant accès au code puisse récupérer le mot de passe et l'utiliser pour accéder au service concerné.

La correction consiste à remplacer le mot de passe écrit directement dans le code par une variable d'environnement :

```typescript
process.env.STRAPI_CLIENT_PASSWORD ?? ""
```

Le mot de passe est maintenant stocké dans le fichier `.env` local, qui n'est pas envoyé sur GitHub.

Le chemin est donc le suivant :

1. Le mot de passe est défini dans `.env`.
2. Docker Compose transmet la variable au conteneur frontend.
3. Le code récupère sa valeur avec `process.env.STRAPI_CLIENT_PASSWORD`.

Même dans un dépôt privé, un secret écrit directement dans le code est dangereux, car il peut être exposé dans l'historique Git, les copies du projet ou les journaux.

**Q22.**

La correction de `CoursList.tsx` a pour objectif de simplifier le code et de supprimer les problèmes de maintenabilité.

Les modifications effectuées sont :

- Simplification des conditions imbriquées dans `describeResults`.
- Utilisation de retours anticipés pour éviter une structure trop complexe.
- Suppression des imports inutilisés, notamment `useEffect` et `sortBySession`.

Ces modifications rendent le code plus lisible et plus facile à maintenir.

**Q23.**

Le taux de duplication initial était de 5,6 % sur 856 lignes analysées.

Le code dupliqué correspond à des instructions ou blocs de code identiques ou similaires présents à plusieurs endroits du projet.

Pour supprimer ces duplications, on pourrait extraire les parties communes dans une fonction ou un composant réutilisable, puis appeler cette fonction aux différents endroits.

**Q24.**

La dette technique représente le temps estimé nécessaire pour corriger les problèmes de qualité du code et améliorer sa maintenabilité.

SonarQube l'exprime sous forme d'un effort de correction estimé, par exemple en minutes, heures ou jours.

Les valeurs exactes avant et après les corrections doivent être relevées dans la rubrique Technical Debt de SonarQube.

**Q25.**

Le New Code correspond au code ajouté ou modifié depuis une période de référence définie dans SonarQube.

Il permet de contrôler la qualité des nouvelles modifications sans être pénalisé uniquement par les problèmes déjà présents dans le projet.

Dans notre configuration du TP3, les conditions du Quality Gate ont été configurées sur le code global (Overall Code) :

- Coverage inférieur à 90 %.
- Vulnerabilities supérieur à 0.

Il n'y avait donc pas de condition spécifique configurée sur le New Code.

SonarQube recommande de contrôler particulièrement le nouveau code afin d'éviter d'introduire de nouveaux problèmes et d'améliorer progressivement la qualité du projet.

**Q26.**

Avec `sonar.qualitygate.wait=true`, le scanner attend le résultat du Quality Gate après l'analyse.

Si le Quality Gate est rouge (Failed), le scanner retourne un code d'erreur.

C'est utile dans un pipeline CI/CD, car cela permet de bloquer automatiquement la validation ou le déploiement d'un code qui ne respecte pas les conditions de qualité et de sécurité.

## Partie 7 — Synthèse

**Q27.**

L'ordre proposé pour le pipeline CI est :

1. Récupération du code source depuis Git.
2. Installation des dépendances.
3. Vérification et compilation du projet.
4. Exécution des tests unitaires.
5. Exécution des tests de composants.
6. Génération du rapport de couverture.
7. Démarrage des services nécessaires avec Docker Compose.
8. Exécution des tests d'intégration.
9. Construction de l'image Docker du frontend.
10. Exécution des tests E2E avec Playwright.
11. Analyse du code avec SonarQube.
12. Vérification du Quality Gate.
13. Déploiement si toutes les étapes précédentes réussissent.

Les tests unitaires et de composants sont exécutés en premier parce qu'ils sont rapides et permettent de détecter les erreurs rapidement. Les tests d'intégration et E2E viennent ensuite, car ils nécessitent plusieurs services.

Enfin, SonarQube vérifie la qualité et la sécurité du code.

Les étapes qui doivent bloquer le merge d'une Pull Request sont :

- La compilation du projet.
- Les tests unitaires et de composants.
- Les tests d'intégration.
- Les tests E2E.
- Le Quality Gate SonarQube.

Cela permet de garantir que le code fusionné fonctionne correctement et respecte les exigences de qualité.