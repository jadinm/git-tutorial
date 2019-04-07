
Subversion
----------

`subversion`_ (ou abrégé `svn(1)`_) est un logiciel qui permet à plusieurs utilisateurs de travailler sur les mêmes documents de type texte. `Subversion`_ est fréquemment utilisé pour gérer du code source développé de façon collaborative, mais il peut aussi servir à gérer n'importe quel ensemble de fichiers (de préférence textes) manipulés par plusieurs personnes.

.. Dans le cadre du cours SINF1252 vous devez vous inscrire à subversion dans le projet ``SINF1252_2012`` en suivant le lien et les instructions sur http://wiki.student.info.ucl.ac.be/index.php/Svn

Pour commencer l'utilisation de `svn(1)`_ vous devriez faire d'abord un ``checkout`` du répertoire:

        .. code-block:: console

                $ svn checkout <url de votre répertoire>
                Checked out revision 1.

Ceci installe votre répertoire (ici, nommé ``my_rep``) dans le dossier courant. Vous pouvez vous déplacer dans le nouveau dossier et créer un nouveau dossier pour cet premier projet. Il faut explicitement ajouter ce dossier à svn avec la commande ``svn add [nom du dossier]``. Chaque fichier et dossier dont vous voulez qu'il fasse partie du contrôle de version doivent être ajoutés avec cette commande.

        .. code-block:: console

                $ cd my_rep
                $ mkdir projet_S1
                $ svn add projet_S1
                A       projet_S1

Ce dossier n'a pas encore été envoyé sur le serveur principal et n'est donc pas encore visible pour d'autres utilisateurs. Pour afficher l'état de votre répertoire utilisez ``svn status``. La lettre ``A`` indique que ceci est un nouveau dossier/fichier pas encore envoyé vers le serveur. ``?`` indique que les fichiers/dossiers ne font pas partie du répertoire svn (on peut les ajouter avec ``svn add``). ``M`` indique que les fichiers sont modifiés localement mais pas encore envoyés vers le serveur. Ces fichiers font partie du répertoire svn.

        .. code-block:: console

                $ svn status
                A       projet_S1
                $ svn commit -m "Projet S1: Initialisation"
                Adding  projet_S1
                Transmitting file data .
                Committed revision 2.

La commande ``svn commit`` permet de pousser les changements locaux et les nouveaux fichiers vers le serveur. La chaîne de charactères entre les ``"`` est le commentaire qu'il faut ajouter au commit. Il est important de commenter vos commits pour que vous puissiez vous retrouvez dans votre historique. L'historique de votre répertoire peut être affiché avec la commande ``svn log``.

Les autres utilisateurs de votre répertoire (c'est-à-dire dans le cadre de ce cours: vôtre binôme du groupe) peuvent à partir de maintenant accéder à ce nouveau dossier en mettant à jour son répertoire local.
Pour mettre à jour le répertoire local, on utilise la commande ``svn update``.

        .. code-block:: console

                $ svn update
                Updating '.':
                A       projet_S1
                Updated to revision 2.

Il est recommandé de toujours faire un ``update`` avant de faire un ``commit``. Lors d'un update il est possible qu'un conflit se crée dans votre dossier local. Ceci peut arriver si vous avez modifié une ligne dans un fichier localement et que cette ligne a aussi été modifiée par le commit d'un autre utilisateur. Pour résoudre le conflit, vous devez éditer le fichier que svn a indiqué être en conflit en cherchant des lignes qui commencent par ``<<<``. Corrigez ce fichier et retournez dans la console et tapez ``r`` pour indiquer à svn que ce conflit a été résolu.

Pour plus d'informations sur svn regardez les commandes ``svn help``, ``svn help [commande]`` ou http://svnbook.red-bean.com/. Une recherche sur Google vous aidera aussi pour résoudre vos problèmes avec subversion.

