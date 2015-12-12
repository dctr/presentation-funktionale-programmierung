# Funktionale Programmierung mit Java 8

... ein pragmatischer Ansatz

Note: Pragmatisch da ohne viel Mathe



# Was bedeuted FP?

Funktionale Programmierung = FP



## Imperative Programmierung

Allgemein bekannt -- hoffentlich

- Beschreibe *imperativ* **wie** etwas getan werden muss
- Schritt für Schritt Anleitung



## Im Gegensatz dazu: Funktionale Programmierung

- Beschreibe *funktional* **was** getan werden muss
- Beschäftigt sich weniger mit dem "wie"

# Herleitung über bekannte Grundlagen

**Das Command Pattern** (siehe Vortrag von Thomas Kopp)

Kapselt konkretes Verhalten hinter einem abstrakten Interface

## Interface

```java
@FunctionalInterface
public interface Command {
    void execute(Object param);
}
```

Annotaion `@FunctionalInterface` aus Java 8 erstmal ignorieren.

## Implementierung

```java
public class MyCommand implements Command {
    void execute(Object param) {
        // Do something meaningful.
    }
}
```

## Aufruf klassisch, mit dedizierter Klasse

```java
public class Main {
    public static void main(String[] args) {
        Executor e = new Executor();
        e.doSomeAction(new MyCommand());
    }
}
```

## Aufruf klassisch, mit anonymer Klasse

Anonyme Klassen statt separater Implementierung

```java
public class Main {
    public static void main(String[] args) {
        Executor e = new Executor();
        e.doSomeAction(new Command() {
            void execute(Object param) {
                // Do something meaningful.
            }
        });
    }
}
```

## Aufruf mit Java 8 Lambda-Ausdruck

```java
public class Main {
    public static void main(String[] args) {
        Executor e = new Executor();
        e.doSomeAction((Object param) -> {
            // Do something meaningful.
        });
    }
}
```

## Kurzschreibweisen

```java
List<String> teamSchadow = Arrays.asList("Dominik", "Sandro", "Jochen", ...);
```

```java
friends.forEach((String name) -> {
    System.out.println(name)
});
```

- Automatische Typinferenz
- Keine geschweiften Klammern bei Einzeilern

```java
teamSchadow.forEach(name -> System.out.println(name));
```

- Methodenreferenz (yey, function pointers!), wenn Parameter 1:1 übernommen wird

```java
teamSchadow.forEach(System.out::println);
```

- Wie auch bei anonymen Klassen in Variablen speicherbar

```java
Consumer<String> myPrintLn = name -> System.out.println(name);
teamSchadow.forEach(myPrintLn);
```

## FunctionalInterface

- Lambda ist quasi nur Kurzschreibweise für anonyme Klasse
- `@FunctionalInterface`s sind Interfaces mit *exakt* einer Methode
- Bei Angabe eines Lambda-Ausdrucks daher automatische Typinferenz möglich

## Anwendungsfälle

- Wie Command Pattern (offensichtlicherweise)
- Definieren von (asynchronen) Callbacks (s. auch mein Vert.X-Vortrag)
- Coole Dinge, die wir in diesem Vortrag sehen werden
- ... u. v. m.

# FP Beispiel

Summiere alle Preise über 20 EUR, ermäßigt um 10 %

## Stufe 1 - Wie in alten Tagen

```java
Collection<BigDecimal> prices = MyApi.getPrices();
BigDecimal price;
BigDecimal totalOfDiscountedPrices = BigDecimal.ZERO;
int i = 0;

while (i < prices.size()) {
    price = prices.get(i);
    if (price.compareTo(BigDecimal.valueOf(20)) > 0) {
        totalOfDiscountedPrices =
                totalOfDiscountedPrices.add(price.multiply(BigDecimal.valueOf(0.9)));
    }
    i++
}

System.out.println("Result: " + totalOfDiscountedPrices);
```

