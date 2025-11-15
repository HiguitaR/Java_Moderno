# ***PARTE 1***


## **Procesamiento de Streams (Flujos)**

    El primer concepto de programación es el procesamiento de streams. Para propósitos introductorios, 
    un stream (flujo) es una secuencia de elementos de datos que conceptualmente se producen uno a la 
    vez. Un programa podría leer elementos de un stream de entrada uno por uno y, de manera similar, 
    escribir elementos en un stream de salida. El stream de salida de un programa bien podría ser el 
    stream de entrada de otro.
    Un ejemplo práctico se encuentra en Unix o Linux, donde muchos programas operan leyendo datos de la
    entrada estándar (stdin en Unix y C, System.in en Java), procesándolos y luego escribiendo sus 
    resultados en la salida estándar (stdout en Unix y C, System.out en Java). Primero, un poco de 
    contexto: el comando Unix cat crea un stream concatenando dos archivos, tr traduce los caracteres 
    en un stream, sort ordena las líneas en un stream, y tail -3 da las últimas tres líneas en un 
    stream. La línea de comandos de Unix permite que tales programas se conecten entre sí con tuberías 
    (pipes, |), dando ejemplos como:

```text
cat file1 file2 | tr "[A-Z]" "[a-z]" | sort | tail -3
```

    El cual (suponiendo que file1 y file2 contienen una sola palabra por línea) imprime las tres 
    palabras de los archivos que aparecen últimas en orden alfabético, después de haberlas traducido 
    primero a minúsculas.
    
    Decimos que sort toma un stream de líneas$^3$ como entrada y produce otro stream de líneas como 
    salida (este último ordenado), como se ilustra en la figura 1.2.
    
    Nótese que en Unix, estos comandos (cat, tr, sort y tail) se ejecutan concurrentemente, de modo 
    que sort puede estar procesando las primeras líneas antes de que cat o tr hayan terminado.
    
    Una analogía más mecánica es la de una cadena de montaje de automóviles donde un flujo de coches se 
    pone en cola entre estaciones de procesamiento, y cada una toma un coche, lo modifica y lo pasa a 
    la siguiente estación para un procesamiento posterior. El procesamiento en estaciones separadas es 
    típicamente concurrente, aunque la cadena de montaje sea físicamente una secuencia.
    
    Java 8 añade una API de Streams (nótese la 'S' mayúscula) en java.util.stream basándose en esta 
    idea; un Stream<T> es una secuencia de elementos de tipo T. Por ahora, se puede considerar como un 
    iterador avanzado. La API de Streams tiene muchos métodos que se pueden encadenar para formar un 
    pipeline complejo, tal como se encadenaban los comandos de Unix en el ejemplo anterior.
    
    La motivación clave para esto es que ahora puedes programar en Java 8 con un nivel de abstracción 
    más alto, estructurando tus pensamientos en cómo convertir un stream de esto en un stream de aquello
    (similar a cómo piensas al escribir consultas a bases de datos), en lugar de procesar un elemento a
    la vez.
        
    Otra ventaja es que Java 8 puede ejecutar de manera transparente tu pipeline de operaciones de 
    Stream en varios núcleos de CPU sobre partes disjuntas de la entrada: esto es paralelismo casi 
    gratuito en lugar del arduo trabajo que implica usar Threads (hilos). Cubriremos la API de Streams 
    de Java 8 en detalle en los capítulos 4 a 7.

## **Pasar código a métodos con behavior parameterization (Parametrización de Comportamiento)**

    El segundo concepto de programación añadido a Java 8 es la capacidad de pasar un fragmento de 
    código a una API. Esto suena terriblemente abstracto.

    En el ejemplo de Unix, podrías querer indicarle al comando sort que utilice un ordenamiento 
    personalizado. Aunque el comando sort admite parámetros de línea de comandos para realizar 
    varios tipos de ordenamiento predefinidos, como el orden inverso, estos son limitados.

    Por ejemplo, digamos que tienes una colección de IDs de factura con un formato similar a 
    2013UK0001, 2014US0002, y así sucesivamente. Los primeros cuatro dígitos representan el año, 
    las siguientes dos letras un código de país, y los últimos cuatro dígitos el ID de un cliente.
    Podrías querer ordenar estos IDs de factura por año, o quizás usando el ID del cliente, o 
    incluso el código del país.

    Lo que realmente se desea es la capacidad de indicarle al comando sort que tome como argumento 
    un ordenamiento definido por el usuario: un fragmento de código separado pasado al comando sort.

    Ahora, como paralelo directo en Java, quieres indicarle a un método sort que compare usando un 
    orden personalizado. Podrías escribir un método compareUsingCustomerId para comparar dos IDs de
    factura, pero, antes de Java 8, ¡no podías pasar este método a otro método!

    Podrías crear un objeto Comparator para pasarlo al método sort, como mostramos al comienzo de 
    este capítulo, pero esto es verboso y oscurece la idea de simplemente reutilizar una pieza de 
    comportamiento existente.

    Java 8 añade la capacidad de pasar métodos (tu código) como argumentos a otros métodos. 
    La Figura 1.3, basada en la figura 1.2, ilustra esta idea. También nos referimos a esto 
    conceptualmente como parametrización de comportamiento (behavior parameterization). 
    ¿Por qué es esto importante? La API de Streams se basa en la idea de pasar código para 
    parametrizar el comportamiento de sus operaciones, justo como pasaste compareUsingCustomerId 
    para parametrizar el comportamiento de sort.

```java
public int compareUsingCustomerId(String inv1, String inv2){}
```

    Resumimos cómo funciona esto en la sección 1.3 de este capítulo, pero dejamos los detalles 
    completos para los capítulos 2 y 3. Los capítulos 18 y 19 analizan cosas más avanzadas que 
    puedes hacer utilizando esta característica, con técnicas provenientes de la comunidad de la 
    programación funcional.

