# Guide svn-precommit-review

`svn-precommit-review` est une compétence Codex Agent qui vérifie une copie de travail SVN/TortoiseSVN avant le commit et rédige un message de commit en coréen. Par défaut, elle effectue la revue et exécute automatiquement `svn add` pour les nouveaux fichiers nécessaires qui respectent les critères de sécurité. Le commit nécessite une demande de l'utilisateur et une seule confirmation finale du périmètre et du message ; seul le périmètre approuvé est committé.

## Installation

```text
%USERPROFILE%\.codex\skills\svn-precommit-review\SKILL.md
```

ou :

```text
<project-root>\.agents\skills\svn-precommit-review\SKILL.md
```

Copiez `svn-precommit-review/SKILL.md` de ce dépôt vers l'emplacement choisi.

## Utilisation

Revue et préparation par défaut des fichiers nécessaires (une demande générale comme « vérifie » ou `검토해줘` fait de même) :

```text
$svn-precommit-review
```

Vérification en lecture seule :

```text
$svn-precommit-review Vérifie uniquement les changements SVN actuels en lecture seule. N'exécute pas svn add/delete/commit.
```

Demander une revue et un commit :

```text
$svn-precommit-review Vérifie les changements de <SVN_WORKING_COPY> et effectue le commit.
```

Si vous avez demandé un commit, vérifiez les chemins inclus et exclus, les add/delete préparés, les risques restants et le message, puis répondez une fois à la question finale. « Effectue le commit » ou `커밋해줘` suffit ; le terme `CLI` n'est pas nécessaire.

```text
네, 진행해
```

## Comportement

- Vérifie `svn status --xml`, l'état distant, les diffs de contenu et de propriétés.
- Vérifie les fichiers générés, les fichiers IDE/utilisateur, les journaux, la configuration locale, les données sensibles, les conflits, les fichiers manquants et les lacunes de tests.
- Lors de la revue par défaut, exécute obligatoirement et automatiquement `svn add` pour les fichiers non versionnés de source, de projet, de ressources et de configuration non réservée à l'environnement local dont la nécessité est établie par des références dans les fichiers suivis ou les changements examinés. Si les critères sont remplis, une simple recommandation ne suffit pas.
- Les demandes explicites de lecture seule, de revue seule, de message seul ou d'absence de changements interdisent toute écriture. Une demande générale de vérification inclut la préparation par défaut.
- Un ajout n'est déclaré terminé qu'après la réussite de la commande add et la confirmation de l'état `A` par `svn status`. Les ajouts terminés figurent dans le résumé des fichiers, pas parmi les avertissements non résolus.
- Un add sûr et indépendant peut se poursuivre malgré un risque restant dans le code. Ce risque reste signalé ; l'ajout n'est pas présenté comme une correction du code.
- N'ajoute pas `bin`, `obj`, fichiers générés, fichiers IDE/utilisateur, journaux, fichiers temporaires, caches, configuration locale, archives, dumps, sauvegardes, captures d'écran, fichiers ignorés, `.env`, fichiers susceptibles de contenir des secrets ou fichiers ambigus. Tout le contenu d'un répertoire est examiné récursivement avant son ajout ; si l'ensemble n'est pas sûr, seuls les chemins précis et sûrs sont ajoutés.
- Si l'utilisateur demande d'inclure les candidats à la suppression examinés, seule la suppression de chemins locaux versioned propres est planifiée.
- Ne supprime pas de chemins modifiés localement, en conflit, switched, external, locked ou ambigus.
- Le commit nécessite une demande et l'approbation finale du périmètre exact et du message. L'approbation antérieure du même périmètre et du même contenu reste valable ; status et les diffs de contenu et de propriétés sont revérifiés juste avant le commit. Une nouvelle revue et une nouvelle confirmation ne sont requises qu'en cas de changement substantiel du périmètre ou du contenu, ou de changement d'un état bloquant.
- Si un commit manuel dans TortoiseSVN est explicitement choisi, termine avec la revue et la liste de contrôle, sans commit CLI ni question pour l'exécuter. La préparation sûre par défaut reste active sauf demande de lecture seule.
- N'utilise pas de `svn add`, `svn delete` ni `svn commit` à large périmètre.

Les indications pendant l'exécution, la revue, le message de commit et les résultats sont toujours rédigés en coréen. Les résumés sont regroupés par intention, sans nombre fixe de points ni détails excessifs. Les tests antérieurs ne sont réutilisés qu'avec des preuves réelles applicables au contenu examiné resté inchangé, et sont signalés comme résultats antérieurs, non réexécutés.

## Messages de commit en coréen

Pour les commits CLI, crée hors de la copie de travail un fichier temporaire unique en UTF-8 sans BOM, avec des fins de ligne LF, et utilise `--file` et `--encoding UTF-8` :

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

Le fichier temporaire est supprimé après chaque tentative et son absence est vérifiée. Lorsque l'accès au dépôt le permet, le message du journal XML de la révision réelle identifiée dans la sortie du commit est comparé caractère par caractère à l'original. Si le coréen stocké est endommagé, la compétence signale un échec de vérification d'encodage et ne réessaie pas le commit ni ne modifie les revision properties.
