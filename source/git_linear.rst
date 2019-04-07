.. -*- coding: utf-8 -*-
.. Copyright |copy| 2013 by Benoit Legat
.. Ce fichier est distribué sous une licence `creative commons <http://creativecommons.org/licenses/by-sa/3.0/>`_


Utilisation linéaire de Git
~~~~~~~~~~~~~~~~~~~~~~~~~~~

On peut utiliser `Git`_ à plusieurs niveaux.
On peut tout à fait avoir un historique linéaire tout en profitant pleinement
de `Git`_.
Pour ceux dont c'est la première fois qu'ils utilisent un système de contrôle
de version,
il vaut peut-être mieux commencer par ne lire que cette partie et
déjà l'utiliser de façon plus basique pour le premier projet et
lire les parties suivantes une fois que vous voulez consolider
votre connaissance et apprendre plus en détails comment ça fonctionne
sans toutefois voir toutes ses possibilités.
Toutefois, la lecture de cette partie n'est pas nécessaire pour comprendre
les parties suivantes donc si vous voulez juste parfaire votre
connaissance, vous pouvez la passer sans crainte.

Créer un historique linéaire
############################

Un historique linéaire est un historique comme on l'imagine avec des versions
l'une après l'autre, chaque version étendant la précédente avec
certaines modifications.
On verra par après qu'il est possible d'avoir un historique non-linéaire
avec `Git`_ mais ce n'est pas indispensable.

Sur `Git`_, on appelle une version un *commit*.
Chacun de ces commits est documenté en fournissant le nom de l'auteur,
son email, un commentaire et une description (optionnelle).
Pour ne pas devoir respécifier le nom et l'email à chaque fois,
on le stocke dans le fichier de configuration de `Git`_ ``~/.gitconfig``.
Bien qu'on peut l'éditer manuellement, on préfère le faire à l'aide de
la commande `git-config(1)`_.

Pour spécifier le commentaire,
`git-commit(1)`_ ouvrira un éditeur de texte.
Pour entrer une description, laissez une ligne vide puis écrivez la.
L'éditeur de texte à ouvrir est déterminé par `Git`_ en fonction de la variable
``core.editor`` du fichier de configuration mentionné plus haut.
Vous pouvez aussi spécifier le commentaire à l'aide de l'option ``-m``
de `git-commit(1)`_ comme on verra dans les exemples par après.

Voici les commandes à exécuter pour configurer le nom, l'email et l'éditeur
de texte.
Vous devez bien entendu remplacer les valeurs par celles qui vous conviennent.

.. code-block:: bash

   $ git config --global user.name "Jean Dupont"
   $ git config --global user.email jean@dupont.com
   $ git config --global core.editor gedit

L'option ``--global`` spécifie qu'on veut que ces configurations s'appliquent
pour tous nos dépôts (`Git`_ éditera le fichier ``~/.gitconfig``).
Sinon, `git-config(1)`_ ne modifie que le fichier
``.git/config`` à l'intérieur du *git directory* du projet en cours.
Ce dernier prône bien entendu sur ``~/.gitconfig`` quand une variable
a des valeurs différentes dans ``~/.gitconfig`` et ``.git/config``.

Vous voilà paré pour créer votre premier dépôt `Git`_
mais avant de voir comment faire des nouveaux commits,
il est impératif de comprendre ce qu'est la *staging area*.

Il y a 3 états dans lequel un changement de fichier peut-être,
 - il peut être dans le *working directory*,
   c'est à dire que c'est le fichier tel qu'il est actuellement dans le code;
 - il peut être dans la *staging area*,
   c'est à dire qu'il sera pris en compte dans le prochain commit;
 - et il peut être dans le *git directory*, c'est à dire sauvegardé dans
   un commit à l'intérieur du dossier ``.git``.

Pour committer des changements, on les met d'abord dans la
*staging area* puis on commit.
Cette flexibilité permet de ne pas committer
tout les changements du *working directory*.

Voyons tout ça avec un programme exemple qui affiche en LaTex
la somme des entiers de :math:`1` à :math:`n`.
On va utiliser les commandes

 * `git-init(1)`_ qui permet de transformer un projet en dépôt `Git`_
   (tout est stocké dans le dossier ``.git``);
 * `git-diff(1)`_ qui donne la différence entre l'état des fichiers dans le
   *working directory* avec leur état dans le *git directory*
   au commit actuel;
 * `git-status(1)`_ qui affiche les fichiers modifiés et ceux qui vont être
   committés;
 * `git-add(1)`_ qui spécifie quels changements doivent faire partie
   du prochain commit en les ajoutant à la *staging area*;
 * `git-commit(1)`_ qui commit les fichiers dans la *staging area*;
 * et `git-log(1)`_ qui montre tous les commits de l'historique.

