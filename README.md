Ce dépôt est fait pour construire un talk promouvant Elm, les deux idées
directrices étant:

* Elm est relativement simple à prendre en main.
* Elm est très efficace en production.

Le but de la présentation 
n'est pas d'apprendre à coder en Elm mais de présenter ses principales
forces. Ceci est un brouillon, faites des PR pour proposer vos idées ou rectifier
mes bêtises.



Qu'est-ce que Elm?
====
Un langage de programmation conçu pour élaborer des interfaces Web, compilant
vers le JS.


Les types: une application fiable, un code proche du métier
==========

Exemple en js: on veut stocker le salaire d'une personne.


```javascript

var salary = 1000;
// salary can be null in the case the person doesn't provide it

/* 
  ...

 200 lignes, un autre fichier, un trou spatio-temporel, peu importe 
  ...
*/

ReactDOM.render(
  <span>Your salary : {salary}!</span>,
  document.getElementById('root')
);

```
Le cas "non renseigné" n'est pas géré et créé un affichage bancal.

En Elm (garder la présentation du code "JS"): (cf )
```elm
type alias Model = {
    salary: Int
    }

init:Model
init = { 
    salary = 1000
    }

view: Model -> Html msg
view model =
    span [] [text ("Your salary: " ++ String.fromInt model.salary)]

```

Dans cette version, on ne peut pas avoir de "salaire indéfini" ; on transforme 
en (le champ "salary" est "peut-être" connu):
```elm
type alias Model = {
    salary: Maybe Int
    }

init:Model
init = { 
    salary = Just 1000
    }

view: Model -> Html msg
view model =
    span [] [text ("Your salary: " ++ String.fromInt model.salary)]
```
Mais ça, sa plante : Montrer le message d'erreur (String attendu, mais 
Maybe String fourni). ON NE PEUT PAS OUBLIER CE "CAS LIMITE".

Du coup, on remodifie : 

```elm
type alias Model = {
    salary: Maybe Int
    }

init:Model
init = { 
    salary = Just 1000
    }

view: Model -> Html msg
view model =
    case model.salary of
        Just salary ->
            span [] [text ("Your salary: " ++ String.fromInt salary)]
```

Re message d'erreur (LE MONTRER ENCORE !!!), il manque une branche:
```elm
type alias Model = {
    salary: Maybe Int
    }

init:Model
init = { 
    salary = Just 1000
    }

view: Model -> Html msg
view model =
    case model.salary of
        Just salary ->
            span [] [text ("Your salary: " ++ String.fromInt salary)]
        Nothing ->
            span [] [text "Unknown salary"]
```
Cool, ça compile enfin ! On fait des tests en changeant le salaire en Nothing
par exple.

Il faut imaginer qu'on a cette sécurité PARTOUT dans le code, pour TOUT le projet.
C'est le "Zero runtime exception" (insérer le graphique de NoRedInk montrant les
runtimeerrors en JS / Elm).


Mais un vrai Elmien ne ferait pas ça: que signifie ce "Nothing"? Au chomage ?
Champ non renseigné ? Ok on a dit dans le commentaire que c'était non renseigné,
on va créer un nouveau:

```elm
type Salary = 
    HaveSalary Int 
    | Unknown

type alias Model = {
    salary: Salary
    }


init:Model
init = { 
    salary = HaveSalary 1000
    }

view: Model -> Html msg
view model =
    case model.salary of
        HaveSalary salary ->
            span [] [text ("Employed by: " ++ String.fromInt salary)]
        Unknown ->
            span [] [text "Unknown salary"]
```
Even cooler ! Le code est maintenant en correspondance exacte avec le métier,
pas de valeur abstraite représentant un truc concret.

