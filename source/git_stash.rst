.. -*- coding: utf-8 -*-
.. Copyright |copy| 2013 by Benoit Legat
.. Ce fichier est dérivé de `Outils Git
    <https://github.com/obonaventure/SystemesInformatiques/blob/master/Outils/git.rst>`_
   by Benoit Legat, used under `creative commons <http://creativecommons.org/licenses/by-sa/3.0/>`_
.. Ce fichier est distribué sous une licence `creative commons <http://creativecommons.org/licenses/by-sa/3.0/>`_


Sauvegarder des modifications hors de l'historique
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

On a vu que certaines opérations comme `git-checkout(1)`_ nécessitent
de ne pas avoir de modifications en conflit avec l'opération.

`git-stash(1)`_ permet de sauvegarder ces modifications pour qu'elles ne soient
plus dans le *working directory* mais qu'elles ne soient pas perdues.
On peut ensuite les réappliquer avec ``git stash apply`` puis les effacer
avec ``git stash drop``.

Reprenons notre exemple de *Changer la branche active* illustré par la figure
suivante

.. figure:: figures/hello_intro.svg
   :align: center

   Historique après avoir ajouté un commentaire d'introduction

.. code-block:: bash

   $ git checkout pid
   Switched to branch 'pid'
   $ echo "42" >> main.c
   $ echo "42" >> .gitignore
   $ git stash
   Saved working directory and index state WIP on pid: b14855e Add .gitignore
   HEAD is now at b14855e Add .gitignore
   $ git checkout master
   Switched to branch 'master'
   $ git stash apply
   Auto-merging main.c
   # On branch master
   # Changes not staged for commit:
   #   (use "git add <file>..." to update what will be committed)
   #   (use "git checkout -- <file>..." to discard changes in working directory)
   #
   #	modified:   .gitignore
   #	modified:   main.c
   #
   no changes added to commit (use "git add" and/or "git commit -a")

On voit que les changements on été appliqués

.. code-block:: diff

   $ git diff
   diff --git a/.gitignore b/.gitignore
   index cba7efc..5df1452 100644
   --- a/.gitignore
   +++ b/.gitignore
   @@ -1 +1,2 @@
    a.out
   +42
   diff --git a/main.c b/main.c
   index 8381ce0..eefabd7 100644
   --- a/main.c
   +++ b/main.c
   @@ -11,3 +11,4 @@ int main () {
      printf("Hello world!\n");
      return EXIT_SUCCESS;
    }
   +42

On peut alors supprimer le *stash*

.. code-block:: bash

   $ git stash drop
   Dropped refs/stash@{0} (ae5b4fdeb8bd751449d73f955f7727f660708225)

.. TODO Ajouter un exercice sur le stashing
