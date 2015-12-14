## Funktionale Programmierung

- Mit Java 8
- Für Pragmatiker

Note: Praxisorientiert, ohne mathematische Aspekte



## FP Teaser

FP = Funktionale Programmierung

Note:
- Erfordert Grundlagen
- "Motivation" später
- Wir erhalten das Mysterium


### Imperative Programmierung

- Beschreibe **wie** etwas getan werden muss
- Schritt für Schritt Anleitung

Note: Allgemein bekannt - hoffentlich


### Funktionale Programmierung

- Beschreibe **was** getan werden muss
- Beschäftigt sich nicht mit dem **wie**



## Voraussetzungen

Grundbaustein: Kapselung dieses **was**

**Das Command Pattern:** Kapselt Verhalten hinter Interface

Note: Wie eben im Vortrag von Thomas gehört


### Interface

```java
@FunctionalInterface
public interface Command {
    void execute(Object param);
}
```

(Annotaion erstmal ignorieren)


### Aufruf mit dedizierter Klasse

```java
public class MyCommand implements Command {
    void execute(Object param) {
        // Do something meaningful.
    }
}

executor.doSomeAction(new MyCommand());
```


### Aufruf mit anonymer Klasse

```java
executor.doSomeAction(new Command() {
    void execute(Object param) {
        // Do something meaningful.
    }
});
```


### Aufruf mit Lambda-Ausdruck

```java
executor.doSomeAction((Object param) -> {
    // Do something meaningful.
});
```

Note: Nebenbei auch Lambdas erklärt, und das mit Absicht.


### Kurzschreibweisen für Lambdas

```java
List<String> teamSchadow =
    Arrays.asList("Dominik", "Sandro", "Jochen", ...);
```

```java
teamSchadow.forEach((String name) -> {
    System.out.println(name)
});
```

```java
teamSchadow.forEach(name -> System.out.println(name));
```

```java
teamSchadow.forEach(System.out::println);
```

```java
Consumer<String> myPrintLn = name -> System.out.println(name);
teamSchadow.forEach(myPrintLn);
```

Note:
- Automatische Typinferenz
- Keine geschweiften Klammern bei Einzeilern
- Methodenreferenz (yey, function pointers!), wenn Parameter 1:1 übernommen wird
- Wie auch bei anonymen Klassen in Variablen speicherbar


### `@FunctionalInterface`

- Lambda nur Kurzschreibweise für anonyme Klasse
- FIs haben *eine* (abstrakte) Methode
- Dadurch automatische Typinferenz möglich

Note:
- Die Annotation von eben
- Interfaces in Java 8 können default Methoden haben


### Anwendungsfälle

- Command Pattern
- Callbacks
- ... u. v. m.
- Coole Dinge in diesem Vortrag - aka FP

Note: Callbacks & asynchrone Programmierung, s. auch mein Vert.X-Vortrag



## Fragen soweit?

- Command Pattern
- Lambda-Ausdrücke
- FunctionalInterface

Note:
- Grundlagen sollten ab hier klar sein
- Grundidee = Kapselung von Logik



## FP Trailer

Note: Ausblick, bevor wir zur "Motivation" kommen.


### FP Theorie

- Deklarativ (*was* statt *wie*)
- Unveränderbarkeit (Lambda-Scope geschlossen)
- Keine Seiteneffekte (durch Unveränderbarkeit)
- Funktionen höherer Ordnung
- Lambda-Kalkül

Note:
- Ausdruck über Anweisung
- keine Mutation externer Variablen, in Java nicht umgesetzt, siehe später
- Funktionen nehmen Funktionen entgegen, s. auch Command


### Vom Command Pattern zu FP

> ... und jetzt nochmal für Pragmatiker, bitte!

**Beispiel:** Summiere Preise über 20 EUR, ermäßigt um 10 %


#### Stufe 1 - Wie in alten Tagen

```java
Collection<BigDecimal> prices = MyApi.getPrices();
BigDecimal price;
BigDecimal totalOfDiscountedPrices = BigDecimal.ZERO;
int i = 0;

while (i < prices.size()) {
    price = prices.get(i);
    if (price.compareTo(BigDecimal.valueOf(20)) > 0) {
        totalOfDiscountedPrices = totalOfDiscountedPrices
            .add(price.multiply(BigDecimal.valueOf(0.9)));
    }
    i++
}

System.out.println("Result: " + totalOfDiscountedPrices);
```


#### Stufe 2 - Syntaktischer Zucker