Et bonus: on veut rajouter le statut "Unemployed" :
```elm
type Salary = 
    HaveSalary Int 
    | Unknown
    | Unemployed

type alias Model = {
    salary: Salary
    }


init:Model
init = { 
    salary = Unemployed
    }

view: Model -> Html msg
view model =
    case model.salary of
        HaveSalary salary ->
            span [] [text ("Employed by: " ++ String.fromInt salary)]
        Unknown ->
            span [] [text "Unknown salary"]
```
Crap, il manque un truc. Mais en lisant le message d'erreur, on voit qu'il faut 
rajouter une branche dans la vue: 

```elm

view: Model -> Html msg
view model =
    case model.salary of
        HaveSalary salary ->
            span [] [text ("Employed by: " ++ String.fromInt salary)]
        Unknown ->
            span [] [text "Unknown salary"]
        Unemployed ->
            span [] [text "You don't work. Cross the road to find one! (E.M.)"]
```

Il faut bien voir qu'à ce stade en JS sur un projet conséquent, on est un peu 
sur des oeufs: ai-je bien traité tous les cas où ce salary apparaît ? On peut
faire des tests, mais :

* les tests eux-mêmes peuvent être incomplet

* bah il faut écrire du code pour faire des tests alors qu'en Elm, le compilo
vérifie toutes ces choses "automatiques" pour nous ; donc beaucoup moins de 
tests à écrire.


Moralité :

1. Le développeur est pris par la main pour corriger ses erreurs => bien pour 
les débutants.

2. Le typage permet de faire des refactors en étant très serein (le compilo 
couvre nos arrières!) => bien pour la prod !

3. Le code reste proche du métier => bien pour les deux !

