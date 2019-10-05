.. -*- coding: utf-8 -*-
.. Copyright |copy| 2013 by Benoit Legat
.. Ce fichier est dérivé de `Outils Git
    <https://github.com/obonaventure/SystemesInformatiques/blob/master/Outils/git.rst>`_
   by Benoit Legat, used under `creative commons <http://creativecommons.org/licenses/by-sa/3.0/>`_
.. Ce fichier est distribué sous une licence `creative commons <http://creativecommons.org/licenses/by-sa/3.0/>`_

.. _rollback_changes:

Modifier un commit récent
~~~~~~~~~~~~~~~~~~~~~~~~~

Si on a oublié d'ajouter des modifications dans le dernier commit et
qu'on ne l'a pas encore *pushé*, on peut facilement les rajouter.
Il suffit de donner l'option ``--amend`` à `git-commit(1)`_.
Il ajoutera alors les modifications au commit actuel au lieu d'en créer un
nouveau.

On peut aussi annuler le dernier commit avec ``git reset HEAD^``.
`Git`_ permet aussi de construire un commit qui a l'effet inverse d'un autre
avec `git-revert(1)`_.
Ce dernier construit un commit qui annulera l'effet d'un autre commit.
Voyons tout ça par un exemple qui pourrait être le code de *Deep Thought*.

On a un fichier ``main.c`` contenant

.. code-block:: c

   #include <stdio.h>
   #include <stdlib.h>

   int main (int argc, char *argv[]) {
     int *n = (int*) malloc(sizeof(int));
     *n = 42;
     printf("%d\n", *n);
     return EXIT_SUCCESS;
   }

un ``Makefile`` contenant

.. code-block:: makefile

   run: answer
       echo "The answer is `./answer`"

   answer: main.c
       gcc -o answer main.c

si bien qu'on a

.. code-block:: bash

   $ make
   gcc -o answer main.c
   echo "The answer is `./answer`"
   The answer is 42
   $ make
   echo "The answer is `./answer`"
   The answer is 42
   $ touch main.c
   $ make
   gcc -o answer main.c
   echo "The answer is `./answer`"
   The answer is 42

et un fichier ``.gitignore`` avec comme seul ligne ``answer``.

Commençons par committer ``main.c`` et ``.gitignore`` en oubliant le
``Makefile``.

.. code-block:: bash

   $ git init
   Initialized empty Git repository in /path/to_project/.git/
   $ git status
   # On branch master
   #
   # Initial commit
   #
   # Untracked files:
   #   (use "git add <file>..." to include in what will be committed)
   #
   #	.gitignore
   #	Makefile
   #	main.c
   nothing added to commit but untracked files present (use "git add" to track)
   $ git add .gitignore main.c
   $ git commit -m "First commit"
   [master (root-commit) 54e48c9] First commit
    2 files changed, 10 insertions(+)
    create mode 100644 .gitignore
    create mode 100644 main.c
   $ git log --stat --oneline
   54e48c9 First commit
    .gitignore | 1 +
    main.c     | 9 +++++++++
    2 files changed, 10 insertions(+)
   $ git status
   # On branch master
   # Untracked files:
   #   (use "git add <file>..." to include in what will be committed)
   #
   #	Makefile
   nothing added to commit but untracked files present (use "git add" to track)

On pourrait très bien faire un nouveau commit contenant le ``Makefile``
mais si, pour une quelconque raison,
on aimerait l'ajouter dans le commit précédent,
on peut le faire comme suit

.. code-block:: bash

   $ git add Makefile
   $ git commit --amend
   [master 1712853] First commit
    3 files changed, 15 insertions(+)
    create mode 100644 .gitignore
    create mode 100644 Makefile
    create mode 100644 main.c
   $ git log --stat --oneline
   1712853 First commit
    .gitignore | 1 +
    Makefile   | 5 +++++
    main.c     | 9 +++++++++
    3 files changed, 15 insertions(+)

On voit qu'aucun commit n'a été créé mais c'est le commit précédent qui
a été modifié.
Ajoutons maintenant un check de la valeur retournée par `malloc(3)`_ pour gérer
les cas limites

.. code-block:: diff

   $ git diff
   diff --git a/main.c b/main.c
   index 39d64ac..4864e60 100644
   --- a/main.c
   +++ b/main.c
   @@ -3,6 +3,10 @@

    int main (int argc, char *argv[]) {
      int *n = (int*) malloc(sizeof(int));
   +  if (*n == NULL) {
   +    perror("malloc");
   +    return EXIT_FAILURE;
   +  }
      *n = 42;
      printf("%d\n", *n);
      return EXIT_SUCCESS;

et committons le

.. code-block:: bash

   $ git add main.c
   $ git commit -m "Check malloc output"
   [master 9e45e79] Check malloc output
    1 file changed, 4 insertions(+)
   $ git log --stat --oneline
   9e45e79 Check malloc output
    main.c | 4 ++++
    1 file changed, 4 insertions(+)
   1712853 First commit
    .gitignore | 1 +
    Makefile   | 5 +++++
    main.c     | 9 +++++++++
    3 files changed, 15 insertions(+)

Essayons maintenant de construire un commit qui retire les lignes qu'on
vient d'ajouter avec `git-revert(1)`_

.. code-block:: bash

   $ git revert 9e45e79
   [master 6c0f33e] Revert "Check malloc output"
    1 file changed, 4 deletions(-)
   $ git log --stat --oneline
   6c0f33e Revert "Check malloc output"
    main.c | 4 ----
    1 file changed, 4 deletions(-)
   9e45e79 Check malloc output
    main.c | 4 ++++
    1 file changed, 4 insertions(+)
   1712853 First commit
    .gitignore | 1 +
    Makefile   | 5 +++++
    main.c     | 9 +++++++++
    3 files changed, 15 insertions(+)

Le contenu de ``main.c`` est alors

.. code-block:: c

   #include <stdio.h>
   #include <stdlib.h>

   int main (int argc, char *argv[]) {
     int *n = (int*) malloc(sizeof(int));
     *n = 42;
     printf("%d\n", *n);
     return EXIT_SUCCESS;
   }

Comme c'est une bonne pratique de vérifier la valeur de retour de `malloc(3)`_,
supprimons ce dernier commit

.. code-block:: bash

   $ git reset HEAD^
   Unstaged changes after reset:
   M	main.c
   $ git log --oneline
   9e45e79 Check malloc output
   1712853 First commit

.. inginious:: git_catastrophy_scenario_1

.. inginious:: git_catastrophy_scenario_2

.. TODO Expliquer git-reflog

.. inginious:: git_catastrophy_scenario_3