```java
Collection<BigDecimal> prices = MyApi.getPrices();
BigDecimal totalOfDiscountedPrices = BigDecimal.ZERO;

for (BigDecimal price : prices) {
    if (price.compareTo(BigDecimal.valueOf(20)) > 0) {
        totalOfDiscountedPrices = totalOfDiscountedPrices
            .add(price.multiply(BigDecimal.valueOf(0.9)));
    }
}

System.out.println("Result: " + totalOfDiscountedPrices);
```

Note: Erlaubt kompaktere Syntax, Denk- und Funktionsweise ist aber gleich.


#### Stufe 3 - Eine neue Welt

```java
final BigDecimal totalOfDiscountedPrices = MyApi
    .getPrices().stream()
    .filter(price -> price.compareTo(BigDecimal.valueOf(20)) > 0)
    .map(price -> price.multiply(BigDecimal.valueOf(0.9)))
    .reduce(BigDecimal.ZERO, BigDecimal::add);

System.out.println("Result: " + totalOfDiscountedPrices);
```

Note:
- WAS filtern wir, bzw. WAS bilden wir ab
- Nicht WIE (Iteration, Collections manipulieren)!
- Auf Einzelelement-Basis



## Motivation

Warum funktionale Programmierung?


### Theoretische Vorteile von FP

- Mathematische Beweisbarkeit von Korrektheit
- ...

Note: Blah, blah, blah ..., wir wollten Pragmatismus


### Pragmatische Nachteile imperativer Programmierung

- Vermischt das
  - *was* (Auszuführende Aktion, aka Command)
  - mit dem *wie* (iterieren, ...)
- Redundanter, manuell erstellter Code (lies: Fehler)
  - Boilerplate-Code (Deklaration von Command-Klassen)
  - Low-Level Code (Verzweigungen, Schleifen, ...)
  - Parallelisierung (Threads starten, Synchronisation, ...)


### Pragmatsche Vorteile von funktionaler Programmierung

- Prägnant und Ausdrucksstark (Boilerplate unter der Haube)
- Näher an natürlicher Sprache (intuitiver, lesbarer, wartbarer)
- Inhärent parallelisierbar (wenn unter der Haube implementiert)
- Weniger Fehleranfällig (weniger Code, keine Variablenmuation)



## Umsetzung - Teil 1

Java Boardmittel (aka die Stream-API) & "Grundverben"


### stream() ⇒ 1:n

`Stream<T> Collection<T>::stream()`

```java
collection.stream()
    .someAction(command);
```

- Erzeugt "Strom" von Elementen
- `someAction(command)` wird für jedes Element ein Mal aufgerufen
- Vgl. Iterator: `iterable.forEach(command)`


### API

- java.util.stream definiert `Stream<T>`; Streams stellen "Grundverben" bereit
- java.util.function definiert `FunctionalInterface`s die von "Grundverben" entgegengenommen werden
  - `Consumer<T>`: Represents an operation that accepts a single input argument and returns no result.
  - `Function<T,R>`: Represents a function that accepts one argument and produces a result.
  - ...

Note: Am Beispiel der neuen Stream API aus Java 8


### Fluent Interface

- Rückgabewert der meisten Stream-Operationen wieder ein Stream
- Aktionen manipulieren Stream
- Bestimmen, was nächste Aktion als Eingangs-Stream erhält

```java
collection.stream()
    .someAction(lambda1) // Receives all items of collection
    .otherAction(lambda2) // Receives items emitted by someAction()
    .furtherAction(lambda3); // Receives items emitted by otherAction()
```


### map() ⇒ n:n

`Stream<R> Stream<T>::map(Function<? super T, ? extends R> mapper)`

```java
teamSchadow.stream()
    .map(name -> name.toUpperCase);

// "DOMINIK", "SANDRO", "JOCHEN", ...
```

- Manipulation von Elementen des Stream durch `Function`
- Rückgabetyp der `Function` bestimmt Typ des anschießenden Streams

Note: Single-Line-Statement, kein Return benötigt


### filter() ⇒ n:m (m < n)

`Stream<T> Stream<T>::filter(Predicate<? super T> predicate)`

```java
teamSchadow.stream()
    .filter(name -> name.startsWith("D"))
    .map(name -> name.toUpperCase);

// "DOMINIK", ...
```

- Return-Typ von `Predicate` ist boolean
- filter() prüft, ob `Predicate` true zurückgibt
- "true"-Elemente werden propagiert


### reduce() ⇒ n:1

`T Stream<T>::reduce(T identity, BinaryOperator<T> accumulator)`

