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