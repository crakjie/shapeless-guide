# Travailler avec les types et les implicites {#sec:type-level-programming}

Dans le chapitre précédent, nous avons vu
l'un des cas d'utilisation les plus fascinants de shapeless :
la déduction automatique d'instance de case class.
Nous allons découvrir qu'il existe plusieurs autres exemples au moins aussi saisissants.
Cependant, avant d'en arriver là, nous devons prendre un peu de temps
pour traiter la théorie que nous avons évité jusqu'ici
et établir un ensemble de patrons de conception pour écrire et débugger du code 
hautement implicite et hautement typé.