## **Paralelismo y datos mutables compartidos**

    El tercer concepto de programación es bastante más implícito y surge de la frase "paralelismo 
    casi gratis" en nuestra discusión anterior sobre el procesamiento de flujos (streams). ¿A qué 
    tienes que renunciar? Puede que tengas que hacer algunos pequeños cambios en la forma en que 
    codificas el comportamiento que pasas a los métodos de los flujos. Al principio, estos cambios 
    pueden parecer un poco incómodos, pero una vez que te acostumbres a ellos, te encantarán. Debes
    proporcionar un comportamiento que sea seguro para ejecutar de forma concurrente en diferentes 
    partes de la entrada. Normalmente, esto significa escribir código que no accede a datos mutables
    compartidos para hacer su trabajo. A veces, a estas se les denomina funciones puras o funciones 
    sin efectos secundarios o funciones sin estado, y las discutiremos en detalle en los capítulos 
    18 y 18 y 19. El paralelismo anterior surge solo al suponer que múltiples copias de tu fragmento
    de código pueden funcionar de forma independiente. Si hay una variable u objeto compartido en el
    que se escribe, entonces las cosas ya no funcionan. ¿Qué pasa si dos procesos quieren modificar 
    la variable compartida al mismo tiempo? (La Sección 1.4 ofrece una explicación más detallada con
    un diagrama). Encontrarás más sobre este estilo a lo largo del libro.
    Los flujos de Java 8 explotan el paralelismo más fácilmente que la API de hilos (Threads) 
    existente de Java, por lo que, aunque es possible usar synchronized para romper la regla de no 
    tener datos mutables compartidos, es ir en contra del sistema, ya que abusa de una abstracción 
    optimizada en torno a esa regla. Usar synchronized en múltiples núcleos de procesamiento suele 
    ser mucho más costoso de lo que esperas, porque la sincronización obliga a que el código se 
    ejecute secuencialmente, lo que va en contra del objetivo del paralelismo.
    Dos de estos puntos (no tener datos mutables compartidos y la capacidad de pasar métodos y 
    funciones —código— a otros métodos) son las piedras angulares de lo que generalmente se describe
    como el paradigma de la programación funcional, que verás en detalle en los capítulos 18 y 19. 
    En contraste, en el paradigma de la programación imperativa, normalmente describes un programa 
    en términos de una secuencia de sentencias que mutan el estado. El requisito de no tener datos 
    mutables compartidos significa que un método se describe perfectamente solo por la forma en que 
    transforma los argumentos en resultados; en otras palabras, se comporta como una función 
    matemática y no tiene efectos secundarios (visibles).

## **Java necesita evolucionar**

    Ya has visto la evolución en Java antes. Por ejemplo, la introducción de genéricos y el uso de 
    List<String> en lugar de solo List pudo haber sido inicialmente irritante. Pero ahora estás 
    familiarizado con este estilo y los beneficios que aporta (detectar más errores en tiempo de 
    compilación y hacer el código más fácil de leer, porque ahora sabes de qué es una lista).
    
    Otros cambios han facilitado la expresión de cosas comunes (por ejemplo, usar un bucle for-each
    en lugar de exponer el código repetitivo del uso de un Iterator). Los principales cambios en 
    Java 8 reflejan un movimiento que se aleja de la orientación a objetos clásica, que a menudo se 
    centra en mutar valores existentes, y se dirige hacia el espectro de la programación de estilo 
    funcional, en la que lo que quieres hacer en términos generales (por ejemplo, crear un valor que
    represente todas las rutas de transporte de A a B por menos de un precio dado) se considera 
    primordial y se separa de cómo puedes lograrlo (por ejemplo, escanear una estructura de datos 
    modificando ciertos componentes). Ten en cuenta que la programación orientada a objetos clásica
    y la programación funcional, como extremos, podrían parecer estar en conflicto. Pero la idea es 
    obtener lo mejor de ambos paradigmas de programación, para que tengas una mejor oportunidad de 
    tener la herramienta adecuada para el trabajo. Discutiremos esto en detalle en las secciones 
    1.3 y 1.4.
    
    Una frase clave podría ser esta: los lenguajes necesitan evolucionar para seguir el ritmo de los
    cambios en el hardware o en las expectativas de los programadores (si necesitas convencerte, 
    considera que COBOL fue en su día uno de los lenguajes más importantes comercialmente). Para 
    perdurar, Java tiene que evolucionar añadiendo nuevas características. Esta evolución no tendrá 
    sentido a menos que se utilicen las nuevas características, por lo que al usar Java 8 estás 
    protegiendo tu forma de vida como programador Java.
    
    Además de eso, tenemos la sensación de que te encantará usar las nuevas características de Java 
    8. ¡Pregúntale a cualquiera que haya usado Java 8 si está dispuesto a volver atrás! Además, las 
    nuevas características de Java 8 12 CAPÍTULO 1 Java 8, 9, 10 y 11: ¿Qué está pasando? podrían, en
    la analogía del ecosistema, permitir que Java conquiste territorio de tareas de programación 
    actualmente ocupado por otros lenguajes, por lo que los programadores de Java 8 estarán aún más 
    demandados.
    Ahora presentaremos los nuevos conceptos en Java 8, uno por uno, señalando los capítulos que 
    cubren estos conceptos con más detalle.

