# ***PARTE 3***

## **Procesamiento de Streams (Flujos)**

    Este capítulo cubre
    * Lambdas en pocas palabras
    * Dónde y cómo usar lambdas
    * El patrón execute-around
    * Interfaces funcionales, inferencia de tipos
    * Referencias a métodos
    * Composición de lambdas

    En el capítulo anterior, viste que pasar código con parametrización de comportamiento es útil 
    para hacer frente a cambios frecuentes en los requisitos de tu código. Te permite definir un 
    bloque de código que representa un comportamiento y luego pasarlo como argumento. Puedes decidir 
    ejecutar ese bloque cuando ocurra un evento determinado (por ejemplo, un clic en un botón) o en 
    puntos específicos de un algoritmo (por ejemplo, un predicado como “solo manzanas con más de 
    150 g” en un algoritmo de filtrado o una operación de comparación personalizada en una ordenación).
    En general, con este concepto puedes escribir código más flexible y reutilizable.

    Pero viste que usar clases anónimas para representar diferentes comportamientos no es satisfactorio:
    es verboso y no incentiva a los programadores a usar la parametrización de comportamiento en la 
    práctica. En este capítulo, aprenderás sobre una nueva característica de Java 8 que resuelve este 
    problema: las expresiones lambda. Estas te permiten representar un comportamiento o pasar código 
    de forma concisa. Por ahora, puedes pensar en las expresiones lambda como funciones anónimas, 
    métodos sin nombre, que también se pueden pasar como argumentos a un método, al igual que con una
    clase anónima.

    Veremos cómo construirlas, dónde usarlas y cómo hacer tu código más conciso con ellas. También 
    explicaremos nuevas características como la inferencia de tipos y las nuevas interfaces importantes
    disponibles en la API de Java 8. Finalmente, introduciremos las referencias a métodos, una 
    característica útil que complementa perfectamente a las expresiones lambda.

    Este capítulo está organizado para enseñarte paso a paso cómo escribir código más conciso y 
    flexible. Al final, reuniremos todos los conceptos en un ejemplo concreto: tomaremos el ejemplo 
    de ordenación del capítulo 2 y lo mejoraremos gradualmente usando expresiones lambda y referencias
    a métodos, para hacerlo más claro y legible. Este capítulo es importante por sí mismo y porque 
    usarás las lambdas extensamente a lo largo del libro.

    LAMNDAS EN POCAS PALABRAS
    Una expresión lambda puede entenderse como una representación concisa de una función anónima que
    puede pasarse como argumento. No tiene un nombre, pero sí una lista de parámetros, un cuerpo, un 
    tipo de retorno y, posiblemente, una lista de excepciones que pueden lanzarse. Desglosemos este 
    concepto:

    * Anónima: No tiene un nombre explícito como un método tradicional, lo que reduce la cantidad de
        código y simplifica su uso.
    * Función: Aunque no está asociada a una clase como un método, tiene parámetros, un cuerpo, un 
        tipo de retorno y puede lanzar excepciones, al igual que un método.
    * Pasa como argumento: Una expresión lambda puede pasarse a un método o almacenarse en una variable.
    * Concisa: Elimina la verbosidad de las clases anónimas, permitiendo escribir código más limpio 
        y directo.

    El término lambda proviene del cálculo lambda, un sistema matemático desarrollado en los años 30
    para describir la computación, donde todas las funciones son anónimas.

    ¿Por qué son importantes? Antes de Java 8, pasar comportamiento requería clases anónimas, lo que
    resultaba engorroso. Las expresiones lambda resuelven este problema al permitir pasar código de 
    forma clara y breve. Aunque no añaden funcionalidad nueva al lenguaje, sí fomentan el uso de la 
    parametrización de comportamiento, haciendo el código más legible y flexible. Por ejemplo, con 
    una lambda puedes crear un Comparator personalizado de forma mucho más simple.

    Antes:
```java
Comparator<Apple> byWeight = new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2){
        return a1.getWeight().compareTo(a2.getWeight());
    }
};
```

    Despues (con expresion Lamnda):
