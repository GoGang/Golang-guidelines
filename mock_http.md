Dummies http en go
===

Utilisation du package test/http/httptest
---
Le package test/http/httptest propose des implémentations prêtes à l'emploi. Deux types :
* Server
* ResponseRecorder

### ResponseRecorder
Comme indiqué dans la documentation de référence, le ResponseRecorder est une implémentation de l'interface ResponseWriter qui enregistre ses modifications, en vue d'une inspection par les tests.

Dans un cas de test, cette implémentation est passée au handler à tester, ainsi qu'une requête forgée pour l'occasion simulant un appel réel. Le ResponseRecorder va stocker notamment dans sa structure :
* le code HTTP renvoyé par le handler
* la liste des headers HTTP
* le corps de la requête

L'instanciation se fait par l'appel à la méthode NewRecorder().

Forger la requête est relativement trivial, dans la mesure où l'URL n'a pas d'importance, puisque seul le header est testé.

### Server
Si on veut tester plus largement, jusqu'a simuler précisément un test d'intégration, l'implémentation Server de httptest permet de démarrer un serveur qui joue le rôle d'un module connexe au module à tester.

Ce serveur utilise un handler ad-hoc servant uniquement a test en cours. On peut alors soit tester en appelant le handler, soit démarrer l'application complète et la soumettre à une requête qui va déclencher
le traitement à tester.

Ce faux serveur répond sur localhost et un port pris au hasard dans les ports disponibles. Pour que l'application en cours de test s'adresse au serveur en question, il faut éventuellement patcher la configuration de votre
application pour qu'elle adresse cette URL.

Mock à la objet
---
Go propose une sous-partie des fonctionnalités d'une couche objet, notamment l'utilisation d'interfaces comme types, de sorte qu'il est possible de déclarer deux structures implémentant une interface données: une 
utilisée dans le code de production, et l'autre dans le code de test.

__Exemple__ :

	// Code de production
	type Connectable interface {
		Connect(string) (int, error)
	}

	var connecteur Connectable = &ConnecteurImpl{}

	type ConnecteurImpl struct {
		Connectable
		// champs de la structure
	}

	func (c *ConnecteurImpl) Connect(url string) (int, error) {
		response, err := nttp.Get(url)
		if err != nil {
			return 0, err
		}
		return response.StatusCode, nil
	}

	func main() {
		status, err := connecteur.Connect("http://example.com")
		if err != nil {
			fmt.Println("La requête a répondu", http.StatusText(status))
		}
		fmt.Println("Argh ! Erreur", err)
	}

	// code de test
	type ConnecteurToujoursOk struct {
		Connectable
	}

	func (c *ConnecteurImpl) Connect(_ string) (int, error) {
		return 200, nil // connecteur en succès
	}
	
	func TestTest(t *testing.T) {
		connecteur := &ConnecteurToujoursOk{}
		// le test proprement dit vient ici
	}

__Avantages__:
  * Ça fait le boulot
  * L'abstraction de l'accès réseau est encapsulé dans un objet dédié.
  
__Inconvénients__:
  * Beaucoup de code, surtout si le connecteur n'utilise pas de données à mettre dans la structure.
  * La méthode du connecteur qui contient l'appel à Get n'est pas facilement testable unitairement, il faut instancier un faux serveur, par exemple.
  * L'effet de bord sur la variable connecteur
  * Pas très idiomatique du langage, on fait pas du java.

Mock de fonction
---
En go les fonctions sont des citoyens de première classe du langage, on peut notamment les affecter à des variable. Il est donc possible de déclarer une variable pointant vers la fonction d'accès au réseau, et de redéfinir cette variable dans le cas de tests pour substituer à l'implémentation "réseau" une implémentation "locale".

__Exemple__:
	// Code de production
	var httpGet = http.Get

	func main() {
		response, err := httpGet("http://example.com")
		if err != nil {
			fmt.Println("La requête a répondu", response.Status)
		}
		fmt.Println("Argh ! Erreur", err)
	}

	// Code de test

	func TestTest(t *testing.T) {
		httpGet = func(url string) (response, error) {
			resp, _ := http.NewResponse(Status: 200}
			return resp, nil
		}
		// le test proprement dit vient ici
	}

__Avantages__:
  * Ça fait le boulot
  * Peu de code
  * Les tests unitaires couvrent jusqu'à l'appel réseau.
  
__Inconvénients__:
  * L'effet de bord sur la variable httpGet
