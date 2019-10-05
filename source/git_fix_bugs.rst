.. -*- coding: utf-8 -*-
.. Copyright |copy| 2013 by Benoit Legat
.. Ce fichier est dérivé de `Outils Git
    <https://github.com/obonaventure/SystemesInformatiques/blob/master/Outils/git.rst>`_
   by Benoit Legat, used under `creative commons <http://creativecommons.org/licenses/by-sa/3.0/>`_
.. Ce fichier est distribué sous une licence `creative commons <http://creativecommons.org/licenses/by-sa/3.0/>`_


Corriger des bugs grâce à Git
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Git permet de garder des traces des nombreux changements qui ont été effectué au
cours de l’évolution d’un programme. Il contient d’ailleurs un outil très
puissant vous permettant de retrouver la source de certaines erreurs, pourvu que
les changements soient faits par petits commits : `git-bisect(1)`_.

Supposez que vous ayez introduit une fonctionnalité dans votre programme. Tout
allait alors pour le mieux. Quelques semaines plus tard, à votre grand dam, vous
vous rendez compte qu’elle ne fonctionne plus. Vous sillonnez tous les fichiers
qui pourraient interagir avec cette fonction, en vain. Dans le désespoir, à
l’approche de la deadline, vous succombez au nihilisme.

Avant de tout abandonner, pourtant, vous réalisez quelque chose de très
important. Ce que vous cherchez, c’est la source de l’erreur ; cela fait, la
corriger sera sans l’ombre d’un doute une tâche aisée. Si seulement il était
possible de voir à partir de quel changement le bug a été introduit…

C’est là que vous repensez à Git ! Git connaît tous les changements qui ont été
effectués, et vous permet facilement de revenir dans le passé pour vérifier si
le bug était présent à un moment donné. En outre, vous vous rappelez vos cours
d’algorithmiques et vous rendez compte que, puisque vous connaissez un point où
le bug était présent et un autre ou il ne l’était pas, vous pouvez à l’aide
d’une recherche binaire déterminer en un temps logarithmique (par rapport aux
nombres de révisions comprises dans l’intervalle) quelle révision a introduit
l’erreur.

C’est exactement l’idée derrière `git-bisect(1)` : vous donnez un intervalle de
commits dans lequel vous êtes certains de pouvoir trouver le vilain commit
responsable de tous vos maux, pour ensuite le corriger. Vous pouvez même
entièrement automatiser cette tâche si vous pouvez, excellent programmeur que
vous êtes, écrire un script qui renvoie 1 si le bug est présent et 0 si tout va
bien.

Pour vous montrez comme utiliser cette fonctionnalité, et vous convaincre que
cela marche vraiment, et pas seulement dans des exemples fabriqués uniquement
dans un but de démonstration, nous allons l’appliquer à un vrai programme C :
mruby, une implémentation d’un langage correspondant à un sous-ensemble de Ruby.

Intéressons nous à `un des problèmes qui a été rapporté par un utilisateur
<https://github.com/mruby/mruby/issues/1583>`_. Si vous lisez cette page, vous
verrez qu’en plus de décrire le problème, il mentionne le commit à partir duquel
il rencontre l’erreur. Si vous regardez aussi le commit qui l’a corrigée, vous
verrez que le développeur a bien dû changer une ligne introduite dans le commit
qui avait été accusé par l’utilisateur.

Mettons nous dans la peau de l’utilisateur qui a trouvé le bug, et tentons nous
aussi d’en trouver la cause, en utilisant Git. D’abord, il nous faut obtenir le
dépôt sur notre machine (vous aurez besoin de Ruby afin de pouvoir tester),
et revenir dans le passé puisque, depuis, l’erreur a été corrigée.

.. code-block:: console

        $ git clone git@github.com:mruby/mruby.git
        (...)
        $ cd mruby
        $ git checkout 5b51b119ca16fe42d63896da8395a5d05bfa9877~1
        (...)

Sauvegardons aussi le fichier de test proposé, par exemple dans
``~/code/rb/test.rb`` :

.. code-block:: ruby

        class A
          def a
            b
          end
          def b
            c
          end
          def c
            d
          end
        end
        x = A.new.a

Vous devriez maintenant être capable de vérifier que la méthode ``A.a`` n’est pas
incluse dans la backtrace :

