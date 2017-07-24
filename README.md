# tipsNtricks
## bisector

Bisector ne renvoie le bon index que si le tableau de référence est trié dans le bon ordre. Si le bisect ne renvoie que les index extremes du tableau c'est que ce dernier n'est pas trié dans le bon ordre.

``` Javascript
var bisect = d3.bisector(function(d){return d}).left;
var valeur = myThis.xScaleLine.invert(d3.mouse(this)[0]);     
var years = myThis.mainService.unique(myThis.interfaceService.dataService.globalDates) //.reverse();

var index = bisect(years, valeur);
```

## Décomposition de tableaux & objets
*Javascript & Typescript*

Pour fusionner deux objets, ou ajouter des attributs à la volée il est courant d'utiliser `Object.assign(...)`.

```JavaScript
const firstObj = {name: 'John', age: 45};
const finalObj = Object.assign(firstObj, {lastname: 'Wick', size: 180});
//finalObj == {name: 'John', age: 45, lastname: 'Wick', size: 180}
```

Il est possible d'utiliser la décomposition de tableaux/objets dans les mêmes cas d'utilisation pour les mêmes résultats.

```JavaScript
const firstObj = {name: 'John', age: 45};
const finalObj = {...firstObj, lastname: 'Wick', size: 180};
//finalObj == {name: 'John', age: 45, lastname: 'Wick', size: 180}
```

Plus facile à lire, moins verbeux.
En cas d'attributs de même nom, les premiers sont remplacés par les suivants.
La décomposition suit le même schéma pour les tableaux.

### Affectation

Sur une même expression, on peut déclarer plusieurs variables et leur affecter à chacune une valeur.

```JavaScript
[a, b] = [5, 9];
//a = 5
//b = 9
```

```JavaScript
{a, b} = {a: 5, b: 9};
//a = 5
//b = 9
```

Pratique lorsque l'on veut récupérer plusieurs valeurs depuis une fonction et les assigner directement.

```JavaScript
function do() {
    return {x: 45, y: 98};
}

{x, y} = do();
//x = 45
//y = 98
```

Néanmoins attention à la lisibilité.

Plus d'infos https://developer.mozilla.org/fr/docs/Web/JavaScript/Reference/Op%C3%A9rateurs/Affecter_par_d%C3%A9composition