La première version sera la suivante

.. code-block:: c

   #include <stdio.h>
   #include <stdlib.h>

   int main (int argc, char *argv[]) {
     long int sum = 0, i, n = 42;
     for (i = 1; i <= n; i++) {
       sum += i;
     }
     printf("\\sum_{i=1}^{ %ld } i = %ld\n", n, sum);
     return EXIT_SUCCESS;
   }

Ce programme fonctionne comme suit

.. code-block:: bash

   $ gcc main.c
   $ ./a.out
   \sum_{i=1}^{42} i = 903

On va sauvegarder un premier commit contenant cette version de ``main.c``

`git-init(1)`_ permet d'initialiser le dépôt `Git`_.
`git-status(1)`_ analyse le contenu du répertoire.
Il indique que le fichier ``main.c`` n'est pas suivi par `Git`_ (`untracked`).
Ce fichier est ajouté avec la commande `git-add(1)`_.
`git-commit(1)`_ sauvegarde cette version du code dans un commit
dont le commentaire, spécifié avec l'option ``-m``, est *First commit*.

.. code-block:: bash

   $ git init
   Initialized empty Git repository in /path/to/project/.git/
   $ git status
   # On branch master
   #
   # Initial commit
   #
   # Untracked files:
   #   (use "git add <file>..." to include in what will be committed)
   #
   #	main.c
   nothing added to commit but untracked files present (use "git add" to track)
   $ git add main.c
   $ git status
   # On branch master
   #
   # Initial commit
   #
   # Changes to be committed:
   #   (use "git rm --cached <file>..." to unstage)
   #
   #	new file:   main.c
   #
   $ git commit -m "First commit"
   [master (root-commit) 3d18efe] First commit
    1 file changed, 11 insertions(+)
    create mode 100644 main.c
   $ git log
   commit 3d18efe4df441ebe7eb2b8d0b78832a3861dc05f
   Author: Benoît Legat <benoit.legat@gmail.com>
   Date:   Sun Aug 25 15:32:42 2013 +0200

       First commit

Modifions maintenant le programme pour qu'il prenne la valeur de
:math:`n` dans ``argv``.
Si on compile le programme après modification, et qu'on exécute avec
en argument :math:`10` puis :math:`9.75`, on obtient ce qui suit

.. code-block:: bash

   $ gcc main.c
   $ ./a.out 10
   \sum_{i=1}^{10} i = 55
   $ ./a.out 9.75
   $ echo $?
   1

On peut maintenant voir avec `git-status(1)`_ que le fichier ``main.c``
a été modifié

.. code-block:: bash

   $ git status
   # On branch master
   # Changes not staged for commit:
   #   (use "git add <file>..." to update what will be committed)
   #   (use "git checkout -- <file>..." to discard changes in working directory)
   #
   #	modified:   main.c
   #
   no changes added to commit (use "git add" and/or "git commit -a")

Avec `git-diff(1)`_, on peut voir quelles sont les lignes qui ont été
retirées (elles commencent par un ``-``) et celles qui ont été ajoutées
(elles commencent par un ``+``).