.. code-block:: console

        $ make && ./bin/mruby ~/code/rb/test.rb
        (...)
        trace:
                [3] /home/kilian/code/rb/test.rb:9:in A.c
                [2] /home/kilian/code/rb/test.rb:6:in A.b
                [0] /home/kilian/code/rb/test.rb:13
        /home/kilian/code/rb/test.rb:9: undefined method 'd' for #<A:0xdf1000> (NoMethodError)

C’est le moment de commencer. Il faut d’abord dire à Git que nous désirons
démarrer une bissection et que le commit actuel est « mauvais », c’est à dire
que le bug est présent. Ceci est fait en utilisant les deux lignes suivantes,
dans l’ordre :

.. code-block:: console

        $ git bisect start
        $ git bisect bad

Regardons ce qu’il en était quelque mois auparavant (remarquez qu’il faut
utiliser ``make clean`` pour s’assurer de tout recompiler ici) :

.. code-block:: console

        $ git checkout 3a27e9189aba3336a563f1d29d95ab53a034a6f5
        Previous HEAD position was 7ca2763... write_debug_record should dump info recursively; close #1581
        HEAD is now at 3a27e91... move (void) cast after declarations
        $ make clean && make && ./bin/mruby ~/code/test.rb
        (...)
        trace:
                [3] /home/kilian/code/rb/test.rb:9:in A.c
                [2] /home/kilian/code/rb/test.rb:6:in A.b
                [1] /home/kilian/code/rb/test.rb:3:in A.a
                [0] /home/kilian/code/rb/test.rb:13
        /home/kilian/code/rb/test.rb:9: undefined method 'd' for #<A:0x165d2c0> (NoMethodError)

Cette fois-ci, tout va bien. Nous pouvons donc en informer Git :

.. code-block:: console

        $ git bisect good
        Bisecting: 116 revisions left to test after this (roughly 7 steps)
        [fe1f121640fbe94ad2e7fabf0b9cb8fdd4ae0e02] Merge pull request #1512 from wasabiz/eliminate-mrb-intern

Ici, Git nous dit combien de révisions il reste à vérifier dans l’intervalle en
plus de nous donner une estimation du nombre d’étapes que cela prendra. Il nous
informe aussi de la révision vers laquelle il nous a déplacé. Nous pouvons donc
réitérer notre test et en communiquer le résultat à Git :

.. code-block:: console

        $ make clean && make && ./bin/mruby ~/code/test.rb
        (...)
        trace:
                [3] /home/kilian/code/rb/test.rb:9:in A.c
                [2] /home/kilian/code/rb/test.rb:6:in A.b
                [1] /home/kilian/code/rb/test.rb:3:in A.a
                [0] /home/kilian/code/rb/test.rb:13
        /home/kilian/code/rb/test.rb:9: undefined method 'd' for #<A:0x165d2c0> (NoMethodError)
        $ git bisect good
        Bisecting: 58 revisions left to test after this (roughly 6 steps)
        [af03812877c914de787e70735eb89084434b21f1] add mrb_ary_modify(mrb,a); you have to ensure mrb_value a to be an array; ref #1554

Si nous réessayons, nous allons nous rendre compte que notre teste échoue à
présent (il manque la ligne ``[1]``): nous somme allés trop loin dans le
futur. Il nous faudra donc dire à Git que la révision est mauvaise.

.. code-block:: console

        $ make clean && make && ./bin/mruby ~/code/test.rb
        (...)
        trace:
                [3] /home/kilian/code/rb/test.rb:9:in A.c
                [2] /home/kilian/code/rb/test.rb:6:in A.b
                [0] /home/kilian/code/rb/test.rb:13
        /home/kilian/code/rb/test.rb:9: undefined method 'd' for #<A:0x165d2c0> (NoMethodError)
        $ git bisect bad
        Bisecting: 28 revisions left to test after this (roughly 5 steps)
        [9b2f4c4423ed11f12d6393ae1f0dd4fe3e51ffa0] move declarations to the beginning of blocks

Si vous continuez à appliquer cette procédure, vous allez finir par trouver la
révision fautive, et Git nous donnera l’information que nous recherchions, comme
par magie :

