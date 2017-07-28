# ReactItem
Comment fonctionne la classe, comment l'utiliser, l'implémenter.

On admettra les bases de Javascript, Typescript et React comme étant connues. 
Les imports ne seront pas affichés.

* [Structure globale](#structure-globale)
  * [Interfaces](#interfaces)
  * [Classe](#classe)
* [Informations préliminaires](#informations-préliminaires)
* [Propriétés](#propriétés)
  * [Props](#props)
  * [State](#state)
* [Fonctions](#fonctions)
  * [Abstraites](#abstraites)
  * [Utilisables](#utilisables)
  * [Moteur](#moteur)
* [ReactItemTransition](#reactitemtransition)

## Structure globale

### Interfaces

```TypeScript
export interface RecursifProp {
    [key: string]: string | number | undefined | RecursifProp;
}
```

RecursifProp représente des propriétés pouvant être imbriquées. 
Comme les attributs HTML possédant un attribut `style` composé des mêmes types d'attributs.

```TypeScript
export interface ChildrenAdded {
    [key: string]: ReactItemData[];
}
```

ChildrenAdded représente les différents enfants ajoutés dynamiquement, définis par leur clef.

```TypeScript
export interface ReactItemData {
    component: ComponentClass<ReactItemProps>;
    props: ReactItemProps;
    children: ReactItemData[];
}
```

ReactItemData représente les propriétés de chacun des enfants, dynamiques ou non, de l'élément.

```TypeScript
export interface ReactItemProps {
    controller: MainController;
    json?: ViewProperties.TreeElement;
    data?: DatasetsView;
    key?: string | number;
    attr: {
        x: number;
        y: number;
        width: number;
        height: number;
        [key: string]: string | number | undefined | RecursifProp;
    };
    transition: number;
    childrenAdded: ChildrenAdded;
    children: ReactItemData[];
    added?: boolean;
}
```

ReactItemProps représente l'ensemble des propriétés qui sont fournis à l'élément par le parent lors de son `render`. 
A noter que cette interface doit forcément être étendue par une interface propre à la classe étendue. 

L'attribut `data` est composé de l'ensemble des data données par crossfilter structurées ainsi `data[nom_dataset][nom_dimension]: liste des data`. Il est absent uniquement lors du premier `render` .
L'attribut `json` compose l'ensemble des propriétés de la vue propres à l'élément. Il est absent uniquement s'il s'agit d'un élément ajouté dynamiquement.
L'attribut `added` défini si l'élément est ajouté dynamiquement ou non.

```TypeScript
interface ScaleGroup {
    group: Grouping<string | Date, Value>[];
    scales: (ScaleBand<string> | ScaleLinear<number, number> | ScaleTime<number, number>)[];
}
```

ScaleGroup représente les données finales, filtrées et définies par l'indicateur souhaité, ainsi que les *scales* calculés, qui serviront aux placement des éléments.

```TypeScript
export interface ReactItemState extends ScaleGroup {
    attr: {
        className: string;
        [key: string]: string | number | undefined | {};
    };
}
```

ReactItemState représente l'ensemble des propriétés qui sont calculées lors de la récupérations de nouvelles `props`, ainsi que les propriétés ajoutées à la volée depuis une classe extérieur (pour le moment aucune).
De la même manière que ReactItemProps, l'interface doit forcément être étendue par une interface propre à la classe étendue.

L'interface possède les attributs de ScaleGroup, ainsi que les `attr` qui seront données en dur à l'élément HTML principal.

```TypeScript
export interface ScaleProps {
    band?: {
        domain: (group: Grouping<string | Date, Value>[]) => string[];
        range: [number | { valueOf(): number }, number | { valueOf(): number }];
    };
    linear?: {
        domain: (group: Grouping<string | Date, Value>[]) => Array<number | { valueOf(): number }>;
        range: number[];
    };
    time?: {
        domain: (group: Grouping<string | Date, Value>[]) => Array<Date | number | { valueOf(): number }>;
        range: number[];
    };
}
```

ScaleProps représente pour un scale donné l'ensemble des scales disponibles avec leur propriétés: 
le `domain` sous forme de fonction avec le group en paramètre,
le `range` sous forme de tableau.

```TypeScript
export interface DataProps {
    group: (group: Grouping<string | Date, Value>[]) => Grouping<string | Date, Value>[];
    scales: ScaleProps[];
}
```

DataProps représente les propriétés utilisées pour calculer les data finales et les scales qui seront utilisées par la classe.
L'attribut `group` est une fonction avec le group en paramètre qui applique les filtres préliminaires.
L'attribut `scales` représente l'ensembles des propriétés des scales en accord avec les propriétés de la vue.

```TypeScript
interface CSSBox {
    left: number;
    right: number;
    top: number;
    bottom: number;
}
```

CSSBox représente les propriétés utilisées par les éléments CSS comme padding ou margin.

### Classe

```TypeScript
export abstract class ReactItem<P extends ReactItemProps = ReactItemProps, S extends ReactItemState = ReactItemState>
    extends Component<P, S> {

    public static readonly ADDED_CLASSNAME: string = 'added';
    public static readonly TRANSITION_DEFAULT: number = 1000;

    protected padding: CSSBox;

    constructor(props: P) {...}

    componentWillReceiveProps(nextProps: P) {...}

    protected setAllPadding(val: number): CSSBox {...}

    protected getInitialState(props: P): S {...}

    protected abstract getDataProps(): DataProps | undefined;

    private getData(props: P): ScaleGroup {...}

}
```

## Informations préliminaires

## Propriétés

### Props

### State

## Fonctions

### Abstraites

### Utilisables

### Moteur

## ReactItemTransition
