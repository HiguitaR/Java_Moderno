# ***PARTE 1***


## *Procesamiento de Streams (Flujos)*

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

### *Pasar código a métodos con behavior parameterization (Parametrización de Comportamiento)*

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

### **Paralelismo y datos mutables compartidos**

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

### **Java necesita evolucionar**

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

### **Funciones en Java**

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