## Stufe 2 - Syntaktischer Zucker

```java
Collection<BigDecimal> prices = MyApi.getPrices();
BigDecimal totalOfDiscountedPrices = BigDecimal.ZERO;

for (BigDecimal price : prices) {
    if (price.compareTo(BigDecimal.valueOf(20)) > 0) {
        totalOfDiscountedPrices =
                totalOfDiscountedPrices.add(price.multiply(BigDecimal.valueOf(0.9)));
    }
}

System.out.println("Result: " + totalOfDiscountedPrices);
```

## Stufe 3 - Eine neue Welt

Java 8: Stream-API und Lambdas = "Funktionale Programmierung" (-artig)

```java
final BigDecimal totalOfDiscountedPrices = MyApi.getPrices().stream()
        .filter(price -> price.compareTo(BigDecimal.valueOf(20)) > 0)
        .map(price -> price.multiply(BigDecimal.valueOf(0.9)))
        .reduce(BigDecimal.ZERO, BigDecimal::add);

System.out.println("Result: " + totalOfDiscountedPrices);
```

# Fragen soweit?

- Grundidee sollte ab hier klar sein
- Verständnisfragen bitte jetzt

# Warum sollte ich das tun?

## Nachteile imperativer Programmierung

- Redundanter, manuell erstellter Code (lies: Fehler)
  - Boilerplate-Code (Deklaration anonymer Klassen VS Lambda-Syntax)
  - Low-Level Code (Verzweigungen, Schleifen, ... VS filter(), forEach(), ...)
  - Parallelisierung (Threads starten, Synchronisation, ...)
- Vermischt was *was* (auszuführende Aktion, aka Command) mit dem *wie* (iterieren, ...)

## Vorteile funktionaler Programmierung

- Prägnant und Ausdrucksstark (Boilerplate Code aus Bibliothek unter der Haube)
- Näher an natürlicher Sprache (intuitiver, lesbarer, wartbarer)
- Inhärent parallelisierbar (da unter der Haube, z. B. bei `map()`)
- Weniger Fehleranfällig (weniger Code, keine Variablenmuation)

# Was bedeutet FP in grauer Theorie)?

- Deklarativ (Ausdruck über Anweisung, das viel wiederholte *was* statt *wie*)
- Unveränderbarkeit (keine Mutation externer Variablen, jedes Lambda gibt Ergebnis an nächstes weiter)
- Keine Seiteneffekte (durch Unveränderbarkeit)
- Funktionen höherer Ordnung (Funktionen nehmen Funktionen entgegen, s. auch Command)

# Wie umsetzen - Teil 1: Java Boardmittel

## Grundverben

Am Beispiel der neuen Stream API aus Java 8

- java.util.stream Interface Stream<T>: Implementierer stellen Grundverben bereit
- java.util.function gibt große Vorauswahl fertiger `FunctionalInterface`s die von Grundverben entgegengenommen werden
  - Consumer<T>: Represents an operation that accepts a single input argument and returns no result.
  - Function<T,R>: Represents a function that accepts one argument and produces a result.
  - ...

## Fluent-API

- Rückgabewert der meisten Grundverben wieder ein Stream
- Aktionen manipulieren Stream
- Bestimmen also, was die nächste Aktion als Eingabe-Stream erhält

```java
collection.stream()
    .someAction(lambda1) // Receives all items of collection
    .otherAction(lambda2) // Receives items emitted by someAction()
    .furtherAction(lambda3); // Receives items emitted by otherAction()
```

### stream() => 1:n

`Stream<T> Collection<T>::stream()`

```java
collection.stream().someAction(lambda);
```

- Stream ist Iterator-ähnliches FP-Pendant seit Java 8
- Vgl. `iterable.forEach(func)`
- `someAction`s lambda wird für jedes Element der Collection ein Mal aufgerufen

### map() => n:n

`<R> Stream<R> Stream<T>::map(Function<? super T, ? extends R> mapper)`

