## La programmation générique, qu'est-ce que c'est ?

Les types sont utiles car ils sont spécifiques :
ils nous aident a comprendre comment les différents morceaux de code
s'imbriquent les uns aux autres, ils nous aident à éviter les bugs
et nous conduisent vers une solution lorsque l'on code.

Pourtant, parfois, les types sont *trop* spécifiques.
Il y a des situations où l'on veut pouvoir profiter des similarités
entres les types pour éviter les répétitions.
Considérons par exemple les définitions suivantes :

```tut:book:silent
case class Employee(name: String, number: Int, manager: Boolean)

case class IceCream(name: String, numCherries: Int, inCone: Boolean)
```

Ces deux *case classes* représentent des données différentes
mais elles on des points communs évidents :
elles possèdent toutes deux des champs du même type.
Supposons que nous voulons implémenter une opération générique,
par exemple les sérialiser en un fichier CSV.
En dépit des similarités, nous devons écrire
une méthode de sérialisation pour chaque type :

```tut:book:silent
def employeeCsv(e: Employee): List[String] =
  List(e.name, e.number.toString, e.manager.toString)

def iceCreamCsv(c: IceCream): List[String] =
  List(c.name, c.numCherries.toString, c.inCone.toString)
```

La programmation générique consiste à venir à bout de ce genre de différences.
Shapeless met a disposition un moyen pratique de convertir des types
spécifiques en une représentation générique, qui sont ensuite
manipulables avec un code commun.

Par exemple, nous pouvons utiliser le code suivant
pour convertir les `Employee` et les `IceCream`
en valeurs du même type.
Ne vous inquiétez pas si vous ne comprenez pas encore cet exemple,
ce sujet sera traité plus tard.

```tut:book:silent
import shapeless._
```

```tut:book
val genericEmployee = Generic[Employee].to(Employee("Dave", 123, false))
val genericIceCream = Generic[IceCream].to(IceCream("Sundae", 1, false))
```

Les deux valeurs sont maintenant du même type.
Elles sont toutes les deux des listes hétérogènes (des `HList`s)
contenant une `String`, un `Int`, et un `Boolean`.
Nous jetterons bientôt un œil aux `HList`s et à l'importance de leur rôle.
Pour l'instant, le plus intéressant est que nous pouvons
sérialiser chaque valeur avec une seule fonction.

```tut:book:silent
def genericCsv(gen: String :: Int :: Boolean :: HNil): List[String] =
  List(gen(0), gen(1).toString, gen(2).toString)
```

```tut:book
genericCsv(genericEmployee)
genericCsv(genericIceCream)
```
Cet exemple est basique mais il pointe du doigt
la nature de la programmation générique.
Nous reformulons les problèmes pour pouvoir les résoudre
avec des constructions génériques et ainsi écrire de petit bloc de code
qui fonctionne avec une large variété de types.
Programmer avec shapeless nous permet d'éliminer une grande quantité de
code *boilerplate*, de rendre les applications Scala plus faciles à lire, écrire
et maintenir.

Votre curiosité a été piquée ? Alors c'est parti !