.. code-block:: console

        $ git bisect bad
        Bisecting: 0 revisions left to test after this (roughly 0 steps)
        [a7c9a71684fccf8121f16803f8e3d758f0dea001] better error position display
        $ make clean && make && ./bin/mruby ~/code/rb/test.rb
        (...)
        trace:
                [3] /home/kilian/code/rb/test.rb:9:in A.c
                [2] /home/kilian/code/rb/test.rb:6:in A.b
                [0] /home/kilian/code/rb/test.rb:13
        /home/kilian/code/rb/test.rb:9: undefined method 'd' for #<A:0x1088160> (NoMethodError)
        $ git bisect bad
        a7c9a71684fccf8121f16803f8e3d758f0dea001 is the first bad commit
        commit a7c9a71684fccf8121f16803f8e3d758f0dea001
        Author: Yukihiro "Matz" Matsumoto <matz@ruby-lang.org>
        Date:   Tue Oct 15 12:49:41 2013 +0900

            better error position display

        :040000 040000 67b00e2d4f6acadc0474e00fc0f5e6e13673c64a 036eb9c3b9960613bde3882b7a88ac6cabc56253 M      include
        :040000 040000 5040dd346fea4d8f476d26ad2ede0dc49ca368cd 903f2d954d8686e7bfa7bcf5d83b80b5beb4899f M      src

Maintenant que nous connaissons la source du problème, il ne faut pas oublier de
confirmer à Git que la recherche est bien terminée, et que nous désirons
remettre le dépôt dans son état normal.

.. code-block:: console

        $ git bisect reset
        Previous HEAD position was a7c9a71... better error position display
        HEAD is now at 7ca2763... write_debug_record should dump info
        recursively; close #1581

Automatisation de la procédure
##############################

Exécuter ce test à la main est cependant répétitif, prône aux erreurs
d’inattention, et surtout très facile à automatiser. Écrivons donc un script qui
vérifie que la ligne mentionnant ``A.a`` est bien présente à chaque fois,
appelons le par exemple ``~/code/sh/Iznogoud.sh``. Il s’agit de renvoyer 0
si tout se passe bien et une autre valeur s’il y a un problème.

.. code-block:: bash

        #!/usr/bin/env bash
        make clean && make && ./bin/mruby ~/code/rb/test.rb 2>&1 | grep A\.a

Puisque ``grep`` renvoie 1 quand il ne trouve pas de ligne contenant le motif
qu’on lui passe en argument et 0 sinon, notre script renvoie bien 1 si la sortie
de mruby ne contient pas la ligne mentionnant ``A.a`` et 0 sinon.

N’oubliez pas de changer les permissions du script pour en permettre l’exécution :

.. code-block:: console

        $ chmod +x ~/code/sh/Iznogoud.sh


Ce test n’est en bien sûr pas infaillible, mais sera suffisant ici. Il faut
d’abord redonner à Git l’intervalle dans lequel se trouve la révision fautive.

.. code-block:: console

        $ git bisect start
        $ git bisect bad
        $ git checkout 3a27e9189aba3336a563f1d29d95ab53a034a6f5
        Previous HEAD position was 7ca2763... write_debug_record should dump info recursively; close #1581
        HEAD is now at 3a27e91... move (void) cast after declarations
        $ git bisect good
        Bisecting: 116 revisions left to test after this (roughly 7 steps)
        [fe1f121640fbe94ad2e7fabf0b9cb8fdd4ae0e02] Merge pull request #1512 from wasabiz/eliminate-mrb-intern

Il suffit maintenant d’utiliser ``git bisect run`` avec le nom du script pour
l’utiliser. Il est possible de rajouter d’autres arguments après le nom du
script, qui seront passés au script lors de chaque exécution. Par exemple, si
vous avez dans votre Makefile une tâche test qui renvoie 0 si tous les tests
passent et 1 si certains échouent, alors ``git bisect run make test``
permettrait de trouver à partir de quand les tests ont cessé de fonctionner.

Si vous exécutez la ligne suivante, vous devriez bien trouver, après quelques
compilations, le même résultat qu’avant :

.. code-block:: console

        $ git bisect run ~/code/sh/Iznogoud.sh
        (...)
        a7c9a71684fccf8121f16803f8e3d758f0dea001 is the first bad commit
        commit a7c9a71684fccf8121f16803f8e3d758f0dea001
        Author: Yukihiro "Matz" Matsumoto <matz@ruby-lang.org>
        Date:   Tue Oct 15 12:49:41 2013 +0900

            better error position display

        :040000 040000 67b00e2d4f6acadc0474e00fc0f5e6e13673c64a 036eb9c3b9960613bde3882b7a88ac6cabc56253 M      include
        :040000 040000 5040dd346fea4d8f476d26ad2ede0dc49ca368cd 903f2d954d8686e7bfa7bcf5d83b80b5beb4899f M      src
        bisect run success

À nouveau, n’oubliez pas d’utiliser ``git bisect reset`` avant de continuer à
travailler sur le dépôt.
