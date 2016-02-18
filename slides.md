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


### Praxis-geprüft

Beim Kunden im Einsatz, weil ...


### Imperative Programmierung

- Beschreibe **wie** etwas getan werden muss
- Schritt für Schritt Anleitung

Note: Allgemein bekannt - hoffentlich


### Funktionale Programmierung

- Beschreibe **was** getan werden muss
- Beschäftigt sich nicht mit dem **wie**
- Nutzt Funktionen höherer Ordnung

Note: Fkt. höherer Ordnung = Fkt. die Fkt. engegennehmen



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

Note: Annotation erstmal ignorieren


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
executor.doSomeAction(param -> {
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

- FIs haben *eine* (abstrakte) Methode
- Dadurch automatische Typinferenz möglich
- Lambda nur Kurzschreibweise für anonyme Klasse

Note: Die Annotation von eben


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



## Motivation


### Theoretische Vorteile von FP

- Lambda-Kalkül
- Mathematische Beweisbarkeit von Korrektheit

Note: ..., aber wir wollten Pragmatismus


### Überleitung zu FP

> ... und jetzt nochmal für Pragmatiker, bitte!

**Beispiel:** Summiere Preise über 20 EUR, ermäßigt um 10 %


#### Wie in alten Tagen

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


#### NUR Syntaktischer Zucker

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

Note: Erlaubt kompaktere Syntax, Funktions- und vor allem Denkweise ist aber gleich.


### Nachteile imperativer Programmierung

- Vermischt das
  - *was* (Auszuführende Aktion, aka Command / Logik)
  - mit dem *wie* (iterieren, verzweigen, ...)
- Redundanter, manuell erstellter Code (lies: Fehler)
  - Boilerplate-Code (Deklaration von Command-Klassen)
  - Low-Level Code (Verzweigungen, Schleifen, ...)
  - Parallelisierung (Threads starten, Synchronisation, ...)


#### Eine neue Welt

```java
final BigDecimal totalOfDiscountedPrices = MyApi
    .getPrices().stream()
    .filter(price -> price.compareTo(BigDecimal.valueOf(20)) > 0)
    .map(price -> price.multiply(BigDecimal.valueOf(0.9)))
    .reduce(BigDecimal.ZERO, BigDecimal::add);

System.out.println("Result: " + totalOfDiscountedPrices);
```

Note:
- WAS filtern wir, bzw. WAS bilden wir ab = Commands
- Kein WIE (Iteration, Collections manipulieren)!
- WIE unter der Haube gekapselt in Verben (filter, map, reduce)
- WAS auf Einzelelement-Basis definiert


### Praktische Vorteile von FP

- Unveränderbarkeit (Lambda-Scope geschlossen)
- Keine Seiteneffekte (durch Unveränderbarkeit)
- Prägnant, ausdrucksstark, näher an natürlicher Sprache
- Weniger Fehleranfällig (da weniger Code)

⇒ wartbarer

Note:
- geschlossener Scope in Java nicht umgesetzt, siehe später
- NatSpr + intuitiver (gewöhnungsbedürftig, wie OO auch)
- Fehleranfälligkeit: weniger Code, (keine Variablenmuation)



## Umsetzung - Teil 1

Java Boardmittel (aka die Stream-API) & die "Grundverben"


### API

- java.util.stream definiert `Stream<T>`
- java.util.function definiert `FunctionalInterface`s
  - `R Function<T,R>::apply(T t)`
  - `boolean Predicate<T>::test(T t)`
  - `void Consumer<T>::accept(T t)`
  - `T Supplier<T>::get()`
  - ...

Note:
- Am Beispiel der neuen Stream API aus Java 8
- Streams stellen "Grundverben" bereit
- FIs werden von "Grundverben" entgegengenommen
- FIs = Commands


### stream() ⇒ 1:n

`Stream<T> Collection<T>::stream()`

```java
collection.stream()
    .someFpVerb(command);
```

- Erzeugt "Strom" von Elementen
- `someFpVerb(command)` wird für jedes Element ein Mal aufgerufen
- Vgl. Iterator: `iterable.forEach(command)`


### map() ⇒ n:n

`Stream<R> Stream<T>::map(Function<? super T, ? extends R> mapper)`

```java
teamSchadow.stream()
    .map((String name) -> name.toUpperCase());

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
    .map(String::toUpperCase);

// "DOMINIK", ...
```

- Return-Typ von `Predicate` ist boolean
- filter() prüft, ob `Predicate` true zurückgibt
- Nur "true"-Elemente werden propagiert

Note: Fluent Interface


### reduce() ⇒ n:1

`T Stream<T>::reduce(T identity, BinaryOperator<T> accumulator)`

```java
teamSchadow.stream()
    .reduce("", (collector, name) -> {
        return collector + ", " + name;
    });

// "Dominik, Sandro, Jochen, ..."
```

- BinaryOperator kombiniert Element mit Ergebnis der vorausgegangenen Operation
- Initialer Wert i. d. R. Identität (s. erster Parameter)
- Rückgabewert ist Element des Typs, kein Stream!

Note: String "", + 0, * 1, ...


### ... und viele mehr

In java.util.stream

- `long count()`
- `Stream<T> sorted(Comparator<? super T> comparator)`
- `Stream<T> limit(long maxSize)`
- ...


### Inhärente Parallelisierbarkeit der Stream-API

`streamParallel()` statt `stream()`, so einfach geht das.

Note: WIE unter der Haube, Threading ist auch so ein WIE


### Inhärente Parallelisierbarkeit!

Die Einfachheit der Anpassung (und damit einhergehend die Folienanzahl) wird der Mächtigkeit kaum gerecht.


### Inhärente Parallelisierbarkeit!!

Daher strecke ich auf drei Folien :-)



## Umsetzung - Teil 2

Bibliotheken, Frameworks & Sprachen


### Andere Java-APIs und Framkeworks

- `Optional<T>` (Javas halbherziger Versuch NPEs zu umschiffen)
- ... und ein paar andere

- [Functional Java](http://www.functionaljava.org/) (viel Conveniece)
- [Google Guava](https://github.com/google/guava) (mit Observer-Pattern)
- ...

Note: APIs die FIs entgegennehmen


### [ReactiveX](http://reactivex.io/) (RxJava, RxJS, Rx.NET, RxPY, ...)

> ReactiveX is a combination of the best ideas from the Observer pattern, the Iterator pattern, and functional programming.

- Bekannt, verbreitet, mächtig
- Umfangreicher als java.util.stream
  - join(), merge(), zip()
  - doOnNext(), doOnError(), doOnCompleted()
  - ...
- Dezent komplexer in Möglichkeiten (und Einarbeitung)

Note:
- @Thomas: Am nächsten Entwicklertag Observer-Pattern?
- Im Projekt beim Kunden erfolgreich im Einsatz


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


### Denken und programmieren in Streams und FIs

- Fluent API anbieten
- `Stream<T>`s erzeugen / zurückgeben
- *Higher order functions* als Parameter akzeptieren
- `FuntionalInterface`s implementieren (um als Parameter genutzt werden zu können)

Note:
- Ausführliche Beispiele würden Rahmen sprengen
- Fokus im Folgenden auf einigen Entwurfsmustern



## Umsetzung - Teil 4

Funktionale Entwurfsmuster


### Dependency Injection

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

Macht auch testen einfacher (z. B. `asset -> throw new EnumConstantNotPresentException("on purpose");`)

Note:
- Dependency hier assetSelector
- Siehe auch Punkt oben "FIs als Parameter akzeptieren"


### Execute Around Method


#### 1. Klassisch

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


#### 2. Funktional

```java
public class FluentMailer {
private FluentMailer() {}
    public FluentMailer from(final String address) { /*... */; return this; }
    public FluentMailer to(final String address) { /*... */; return this; }
    public FluentMailer subject(final String line) { /*... */; return this; }
    public FluentMailer body(final String message) { /*... */; return this; }
    public static void send(final Consumer<FluentMailer> mailerConsumer) {
        final FluentMailer mailer = new FluentMailer();
        mailerConsumer.accept(mailer);
        mailer.privateValidate();
        mailer.privateSend();
    }
}
```

```java
FluentMailer.send(mailer -> mailer
    .from("build@agiledeveloper.com")
    .to("venkats@agiledeveloper.com")
    .subject("build notification")
    .body("...much better..."));
```

Note:
- Fluent (return this)
- send() aktzeptiert FI
- send() managed mailer


#### Vorteile des "Execute Around Method"-Pattern

- Fluent ⇒ Keine Wiederholung von `mailer.*`
- Gekapselter Scope & Lifetime
  - wäre es in 1. erlaubt, mailer zu speichern und send() mehrfach aufzurufen?
  - kein Zugriff auf mailer nach send() in 2.
- Auch gut zum Resourcen-Management (DB-Verbindung oder Dateien öffnen und schließen, Lock-Verwaltung, Exception-Handling, etc.)
  - Spart Boilerplate-Aufräum-Code
  - Spart Boilerplate try/finally-Blöcke
  - Beugt vergessen dieses Boilerplate-Codes vor


### Lazy Evaluation

- Kostspielige Operationen sollten nur ausgeführt werden wenn nötig
- Eventuell entscheidet eine leichtgewichtige Vorbedingung


#### 1. JVM-funktionsweise ausnutzen

```java
lightBoolean && heavy1() && heavy2()
```

- Möglich bei boolean-Ausdrücken
- Nicht möglich bei z. B. Funktionsparametern

```java
myMethod(lightBoolean, heavy1(), heavy2());
```


#### 1. Auswertung in Lambdas kapseln

Lambdas werden "vor Ort" ausgewertet, aber nicht aufgerufen

```java
Supplier<Boolean> lightLambda1 = () -> heavy1();
Supplier<Boolean> lightLambda2 = () -> heavy2();

myMethod(lightBoolean, lightLambda1, lightLambda2);
```


#### 2. Stream<T> ist inhärent faul

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


#### 2. Intermediate VS terminal

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

Note:
- Reduce fragt natürlich ab bis Ende
- Terminals sind idR die, die keinen Stream<T> zurückgeben, sondern T



## ABER


### Probleme mit "reiner" FP in Java

Nicht 100%-ig umgesetzt bzw. überhaupt umsetzbar

- Dinge fehlen
- Lambda-Scope nicht geschlossen
- Mischbetrieb mit OOP möglich
- ...

⇒

- Erlaubt Seiteneffekte (Lambdas können z. B. aüßere Variablen zugreifen)
- Kann Code unübersichtlicher machen als vorher

Aber FP ist in, daher muss Java es können...


### Performance in Java

- Nicht so performant wie "echte" FP-Sprachen
- Lambdas sind implizit anonyme Klassen

Note: Da Java stark auf sein Typsystem ausgerichtet ist.



## Fazit

Nichtsdestotrotz sehr hilfreich für den Alltag!

- Denkweise (subjektiv) eingängig und angenehm
- Lesbarkeit
- Parallelisierbarkeit
- Vorbereitung auf andere Sprachen
- Macht Spaß!

Note:
- Spaß = Einarbeiten in neue Denkweisen
