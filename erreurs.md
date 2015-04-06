Erreurs
=======

Erreurs et panics
-----------------

En go, il y a deux façons de déclarer des erreurs: les erreurs et les panics. Les erreurs sont communiquées au code appelant comme tout autre valeur, les panics sont des valeurs de n'importe quel type qui remontent par l'appel à la méthode "panic()", et la récupération du message par la méthode "recover()"

### Erreurs

On entend par « erreur » n'importe quel type qui respecte l'interface « error », c'est à dire qu'elle déclare une méthode Error(). 

Les erreurs sont utilisées dans le cas général d'une exécution « non nominale ». On retourne l'erreur à la méthode appelante, qui, elle, aura suffisamment de contexte pour savoir comment intégrer cette erreur dans le comportement de l'application. Imaginons une fonction qui renvoie la réponse à une question posée à l'utilisateur. Si la méthode de traitement de cette réponse récupère une réponse dont elle ne peut rien faire, elle retourne une erreur, pour que le code appelant répète la question en indiquant que la réponse précédente n'a pas été comprise.
esp
Le type d'erreur le plus simple est le type stringError, que l'on instancie via errors.New ou fmt.Errorf. Le type stringError est le type d'erreur le plus rudimentaire de la librairie standard, et elle ne devrait être utilisée que dans les cas les plus triviaux. Pour les cas plus classiques, chaque package de la librairie standard expose sa propre implémentation d'erreur, qu'il est bon de connaître et d'utiliser dès que possible. Un bon réflexe est de rechercher le type erreur de chaque nouveau package que l'on utilise, au même titre que n'importe quel type ou méthode qu'elle expose. 

Ça va sans dire, mais les erreurs doivent être utilisées. Si vous ne lisez pas l'erreur qui est retournée, vous prenez le risque d'utiliser un objet à zéro, ou un pointeur nil dans un cas d'utilisation nominal. Vous aurez de grandes chances de déclancher un panic en utilisant un pointeur nul par exemple, et vous occasionnerez donc le crash du programme. Dans le code de production, ne pas lire, et traiter, une erreur suceptible d'être retournée par une fonction relève de l'inconscience. 

L'utilitaire errcheck vérifie que les erreurs sont traitées. Abusez-en.

Go promeut le retour des erreurs et le traitement par le code appelant plutôt que l'utilisation des panics.

### Panics

Les panics ressemblent à ce qui se fait en java avec le système d'exceptions: on shunte une partie de la pile d'appel en espérant que du code rattrappe l'exception, par l'utilisation de recover (ou catch en java).

Dans la librairie standard, les panics sont utilisées dans le cas d'erreurs bas niveau qui relèvent de l'erreur de programmation, par exemple l'utilisation d'un pointeur nul ou l'accès à un élément de tableau qui se situerai hors de la plage d'indexes. Ce sont des cas de problèmes très techniques, bien loin des cas "métiers" de valeurs erronnées, de connection réseau mal fichue, qui relèvent de l'erreur et non de la panic.

### Erreurs ou panics

Si les panics sont peu utilisées, c'est pour plusieurs raisons. 
* Si l'erreur est traitable par le code appelant, parce qu'il a plus de contexte, l'erreur est plus utile.
* Si on panic, c'est que le traitement de l'erreur sera effectué loin du moment de l'erreur haut dans la pile, et qu'il faudra sans doute déployer certains efforts pour remonter suffisamment de contexte au recover pour traiter le cas correctement. Mais j'ai tendance à considérer cette pratique comme révélatrice d'un problème de conception.
* On privilégie donc le panic pour le cas désespéré, quand on ne peut plus rien récupérer et que la seule solution est d'afficher un gros « désolé on a eu un souci » à l'écran.
* Les erreurs sont des valeurs normales du langage.