```java
teamSchadow.stream()
    .reduce("", (collector, name) -> {
        return String.join(", ", collector, name)
    });

// "Dominik, Sandro, Jochen, ..."
```

- BinaryOperator kombiniert Element mit Ergebnis der vorausgegangenen Operation
- Initialer Wert i. d. R. Identität (s. erster Parameter)
- Rückgabewert ist Element des Typs, kein Stream

Note: String "", + 0, * 1, ...


### collect() ⇒ n:1

`R Stream<T>::collect(Collector<? super T,A,R> collector)`

- Vorgefertigte Kollektoren in java.util.stream.Collectors
- Implementieren FI `Collector<T,A,R>`
- sum, avg, join, toList, ...


### ... und viele mehr

In java.util.stream

- `long count()`
- `Stream<T> sorted(Comparator<? super T> comparator)`
- `Stream<T> limit(long maxSize)`
- ...


### Inhärente Parallelisierbarkeit der Stream-API

`streamParallel()` statt `stream()`, so einfach geht das.


### Inhärente Parallelisierbarkeit!

Die Einfachheit der Anpassung (und damit einhergehend die Folienanzahl) wird der Mächtigkeit kaum gerecht.


### Inhärente Parallelisierbarkeit!!

Daher strecke ich auf mehrere Folien :-)



## Umsetzung - Teil 2

Bibliotheken & Sprachen


### Andere Java-APIs mit FP-API

- `Optional<T>` (Javas halbherziger Versuch NPEs zu umschiffen)
- ... ein paar andere

Note: APIs die FIs entgegennehmen


### [ReactiveX](http://reactivex.io/) (RxJava, RxJS, Rx.NET, RxPY, ...)

> ReactiveX is a combination of the best ideas from the Observer pattern, the Iterator pattern, and functional programming.

- Bekannt, verbreitet und mächtig
- Umfangreicher als java.util.stream
  - join(), merge(), zip()
  - doOnNext(), doOnError(), doOnCompleted()
  - ...
- Dezent komplexer in Möglichkeiten (und Einarbeitung)

Note:
- @Thomas: Am nächsten Entwicklertag Observer-Pattern?
- Im Projekt beim Kunden erfolgreich im Einsatz


### ... und weitere Bibliotheken

