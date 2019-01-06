# Les Génériques en Java

Les *[Génériques](https://en.wikipedia.org/wiki/Generics_in_Java)* ont été ajoutés dans Java 5 afin de permettre *la vérification des types* au moment de la compilation et la suppression des risques de [ClassCastException](https://docs.oracle.com/javase/7/docs/api/java/lang/ClassCastException.html) qui étaient courants lors de l'utilisation des classes *[Collection](https://docs.oracle.com/javase/7/docs/api/java/util/Collection.html)*. L'ensemble du framework *Collection* a été réécrit pour utiliser les *Génériques* afin d'être *[type-safety](https://en.wikipedia.org/wiki/Type_safety#Java)*. Voyons comment les *Génériques* nous aident à utiliser les classes *Collection* en toute sécurité.

## Avant les *Génériques*
```java
List list = new ArrayList();
list.add("abc");
list.add(new Integer(5)); // OK

for (Object obj : list) {
    // type casting leading to ClassCastException at runtime
    String str = (String) obj; 
}
```

Pour utiliser un élément de la liste, il fallait faire un *cast* qui peut mener à une exception de type *ClassCastException*.
L'exemple ci-dessus compile bien mais génère une exception *ClassCastException* à l’exécution car nous essayons de *caster* un Integer en String.

Après Java 5, l'usage des *Collections* se fait comme ceci.
```java
List<String> list1 = new ArrayList<String>(); // ==> java 5
List<String> list2 = new ArrayList<>();       // ==> java 7
list1.add("abc");
list1.add(new Integer(5)); // compiler error

for (String str : list1) {
    // no type casting needed, avoids ClassCastException
}
```

## Classe Générique

Nous pouvons définir nos propres classes avec des types *Génériques*. Pour cela, nous utilisons des *<>* pour définir le paramètre *type*.
Pour comprendre son utilité, voici un exemple:

```java
public class GenericsType<T> {
    private T t;

    public T get() {
        return this.t;
    }

    public void set(T t1) {
        this.t = t1;
    }

    public static void main(String args[]) {
        GenericsType<String> type = new GenericsType<>();
        type.set("Toto"); // valid

        GenericsType type1 = new GenericsType(); // raw type
        type1.set("Toto"); // valid
        type1.set(10); // valid and autoboxing support
    }
}
```

## Interface Générique

*[Comparable](https://docs.oracle.com/javase/7/docs/api/java/lang/Comparable.html)* est un très bon exemple d'utilisation des *Génériques* dans une interface.

```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```

De la même façon, nous pouvons créer nos propres interfaces *Génériques*. Nous pouvons également avoir plusieurs types de paramètres *Génériques* comme dans l'interface *Map* (```java.util.Map<K, V>```).
Il est également possible d'imbriquer des types *Génériques* les uns dans les autres, par exemple ```HashMap<String, List<String>>();```

## Type Générique

Les conventions de nommage nous aident à mieux comprendre le code et c'est une très bonne pratique de programmation. Les *Génériques* ont donc leurs propres conventions de nommage. Généralement, les types sont définis avec une seule lettre en majuscule afin de faciliter la lecture du code. Nous pouvons ainsi les dissocier des variables java. Les nommages les plus communs sont les suivants :

* E – Element (très utilisé par le Framework Collection de Java, par exemple ArrayList, Set etc.)
* K – Key (Utilisé dans Map)
* N – Number
* T – Type
* V – Value (Utilisé dans Map)
* S,U,V etc. – 2ème, 3ème, 4ème types

## Méthode Générique

Parfois, il n'est pas nécessaire d'avoir toute la classe *Générique*, mais seulement quelques méthodes. Depuis que le constructeur est une méthode *spécifique*, nous pouvons appliquer les *Génériques* dessus.

```java
public class GenericsMethods {

    //Java Generic Method
    public static <T> boolean isEqual(GenericsType<T> g1, GenericsType<T> g2) {
        return g1.get().equals(g2.get());
    }

    public static void main(String args[]) {
        GenericsType<String> g1 = new GenericsType<>();
        g1.set("Toto");

        GenericsType<String> g2 = new GenericsType<>();
        g2.set("Toto");

        boolean isEqual = GenericsMethods.<String>isEqual(g1, g2);
        // above statement can be written simply as
        isEqual = GenericsMethods.isEqual(g1, g2);
        // This feature, known as type inference, allows you to invoke a generic method
        // as an ordinary method, without specifying a type between angle brackets.
        // Compiler will infer the type that is needed
    }
}
```

## <? et *limitation*>

Supposons que nous voulons restreindre le type des objets que nous voulons utiliser dans le paramétrage du *type*. Par exemple, dans une méthode qui compare deux objets entre eux et que nous souhaitons que ces objets héritent de *Comparable*.
Pour déclarer un type avec *limitation*, il faut indiquer le nom du type, suivi du mot-clé *extends*, suivi de sa *limitation*, comme dans la méthode ci-dessous.

```java
public static <T extends Comparable<T>> int compare(T t1, T t2) {
    return t1.compareTo(t2);
}
```

## <? et *limitation supérieure*>

Les *limitations supérieures* sont utilisées pour assouplir la restriction sur le type de variable dans une méthode. Supposons que nous voulions écrire une méthode qui renvoie la somme des nombres de la liste, de sorte que notre implémentation ressemble à ceci.

```java
public static double sum(List<Number> list) {
    double sum = 0;
    for (Number n : list) {
        sum += n.doubleValue();
    }
    return sum;
}
```

Maintenant, le problème avec l’implémentation ci-dessus est qu’elle ne fonctionnera pas avec une liste d'*Integer* ou de *Double* car nous savons que List\<Integer\> et List\<Double\> ne sont pas liés, c’est à ce moment que la *limitation supérieure* est utile. Nous utilisons le caractère *'?'* avec le mot-clé *extends* et la classe ou l’interface de la *limitation supérieure* qui nous permettra de passer les types d’arguments de la *limitation supérieure* ou de ses sous-classes.

L'implémentation ci-dessus peut être modifiée comme ceci.

```java
public class GenericsWildcards {

    public static void main(String[] args) {
        List<Integer> ints = new ArrayList<>();
        ints.add(3); ints.add(5); ints.add(10);
        double sum = sum(ints);
        System.out.println("Sum of ints="+sum);
    }

    public static double sum(List<? extends Number> list) {
        double sum = 0;
        for (Number n : list) {
            sum += n.doubleValue();
        }
        return sum;
    }
}
```

## <? et *limitation inférieure*>

Supposons que nous voulions ajouter des entiers à une liste d'entiers d'une méthode, nous pouvons conserver le type d'argument sous la forme List\<Integer\>, mais il sera lié à des entiers alors que List\<Number\> et List\<Object\> peuvent également contenir des entiers. Nous pouvons utiliser une *limitation inférieure* pour y parvenir. Pour ce faire, nous utilisons le caractère *'?'* avec le mot clé *super* et une classe de *limitation inférieure*.

Dans ce cas, le compilateur java permet d'ajouter des types d'objet de la *limitation inférieure* à la liste.

```java
public static void addIntegers(List<? super Integer> list) {
    list.add(new Integer(50));
}
```

Sous-typage des *Génériques* à l'aide de la *limitation supérieure*
```java
List<? extends Integer> intList = new ArrayList<>();
List<? extends Number>  numList = intList;  // OK. List<? extends Integer> is a subtype of List<? extends Number>
```