```java
teamSchadow.stream()
    .map(name -> name.toUpperCase);

// "DOMINIK", "SANDRO", "JOCHEN", ...
```

- Manipulation von Elementen des Stream durch Function
- Rückgabetyp der Function bestimmt Typ des anschießenden Streams
- Single-Line-Statement => kein Return benötigt

### filter() => n:m (m < n)

`Stream<T> Stream<T>::filter(Predicate<? super T> predicate)`

```java
teamSchadow.stream()
    .filter(name -> name.startsWith("D"))
    .map(name -> name.toUpperCase);

// "DOMINIK", ...
```

- Return-Typ von Predicate ist boolean
- filter() prüft, ob Predicate true zurückgibt
- "true"-Elemente werden propagiert

### reduce() / collect() => n:1

`T Stream<T>::reduce(T identity, BinaryOperator<T> accumulator)`

```java
teamSchadow.stream()
    .reduce("", (collector, name) -> {
        return String.join(", ", collector, name)
    });

// "Dominik, Sandro, Jochen, ..."
```

- BinaryOperator kombiniert jedes Element des Streams mit dem Ergebnis der vorausgegangenen Operation von BinaryOperator
- Initialer Wert bei Eingang des ersten Elements sollte Identität sein (s. erster Parameter von reduce)
- Rückgabewert ist einzelnes Element des Typs, kein neuer Stream

### ... und viele mehr

In java.util.stream

- `long count()`
- `Stream<T> sorted(Comparator<? super T> comparator)`
- `Stream<T> limit(long maxSize)`
- ...

In java.util.stream.Collectors

- Viele vorgefertigte Kollektoren für reduce-ähnliches `collect()` (sum, avg, join, toList, ...)

java.util.Optional<T> (Javas halbherziger Versuch NPEs zu umschiffen)

## Inhärente Parallelisierbarkeit

`streamParallel()` statt `stream()`, so einfach geht das.

### Inhärente Parallelisierbarkeit II

Die Einfachheit der Anpassung (und damit einhergehend die Folienanzahl) wird der Mächtigkeit kaum gerecht.

### Inhärente Parallelisierbarkeit III

Daher strecke ich auf mehrere Folien :-)

# Wie umsetzen - Teil 2: Zusätzliche Bibliotheken

## [ReactiveX](http://reactivex.io/) (RxJava, RxJS, Rx.NET, RxPY, ...)

> ReactiveX is a combination of the best ideas from the Observer pattern, the Iterator pattern, and functional programming.

- Bekannt, verbreitet und mächtig
- Im Projekt beim Kunden erfolgreich im Einsatz
- Umfangreicher als java.util.stream
  - join(), merge(), zip()
  - doOnNext(), doOnError(), doOnCompleted()
  - ...
- Dezent komplexer in Möglichkeiten und Einarbeitung

@Thomas: Am nächsten Entwicklertag Observer-Pattern?

## und weitere