.. code-block:: diff

   $ git diff
   diff --git a/main.c b/main.c
   index 86601ed..a9e4c4a 100644
   --- a/main.c
   +++ b/main.c
   @@ -2,7 +2,12 @@
    #include <stdlib.h>

    int main (int argc, char *argv[]) {
   -  long int sum = 0, i, n = 42;
   +  long int sum = 0, i, n;
   +  char *end = NULL;
   +  n = strtol(argv[1], &end, 10);
   +  if (*end != '\0') {
   +    return EXIT_FAILURE;
   +  }
      for (i = 1; i <= n; i++) {
        sum += i;
      }

Ajoutons ``main.c`` aux modifications à mettre dans le prochain commit puis
créons ce commit

.. code-block:: bash

   $ git add main.c
   $ git commit -m "Read n from argv"
   [master 56ce59c] Read n from argv
    1 file changed, 6 insertions(+), 1 deletion(-)

On peut maintenant voir le nouveau commit dans l'historique affiché par
`git-log(1)`_

.. code-block:: bash

   $ git log
   commit 56ce59c54726399c18b3f87ee23a45cf0d7f015d
   Author: Benoît Legat <benoit.legat@gmail.com>
   Date:   Sun Aug 25 15:37:51 2013 +0200

       Read n from argv

   commit 3d18efe4df441ebe7eb2b8d0b78832a3861dc05f
   Author: Benoît Legat <benoit.legat@gmail.com>
   Date:   Sun Aug 25 15:32:42 2013 +0200

       First commit

On va maintenant s'occuper d'un *segmentation fault* qui arrive
quand il n'y a pas d'argument.

.. code-block:: bash

   $ gcc main.c
   $ ./a.out
   Segmentation fault (core dumped)

Pour cela, on va simplement vérifier la valeur de ``argc`` et utiliser :math:`42` comme
valeur par défaut.
`git-diff(1)`_ nous permet de voir les changements qu'on a fait

.. code-block:: diff

   $ git diff
   diff --git a/main.c b/main.c
   index a9e4c4a..e906ea1 100644
   --- a/main.c
   +++ b/main.c
   @@ -2,11 +2,13 @@
    #include <stdlib.h>

    int main (int argc, char *argv[]) {
   -  long int sum = 0, i, n;
   +  long int sum = 0, i, n = 42;
      char *end = NULL;
   -  n = strtol(argv[1], &end, 10);
   -  if (*end != '\0') {
   -    return EXIT_FAILURE;
   +  if (argc > 1) {
   +    n = strtol(argv[1], &end, 10);
   +    if (*end != '\0') {
   +      return EXIT_FAILURE;
   +    }
      }
      for (i = 1; i <= n; i++) {
        sum += i;

On va maintenant committer ces changement
dans un commit au commentaire *Fix SIGSEV*

.. code-block:: bash

   $ git add main.c
   $ git commit -m "Fix SIGSEV"
   [master 7a26c63] Fix SIGSEV
    1 file changed, 6 insertions(+), 4 deletions(-)
   $ git log
   commit 7a26c6338c38614ce1c4ff00ac0a6895b57f15cb
   Author: Benoît Legat <benoit.legat@gmail.com>
   Date:   Sun Aug 25 15:39:49 2013 +0200

       Fix SIGSEV

   commit 56ce59c54726399c18b3f87ee23a45cf0d7f015d
   Author: Benoît Legat <benoit.legat@gmail.com>
   Date:   Sun Aug 25 15:37:51 2013 +0200

       Read n from argv

   commit 3d18efe4df441ebe7eb2b8d0b78832a3861dc05f
   Author: Benoît Legat <benoit.legat@gmail.com>
   Date:   Sun Aug 25 15:32:42 2013 +0200

       First commit

.. inginious:: git-add

.. inginious:: git-commit



Travailler à plusieurs sur un même projet
#########################################

`Git`_ est déjà un outil très pratique à utiliser seul mais c'est quand
on l'utilise pour se partager du code qu'il devient vraiment indispensable.
On se partage le code par l'intermédiaire de *remotes*.
Ce sont en pratique des serveurs auxquels on peut avoir l'accès lecture et/ou
écriture.
On va traiter ici le cas où deux développeurs, Alice et Bob,
ont l'accès lecture et écriture.

Alice va créer le projet avec

.. code-block:: bash

   $ git init
   Initialized empty Git repository in /path/to/project/.git/

puis elle créera une *remote*, c'est à dire un autre dépôt `Git`_ que celui
qu'ils ont en local, avec lequel ils vont pouvoir synchroniser leur
historique.
Supposons qu'ils aient un projet *projectname* sur Github.
Vous pouvez créer le *remote* comme suit

.. code-block:: bash

   $ git remote add https://github.com/alice/projectname.git

Ensuite, vous pourrez obtenir les modifications de l'historique du *remote*
avec ``git pull origin master``
et ajouter vos modifications avec ``git push origin master``.

Si vous exécutez ``git pull origin master``, que vous faites quelques
commits et puis que vous essayer de mettre *origin* à jour avec
``git push origin master``,
il faut qu'aucun autre développeur n'ait pushé de modification entre temps.
S'il en a pushé, `Git`_ ne saura pas effectuer votre *push*.
Il vous faudra alors faire un *pull*.
`Git`_ tentera alors de fusionner vos changements avec ceux d'*origin*.
Si ces derniers sont à une même ligne d'un même fichier, il vous demandera
de résoudre le conflit vous-même.
Il est important pour cela que vous ayez committé vos changements avant
le *pull* sinon `Git`_ l'abandonnera car il ne sait que fusionner des commits.
C'est à dire que ce qu'il y a dans le *git directory*,
pas ce qu'il y a dans le *working directory* ni dans la *staging area*.

Prenons un exemple où Bob *push* en premier puis Alice doit résoudre
un conflit.
Alice commence avec le fichier ``main.c`` suivant

.. code-block:: c

   #include <stdio.h>
   #include <stdlib.h>

   int main (int argc, char *argv[]) {
   }

Elle fait le premier commit du projet

.. code-block:: bash

   $ git add main.c
   $ git commit -m "Initial commit"
   [master (root-commit) 80507e3] Initial commit
    1 file changed, 5 insertions(+)
    create mode 100644 main.c

et va maintenant le *pusher* sur le serveur

.. code-block:: bash

   $ git push origin master
   Counting objects: 3, done.
   Delta compression using up to 4 threads.
   Compressing objects: 100% (2/2), done.
   Writing objects: 100% (3/3), 282 bytes, done.
   Total 3 (delta 0), reused 0 (delta 0)
   To https://github.com/alice/projectname.git
   * [new branch]      master -> master

Bob clone alors le projet pour en avoir une copie en local
ainsi que tout l'historique et la remote *origin* déjà configurée

.. code-block:: bash

   $ git clone https://github.com/alice/projectname.git
   Cloning into 'projectname'...
   remote: Counting objects: 3, done.
   remote: Compressing objects: 100% (2/2), done.
   remote: Total 3 (delta 0), reused 3 (delta 0)
   Unpacking objects: 100% (3/3), done.
   $ git remote -v
   origin	https://github.com/alice/projectname.git (fetch)
   origin	https://github.com/alice/projectname.git (push)

Ensuite, il ajoute ses modifications

.. code-block:: diff

   $ git diff
   diff --git a/main.c b/main.c
   index bf17640..0b0672a 100644
   --- a/main.c
   +++ b/main.c
   @@ -2,4 +2,5 @@
    #include <stdlib.h>

    int main (int argc, char *argv[]) {
   +  return 0;
    }

et les commit

.. code-block:: bash

   $ git add main.c
   $ git commit -m "Add a return statement"
   [master 205842a] Add a return statement
    1 file changed, 1 insertion(+)

et les push sur le serveur

.. code-block:: bash

   $ git push origin master
   Counting objects: 5, done.
   Delta compression using up to 4 threads.
   Compressing objects: 100% (2/2), done.
   Writing objects: 100% (3/3), 291 bytes, done.
   Total 3 (delta 1), reused 0 (delta 0)
   To https://github.com/alice/projectname.git
      80507e3..205842a  master -> master

Pendant ce temps là, Alice ne se doute de rien et
fait ses propres modifications

.. code-block:: diff

   $ git diff
   diff --git a/main.c b/main.c
   index bf17640..407cd8a 100644
   --- a/main.c
   +++ b/main.c
   @@ -2,4 +2,5 @@
    #include <stdlib.h>

    int main (int argc, char *argv[]) {
   +  return EXIT_SUCCESS;
    }

puis les commit

.. code-block:: bash

   $ git add main.c
   $ git commit -m "Add missing return statement"
   [master 73c6a3a] Add missing return statement
    1 file changed, 1 insertion(+)

puis essaie de les pusher

.. code-block:: bash

   $ git push origin master
   To https://github.com/alice/projectname.git
    ! [rejected]        master -> master (non-fast-forward)
   error: failed to push some refs to 'https://github.com/alice/projectname.git'
   hint: Updates were rejected because the tip of your current branch is behind
   hint: its remote counterpart. Merge the remote changes (e.g. 'git pull')
   hint: before pushing again.
   hint: See the 'Note about fast-forwards' in 'git push --help' for details.

mais `Git`_ lui fait bien comprendre que ce n'est pas possible.
En faisant le *pull*, on voit que `Git`_ fait de son mieux pour
fusionner les changements mais qu'il préfère nous laisser
choisir quelle ligne est la bonne

.. code-block:: bash

   $ git pull origin master
   remote: Counting objects: 5, done.
   remote: Compressing objects: 100% (1/1), done.
   remote: Total 3 (delta 1), reused 3 (delta 1)
   Unpacking objects: 100% (3/3), done.
   From https://github.com/alice/projectname
      80507e3..205842a  master     -> origin/master
   Auto-merging main.c
   CONFLICT (content): Merge conflict in main.c
   Automatic merge failed; fix conflicts and then commit the result.

Il marque dans ``main.c`` la ligne en conflit et ce qu'elle vaut
dans les deux commits

.. code-block:: c

   #include <stdio.h>
   #include <stdlib.h>

   int main (int argc, char *argv[]) {
   <<<<<<< HEAD
     return EXIT_SUCCESS;
   =======
     return 0;
   >>>>>>> 205842aa400e4b95413ff0ed21cfb1b090a9ef28
   }

On peut retrouver les fichiers en conflits dans
``Unmerged paths``

.. code-block:: bash

   $ git status
   # On branch master
   # You have unmerged paths.
   #   (fix conflicts and run "git commit")
   #
   # Unmerged paths:
   #   (use "git add <file>..." to mark resolution)
   #
   #	both modified:      main.c
   #
   no changes added to commit (use "git add" and/or "git commit -a")

Il nous suffit alors d'éditer le fichier pour lui donner le contenu
de la fusion

.. code-block:: c

   #include <stdio.h>
   #include <stdlib.h>

   int main (int argc, char *argv[]) {
     return EXIT_SUCCESS;
   }

puis de le committer

.. code-block:: bash

   $ git add main.c
   $ git commit
   [master eede1c8] Merge branch 'master' of https://github.com/alice/projectname

On peut alors mettre le serveur à jour

.. code-block:: bash

   $ git push origin master
   Counting objects: 8, done.
   Delta compression using up to 4 threads.
   Compressing objects: 100% (3/3), done.
   Writing objects: 100% (4/4), 478 bytes, done.
   Total 4 (delta 2), reused 0 (delta 0)
   To https://github.com/alice/projectname.git
      205842a..eede1c8  master -> master

Paul peut alors récupérer les changements avec

.. code-block:: bash

   $ git pull origin master
   remote: Counting objects: 8, done.
   remote: Compressing objects: 100% (1/1), done.
   remote: Total 4 (delta 2), reused 4 (delta 2)
   Unpacking objects: 100% (4/4), done.
   From https://github.com/alice/projectname
      205842a..eede1c8  master     -> origin/master
   Updating 205842a..eede1c8
   Fast-forward
    main.c | 2 +-
    1 file changed, 1 insertion(+), 1 deletion(-)

La plupart des fusions ne demande pas d'intervention manuelle mais
dans les cas comme celui-ci,
`Git`_ n'a pas d'autre choix que de nous demander notre avis.

.. inginious:: git-clone

.. inginious:: git-push

.. inginious:: git-pull

.. inginious:: git-merge-conflict

Contribuer au syllabus
######################

Dans le cas du syllabus, vous n'avez pas l'accès écriture.
La manière dont Github fonctionne pour règler ça c'est que vous *forkez* le
projet principal.
C'est à dire que vous en faites un copie indépendante à votre nom.
À celle là vous avez l'accès écriture.
Vous allez ensuite soumettre vos changements sur celle là puis les
proposer à travers l'interface de Github qu'on appelle *Pull request*.
Conventionnellement, on appelle la *remote* du dépôt principal *upstream*
et la votre *origin*.

Commencez donc par vous connecter sur Github, allez à
l'`adresse du code du syllabus
<https://github.com/obonaventure/SystemesInformatiques/>`_ et cliquez
sur *Fork*.

Vous pouvez maintenant obtenir le code du syllabus avec la commande
`git-clone(1)`_
(remplacez ``username`` par votre nom d'utilisateur sur Github)

.. code-block:: bash

   $ git clone https://github.com/username/SystemesInformatiques.git

Vous pouvez alors faire les changements que vous désirez puis les committer
comme expliqué à la section précédente.
Il est utile de garder le code à jour avec *upstream*.
Pour cela, il faut commencer par ajouter la remote

.. code-block:: bash

   $ git remote add upstream https://github.com/obonaventure/SystemesInformatiques.git

À chaque fois que vous voudrez vous mettre à jour, utilisez `git-pull(1)`_

.. code-block:: bash

   $ git pull upstream master

Une fois vos changements commités, vous pouvez les ajouter à *origin* avec
`git-push(1)`_

.. code-block:: bash

   $ git push origin master

Votre amélioration devrait normalement être visible via
`https://github.com/obonaventure/SystemesInformatiques/network <https://github.com/obonaventure/SystemesInformatiques/network>`_.
Vous pouvez maintenant aller sur Github à la page de votre fork et
cliquer sur *Pull Requests* puis *New pull request* et expliquer
vos changements.

Si plus tard vous voulez encore modifier le syllabus,
il vous suffira de mettre à jour le code en local

.. code-block:: bash

   $ git pull upstream master

committer vos changements, les ajouter à *origin*

.. code-block:: bash

   $ git push origin master

puis faire un nouveau pull request.