## **Funciones en Java**

    La palabra función en los lenguajes de programación se usa comúnmente como sinónimo de método, 
    particularmente de un método estático; esto se suma a su uso para función matemática, aquella 
    sin efectos secundarios. Afortunadamente, como verás, cuando Java 8 se refiere a funciones, 
    estos usos casi coinciden.
    
    Java 8 añade funciones como nuevas formas de valor. Esto facilita el uso de streams (flujos), 
    cubiertos en la sección 1.4, que Java 8 proporciona para explotar la programación paralela en 
    procesadores multinúcleo. Comenzaremos mostrando que las funciones como valores son útiles por 
    sí mismas.
    
    Piensa en los posibles valores que manipulan los programas Java. En primer lugar, hay valores 
    primitivos como 42 (de tipo int) y 3.14 (de tipo double). En segundo lugar, los valores pueden 
    ser objetos (más estrictamente, referencias a objetos). La única forma de obtener uno de estos 
    es usando new, quizás a través de un método factory o una función de biblioteca; las referencias
    a objetos apuntan a instancias de una clase. Los ejemplos incluyen "abc" (de tipo String), new 
    Integer(1111) (de tipo Integer) y el resultado new HashMap<Integer, String>(100) de llamar 
    explícitamente a un constructor para HashMap. Incluso los arrays (arreglos) son objetos. ¿Cuál 
    es el problema?
    
    Para ayudar a responder esto, notaremos que el objetivo principal de un lenguaje de programación
    es manipular valores, que, siguiendo la tradición histórica de los lenguajes de programación, se
    denominan por lo tanto valores de primera clase (o ciudadanos, en la terminología tomada del 
    movimiento de derechos civiles de los años 60 en Estados Unidos). Otras estructuras en nuestros 
    lenguajes de programación, que tal vez nos ayuden a expresar la estructura de los valores pero 
    que no pueden pasarse durante la ejecución del programa, son ciudadanos de segunda clase. Los 
    valores enumerados anteriormente son ciudadanos Java de primera clase, pero otros conceptos de 
    Java, como métodos y clases, ejemplifican los ciudadanos de segunda clase. Los métodos están 
    bien cuando se usan para definir clases, que a su vez pueden ser instanciadas para producir 
    valores, pero ni los métodos ni las clases son valores en sí mismos. ¿Importa esto? Sí, resulta 
    que poder pasar métodos durante el tiempo de ejecución, y por lo tanto convertirlos en 
    ciudadanos de primera clase, es útil en la programación, por lo que los diseñadores de Java 8 
    agregaron la capacidad de expresar esto directamente en Java. Por cierto, podrías preguntarte si
    convertir a otros ciudadanos de segunda clase, como las clases, en valores de primera clase 
    también podría ser una buena idea. Varios lenguajes como Smalltalk y JavaScript han explorado 
    este camino.

## **Métodos y lambdas como ciudadanos de primera clase**

    Experimentos en otros lenguajes, como Scala y Groovy, han demostrado que permitir que conceptos como
    los métodos se utilicen como valores de primera clase facilita la programación al ampliar el 
    conjunto de herramientas disponibles para los programadores. Y una vez que los programadores se 
    familiarizan con una característica potente, se muestran reacios a usar lenguajes que no la incluyen.
    Los diseñadores de Java 8 decidieron permitir que los métodos fueran valores, con el fin de facilitar
    la programación. Además, la característica de Java 8 que trata a los métodos como valores constituye
    la base de varias otras funcionalidades de Java 8 (como Streams).
    
    La primera característica nueva de Java 8 que presentamos es la de las referencias a métodos. 
    Supongamos que deseas filtrar todos los archivos ocultos en un directorio. Necesitas escribir un 
    método que, dado un objeto File, te indique si está oculto. Afortunadamente, existe tal método en la
    clase File llamado isHidden. Este puede verse como una función que toma un File y devuelve un boolean.
    Pero para usarlo en un filtro, necesitas envolverlo en un objeto FileFilter, que luego pasas al 
    método File.listFiles, de la siguiente manera:

```java
File[] hiddenFilter = new File(".").listFiles(new FileFilter() {
    public boolean accept(File file) {
        return file.isHidden();    //<-- Filtering hidden files!
    }
});
```

    ¡Uf! Eso es horrible. Aunque solo son tres líneas importantes, son tres líneas opacas: todos 
    recordamos habernos preguntado "¿Realmente tengo que hacerlo así?" al encontrarlo por primera vez.
    Ya tienes el método isHidden que podrías usar. ¿Por qué tienes que envolverlo en una clase 
    FileFilter tan verbosa y luego instanciarla? Porque eso era lo que tenías que hacer antes de Java 8.

    Ahora, puedes reescribir ese código de la siguiente manera:

