# tipsNtricks
## bisector

Bisector ne renvoie le bon index que si le tableau de référence est trié dans le bon ordre. Si le bisect ne renvoie que les index extremes du tableau c'est que ce dernier n'est pas trié dans le bon ordre.

``` Javascript
var bisect = d3.bisector(function(d){return d}).left;
var valeur = myThis.xScaleLine.invert(d3.mouse(this)[0]);     
var years = myThis.mainService.unique(myThis.interfaceService.dataService.globalDates) //.reverse();

var index = bisect(years, valeur);
```
