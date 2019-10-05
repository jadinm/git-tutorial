.. -*- coding: utf-8 -*-
.. Copyright |copy| 2019 by Benoit Legat et Mathieu Jadin
.. Ce fichier est dérivé de `Outils Git
    <https://github.com/obonaventure/SystemesInformatiques/blob/master/Outils/git.rst>`_
   by Benoit Legat, used under `creative commons <http://creativecommons.org/licenses/by-sa/3.0/>`_
.. Ce fichier est distribué sous une licence `creative commons <http://creativecommons.org/licenses/by-sa/3.0/>`_


Ignorer des fichiers
~~~~~~~~~~~~~~~~~~~~

De base, les fichiers non-trackés apparaissent quand on utilise `git-status(1)`_.
Cela est un problème quand on travaille à plusieurs car il n'y a aucun intérêt à synchroniser
des fichiers compilés ou des fichiers de configuration d'un IDE.
Par exemple,

.. code-block:: bash

   $ git status
   # On branch master
   # Changes to be committed:
   #   (use "git reset HEAD <file>..." to unstage)
   #
   #	modified:   main.c
   #	new file:   file.c
   #
   # Changes not staged for commit:
   #   (use "git add <file>..." to update what will be committed)
   #   (use "git checkout -- <file>..." to discard changes in working directory)
   #
   #    modified:   main.c
   #	modified:   Makefile
   #
   # Untracked files:
   #   (use "git add <file>..." to include in what will be committed)
   #
   #	main.o
   #	file.o
   #	a.out

On peut choisir de les ignorer en les ajoutant dans dans le fichier ``.gitignore``.
En effet, on peut s'imaginer que dans un gros projet, la partie
``Untracked files`` peut devenir assez imposante et on ne sait plus
distinguer les fichiers qu'il faut penser à ajouter de ceux qu'il faut
ignorer une fois de plus.
La syntaxe est très simple, on spécifie un fichier par ligne,
on utilise un ``*`` pour spécifier n'importe
quelle chaine de charactères, les commentaires commencent par un ``#``
comme en Bash et si la ligne commence par un ``!``,
on demande de ne pas ignorer ce fichier à l'intérieur du dossier même
si un ``.gitignore`` d'un dossier parent dit le contraire.
Dans notre exemple, ``.gitignore`` aura le contenu suivant

.. code-block:: bash

   # Object files
   *.o
   # Executable
   a.out

.. inginious:: git-gitignore
