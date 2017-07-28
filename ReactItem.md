# ReactItem :fire:
Comment fonctionne la classe, comment l'utiliser, l'implémenter.

On admettra les bases de Javascript, Typescript, React et ReactTransitionGroup comme étant connues. 
Les imports ne seront pas affichés.

---

* [Structure globale](#structure-globale)
  * [Interfaces](#interfaces)
  * [Classe](#classe)
* [Informations préliminaires](#informations-préliminaires)
  * [Structure modèle](#structure-modèle)
* [Propriétés](#propriétés)
  * [Props](#props)
  * [State](#state)
* [Fonctions](#fonctions)
  * [Abstraites](#abstraites)
  * [Utilisables](#utilisables)
  * [Moteur](#moteur)
* [ReactItemTransition](#reactitemtransition)
  * [Indications](#indications)
  * [Structure globale](#structure-globale-1)
  * [Structure modèle](#structure-modèle-1)
  * [Fonctions](#fonctions-1)


---

## Structure globale

### Interfaces

Elément propre à typescript, les interfaces sont très utilisées pour homogénéiser les structures des données et en informer le développeur. Les ReactItem en utilise plusieurs.

Pour la suite du document, il vous sera nécessaire de revenir plusieurs fois dans cette section.

---

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

```TypeScript
interface ScaleGroup {
    group: Grouping<string | Date, Value>[];
    scales: (ScaleBand<string> | ScaleLinear<number, number> | ScaleTime<number, number>)[];
}
```

ScaleGroup représente les données finales, filtrées et définies par l'indicateur souhaité, ainsi que les scales calculés, qui serviront aux placement des éléments.

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

    public static readonly ADDED_CLASSNAME: string = ...;
    public static readonly TRANSITION_DEFAULT: number = ...;

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

ReactItem est une classe abstraite, en cela elle doit forcément être **étendue** par une classe créée dans un autre fichier. 

Il en est de même pour les interfaces ReactItemProps et ReactItemState. 
Une interface ne pouvant pas être abstraite, c'est au développeur de faire attention à bien les étendre via des interfaces créées dans le même fichier que la classe, quand bien même ces interfaces seraient vides.

Etendre ces différents composants est nécessaire afin de garder une certaine indépendance entre les éléments.

Il est impératif d'**exporter** la nouvelle classe tout en utilisant le mot-clef `default`.

### Structure modèle

Admettons la création d'une classe `Circle` étendant ReactItem.

```TypeScript
interface CircleProps extends ReactItemProps {...}

interface CircleState extends ReactItemState {...}

export default class Circle extends ReactItem<CircleProps, CircleState> {

    constructor(props: CircleProps) {
        super(props);
    }

    render(): JSX.Element {...}

    protected getDataProps(): DataProps | undefined {...}

}
```

## Propriétés

Les propriétés fournies et calculées par l'élément ReactItem sont possédées par l'attribut `props` et l'attribut `state`. Néanmoins il existe d'autres propriétés :
* `padding: CSSBox` qui représente les valeur de padding utilisées;
* `ADDED_CLASSNAME` (static !) qui représente le nom de classe HTML des éléments ajoutés dynamiquement;
* `TRANSITION_DEFAULT` (static !) qui représente la durée de transition par défaut;


### Props

Accessible depuis `this.props`.

* `controller: MainController`

  Controleur

* `json?: ViewProperties.TreeElement`
                                    
  Les propriétés définies par le fichier de propriété de la vue, concernant l'élément.
  Non défini s'il s'agit d'un élément ajouté dynamiquement.

* `data?: DatasetsView`
                                                          
  Ensemble des data fournies par crossfilter structurées ainsi `data[nom_dataset][nom_dimension]: liste des data`. 
  Non défini lors du premier `render`.

* `key?: string | number`
                                                            
  Index de l'élément, utilisé par React lors de l'affichage d'une liste d'élément.
  Non défini si l'élément n'est pas affiché parmi une liste.

* `attr: {
        x: number;
        y: number;
        width: number;
        height: number;
        [key: string]: string | number | undefined | RecursifProp;
    }`
                                         
  Attributs HTML fournis à l'élément HTML principal.

* `transition: number`
                                                         
  Durée des transitions.

* `childrenAdded: ChildrenAdded`
                                                                   
  Liste des éléments ajoutés dynamiquement, définis par leur clef.

* `children: ReactItemData[]`
                                                                
  Liste des éléments enfant.

* `added?: boolean`
                                                      
  Défini s'il s'agit d'un élément ajouté dynamiquement ou non.


### State

Accessible depuis `this.state`.

* `group: Grouping<string | Date, Value>[]`

  Représente les data finales, précédemment filtrées et sélectionnées sur l'indicateur souhaité.

* `scales: (ScaleBand<string> | ScaleLinear<number, number> | ScaleTime<number, number>)[]`
                                                                                          
  Représente les scales d3 paramétrés et utilisables par la vue.

* `attr: {
        className: string;
        [key: string]: string | number | undefined | {};
    }`
     
  Représente les attributs HTML intégrés à l'élément HTML principal.

## Fonctions

Plusieurs fonctions sont présentes dans chaque ReactItem.

Certaines d'entre-elles sont abstraites et doivent être implémentées par la classe étendue.
D'autres sont simplement utilisables. Enfin, certaines se limitent à la classe ReactItem et ne peuvent pas être réutilisées.

### Abstraites

Ces fonctions doivent être implémentées par la classe étendue.

```TypeScript
render(): JSX.Element {...}
```

Fonction principale de React, elle défini la représentation HTML de l'élément. 
La fonction doit retourner un élément JSX soit via la syntaxe du même nom (en utilisant des balises), soit via la fonction `React.createElement()`.

Pour des raisons de performance il est conseillé (mais pas forcé) d'effectuer les éventuels calculs dans d'autres fonctions (comme `componentDidUpdate()`) afin de limiter l'action de la fonction au simple rendu visuel.

```TypeScript
protected getDataProps(): DataProps | undefined { ... }
```

Fonction utilisée par ReactItem pour définir les data finales ainsi que les scales utilisés par la vue.

Si l'élément ne compte utiliser ni data ni scales, il est alors possible de retourner `undefined`.
Dans le cas contraire, la fonction doit retourner un objet composé d'une fonction appliquant les filtres préliminaires aux data, 
ainsi qu'un tableau composé d'objet définissant les scales disponibles pour chaque scale défini dans le fichier des propriétés de la vue.

### Utilisables

Ces fonctions sont utilisables depuis n'importe quelle classe étendant ReactItem. On négligera les fonctions propres à React.

:exclamation: **ATTENTION** :exclamation:
A moins de souhaiter volontairement briser leur fonctionnement, **en cas de surcharge du `constructor` ou de la fonction `componentWillReceiveProps()` vous devez absolument appeler le `super.` de la fonction !**

```TypeScript
protected setAllPadding(val: number): CSSBox {...}
```

Cette fonction permet simplement d'affecter une même valeur pour l'ensemble des attributs de la propriété `padding`.

```TypeScript
protected getInitialState(props: P): S {...}
```

Cette fonction, appelée une seule fois par le constructeur uniquement, prépare et retourne les données initiales du state (notamment en appelant la fonction `getData()`).

A priori il n'est pas utile de réutiliser cette fonction, mais il peut être utile, dans de rares cas, de surcharger la fonction pour modifier la valeur de retour.

### Moteur

Ces fonctions se limitent à la classe ReactItem, les classes étendues ne peuvent pas les utiliser. 
La connaissance de leur fonctionnement n'est pas indispensable pour l'utilisation de la classe.

```TypeScript
private getData(props: P): ScaleGroup {...}
```

Cette fonction retourne les data finales et scales qui pourront être utilisés par la vue.

## ReactItemTransition

ReactItemTransition est une classe abstraite étendue de ReactItem. Le principe de la classe est de mettre l'accent sur les transitions de l'élément.

### Indications

Il est important de noter que cette classe ne permet de faire aucune action en plus qu'une classe étendant ReactItem, 
il s'agit juste de guider le développeur en le "forcant" à implémenter certaines fonctions liées aux transitions.

Ce que vous pouvez faire avec une classe étendant ReactItemTransition, vous pouvez le faire avec une classe étendant ReactItem.

Pour profiter des spécificités de la classe, **il est indispensable que la fonction `render()` retourne un élément ReactTransitionGroup**.

### Structure globale

```TypeScript
export abstract class ReactItemTransition<P extends ReactItemProps = ReactItemProps, S extends ReactItemState = ReactItemState>
    extends ReactItem<P, S> {

    constructor(props: P) {
        super(props);
    }

    componentWillAppear(callback: () => void): void {...}

    protected abstract willAppear(node: Selection<Element, {}, null, undefined>, isAdded: boolean,
                                  addedNodes: Selection<BaseType, {}, BaseType, {}>, transition: number, callback: () => void): void;

    componentDidAppear(): void {...}

    protected abstract didAppear(node: Selection<Element, {}, null, undefined>, isAdded: boolean,
                                 addedNodes: Selection<BaseType, {}, BaseType, {}>, transition: number): void;

    componentWillEnter(callback: () => void): void {...}

    protected abstract willEnter(node: Selection<Element, {}, null, undefined>, isAdded: boolean,
                                 addedNodes: Selection<BaseType, {}, BaseType, {}>, transition: number, callback: () => void): void;

    componentDidEnter(): void {...}

    protected abstract didEnter(node: Selection<Element, {}, null, undefined>, isAdded: boolean,
                                addedNodes: Selection<BaseType, {}, BaseType, {}>, transition: number): void;

    componentDidUpdate() {...}

    protected abstract didUpdate(node: Selection<Element, {}, null, undefined>, isAdded: boolean,
                                 addedNodes: Selection<BaseType, {}, BaseType, {}>, transition: number): void;

    componentWillLeave(callback: () => void): void {...}

    protected abstract willLeave(node: Selection<Element, {}, null, undefined>, isAdded: boolean,
                                 addedNodes: Selection<BaseType, {}, BaseType, {}>, transition: number, callback: () => void): void;

    componentDidLeave(): void {...}

    protected abstract didLeave(node: Selection<Element, {}, null, undefined>, isAdded: boolean,
                                addedNodes: Selection<BaseType, {}, BaseType, {}>, transition: number): void;

}
```

### Structure modèle

Admettons la création d'une classe Circle étendant ReactItemTransition. Les différences par rapport à ReactItem se limitent au nom de la classe étendue et à l'implémentation des fonctions.

```TypeScript
interface CircleProps extends ReactItemProps {...}

interface CircleState extends ReactItemState {...}

export default class Circle extends ReactItemTransition<CircleProps, CircleState> {

    constructor(props: CircleProps) {
        super(props);
    }

    render(): JSX.Element {...}

    protected willAppear(node: Selection<Element, {}, null, undefined>, isAdded: boolean, addedNodes: Selection<BaseType, {}, BaseType, {}>, transition: number, callback: () => void): void {...}

    protected didAppear(node: Selection<Element, {}, null, undefined>, isAdded: boolean, addedNodes: Selection<BaseType, {}, BaseType, {}>, transition: number): void {...}

    protected willEnter(node: Selection<Element, {}, null, undefined>, isAdded: boolean, addedNodes: Selection<BaseType, {}, BaseType, {}>, transition: number, callback: () => void): void {...}

    protected didEnter(node: Selection<Element, {}, null, undefined>, isAdded: boolean, addedNodes: Selection<BaseType, {}, BaseType, {}>, transition: number): void {...}

    protected didUpdate(node: Selection<Element, {}, null, undefined>, isAdded: boolean, addedNodes: Selection<BaseType, {}, BaseType, {}>, transition: number): void {...}

    protected willLeave(node: Selection<Element, {}, null, undefined>, isAdded: boolean, addedNodes: Selection<BaseType, {}, BaseType, {}>, transition: number, callback: () => void): void {...}

    protected didLeave(node: Selection<Element, {}, null, undefined>, isAdded: boolean, addedNodes: Selection<BaseType, {}, BaseType, {}>, transition: number): void {...}

    protected getDataProps(): DataProps | undefined {...}

}
```

### Fonctions

L'ensemble des fonctions de ReactItem garde leur visibilié et leur fonctionnement, aucun changement leur est fait.

ReactItemTransition ajoute plusieurs fonctions liées aux transitions: 7 fonctions React et ReactTransitionGroup surchargées, et le même nombre créées qui doivent être implémentées.

Les fonctions React surchargées sont sous la forme `componentDid---()` ou `componentWill---()`. Pour chacune d'entre elles, une fonction abstraite est présente.

Pour `componentDidUpdate()`, il existe une fonction `didUpdate()`. Pour `componentWillEnter()`, il existe une fonction `willEnter()`. Et ainsi de suite.

Ainsi chaque fonction React débutant par `component` prépare certaines variables puis appelle la fonction correspondante avec ces variables en paramètre.

Les paramètres de ces nouvelles fonctions sont (dans l'ordre): 
* node: l'élément sélectionné par d3;
* isAdded: un boolean qui défini si l'élément est un élément ajouté dynamiquement;
* addedNodes: une sélection d3 de l'ensemble des éléments enfants ajoutés dynamiquement;
* transition: la durée des transitions;
* callback (pour les fonctions débutant par `will`): la fonction de callback à appeler lorsque la transition est terminée;

:exclamation: **ATTENTION** :exclamation:
A moins de souhaiter volontairement briser leur fonctionnement, **en cas de surcharge des fonction React (débutant par `component`) vous devez absolument appeler le `super.` de la fonction !**

---

Si vous avez tout lu d'une traite vous méritez un :cookie: 

[Retour en haut :arrow_up:](#reactitem-fire)