- [Functional Java](http://www.functionaljava.org/) (viel Conveniece)
- [Google Guava](https://github.com/google/guava) (auch mit Observer)
- ...


### Andere (JVM-)Sprachen

- [Clojure](http://clojure.org/)
  - Lisp-Dialekt
  - JVM, CLR, JS
- [Scala](http://www.scala-lang.org/)
  - Nicht strikt-funktional, wie Java auch
  - Aber im Vgl. "optimierter" für FP
  - JVM, JS
- ...



## Umsetzung - Teil 3

Eigenen Code funktional zugänglich machen


### Denken in Streams und FIs I

- Fluent API
  - `public void myMethod(param)` -> `public MyClass myMethod(param)`
  - `return this;`
- `return Stream<T>`
  - `return collection.stream();`
  - `return Stream<T>.of(T... values)`


### Denken in Streams und FIs II

- Akzeptiere *higher order functions* als Parameter
- Implementiere FuntionalInterfaces und nutze sie als Parameter
  - `someStream.map(this::myExtractedFunction)``
- Erzeuge Streams
  - Von anderen Streams (Arrays, Datei Input, ...)
  - Implementiere `Supplier<T>`, aus dem sich Stream erzeugen lässt


### Denken in Streams und FIs III

- Ausführliche Beispiele würden Rahmen sprengen
- Fokus im Folgenden auf einigen Entwurfsmustern



## Umsetzung - Teil 4

Funktionale Entwurfsmuster


### Delegates / Dependency Injection

```java
public static int totalAssetValues(final List<Asset> assets,
    final Predicate<Asset> assetSelector) {
    return assets.stream()
        .filter(assetSelector)
        .mapToInt(Asset::getValue)
        .sum();
}

System.out.println("Total of assets: " + totalAssetValues(assets,
    asset -> true));
System.out.println("Total of bonds: " + totalAssetValues(assets,
    asset -> asset.getType() == AssetType.BOND));
System.out.println("Total of stocks: " + totalAssetValues(assets,
    asset -> asset.getType() == AssetType.STOCK));
```

Macht auch testen einfacher (z. B. `asset -> throw new EnumConstantNotPresentException("damn");`)

Note:
- Dependency hier assetSelector
- Siehe auch Punkt oben "FIs als Parameter akzeptieren"


### Decorator


#### Ausgangsklasse

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

Note: Wichtiger Teil: Nimmt **einen** Filter entgegen


#### Decorator für Multi-Filter - klassisch

- Ableitung mit Überladung: `public Color capture(final Color... inputColors)`?
> Composition over Inheritance

- Wrapper-Klasse die Camera-Instanz entgegennimmt?
> Forwarding Boilerplate


#### Decorator für Multi-Filter - FP-Kompositionen

```java
Function<Color, Color> myFilters =
    Filters::brigther.compose(Filters::sepia)
Camera multiFilterCamera = new Camera(myFilters);
```

- Consumer<T>: andThen(...)
- Function<T,R>: andThen(...), compose(...)
- Predicate<T>: and(...), or(...), negate()


### Execute Around Method


#### Klassisch

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


#### Funktional

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


#### Vorteile des "Execute Around Method"-Pattern

- Fluent ⇒ Keine Wiederholung von `mailer.*`
- Gekapselter Scope & Lifetime
  - wäre es im 1. Bsp erlaubt, mailer zu speichern und send() mehrfach aufzurufen?
  - kein Zugriff auf mailer nach send()
- Auch gut zum Resourcen-Management (DB-Verbindung oder Dateien öffnen und schließen, Lock-Verwaltung, Exception-Handling, etc.)
  - Spart Boilerplate-Aufräum-Code
  - Spart Boilerplate try/finally-Blöcke
  - Beugt vergessen dieses Boilerplate-Codes vor

Note: Im FP-Beispiel wird die mailer-Resource von send() gemananaget


### Lazy Evaluation

- Kostspielige Operationen sollten nur ausgeführt werden wenn nötig
- Eventuell entscheidet eine leichtgewichtige Vorbedingung

```java
lightBoolean && heavy1() && heavy2()
```

- Möglich bei boolean-Ausdrücken
- Nicht möglich bei z. B. Funktionsparametern

```java
myMethod(lightBoolean, heavy1(), heavy2());
```


#### 1. Auswertung aufschieben

Lambdas werden "vor Ort" ausgewertet, aber nicht aufgerufen

```java
Supplier<Boolean> lightLambda1 = () -> heavy1();
Supplier<Boolean> lightLambda2 = () -> heavy2();

myMethod(lightBoolean, lightLambda1, lightLambda2);
```


#### 2. Steam<T> ist inhärent faul

```java
List<String> names = Arrays.asList("Brad", "Kate", "Kim",
    "Jack", "Joe", "Mike", "Susan", "George",
    "Robert", "Julia", "Parker", "Benson");

final String firstNameWith3Letters = names.stream()
    .filter(name -> length(name) == 3)
    .map(name -> toUpper(name))
    .findFirst()
    .get();
```

Was wäre Log-Ausgabe der Lambdas?


#### Internmediate VS terminal

```
getting length for "Brad"
getting length for "Kate"
getting length for "Kim"
converting "Kim" to uppercase
KIM
```

- Intermediates (filter(), map(), ...) geben einen Stream zurück, der die Operation "cachet"
- Terminals (findFirst(), reduce(), ...) fragen nur so viele Elemente an, wie benötigt
- Erlaubt effizienten Umgang mit **infiniten** Streams (z. B. `Stream<Integer> pimeNumbers`)

Note: Reduce fragt natürlich ab bis Ende



## ABER


### Probleme mir "reiner" FP in Java

Nicht 100%-ig umgesetzt bzw. überhaupt umsetzbar

- Dinge fehlen
- Mischbetrieb mit OOP
  - Erlaubt Seiteneffekte (Lambdas können z. B. aüßere Variablen zugreifen)
  - Mischbetrieb kann Code noch unübersichtlicher machen als vorher
- FP ist in, daher muss Java es können...

Note: keine sog. "Pure Function"s


### Performance

- Nicht so performant wie "echte" FP-Sprachen
- Lambdas sind implizit anonyme Klassen

Note: Da Java stark auf sein Typsystem ausgerichtet ist.


### Randnotizen

- Effective final (was von außen in Lambda verwendet wird, wird `final`)
- Keine checked Exceptions in Lambdas (FIs haben keine throws-Klausel)
  - `throw new RuntimeException(e)` evtl. problematisch bei paralleler Ausführung
  - Komplexe Thematik, übersteigt diesen Vortrag
  - ReactiveX hat einen Mechanismus dafür



## Fazit

Nichtsdestotrotz sehr hilfreich für den Alltag!

- Lesbarkeit
- Parallelisierbarkeit
- Vorbereitung auf andere Sprachen
- Macht Spaß!

Note:
- Spaß = Einarbeiten in neue Denkweisen