- [Functional Java](http://www.functionaljava.org/) (viel Conveniece)
- [Google Guava](https://github.com/google/guava) (auch mit Observer)
- ...


# Wie umsetzen - Teil 3: Andere JVM-Sprachen

- [Clojure](http://clojure.org/) (Lisp-Dialekt; JVM, CLR, JS)
- [Scala](http://www.scala-lang.org/) (nicht strikt-funktional, wie Java auch, aber "optimierter" für FP; JVM, JS)
- ...





# Funktionale Entwurfsmuster

## Delegates / Dependency Injection

```java
public static int totalAssetValues(final List<Asset> assets, final Predicate<Asset> assetSelector) {
    return assets.stream()
        .filter(assetSelector)
        .mapToInt(Asset::getValue)
        .sum();
}

// ...

System.out.println("Total of all assets: " + totalAssetValues(assets, asset -> true));
System.out.println("Total of bonds: " + totalAssetValues(assets, asset -> asset.getType() == AssetType.BOND));
System.out.println("Total of stocks: " + totalAssetValues(assets, asset -> asset.getType() == AssetType.STOCK));
```

Dependency Injection; macht auch testen einfacher (z. B. `asset -> throw new EnumConstantNotPresentException("damn");`)

## Decorator

### Ausgangsklasse

```java
public class Camera {
    private Function<Color, Color> filter;

    public Camera() {
        this.filter = Function::identity;
    }

    public Camera(Function<Color, Color> filter) {
        this.filter = filter;
    }

    public Color capture(final Color inputColor) {
        return filter.apply(inputColor);
    }
}
```

### Decorator für Multi-Filter - klassisch

Ableitung mit Überladung `public Color capture(final Color... inputColors)`?

> Composition over Inheritance

Wrapper-Klasse die Camera-Instanz entgegennimmt?

### Decorator für Multi-Filter - FP-Kompositionen

```java
Function<Color, Color> myFilters = Filters::brigther.compose(Filters::sepia)
Camera multiFilterCamera = new Camera(myFilters);
```

- Consumer<T>: andThen(...)
- Function<T,R>: andThen(...), compose(...)
- Predicate<T>: and(...), or(...), negate()

## Execute Around Method

Anwendungsgebiet: Resourcen-Management

### Klassisch

```java
public class Mailer {
    public void from(final String address) { /*... */ }
    public void to(final String address) { /*... */ }
    public void subject(final String line) { /*... */ }
    public void body(final String message) { /*... */ }
    public void send() { privateValidate(); privateSend(); }
}
```

```java
Mailer mailer = new Mailer();
mailer.from("build@agiledeveloper.com");
mailer.to("venkats@agiledeveloper.com");
mailer.subject("build notification");
mailer.body("...your code sucks...");
mailer.send();
```

### Funktional

```java
public class FluentMailer {
private FluentMailer() {}
    public FluentMailer from(final String address) { /*... */; return this; }
    public FluentMailer to(final String address) { /*... */; return this; }
    public FluentMailer subject(final String line) { /*... */; return this; }
    public FluentMailer body(final String message) { /*... */; return this; }
    public static void send(final Consumer<FluentMailer> block) {
        final FluentMailer mailer = new FluentMailer();
        block.accept(mailer);
        mailer.privateValidate();
        mailer.privateSend();
    }
}
```

```java
FluentMailer.send(mailer ->
    mailer
        .from("build@agiledeveloper.com")
        .to("venkats@agiledeveloper.com")
        .subject("build notification")
        .body("...much better..."));
```

### Vorteile des "Execute Around Method"-Pattern

- Fluent => Keine Wiederholung von `mailer.*`
- Gekapselter Scope & Lifetime
  - wäre es im 1. Bsp erlaubt, mailer zu speichern und send() mehrfach aufzurufen?
  - kein Zugriff auf mailer nach send()
- Auch gut zum Resourcen-Management (DB-Verbindung oder Dateien öffnen und schließen, Lock-Verwaltung, Exception-Handling, etc.)
  - Spart Boilerplate-Aufräum-Code
  - Spart Boilerplate try/finally-Blöcke
  - Beugt vergessen dieses Boilerplate-Codes vor

## Lazy Evaluation

- Kostspielige Operationen sollten nur ausgeführt werden wenn nötig.
- Eventuell entscheidet eine leichtgewichtige Vorbedingung, ob eine kostspielige Operation ausgeführt wird.

```java
lightBoolean && heavy1() && heavy2()
```

- Möglich bei boolean-Ausdrücken
- Nicht möglich bei z. B. Funktionsparametern

```java
myMethod(lightBoolean, heavy1(), heavy2());
```

### Lambdas

Lambdas werden "vor Ort" ausgewertet, aber nicht aufgerufen-

```java
Supplier<Boolean> lightLambda1 = () -> heavy1();
Supplier<Boolean> lightLambda2 = () -> heavy2();

myMethod(lightBoolean, lightLambda1, lightLambda2);
```

### Steam<T> ist inhärent faul

Internmediate VS terminal

```java
List<String> names = Arrays.asList("Brad", "Kate", "Kim", "Jack", "Joe", "Mike", "Susan", "George", "Robert", "Julia", "Parker", "Benson");

final String firstNameWith3Letters = names.stream()
    .filter(name -> length(name) == 3)
    .map(name -> toUpper(name))
    .findFirst()
    .get();
```

> getting length for Brad
> getting length for Kate
> getting length for Kim
> converting to uppercase: Kim
> KIM

- Intermediates (filter() und map()) werden nur vom Stream gecachet
- Terminals (findFirst()) fragen nur so viele Elemente an, wie benötigt
- Erlaubt effizienten Umgang mit **infinit**en Streams (z. B. Stream<Integer> pimeNumbers)

# Eigenen Code funktional zugänglich machen

## Fluent API

### return this

`public void myMethod(param)` -> `public MyClass myMethod(param)`

### return Stream<T>

`public Collection<T> myMethod(param)` -> `public Stream<T> myMethod(param)`

`return collection;` -> `return collection.stream();`

## Akzeptiere FunctionalInterfaces

## Implementiere FuntionalInterfaces

## Eigene Methodenreferenzen

someStream.map(this::myExtractedFunction)

## Implementiere Stream<T>

### Statisch

- Vorab bekannte, fixe Anzahl an Elementen
- Von Einzelwerten: `static <T> Stream<T> of(T... values)`
- Von anderen Streams (Arrays, Datei Input, ...)

### Dynamisch

- Infinit: Implementiere `Supplier<T>`
  - `Supplier<T>::get()` muss als blockierender Aufruf infiniten Stream von Elementen erzeugen
  - Erzeugung eines Streams clientseitig durch `static <T> Stream<T> generate(Supplier<T> s)`
  - Client sollte `Stream<T> limit(long maxSize)` benutzen
- Finit: Gebe direkt den Stream zurück und setzte das limit() selbst

# Randnotizen

- Effective final (was von außen in einem Lambda-Ausdruck verwendet wird, ist `final`)
- Keine checked Exceptions in Lambdas (`Function<T,R>::apply(T t)` etc. haben keine throws-Klausel)
  - `throw new RuntimeException(e)` evtl. problematisch bei paralleler Ausführung
  - Komplexe Thematik, übersteigt diesen Vortrag
  - ReactiveX hat einen Mechanismus dafür















# ABER

- Definition von FP hier nur oberflächlich angekratzt
  - Ist sehr mathematisch
  - Beschäftigt sich auch stark mit Beweisbarkeit von Korrektheit
- In Java auch nicht 100% umsetzbar
  - Dinge fehlen, nur grundlegende FP-Konzepte umgesetzt
  - Mischung mit OOP erlaubt "aufbrechen" des Konzepts (z. B. in Lambdas auf Variablen außerhalb zugreifen => Seiteneffekt => keine "Pure Function")
  - FP ist in, daher muss Java es können...
- Nicht ganz so performant wie "echte" FP-Sprachen
  - Lambdas sind implizit anonyme Klassen

## Nichtsdestotrotz

Sehr hilfreich für den Alltag!

- Lesbarkeit
- Parallelisierbarkeit
- Sich mit FP bekannt machen (und fit für die Zukunft)

## Welche konzepte funktionaler Programmierung kann Java nicht?

## Problem

Man kann an FP "vorbeiprogrammieren", die Konzepte mischen und so noch unübersichtlicheren Code erzeugen als vorher

- Nicht alle Aspekte in Java strikt umsetzbar
- Funktionalität nicht forciert => Mischbetrieb möglich
- Einiges auch schon mit Java 7 möglich (anonymous inner classes), aber warum sollten wir?

# Recap

<Recaps aus FP>