4. On a vu 80% de la syntaxe de Elm, le langage possède très peu de construction
=> bien pour les débutants. (En JS on a beaucoup plus de notions: var/let/const, function/=> (ce ne sont pas
exactement les mêmes constructions!), objets, prototype (avec 42 façons de faire
de l'héritage) ; et par dessus tout ça on rajoute plusieurs couches de
bibliothèques et parfois on enrichit le langage avec Typescript !)

>Est-ce que ça vaut le coup de parler :
>
>* du shadowing (en expliquant pourquoi sur un projet sur le long
>terme, ça peut créer des bugs "bêtes", cf la page https://elm-lang.org/0.19.0/shadowing
>) -> du coup un gros plus en prod ?
>
>* du elm format "ta gueule" qui permet d'avoir la même présentation du code pour
>tout le monde, évite de perdre du temps à choisir une conf (qqsoit la config 
>choisie, il y aura tjs qqn à qui ça ne plaît pas)?



The Elm Architecture: un monde déjà connu
====================


Présenter TEA avec un petit diagramme qui montre la circulation
des données (sans parler des Cmd), indiquer que dans la partie
précédente, on a vu la partie `model---> view`:

```
 Model ---> view ---> Runtime ---> update 
   ^                                 |
   |                                 v
   -----------------------------------
```
 Faire le parallèle à l'oral avec Redux 
(en disant que quand même ça vient de Elm !)

On reprend notre première appli de salaire, on va se donner le droit de l'ajuster
(vous en reviez, avouez!). Pour ça, on va créer deux messages possibles: 
augmente et diminue :

```elm
type alias Model = {
    salary: Int
    }

init:Model
init = { 
    salary = 1000
    }

type Msg =
    Increment
    | Decrement

view: Model -> Html Msg
view model =
    span [] [text ("Your salary: " ++ String.fromInt model.salary)]

```    

Puis on rajoute des boutons, en liant les événements:

```elm
type alias Model = {
    salary: Int
    }

init:Model
init = { 
    salary = 1000
    }

type Msg =
    Increment
    | Decrement

view: Model -> Html Msg;
view model =
    div []
        [ button [onClick Increment] [text "+"]
        , span [] [text ("Your salary: " ++ String.fromInt model.salary)]
        , button [onClick Decrement] [text "-"]
        ]
```


Pour l'instant, on clique, il ne se passe rien, il faut récupérer les événéments
du runtime :

```
 Model ---> view ---> Runtime ---> update 
   ^                                 |
   |                                 v
   -----------------------------------
```

On s'occupe de la fonction `update`:

```elm
type alias Model = {
    salary: Int
    }


init:Model
init = { 
    salary = 1000
    }

type Msg =
    Increment
    | Decrement

view: Model -> Html Msg
view model =
    div []
        [ button [onClick Increment] [text "+"]
        , span [] [text ("Your salary: " ++ String.fromInt model.salary)]
        , button [onClick Decrement] [text "-"]
        ]

update : Msg -> Model
update msg model = 
    case msg of
        Increment -> { model | salary = model.salary + 100 }
        Decrement -> { model | salary = model.salary - 100 }
```

1. on ne mute rien ! D'ailleurs le langage ne le permet pas. Ce que fait
par exemple immutable.js : http://facebook.github.io/immutable-js/. 

2. Approche fonctionnelle, comme les outils qu'apportent lodash https://lodash.com/
et underscore.js: https://underscorejs.org/ par exemple


Conclusion : au final Elm regroupe en un seul outil cohérent toutes les 
découvertes des "bonnes pratiques" de ces dernières années : Circulation
à sens unique des données, immutabilité, fonctionnel. Plus besoin de dépendre
de plus couches d'abstractions pour JS.


La particularité de Elm: on ne se mélange pas!
==========

Lorsqu'on construit un nouveau langage, les deux stratégies classiques :
1. **Compatibilité en arrière complète**. Par exemple C++ est un sur ensemble
de C, tout comme TypeScript est un sur-ensemble de JS. => permet de réutiliser
directement ce qui existe déjà.

2. **Foreign Function Interface**: en permettant de faire liaisons directes 
avec le langage "hôte". Par exemple Scala peut appeler des fonction Java
directement. Idem avec Python/C, Haskell/C...  => permet de réutiliser
quasi-directement l'existant.

Mais ce faisant, on perd les garanties que peuvent nous apporter les nouveaux
langages. Typiquement en Elm, on a la grosse garantie "No Runtime Exception"...
qui ne serait plus respectée si on était autorisé à exécuter du JS quelconque
(`undefined is not a function` es-tu là?). Elm ne fait donc aucun des 2.

Cependant, on peut quand même **communiquer** de JS vers Elm et inversement : 
on envoit des messages grâce à des "ports". Ainsi, depuis un programme Elm,
JS est vu comme un "service distant" qui n'interfère pas avec le code Elm.

Les perfs
====

Conséquence de cette isolation : le compilateur peut faire de fortes hypothèses
sur le code qu'il compile. Le code produit est alors rapide et petit
(mettre les graphes de perfs).

Les performances sur la taille de fichiers sont obtenue en partie grâce au fait
que même si on inclut énormément de bibliothèques, seules les fonctions utiles 
sont compilées (on part du `main` et on regarde celles qui sont appellées).
En JS ce genre d'optimisation est impossible car on peut faire:

```javascript
if( var < 1) {
    obj["add" + var] = function (a,b){...};
} else {
    obj["add" + var] =function(a,b){...};
}
```
On ne peut pas savoir ce que contient `obj["add" + var]` à la compilation....
(ceci est inhérent à tous les langages permettant de la méta-programmation).

Et le must: il n'y a rien à configurer, si ce n'est passer l'option `--optimize`
à `elm make`.

**Conclusion**: de bonne perfs (bien pour la prod) sans se fouler (bien pour
les débutants.... mais pas que !)

Ces performances comptent ! (Insérer ici des stats sur le "décrochage" des 
internautes en fonction du temps de chargement ; retrouver l'article (Medium ?)
où un mec
qu'il a passé son serveur de Ruby (?) à Elixir, en gardant les même fonctionnalités
mais avec un temps de réponse drastiquement réduit et que le retour des utilisateurs
était que "ça marche mieux", alors que c'est juste "plus rapide").