```java
File[] hiddenFiles = new File(".").listFiles(File::isHidden);
```

    ¡Vaya! ¿No es genial? Ya tienes disponible la función isHidden, así que la pasas al método listFiles
    usando la sintaxis :: de referencias a métodos de Java 8 (que significa "usa este método como 
    un valor"); nótese que también hemos empezado a usar la palabra función para referirnos a métodos.
    Explicaremos más adelante cómo funciona mecánicamente.

    Una ventaja es que tu código ahora se lee más parecido al enunciado del problema.

    Aquí tienes un adelanto de lo que viene: los métodos ya no son valores de segunda clase. Análogo
    a usar una referencia de objeto cuando pasas un objeto (y las referencias de objeto se crean con new),
    en Java 8, cuando escribes File::isHidden, creas una referencia a un método, que puede pasarse 
    de forma similar. Este concepto se analiza en detalle en el capítulo 3. Dado que los métodos 
    contienen código (el cuerpo ejecutable de un método), el uso de referencias a métodos permite 
    pasar código como valor, como se muestra en la figura 1.3. La figura 1.4 ilustra el concepto. 
    Verás un ejemplo concreto (seleccionar manzanas de un inventario) en la siguiente sección.

##   **Lambdas: Funciones Anonimas**
    Además de permitir que los métodos (nombrados) sean valores de primera clase, Java 8 introduce 
    un concepto más rico: las funciones como valores, incluyendo lambdas (o funciones anónimas). Por
    ejemplo, puedes escribir (int x) -> x + 1 para expresar "la función que, al recibir un argumento 
    x, devuelve x + 1".

    Podrías preguntarte por qué es necesario, ya que podrías definir un método add1 en una clase 
    MyMathsUtils y usar MyMathsUtils::add1. Sí, es posible, pero la sintaxis lambda es más concisa 
    cuando no tienes un método o clase útil disponible.

    Los programas que usan estos conceptos se consideran escritos en estilo de programación funcional,
    lo que significa "escribir programas que pasan funciones como valores de primera clase".

```java
//Old way of filtering hidden files
File[] hiddenFiles = new File(".").listFiles(new FileFilter() {
            public boolean accept(File file) {
                return file.isHidden();
            }  //Filtering files with the isHidden method requires wrapping the method inside a FileFilter
                //object before passing it to the File.listFiles method.
        });

//Java 8 style
File[] hiddenFiles = new File(".").listFiles(File::isHidden);
// In Java 8 you can pass the isHidden function to the listFiles method using the method reference :: syntax.
```

## **Pasar código: un ejemplo**
    Veamos un ejemplo de cómo esto te ayuda a escribir programas (analizado con más detalle en el 
    capítulo 2). Todo el código de los ejemplos está disponible en un repositorio de GitHub y como 
    descarga desde la página web del libro. Ambos enlaces se pueden encontrar en www.manning.com/books/modern-java-in-action.

    Supón que tienes una clase Apple con un método getColor y una variable inventory que contiene 
    una lista de manzanas. Quizás quieras seleccionar todas las manzanas verdes (usando un tipo enum
    Color que incluye los valores GREEN y RED) y devolverlas en una lista. La palabra filter (filtrar)
    se usa comúnmente para expresar este concepto.

    Antes de Java 8, podrías escribir un método como filterGreenApples:

```java
public static List<Apple> filterGreenApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();  // The result list accumulates the result; it starts as empty, and then green apples are added one by one.
    for (Apple apple : inventory) {
        if (GREEN.equals(apple.getColor())) {  // This line text selects only green apples.
            result.add(apple);
        }
        return result;
    }
}
```

    Pero luego, alguien querría la lista de manzanas pesadas (por ejemplo, más de 150 g), y entonces,
    con pesar, escribirías el siguiente método para lograrlo (quizás incluso usando copiar y pegar):

```java
public static List<Apple> filterHeavyApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (apple.getWeight() > 150) {  // This line text selects only heavy apples.
            result.add(apple);
        }
    }
    return result;
}
```

    Todos conocemos los peligros del copiar y pegar en ingeniería de software (actualizaciones y 
    correcciones de errores aplicadas a una variante pero no a otra). Además, estos dos métodos solo
    difieren en una línea: la condición resaltada dentro del if. Si la diferencia entre las llamadas 
    a los métodos hubiera sido el rango de peso aceptable, podrías haber pasado como argumentos los
    límites inferior y superior al filtro (por ejemplo, (150, 1000) para seleccionar manzanas pesadas 
    o (0, 80) para manzanas ligeras).

    Pero como mencionamos antes, Java 8 permite pasar el código de la condición como argumento, evitando
    así la duplicación del método filter. Ahora puedes escribir:

```java
public static boolean isGreenApple(Apple apple) {
    return GREEN.equals(apple.getColor());
}
public static boolean isHeavyApple(Apple apple) {
    return apple.getWeight() > 150;
}
public interface Predicate<T>{  // Included for clarity (normally imported from java.util.function)
    boolean test(T t);
}
static List<Apple> filterApples(List<Apple> inventory, Predicate<Apple> p) { // A method is passed as a Predicate parameter named p (see the sidebar “What’s a Predicate?”).
    List<Apple> result = new ArrayList<>();
    for (Apple apple: inventory){
        if (p.test(apple)) {  // Does the apple match the condition represented by p?
            result.add(apple);
        }
    }
    return result;
}

```

    Y para usarlo, llamas a cualquiera de los siguientes:

```java
filterApples(inventory, Apple::isGreenApple);
//or
filterApples(inventory, Apple::isHeavyApple);
```

    Nosotros explicamos cómo funciona esto en detalle en los próximos dos capítulos. La idea clave 
    que debes tener en cuenta por ahora es que puedes pasar un método como si fuera un valor en Java 8.

    ¿Qué es un Predicate?
    El código anterior pasó un método Apple::isGreenApple (que recibe una Apple como argumento y 
    devuelve un boolean) al método filterApples, que esperaba un parámetro de tipo Predicate<Apple>.
    La palabra predicate (predicado) se usa a menudo en matemáticas para referirse a algo similar a 
    una función que toma un valor como argumento y devuelve true o false. Como verás más adelante, 
    Java 8 también te permitiría escribir Function<Apple, Boolean> —algo más familiar para quienes 
    aprendieron sobre funciones pero no sobre predicados en la escuela—, pero usar Predicate<Apple> 
    es más estándar (y ligeramente más eficiente porque evita el boxing de un boolean en un Boolean).


## **De pasar métodos a lambdas**
    Pasar métodos como valores es claramente útil, pero es molesto tener que escribir una definición
    para métodos cortos como isHeavyApple e isGreenApple cuando se usan quizás solo una o dos veces.
    Pero Java 8 también ha resuelto esto. Introduce una nueva notación (funciones anónimas, o lambdas)
    que te permite escribir simplemente
```java
filterApples(inventory, (Apple a) -> GREEN.equals(a.getColor()) );
//or
filterApples(inventory, (Apple a) -> a.getWeight() > 150 );
//or even
filterApples(inventory, (Apple a) -> a.getWeight() < 80 ||RED.equals(a.getColor()) );
```

    Ni siquiera necesitas escribir una definición de método que se use solo una vez; el código es más
    claro y conciso porque no tienes que buscar dónde está definido el código que estás pasando.

    Pero si una lambda supera unas pocas líneas de longitud (de modo que su comportamiento no sea 
    inmediatamente claro), deberías usar mejor una referencia a un método con un nombre descriptivo,
    en lugar de una lambda anónima. La claridad del código debe ser tu guía.

    Los diseñadores de Java 8 casi podrían haber terminado aquí, y quizás lo habrían hecho si no fuera
    por las CPU multinúcleo. La programación en estilo funcional, tal como se ha presentado hasta ahora,
    resulta ser muy poderosa, como verás. Java podría haberse completado simplemente añadiendo métodos
    genéricos como filter y algunos similares en sus bibliotecas.

```java
static <T> Collection<T> filter(Collection<T> c, Predicate<T> p);
```
    Ni siquiera tendrías que escribir métodos como filterApples, porque, por ejemplo, la llamada 
    anterior
```java
filterApples(inventory, (Apple a) -> a.getWeight() > 150 );
```
    podría escribirse como una llamada al método de biblioteca filter:
```java
filter(inventory, (Apple a) -> a.getWeight() > 150 );
```
    Pero, por razones relacionadas con aprovechar mejor el paralelismo, los diseñadores no hicieron 
    esto. En su lugar, Java 8 incluye una nueva API similar a las colecciones llamada Stream, que 
    contiene un conjunto completo de operaciones similares a filter (como map y reduce), conocidas 
    por programadores funcionales, junto con métodos para convertir entre colecciones y streams, que
    ahora exploraremos.

## **Streams**
    Nearly every Java application makes and processes collections. But working with collec-tions 
    isn’t always ideal. For example, let’s say you need to filter expensive transactionsfrom a list
    and then group them by currency. You’d need to write a lot of boilerplate code to implement 
    this data-processing query, as shown here:

```java
    // Creates the Map where the grouped transaction will be accumulated
Map<Currency, List<Transaction>> transactionsByCurrencies = new HashMap<>();
for (Transaction transaction : transactions){  //Iterates the List of transactions
        if(transaction.getPrice() >1000){  //Filters expensive transactions
            Currency currency = transaction.getCurrency();  //Extracts the transaction’s currency
            List<Transaction> transactionsForCurrency = transactionsByCurrencies.get(currency);
            if(transactionsForCurrency ==null){  //If there isn't an entry in the grouping Map for this currency create it
                transactionsForCurrency =new ArrayList<>();
                transactionsByCurrencies.put(currency, transactionsForCurrency);
            }
            transactionsForCurrency.add(transaction);  //Adds the currently traversed transaction to the List of transactions with the same currency
        }
}
```

    Además, es difícil entender de un vistazo qué hace el código debido a las múltiples sentencias 
    anidadas de control de flujo. Usando la API de Streams, puedes resolver este problema de la 
    siguiente manera:

```java
import static java.util.stream.Collectors.groupingBy;
Map<Currency, List<Transaction>> transactionsByCurrencies =
    transactions.stream()
        .filter((Transaction t) -> t.getPrice() > 1000)  // Filters expensive transactions
        .collect(groupingBy(Transaction::getCurrency));  // Groups them by currency
```

    No te preocupes por este código por ahora, porque puede parecer un poco mágico. Los capítulos 4 
    a 7 están dedicados a explicar cómo entender la API de Streams. Por ahora, es importante notar 
    que la API de Streams ofrece una forma diferente de procesar datos en comparación con la API de 
    Colecciones. Al usar una colección, tú gestionas el proceso de iteración manualmente: debes 
    recorrer los elementos uno por uno con un bucle for-each y procesarlos secuencialmente. A esto 
    se llama iteración externa. En cambio, al usar la API de Streams, no necesitas pensar en bucles:
    el procesamiento ocurre internamente dentro de la biblioteca. A esto se llama iteración interna.
    Volveremos a estos conceptos en el capítulo 4.

    Como segundo problema al trabajar con colecciones, piensa por un momento en cómo procesarías la 
    lista de transacciones si tuvieras una cantidad muy grande; ¿cómo podrías procesar esta enorme 
    lista? Una sola CPU no podría manejar esa cantidad de datos, pero probablemente tienes una 
    computadora con múltiples núcleos. Idealmente, querrías distribuir el trabajo entre los diferentes 
    núcleos disponibles para reducir el tiempo de procesamiento. En teoría, si tienes ocho núcleos, 
    deberían poder procesar los datos ocho veces más rápido que con un solo núcleo, ya que trabajan 
    en paralelo.

## **Ordenadores multinúcleo**
    Todos los ordenadores de sobremesa y portátiles nuevos son ordenadores multinúcleo. En lugar de 
    tener una sola CPU, cuentan con cuatro, ocho o más CPUs (normalmente llamadas núcleos). El 
    problema es que un programa clásico en Java utiliza solo uno de estos núcleos, desperdiciando el
    potencial de los demás. De forma similar, muchas empresas usan clústeres informáticos
    (computadoras conectadas entre sí mediante redes rápidas) para procesar grandes cantidades de 
    datos de forma eficiente. Java 8 facilita nuevos estilos de programación que permiten aprovechar
    mejor este tipo de hardware.

    El motor de búsqueda de Google es un ejemplo de un programa demasiado grande para ejecutarse en 
    un solo ordenador. Lee cada página de internet y crea un índice que asocia cada palabra que 
    aparece en cualquier página web con todas las URL que contienen esa palabra. Luego, cuando 
    realizas una búsqueda en Google con varias palabras, el software usa este índice para mostrarte 
    rápidamente un conjunto de páginas web que contienen dichas palabras. Intenta imaginar cómo 
    codificarías este algoritmo en Java (incluso para un índice más pequeño que el de Google, 
    necesitarías aprovechar todos los núcleos de tu computadora).

## **El multihilos es difícil**
    El problema es que aprovechar el paralelismo escribiendo código multihilo (usando la API de hilos
    de versiones anteriores de Java) es difícil. Tienes que pensar de forma diferente: los hilos 
    pueden acceder y actualizar variables compartidas al mismo tiempo. Como resultado, los datos 
    podrían cambiar inesperadamente si no se coordinan adecuadamente. Este modelo es más difícil de 
    entender que un modelo secuencial paso a paso. Por ejemplo, la figura 1.5 muestra un posible 
    problema con dos hilos intentando añadir un número a una variable compartida sum si no están 
    sincronizados correctamente.

    Java 8 aborda ambos problemas (el código repetitivo y la complejidad al procesar colecciones, y 
    la dificultad para aprovechar los multicore) con la API Streams (java.util.stream). El primer 
    motivo de diseño es que existen muchos patrones de procesamiento de datos (similares a filterApples
    en la sección anterior u operaciones conocidas de lenguajes de consulta de bases de datos como 
    SQL) que se repiten constantemente y que se beneficiarían de formar parte de una biblioteca: 
    filtrar datos según un criterio (por ejemplo, manzanas pesadas), extraer datos (por ejemplo, 
    extraer el campo de peso de cada manzana en una lista) o agrupar datos (por ejemplo, agrupar una
    lista de números en listas separadas de pares e impares), entre otros. El segundo motivo es que 
    tales operaciones a menudo pueden paralelizarse. Por ejemplo, como se ilustra en la figura 1.6, 
    filtrar una lista en dos CPUs podría hacerse pidiendo a una CPU que procese la primera mitad de 
    la lista y a la segunda CPU que procese la otra mitad. Esto se llama paso de división (1). Las 
    CPUs luego filtran sus respectivas mitades de la lista (2). Finalmente (3), una CPU combinaría 
    los dos resultados. (Esto está estrechamente relacionado con cómo Google realiza búsquedas tan 
    rápidamente, utilizando muchos más de dos procesadores).

    Por ahora, simplemente diremos que la nueva API Streams se comporta de forma similar a la API de
    Colecciones existente en Java: ambas proporcionan acceso a secuencias de elementos de datos. Pero
    es útil tener en cuenta que las colecciones se centran principalmente en almacenar y acceder a 
    datos, mientras que los flujos (streams) se enfocan sobre todo en describir cálculos sobre los 
    datos. El punto clave aquí es que la API Streams permite y fomenta que los elementos de un flujo 
    sean procesados en paralelo. Aunque al principio pueda parecer extraño, a menudo la forma más 
    rápida de filtrar una colección (por ejemplo, usar filterApples en la sección anterior sobre una 
    lista) es convertirla en un flujo, procesarla en paralelo y luego convertirla nuevamente en una 
    lista. Nuevamente, diremos simplemente “paralelismo casi gratuito” y daremos una muestra de cómo 
    puedes filtrar manzanas pesadas de una lista de forma secuencial o en paralelo usando flujos y una
    expresión lambda.
    Este es un ejemplo de procesamiento secuencial:

```java
import static java.util.stream.Collectors.toList;
List<Apple> heavyApples =
    inventory.stream().filter((Apple a) -> a.getWeight() > 150)
    .collect(toList());
```

    Y aquí está usando procesamiento paralelo:

```java
import static java.util.stream.Collectors.toList;
List<Apple> heavyApples =
    inventory.parallelStream().filter((Apple a) -> a.getWeight() > 150)
    .collect(toList());
```

## **Paralelismo en Java y sin estado mutable compartido**
    Siempre se ha dicho que el paralelismo en Java es difícil, y todo lo relacionado con synchronized
    es propenso a errores. ¿Dónde está la solución mágica en Java 8?
    Hay dos soluciones clave. Primero, la biblioteca se encarga de la partición: divide un flujo grande
    en varios más pequeños para procesarlos en paralelo automáticamente. Segundo, este paralelismo 
    casi gratuito que ofrecen los flujos (streams) solo funciona si los métodos pasados a funciones 
    como filter no interactúan entre sí (por ejemplo, mediante objetos compartidos mutables). Pero 
    resulta que esta restricción se siente natural para un programador (véase, por ejemplo, nuestro 
    caso Apple::isGreenApple). Aunque el significado principal de "funcional" en programación funcional
    sea "usar funciones como valores de primera clase", a menudo tiene un matiz secundario de "sin 
    interacción durante la ejecución entre componentes".

    El capítulo 7 explora con mayor detalle el procesamiento paralelo de datos en Java 8 y su rendimiento. 
    Uno de los problemas prácticos que los desarrolladores de Java 8 encontraron al evolucionar el 
    lenguaje con todas estas nuevas funcionalidades fue cómo evolucionar interfaces existentes. Por 
    ejemplo, el método Collections.sort pertenece a la interfaz List, pero nunca fue incluido. 
    Idealmente, querrías hacer list.sort(comparator) en lugar de Collections.sort(list, comparator). 
    Esto puede parecer trivial, pero antes de Java 8 solo podías actualizar una interfaz si actualizabas
    todas las clases que la implementaban: ¡una pesadilla logística! Este problema se resuelve en Java 
    8 mediante los métodos por defecto (default methods).

## **Métodos por defecto y módulos Java**
    Como mencionamos anteriormente, los sistemas modernos tienden a construirse a partir de componentes,
    quizás adquiridos desde otros lugares. Históricamente, Java ofrecía poco soporte para esto, aparte
    de un archivo JAR que contiene un conjunto de paquetes Java sin una estructura particular. Además,
    era difícil evolucionar las interfaces de esos paquetes: cambiar una interfaz en Java implicaba 
    cambiar todas las clases que la implementaban. Java 8 y 9 han comenzado a abordar este problema.

    Primero, Java 9 proporciona un sistema de módulos que ofrece sintaxis para definir módulos que 
    contienen colecciones de paquetes, permitiendo un control mucho mejor sobre la visibilidad y los 
    espacios de nombres. Los módulos enriquecen un componente simple tipo JAR con estructura, tanto 
    para documentación del usuario como para verificación automática; los explicamos en detalle en el 
    capítulo 14. Segundo, Java 8 añadió métodos por defecto para apoyar interfaces evolutivas. Los 
    tratamos en profundidad en el capítulo 13. Son importantes porque cada vez será más común 
    encontrarlos en interfaces, pero como relativamente pocos programadores necesitarán escribir métodos
    por defecto por sí mismos, y dado que facilitan la evolución del software más que ayudar a escribir 
    un programa específico, mantenemos aquí la explicación breve y basada en ejemplos.

    En la sección 1.4, dimos el siguiente ejemplo de código en Java 8:
```java
List<Apple> heavyApples1 =
    inventory.stream().filter((Apple a) -> a.getWeight() > 150)
    .collect(toList());
List<Apple> heavyApples2 =
    inventory.parallelStream().filter((Apple a) -> a.getWeight() > 150)
            .collect(toList());
```

    Pero aquí surge un problema: una List<T> anterior a Java 8 no tiene los métodos stream ni 
    parallelStream, y tampoco los tiene la interfaz Collection<T> que implementa, porque estos métodos 
    aún no se habían concebido. Y sin estos métodos, este código no compilará. La solución más simple,
    que podrías aplicar en tus propias interfaces, habría sido que los diseñadores de Java 8 añadieran 
    el método stream a la interfaz Collection y agregaran su implementación en la clase ArrayList.

    Pero hacer esto habría sido un desastre para los usuarios. Muchos frameworks alternativos de 
    colecciones implementan interfaces de la API Collections. Añadir un nuevo método a una interfaz 
    significa que todas las clases concretas deben proporcionar una implementación para él. Los 
    diseñadores del lenguaje no tienen control sobre las implementaciones existentes de Collection, 
    por lo que surge un dilema: ¿cómo puedes evolucionar interfaces publicadas sin romper 
    implementaciones existentes?

    La solución en Java 8 es romper este último vínculo: ahora una interfaz puede contener firmas de 
    métodos para los cuales una clase implementadora no proporciona una implementación. Entonces, 
    ¿quién los implementa? Los cuerpos de los métodos faltantes se definen directamente en la interfaz 
    (de ahí el nombre de implementaciones por defecto), en lugar de en la clase que la implementa.

    Esto permite que el diseñador de una interfaz amplíe la interfaz más allá de los métodos 
    originalmente planeados, sin romper el código existente. Java 8 permite usar la palabra clave 
    default existente en las especificaciones de interfaz para lograr esto. Por ejemplo, en Java 8 
    puedes llamar al método sort directamente sobre una lista. Esto es posible gracias al siguiente 
    método por defecto en la interfaz List de Java 8, que invoca al método estático Collections.sort:

```java
default void sort(Comparator<? super E> c) {
    Collections.sort(this, c);
}
```
    Esto significa que las clases concretas de List no tienen que implementar explícitamente sort, 
    mientras que en versiones anteriores de Java, dichas clases concretas habrían fallado al 
    recompilarse a menos que proporcionaran una implementación para sort.

    Pero un momento. Una sola clase puede implementar múltiples interfaces, ¿verdad? Si tienes varias
    implementaciones por defecto en distintas interfaces, ¿eso significa que tienes una forma de 
    herencia múltiple en Java? Sí, en cierta medida. Mostramos en el capítulo 13 que existen algunas 
    reglas que evitan problemas como el famoso problema de herencia en diamante presente en C++.

## **Otras buenas ideas de la programación funcional**
    Las secciones anteriores presentaron dos ideas centrales de la programación funcional que ahora 
    forman parte de Java: usar métodos y lambdas como valores de primera clase, y la idea de que las 
    llamadas a funciones o métodos pueden ejecutarse de forma eficiente y segura en paralelo si no 
    hay estado compartido mutable. Ambas ideas son aprovechadas por la nueva API Streams descrita 
    anteriormente.

    Lenguajes funcionales comunes (SML, OCaml, Haskell) también ofrecen otros mecanismos para ayudar 
    a los programadores. Uno de ellos es evitar null mediante el uso explícito de tipos de datos más 
    descriptivos. Tony Hoare, una figura destacada de la informática, dijo en una presentación en 
    QCon Londres 2009:

        --"Lo llamo mi error de mil millones de dólares. Fue la invención de la referencia nula en 1965.
        [...] No pude resistir la tentación de incluir una referencia nula, simplemente porque era muy 
        fácil de implementar."--

    Java 8 introdujo la clase Optional<T>, que, si se usa de forma consistente, puede ayudar a evitar
    las excepciones de puntero nulo (null-pointer exceptions). Es un objeto contenedor que puede o no 
    contener un valor. Optional<T> incluye métodos para manejar explícitamente el caso en que un valor
    esté ausente, evitando así dichas excepciones. Utiliza el sistema de tipos para permitirte indicar 
    cuándo se espera que una variable pueda tener un valor faltante. Analizamos Optional<T> en detalle 
    en el capítulo 11.

    Una segunda idea es la del pattern matching (coincidencia de patrones) estructural. Esto se usa 
    en matemáticas. Por ejemplo:
```math
    f(0) = 1
    f(n) = n*f(n-1) otherwise
```

    En Java, escribirías una sentencia if-then-else o switch. Otros lenguajes han mostrado que, para
    tipos de datos más complejos, el pattern matching puede expresar ideas de programación de forma 
    más concisa que usar if-then-else. Para tales tipos de datos, también podrías usar polimorfismo 
    y sobrescritura de métodos como alternativa a if-then-else, aunque existe una discusión continua 
    en el diseño del lenguaje sobre cuál es más adecuado. Diríamos que ambas son herramientas útiles 
    y que deberías tenerlas a tu disposición. Desafortunadamente, Java 8 no tiene soporte completo 
    para pattern matching, aunque mostramos cómo se puede expresar en el capítulo 19. Actualmente se 
    está discutiendo una propuesta de mejora para Java (JEP) que añadiría pattern matching en una 
    futura versión (ver http://openjdk.java.net/jeps/305). Mientras tanto, ilustraremos con un ejemplo 
    escrito en el lenguaje de programación Scala (otro lenguaje similar a Java que usa la JVM y que 
    ha inspirado algunos aspectos de la evolución de Java; ver capítulo 20). Supongamos que quieres 
    escribir un programa que realice simplificaciones básicas en un árbol que representa una expresión 
    aritmética. Dado un tipo de datos Expr que representa dichas expresiones, en Scala puedes escribir 
    el siguiente código para descomponer un Expr en sus partes y luego devolver otro Expr:

```Scala
def simplifyExpression(expr: Expr): Expr = expr match {
    case BinOp("+", e, Number(0)) => e  // add 0
    case BinOp("-", e, Number(0)) => e  // substract 0
    case BinOp("*", e, Number(1)) => e  // multiplies by 1
    case BinOp("/", e, Number(1)) => e  // divides by 1
    case _ => expr  // Can’t be simplified with these cases, so leave alone
}
```

    En Scala, la sintaxis expr match corresponde al switch de Java. No te preocupes por este código 
    por ahora: aprenderás más sobre el pattern matching en el capítulo 19. Por ahora, puedes pensar 
    en el pattern matching como una forma extendida de switch que puede descomponer un tipo de dato 
    en sus componentes al mismo tiempo.

    ¿Por qué el switch en Java debería limitarse a valores primitivos y cadenas? Los lenguajes 
    funcionales suelen permitir usar switch en muchos más tipos de datos, incluyendo pattern matching 
    (en Scala, esto se logra con la operación match). En el diseño orientado a objetos, el patrón 
    visitor es común para recorrer una familia de clases (como los componentes de un coche: ruedas, 
    motor, chasis, etc.) y aplicar una operación a cada objeto visitado. Una ventaja del pattern 
    matching es que el compilador puede detectar errores comunes, como: “La clase Brakes forma parte 
    de la jerarquía de clases que representan componentes de Car. Olvidaste manejarla explícitamente”.

    Los capítulos 18 y 19 ofrecen una introducción completa a la programación funcional y a cómo 
    escribir programas en estilo funcional en Java 8, incluyendo las herramientas de funciones de su 
    biblioteca. El capítulo 20 compara las características de Java 8 con las de Scala, un lenguaje que,
    como Java, se ejecuta sobre la JVM y que ha evolucionado rápidamente, desafiando ciertos aspectos 
    del nicho de Java en el ecosistema de lenguajes de programación. Este contenido se sitúa al final 
    del libro para ofrecer una visión más clara del porqué se añadieron las nuevas funciones en 
    Java 8 y 9.

## **Características de Java 8, 9, 10 y 11: ¿Por dónde empezar?**
    Java 8 y Java 9 introdujeron actualizaciones significativas, pero como programador Java, lo más 
    probable es que sean las novedades de Java 8 las que más afecten tu trabajo diario a nivel de 
    código: pasar métodos o lambdas se ha convertido en un conocimiento esencial. En cambio, las 
    mejoras de Java 9 potencian la capacidad de definir y usar componentes a mayor escala, ya sea 
    estructurando sistemas con módulos o usando herramientas de programación reactiva. Java 10 
    representa un avance más pequeño, centrado en la inferencia de tipos para variables locales (var),
    y en Java 11 se amplía la sintaxis de los parámetros de expresiones lambda.

## **Resumen**
- Ten en cuenta la idea del ecosistema de lenguajes y la presión de evolucionar o decaer. Aunque Java
  pueda estar muy saludable ahora, recordamos otros lenguajes sanos como COBOL que no evolucionaron.
- Las principales novedades de Java 8 aportan conceptos y funcionalidades nuevas que facilitan 
  escribir programas efectivos y concisos.
- Los procesadores multicore no se aprovechan completamente con las prácticas de programación 
  anteriores a Java 8.
- Las funciones son valores de primera clase; recuerda cómo se pasan métodos como valores funcionales
  y cómo se escriben funciones anónimas (lambdas).
- El concepto de streams en Java 8 generaliza muchos aspectos de las colecciones, permite un código 
  más legible y posibilita procesar elementos en paralelo.
- La programación a gran escala basada en componentes y la evolución de interfaces no fueron 
  históricamente bien atendidas por Java. Ahora puedes definir módulos en Java 9 y usar métodos por 
  defecto para mejorar interfaces sin modificar todas las clases que las implementan.
- Otras ideas interesantes de la programación funcional incluyen el manejo de null y el uso de pattern
  matching.