```java
Comparator<Apple> byWeight =
    (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```
    Debes admitir que el código se ve más claro. No te preocupes si aún no entiendes todas las partes
    de la expresión lambda; explicaremos cada componente en breve. Por ahora, observa que estás pasando
    literalmente solo el código necesario para comparar dos manzanas por su peso, como si estuvieras 
    pasando el cuerpo del método compare. Pronto aprenderás a simplificar aún más tu código. En la 
    siguiente sección explicaremos exactamente dónde y cómo puedes usar expresiones lambda.

    La expresión lambda que acabamos de mostrar tiene tres partes:
    * Parametros Lamnda --> (apple a1, apple a2)
    * Cuerpo Lamnda --> (a1.getWeight().compareTo(a2.getWeight())

    * Una lista de parámetros: En este caso, coincide con los parámetros del método compare de un 
        Comparator: dos objetos Apple.
    * Una flecha: La flecha -> separa la lista de parámetros del cuerpo de la lambda.
    * El cuerpo de la lambda: Compara dos manzanas usando sus pesos. La expresión se considera el valor
        devuelto por la lambda.

```java
(String s) -> s.lenght() //Toma un parámetro de tipo String y devuelve un int.No tiene una sentencia
                         // return porque el valor de retorno se infiere automáticamente cuando el 
                         // cuerpo de la lambda es una única expresión.
(Apple a) -> a.getWeight() > 150 // Toma un parámetro de tipo Apple y devuelve un valor booleano 
                                 // (si la manzana pesa más de 150 g).
(int x, int y) -> {
    system.out.println("Result: ");  // Toma 2 parametros de enteros devuelve ningun valor
    system.out.println(x + y);
}
() -> 42  // no hay parametros que evaluar retorna el entero 42

```

    Esta sintaxis fue elegida por los diseñadores del lenguaje Java porque fue bien recibida en otros
    lenguajes, como C# y Scala. JavaScript tiene una sintaxis similar. La sintaxis básica de una 
    lambda es la siguiente (conocida como lambda de estilo expresión):

    Parámetros → Especificados entre paréntesis, por ejemplo (x, y).
    Flecha lambda (->) → Separa los parámetros del cuerpo.
    Cuerpo → Contiene la expresión o bloque de instrucciones a ejecutar.

    Cuestionario: Sintaxis de lambda
    Basado en las reglas de sintaxis mostradas, ¿cuáles de las siguientes expresiones no son válidas?

    1. () -> {}
    2. () -> "Raoul"
    3. () -> { return "Mario"; }
    4. (Integer i) -> return "Alan" + i;
    5. (String s) -> { "Iron Man"; }
    Respuesta:
    Las expresiones 4 y 5 son inválidas; las demás son válidas. Detalles:

    1. Válida: Lambda sin parámetros y sin retorno (void), equivalente a un método vacío.
    2. Válida: Sin parámetros, devuelve una cadena como expresión.
    3. Válida: Sin parámetros, devuelve una cadena usando return dentro de un bloque.
    4. Inválida: return es una sentencia de control y requiere llaves. Debe escribirse como:
        (Integer i) -> { return "Alan" + i; };
    5. Inválida: "Iron Man" es una expresión, no una sentencia. Para que sea válida, debe eliminarse 
        el bloque:
            (String s) -> "Iron Man" o usar return dentro de llaves:
        (String s) -> { return "Iron Man"; }

    Dónde y cómo usar lambdas
    Quizás ahora te estés preguntando dónde se pueden usar expresiones lambda. En el ejemplo anterior,
    asignaste una lambda a una variable de tipo Comparator<Apple>. También podrías usar otra lambda 
    con el método filter que implementaste en el capítulo anterior:
```java
List<Apple> greenApples =
    filter(inventory, (Apple a) -> GREEN.equals(a.getColor()));
```
    ¿Dónde exactamente puedes usar lambdas? Puedes usar una expresión lambda en el contexto de una 
    interfaz funcional. En el código mostrado aquí, puedes pasar una lambda como segundo argumento 
    al método filter porque espera un objeto de tipo Predicate<T>, que es una interfaz funcional. 
    No te preocupes si esto suena abstracto; ahora explicaremos en detalle qué significa esto y qué 
    es una interfaz funcional.

    Interfaz funcional
    Recuerda la interfaz Predicate<T> que creaste en el capítulo 2 para parametrizar el comportamiento
    del método filter? ¡Es una interfaz funcional! ¿Por qué? Porque Predicate especifica un solo 
    método abstracto: boolean test(T t).
```java
public interface Predicate<T> {
    boolean test (T t);
}
```
    En pocas palabras, una interfaz funcional es una interfaz que declara exactamente un solo método
    abstracto. Ya conoces varias interfaces funcionales en la API de Java, como Comparator y Runnable,
    que vimos en el capítulo 2.
```java
public interface Comparator<T> { //java.util.Comparator
    int compare(T o1, T o2);
}
public interface Runnable {  //java.lang.Runnable
    void run();
}
public interface ActionListener extends EventListener {   //java.awt.event.ActionListener
    void actionPerformed(ActionEvent e);
}
public interface Callable<V> {  //java.util.concurrent.Callable
    V call() throws Exception;
}
public interface PrivilegedAction<T> {  //java.security.PrivilegedAction
    T run();
}
```

    NOTA: Verás que las interfaces ahora también pueden tener métodos por defecto (un método con 
    cuerpo que proporciona una implementación predeterminada en caso de que no sea implementado por 
    una clase). Una interfaz sigue siendo una interfaz funcional si tiene varios métodos por defecto, 
    siempre que especifique un solo método abstracto.
    Para comprobar tu comprensión, el siguiente cuestionario  debería ayudarte a determinar si has 
    comprendido el concepto de interfaz funcional.

    Cuestionario: Interfaz funcional
    ¿Cuáles de estas interfaces son interfaces funcionales?
```java
public interface Adder {
    int add(int a, int b);
}

public interface SmartAdder extends Adder {
    int add(double a, double b);
}

public interface Nothing {
}
```
    Respuesta:
    * Solo Adder es una interfaz funcional.
    * SmartAdder no lo es porque declara dos métodos abstractos (hereda add(int, int) de Adder y define 
        add(double, double)).
    * Nothing no es una interfaz funcional porque no declara ningún método abstracto.

    ¿Qué puedes hacer con interfaces funcionales? Las expresiones lambda te permiten proporcionar 
    directamente la implementación del método abstracto de una interfaz funcional en línea, y tratar 
    toda la expresión como una instancia de dicha interfaz funcional (más técnicamente, una instancia
    de una implementación concreta de la interfaz funcional). Puedes lograr lo mismo con una clase 
    interna anónima, aunque es más engorrosa: proporcionas una implementación y la instancias 
    directamente en línea. El siguiente código es válido porque Runnable es una interfaz funcional que
    define un único método abstracto, run:
```java
Runnable r1 = () -> System.out.println("Hello World 1"); //Uso de una lambda
Runnable r2 = new Runnable() {      //Uso de una clase anonima
    public void run() {
        System.out.println("Hello World 2");
    }
};
public static void process(Runnable r) {
    r.run();
}
process(r1);    //imprime "Hello World 1"
process(r2);    //imprime "Hello World 2"
process(() -> System.out.println("Hello World 3")); //imprime "Hello World 3"
```

    Descripcion Funcional
    La firma del método abstracto de la interfaz funcional describe la firma de la expresión lambda.
    A este método abstracto lo llamamos descriptor de función. Por ejemplo, la interfaz Runnable puede
    verse como la firma de una función que no acepta ningún parámetro y no devuelve nada (void), porque
    tiene un único método abstracto llamado run, que no acepta parámetros y no devuelve nada (void).

    Utilizamos una notación especial a lo largo de este capítulo para describir las firmas de las 
    expresiones lambda y de las interfaces funcionales. La notación () -> void representa una función 
    con una lista de parámetros vacía que devuelve void. Esto es exactamente lo que representa la 
    interfaz Runnable. Como otro ejemplo, (Apple, Apple) -> int denota una función que toma dos objetos 
    Apple como parámetros y devuelve un entero (int). Proporcionaremos más información sobre los 
    descriptores de función más adelante en el capítulo.

    Quizás ya te estés preguntando cómo se verifica el tipo de las expresiones lambda. Explicamos con 
    detalle cómo el compilador comprueba si una lambda es válida en un contexto determinado. Por 
    ahora, basta con entender que una expresión lambda se puede asignar a una variable o pasar a un 
    método que espera una interfaz funcional como argumento, siempre que la expresión lambda tenga la 
    misma firma que el método abstracto de la interfaz funcional. Por ejemplo, en nuestro ejemplo 
    anterior, podrías pasar una lambda directamente al método process de la siguiente manera:
```java
public void process(Runnable r) {
    r.run();
}
process(() -> System.out.println("This is awesome!!"));
```
    Este código, al ejecutarse, imprimirá “This is awesome!!”. La expresión lambda () -> System.out.println("This is awesome!!") 
    no toma parámetros y devuelve void. Esta es exactamente la firma del método run definido en la 
    interfaz Runnable.

    Lambdas y llamada a métodos void
    Aunque pueda parecer extraño, la siguiente expresión lambda es válida:
    process(() -> System.out.println("This is awesome"));
    Después de todo, System.out.println devuelve void, por lo que claramente no es una expresión. 
    ¿Por qué no es necesario encerrar el cuerpo entre llaves así?
    process(() -> { System.out.println("This is awesome"); });
    Resulta que existe una regla especial en la Especificación del Lenguaje Java: no es necesario 
    encerrar una única llamada a un método que devuelve void entre llaves. Esta simplificación 
    sintáctica permite una escritura más concisa cuando la lambda consiste únicamente en una invocación
    de método void.

    ¿Por qué solo se pueden pasar lambdas donde se espera una interfaz funcional?
    Los diseñadores del lenguaje Java optaron por vincular las expresiones lambda a interfaces 
    funcionales —es decir, interfaces con un solo método abstracto— porque esta solución encaja de forma
    natural en el modelo existente sin añadir complejidad. No fue necesario introducir nuevos tipos 
    de funciones en el lenguaje.

    Esta decisión se basó en varios factores clave:
    Compatibilidad y evolución: Interfaces como Runnable, Comparator o Callable ya eran ampliamente 
    usadas antes de Java 8 y cumplen con el patrón de un solo método abstracto. Al aprovecharlas, 
    las lambdas pudieron integrarse sin modificar APIs existentes.
    Familiaridad: Muchos programadores ya estaban acostumbrados a usar clases anónimas con interfaces 
    de un solo método, especialmente en manejadores de eventos.
    Migración suave: Al no requerir cambios en el código base, se facilitó la adopción de lambdas en 
    proyectos antiguos.
    Simplicidad del modelo: En lugar de añadir nuevos tipos funcionales, se reutilizó un mecanismo ya 
    conocido, manteniendo la coherencia del lenguaje.
    La anotación @FunctionalInterface ayuda a garantizar que una interfaz mantenga esta propiedad, 
    generando un error de compilación si se añaden más métodos abstractos.
    
    Quiz: ¿Dónde se pueden usar lambdas?
    ¿Cuáles de los siguientes son usos válidos de expresiones lambda?
    1 execute(() -> {});
        public void execute(Runnable r) {
            r.run();
        }
    2 public Callable<String> fetch() {
        return () -> "Tricky example ;-)";
    }
    3 Predicate<Apple> p = (Apple a) -> a.getWeight();

    Respuesta:
    Solo 1 y 2 son válidos.
    El primer ejemplo es válido porque la lambda () -> {} tiene la firma () -> void, que coincide con
    el método abstracto run definido en Runnable. Aunque el cuerpo de la lambda está vacío, es 
    sintácticamente correcto y no genera errores.
    El segundo ejemplo también es válido. El tipo de retorno del método fetch es Callable<String>, 
    cuyo método call() tiene la firma () -> String. La lambda () -> "Tricky example ;-)" coincide 
    exactamente con esta firma, por lo que es compatible.
    El tercer ejemplo es inválido porque la expresión lambda (Apple a) -> a.getWeight() tiene la firma
    (Apple) -> Integer, mientras que el método test de Predicate<Apple> espera una firma (Apple) -> boolean.
    Como getWeight() devuelve un entero y no un valor booleano, no se cumple la firma requerida.

    ¿Qué hay sobre @FunctionalInterface?
    La anotación @FunctionalInterface se utiliza para indicar que una interfaz está diseñada para ser
    una interfaz funcional, es decir, que debe tener exactamente un método abstracto. Aunque no es 
    obligatoria, es una buena práctica usarla porque:

    Documentación clara: Indica a otros desarrolladores que la interfaz está destinada a usarse con 
    expresiones lambda o referencias a métodos.
    Verificación en tiempo de compilación: Si se añade más de un método abstracto por error, el 
    compilador genera un error, como: "Multiple non-overriding abstract methods found", ayudando a 
    mantener la integridad de la interfaz.
    Una interfaz funcional puede tener métodos default, static y métodos abstractos heredados de Object
    (como equals()), sin que afecten su estatus funcional.

    Se puede comparar con la anotación @Override: no es obligatoria, pero ayuda a prevenir errores y 
    mejora la claridad del código.

    Poniendo en práctica las lambdas: el patrón execute-around
    Veamos un ejemplo de cómo las expresiones lambda, junto con la parametrización del comportamiento,
    pueden usarse en la práctica para hacer que el código sea más flexible y conciso. Un patrón común 
    al procesar recursos (por ejemplo, archivos o bases de datos) consiste en abrir un recurso, 
    realizar un procesamiento sobre él y luego cerrarlo. Las fases de configuración y limpieza siempre
    son similares y rodean el código importante que realiza el procesamiento. Esto se conoce como el 
    patrón execute-around.

    Por ejemplo, en el siguiente código, las líneas resaltadas muestran el código repetitivo necesario
    para leer una línea de un archivo (obsérvese también que se usa la sentencia try-with-resources 
    de Java 7, que ya simplifica el código, ya que no es necesario cerrar el recurso explícitamente):
```java
public String processFile() throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
        return br.readLine();   //Esta es la línea que realiza un trabajo útil.
    }
}
```
    Paso 1: Recuerda la parametrización del comportamiento
    Este código actual es limitado. Solo puedes leer la primera línea del archivo. ¿Qué tal si 
    quisieras devolver las dos primeras líneas, o incluso la palabra más frecuente? Idealmente, te 
    gustaría reutilizar el código que realiza la configuración y limpieza, y decirle al método 
    processFile que realice diferentes acciones sobre el archivo. ¿Te suena familiar? Sí, necesitas 
    parametrizar el comportamiento de processFile. Necesitas una forma de pasar un comportamiento a 
    processFile para que pueda ejecutar diferentes acciones usando un BufferedReader.

    Pasar comportamientos es exactamente para lo que sirven las expresiones lambda. ¿Cómo debería 
    verse el nuevo método processFile si quisieras leer dos líneas a la vez? Necesitas una lambda 
    que tome un BufferedReader y devuelva un String. Por ejemplo, así es como imprimirías dos líneas
    de un BufferedReader:
```java
String result = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```
    Paso 2: Usa una interfaz funcional para pasar comportamientos
    Explicamos anteriormente que las expresiones lambda solo se pueden usar en el contexto de una 
    interfaz funcional. Necesitas crear una que coincida con la firma BufferedReader -> String y que 
    pueda lanzar una IOException. Llamemos a esta interfaz BufferedReaderProcessor:
```java
@FunctionalInterface
public interface BufferedReaderProcessor {
    String process(BufferedReader b) throws IOException;
}
```
    Ahora puedes usar esta interfaz como argumento para tu nuevo método processFile:
```java
public String processFile(BufferedReaderProcessor p) throws IOException {
    …
}
```
    Paso 3: ¡Ejecuta un comportamiento!
    Cualquier expresión lambda del tipo BufferedReader -> String puede pasarse como argumento, ya 
    que coincide con la firma del método definido en la interfaz BufferedReaderProcessor. Ahora solo
    necesitas una forma de ejecutar el código representado por la lambda dentro del cuerpo de 
    processFile. Recuerda, las expresiones lambda te permiten proporcionar directamente la 
    implementación del método abstracto de una interfaz funcional, y tratan toda la expresión como 
    una instancia de dicha interfaz. Por lo tanto, puedes llamar al método process sobre el objeto 
    BufferedReaderProcessor dentro del cuerpo de processFile para realizar el procesamiento:
```java
public String processFile(BufferedReaderProcessor p) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
        return p.process(br);  //Procesa el objeto BufferedReader
    }
}
```
    Paso 4: Pasa lambdas
    Ahora puedes reutilizar el método processFile y procesar archivos de diferentes formas pasando 
    diferentes lambdas.
    A continuación, se muestra cómo procesar una línea:
```java
String oneLine = processFile((BufferedReader br) -> br.readLine());
```
    A continuación se muestra cómo procesar dos líneas:
```java
String twoLines = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```
    Resume los cuatro pasos realizados para hacer el método processFile más flexible.
    Hemos mostrado cómo puedes utilizar interfaces funcionales para pasar expresiones lambda. Pero 
    tuviste que definir tus propias interfaces. En la siguiente sección, exploramos las nuevas 
    interfaces que se agregaron en Java 8 que puedes reutilizar para pasar múltiples lambdas 
    diferentes.

    Usando Interfaces Funcionales:
    1.
```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

public String processFile() throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))){
        return br.readLine();
    }
}
```
    2.
```java
import java.io.BufferedReader;
import java.io.IOException;

public interface BufferedReaderProcesor {
    String process(BufferedReader b) throws IOException;
}
public String processFile(BufferedReaderProcesor p) throws IOException{
    ---
}
```
    3.
```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

public String processFile(BufferedReaderProcessor p) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))){
        return p.process(br);
    }
}
```
    4.

```java
import java.io.BufferedReader;

String Online = processFile((BufferedReader br) -> br.readLine());
String twoLines = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```
