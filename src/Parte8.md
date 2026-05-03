# Parte 3

# Programación efectiva con streams y lambdas
La tercera parte de este libro explora varios temas de Java 8 y Java 9 que te harán más efectivo al 
usar Java y mejorarán tu código base con idiomás modernos. Como está orientado hacia ideas de 
programación más avanzadas, nada más adelante en el libro depende de las técnicas descritas aquí.
El Capítulo 8 es un nuevo capítulo para la segunda edición y explora las mejoras de la Collection 
API de Java 8 y Java 9, cubriendo el uso de collection factories y el aprendizaje de nuevos patrones
idiomáticos para trabajar con colecciones List y Set, junto con patrones idiomáticos que involucran 
Map.
El Capítulo 9 explora cómo puedes mejorar tu código existente usando nuevas características de 
Java 8 y algunas recetas. Además, explora técnicas vitales de desarrollo de software como patrones 
de diseño, refactorización, pruebas y depuración.
El Capítulo 10 es nuevamente nuevo para la segunda edición. Explora la idea de basar una API en un 
DSL (Domain-Specific Language). Esta no es solo una forma poderosa de diseñar APIs, sino también una
que está siendo cada vez más popular y ya visible en Java, como en las interfaces Comparator, Stream
y Collector.

# Capitulo 8
# Mejoras en la Collection API

### Este capítulo cubre
- Usar collection factories
- Aprender nuevos patrones idiomáticos para usar con List y Set
- Aprende patrones idiomáticos para trabajar con Map

Tu vida como desarrollador de Java sería bastante solitaria sin la Collection API. Las colecciones 
se usan en cada aplicación de Java. En capítulos anteriores, viste qué tan útil es la combinación de
Collections con la Streams API para expresar consultas de procesamiento de datos. Sin embargo, la 
Collection API tenía varias deficiencias, que a veces hacía que fuera verbosa y propensa a errores.
En este capítulo, aprenderás sobre las nuevas adiciones a la Collection API en Java 8 y Java 9 que 
facilitarán tu vida. Primero, aprenderás sobre los collection factories en Java 9—nuevas adiciones 
que simplifican el proceso de crear pequeñas listas, sets y mapas. Luego, aprenderás cómo aplicar 
patrones idiomáticos de eliminación y reemplazo en listas y sets gracias a las mejoras de Java 8. 
Finalmente, aprenderás sobre nuevas operaciones convenientes disponibles para trabajar con maps.
El Capítulo 9 explora un rango más amplio de técnicas para refactorizar código Java de estilo antiguo.

## 8.1 Collection factories
Java 9 introdujo algumas formas convenientes de crear pequeños objetos de colección. Primero, 
revisaremos por qué los programadores necesitaban una mejor manera de hacer las cosas; luego, te 
mostraremos cómo usar los nuevos métodos de fábrica.

¿Cómo crearías una pequeña lista de elementos en Java? Podrías wanting agrupar los nombres de tus 
amigos que van de vacaciones, por ejemplo. Aquí hay una forma:
```java
List<String> friends = new ArrayList<>();
friends.add("Raphael");
friends.add("Olivia");
friends.add("Thibaut");
```
¡Pero eso son bastante líneas para almacenar tres cadenas! Una forma más conveniente de escribir 
este código es usar el método de fábrica Arrays.asList():
```java
List<String> friends = Arrays.asList("Raphael", "Olivia", "Thibaut");
```
Obtienes una lista de tamaño fijo que puedes actualizar, pero no agregar elementos ni eliminar. 
Intentar agregar elementos, por ejemplo, resulta en un UnsupportedOperationException, pero actualizar
usando el método set está permitido:
```java
List<String> friends = Arrays.asList("Raphael", "Olivia");
friends.set(0, "Richard");
friends.add("Thibaut"); // throws UnsupportedOperationException
```
Este comportamiento parece ligeramente sorprendente porque la lista subyacente está respaldada por 
un arreglo mutable de tamaño fijo.
¿Qué pasa con un Set? Desafortunadamente, no existe un método de fábrica Arrays.asSet(), por lo que 
necesitas otro truco. Puedes usar el constructor de HashSet, que acepta una List:
```java
Set<String> friends = new HashSet<>(Arrays.asList("Raphael", "Olivia", "Thibaut"));
```
Alternativamente, podrías usar la Streams API:
```java
Set<String> friends = Stream.of("Raphael", "Olivia", "Thibaut")
.collect(Collectors.toSet());
```
Sin embargo, ambas soluciones están lejos de ser elegantes e involucran asignaciones de objetos 
innecesarias detrás de escena. También ten en cuenta que obtienes un Set mutable como resultado.
¿Qué pasa con Map? No hay una forma elegante de crear pequeños mapas, pero no te preocupes; Java 9 
agregó métodos de fábrica para simplificar tu vida cuando necesitas crear pequeñas listas, sets y 
mapas.
Comenzamos el recorrido de nuevas formas de crear colecciones en Java mostrándote qué hay de nuevo 
con Lists.

### Literales de colecciones
Algunos lenguajes, incluyendo Python y Groovy, soportan literales de colecciones, lo cual te permite
crear colecciones usando sintaxis especial, como 42, 1, 5 para crear una lista de tres números. Java
no provee soporte sintáctico porque los cambios en el lenguaje vienen con un alto costo de 
mantenimiento y restringen el uso futuro de una sintaxis posible.
En cambio, Java 9 agrega soporte mejorando la Collection API.

### 8.1.1 List factory
Puedes crear una lista simplemente llamando al método de fábrica List.of:
```java
List<String> friends = List.of("Raphael", "Olivia", "Thibaut");
System.out.println(friends);// [Raphael, Olivia, Thibaut]
```
Sin embargo, notarás algo extraño. Intenta agregar un elemento a tu lista de amigos:
```java
List<String> friends = List.of("Raphael", "Olivia", "Thibaut");
friends.add("Chih-Chun");
```
Ejecutar este código resulta en un java.lang.UnsupportedOperationException. De hecho, la lista 
producida es inmutable. Reemplazar un elemento con el método set() lanza una excepción similar. No 
podrás modificarla usando el método set tampoco.
Esta restricción es algo bueno, sin embargo, ya que te protege de mutaciones no deseadas de las 
colecciones. Nada te impide tener elementos que sean mutables ellos mismos. Si necesitas una lista 
mutable, todavía puedes instanciar una manualmente. Finalmente, ten en cuenta que para prevenir bugs
inesperados y habilitar una representación interna más compacta, los elementos nulos no son permitidos.

### Sobrecarga vs. varargs
Si inspectas más la interfaz List, notarás varias variantes sobrecargadas de List.of:
```java
static <E> List<E> of(E e1, E e2, E e3, E e4)
static <E> List<E> of(E e1, E e2, E e3, E e4, E e5)
```
Podrías preguntarte por qué la API de Java no tuvo un método que use varargs para aceptar un número 
arbitrario de elementos en el siguiente estilo:
```java
static <E> List<E> of(E... elements)
```
Por debajo, la versión varargs allocates un arreglo extra, que es envuelto dentro de una lista. 
Pagas el costo de asignar un arreglo, inicializarlo, y hacer que sea colectado como basura después.
Al proveeer un número fijo de elementos (hasta diez) a través de una API, no pagas este costo. Ten 
en cuenta que todavía puedes crear List.of usando más de diez elementos, pero en ese caso la firma 
varargs es invocada. También ves este patrón aparecen con Set.of y Map.of.

Podrías preguntarte si deberías usar la Streams API en lugar de los nuevos métodos de fábrica de 
colecciones para crear tales listas. Después de todo, viste en capítulos anteriores que puedes usar 
el colector Collectors.toList() para transformar un stream en una lista.
A menos que necesites configurar alguna forma de procesamiento de datos y transformación de los 
datos, recomendamos que uses los métodos de fábrica; son más simples de usar, y la implementación de
los métodos de fábrica es más simple y adecuada.
Ahora que has aprendido sobre un nuevo método de fábrica para List, en la siguiente sección, 
trabajarás con Set.

### 8.1.2 Set factory
Al igual que con List.of, puedes crear un Set inmutable a partir de una lista de elementos:
```java
Set<String> friends = Set.of("Raphael", "Olivia", "Thibaut");
System.out.println(friends);
```
Si intentas crear un Set proporcionando un elemento duplicado, recibes un IllegalArgumentException. 
Esta excepción refleja el contrato que los sets imponen de unicidad de los elementos que contienen:
```java
//java.lang.IllegalArgumentException: duplicate element: Olivia
Set<String> friends = Set.of("Raphael", "Olivia", "Olivia");
```
Otra estructura de datos popular en Java es Map. En la siguiente sección, aprenderás sobre nuevas 
formas de crear Map.

### 8.1.3 Map factories
Crear un map es un poco más complicado que crear listas y sets porque tienes que incluir tanto la 
clave como el valor. Tienes dos formas de inicializar un map inmutable en Java 9. Puedes usar el 
método de fábrica Map.of, que alterna entre claves y valores:
```java
Map<String, Integer> ageOfFriends = Map.of("Raphael", 30, "Olivia", 25, "Thibaut", 26);
System.out.println(ageOfFriends); //{Olivia=25,Raphael=30,Thibaut=26}
```
Este método es conveniente si quieres crear un pequeño map de hasta diez claves y valores. Para ir 
más allá, usa el método de fábrica alternativo llamado Map.ofEntries, que toma objetos 
Map.Entry<K, V> pero está implementado con varargs. Este método requiere asignaciones de objetos 
adicionales para envolver una clave y un valor:
```java
import static java.util.Map.entry;
Map<String, Integer> ageOfFriends
= Map.ofEntries(entry("Raphael", 30),
entry("Olivia", 25),
entry("Thibaut", 26));
System.out.println(ageOfFriends);// -> {Olivia=25,Raphael=30,Thibaut=26}
```
Map.entry es un nuevo método de fábrica para crear objetos Map.Entry.

### Quiz 8.1
¿Qué pensás que es la salida del siguiente fragmento de código?
```java
List<String> actors = List.of("Keanu", "Jessica");
actors.set(0, "Brad");
System.out.println(actors);
```
### Respuesta:
Se lanza una UnsupportedOperationException. La colección producida por List.of es inmutable.

Hasta ahora, has visto que los nuevos métodos de fábrica de Java 9 te permiten crear colecciones
de manera más simple. Pero en la práctica, tenés que procesar las colecciones. En la siguiente 
sección, aprendés sobre algunas mejoras nuevas en List y Set que implementan patrones comunes de 
procesamiento directamente.

## 8.2 Trabajando con List y Set
Java 8 introdujo un par de métodos en las interfaces List y Set:
- removeIf elimina elementos que coincidan con un predicado. Está disponible en todas las clases que
implementan List o Set (y es heredado de la interfaz Collection).
- replaceAll está disponible en List y reemplaza elementos usando una función (UnaryOperator).
- sort también está disponible en la interfaz List y ordena la lista misma.

Todos estos métodos mutan las colecciones en las que se invocan. En otras palabras, cambian la 
colección misma, a diferencia de las operaciones de streams, que producen un nuevo resultado 
(copiado). ¿Por qué se agregaron tales métodos? Modificar colecciones puede ser propenso a errores y
verboso. Por lo tanto, Java 8 agregó removeIf y replaceAll para ayudar.

### 8.2.1 removeIf
Considerá el siguiente código, que intenta eliminar transacciones que tienen un código de referencia
que comienza con un dígito:
```java
for (Transaction transaction : transactions){
        if(Character.isDigit(transaction.getReferenceCode().charAt(0))){
            transactions.remove(transaction);
        }
}
```
¿Podés ver el problema? Lamentablemente, este código puede resultar en una 
ConcurrentModificationException. ¿Por qué? Internamente, el bucle for-each usa un objeto Iterator, 
por lo que el código ejecutado es el siguiente:
```java
for (Iterator<Transaction> iterator = transactions.iterator();
    iterator.hasNext(); ){
    Transaction transaction = iterator.next();
    if(Character.isDigit(transaction.getReferenceCode().charAt(0))){
        transactions.remove(transaction);//El problema es que estamos iterando y modificando la colección a través de dos objetos separados.
    }
}
```
Notá que dos objetos separados gestionan la colección:
- El objeto Iterator, que consulta la fuente usando next() y hasNext()
- El objeto Collection mismo, que elimina el elemento llamando a remove()

Como resultado, el estado del iterator ya no está sincronizado con el estado de la colección, y 
viceversa. Para resolver este problema, tenés que usar el objeto Iterator explícitamente y llamar a 
su método remove():
```java
for (Iterator<Transaction> iterator = transactions.iterator();
    iterator.hasNext(); ){
    Transaction transaction = iterator.next();
    if(Character.isDigit(transaction.getReferenceCode().charAt(0))){
        iterator.remove();
    }
}
```
Este código se ha vuelto bastante verboso de escribir. Este patrón de código ahora se puede expresar
directamente con el método removeIf de Java 8, que no solo es más simple sino que también te protege
de estos bugs. Toma un predicado que indica qué elementos eliminar:
```java
transactions.removeIf(transaction ->Character.isDigit(transaction.getReferenceCode().charAt(0)));
```
Sin embargo, a veces, en lugar de eliminar un elemento, querés reemplazarlo. Para este propósito, 
Java 8 agregó replaceAll.

### 8.2.2 replaceAll
El método replaceAll en la interfaz List te permite reemplazar cada elemento en una lista con uno 
nuevo. Usando la Streams API, podrías resolver este problema de la siguiente manera:
```java
referenceCodes.stream().map(code -> Character //[a12, C14, b13]
        .toUpperCase(code.charAt(0)) + 
        code.substring(1))
            .collect(Collectors.toList())
            .forEach(System.out::println);//outputs A12,C14, B13
```
Sin embargo, este código resulta en una nueva colección de strings. Querés una forma de actualizar 
la colección existente. Podés usar un objeto ListIterator de la siguiente manera (soportando un 
método set() para reemplazar un elemento):
```java
for (ListIterator<String> iterator = referenceCodes.listIterator();
        iterator.hasNext(); ){
    String code = iterator.next();
    iterator.set(Character.toUpperCase(code.charAt(0))+code.substring(1));
}
```
Como pueden ver, este código es bastante verboso. Además, como explicamos anteriormente, usar objetos
Iterator en conjunto con objetos de colección puede ser propenso a errores al mezclar iteración y 
modificación de la colección. En Java 8, simplemente podés escribir:
```java
referenceCodes.replaceAll(code -> Character.toUpperCase(code.charAt(0)) +
code.substring(1));
```
Aprendiste qué hay de nuevo con List y Set, pero no te olvides de Map. Las nuevas adiciones a la 
interfaz Map se cubren en la siguiente sección.

## 8.3 Trabajando con Map
Java 8 introdujo varios métodos default soportados por la interfaz Map. (Los métodos default se 
cubren en detalle en el capítulo 13, pero aquí podés pensar en ellos como métodos preimplementados
en una interfaz.) El propósito de estas nuevas operaciones es ayudarte a escribir código más conciso
usando un patrón idiomático disponible en lugar de implementarlo vos mismo. Мы veamos estas 
operaciones en las siguientes secciones, comenzando con el nuevo forEach.

### 8.3.1 forEach
Iterar sobre las claves y valores de un Map tradicionalmente ha sido incómodo. De hecho, necesitabas
usar un iterator de Map.Entry<K, V> sobre el entry set de un Map:
```java
for(Map.Entry<String, Integer> entry: ageOfFriends.entrySet()) {
    String friend = entry.getKey();
    Integer age = entry.getValue();
    System.out.println(friend + " is " + age + " years old");
}
```
Desde Java 8, la interfaz Map ha soportado el método forEach, que acepta un BiConsumer, tomando la 
clave y el valor como argumentos. Usar forEach hace que tu código sea más conciso:
```java
ageOfFriends.forEach((friend, age) -> System.out.println(friend + " is " +
    age + " years old"));
```
Una preocupación relacionada con iterar sobre datos es ordenarlos. Java 8 introdujo un par de formas
convenientes para comparar entradas en un Map.

Dos nuevas utilidades te permiten ordenar las entradas de un map por valores o claves:
- Entry.comparingByValue
- Entry.comparingByKey

El código:
```java
Map<String, String> favouriteMovies = Map.ofEntries(entry("Raphael", "Star Wars"),
    entry("Cristina", "Matrix"),
    entry("Olivia",
    "James Bond"));
favouriteMovies
    .entrySet()
    .stream()
    .sorted(Entry.comparingByKey())
    .forEachOrdered(System.out::println);//Procesa los elementos del stream en orden alfabético basado en el nombre de la persona.
```
produce, en orden:
```terminaloutput
Cristina=Matrix
Olivia=James Bond
Raphael=Star Wars
```
### HashMap y Rendimiento
La estructura interna de un HashMap fue actualizada en Java 8 para mejorar el rendimiento. Las 
entradas de un map típicamente se almacenan en buckets accedidos por el hashcode generado de la 
clave. Pero si muchas claves devuelven el mismo hashcode, el rendimiento se deteriora porque los 
buckets se implementan como LinkedLists con recuperación O(n). Hoy en día, cuando los buckets se 
vuelven muy grandes, se reemplazan dinámicamente con árboles ordenados, que tienen recuperación 
O(log(n)) y mejoran la búsqueda de elementos en colisión. Tené en cuenta que este uso de árboles 
ordenados es posible solo cuando las claves son Comparable (como clases String o Number).
Otro patrón común es cómo actuar cuando la clave que estás buscando en el Map no está presente. El 
nuevo método getOrDefault puede ayudar.

### 8.3.3 getOrDefault
Cuando la clave que buscas no está presente, recibís una referencia null que tenés que verificar para
prevenir una NullPointerException. Un estilo de diseño común es proporcionar un valor predeterminado
en su lugar. Ahora podés codificar esta idea de manera más simple usando el método getOrDefault. Este
método toma la clave como primer argumento y un valor predeterminado (para usar cuando la clave está
ausente del Map) como segundo argumento:
```java
Map<String, String> favouriteMovies = Map.ofEntries(entry("Raphael", "Star Wars"),
    entry("Olivia", "James Bond"));
System.out.println(favouriteMovies.getOrDefault("Olivia", "Matrix"));
System.out.println(favouriteMovies.getOrDefault("Thibaut", "Matrix"));
```
Tené en cuenta que si la clave existía en el Map pero estaba asociada accidentalmente con un valor 
null, getOrDefault aún puede devolver null. También tené en cuenta que la expresión que pasás como 
alternativa siempre se evalúa, independientemente de si la clave existe o no.
Java 8 también incluyó algunos patrones más avanzados para Dealing with la presencia y ausencia de 
valores para una clave dada. Aprenderás sobre estos nuevos métodos en la siguiente sección.

### 8.3.4 Patrones de compute
A veces, querés realizar una operación condicionalmente y almacenar su resultado, dependiendo de si 
una clave está presente o ausente en un Map. Podrías querer cachear el resultado de una operación 
costosa dada una clave, por ejemplo. Si la clave está presente, no hay necesidad de recalcular el 
resultado. Tres nuevas operaciones pueden ayudar:
- computeIfAbsent—Si no hay ningún valor especificado para la clave dada (está ausente o su valor es
null), calcula un nuevo valor usando la clave y agrégalo al Map.
- computeIfPresent—Si la clave especificada está presente, calcula un nuevo valor para ella y 
agrégalo al Map.
- compute—Esta operación calcula un nuevo valor para una clave dada y lo almacena en el Map.

Un uso de computeIfAbsent es para cachear información. Supone que analizás cada línea de un conjunto
de archivos y calculás su representación SHA-256. Si ya procesaste los datos anteriormente, no hay 
necesidad de recalcularlos.
Ahora supone que implementás un cache usando un Map, y usás una instancia de MessageDigest para 
calcular hashes SHA-256:
```java
Map<String, byte[]> dataToHash = new HashMap<>();
MessageDigest messageDigest = MessageDigest.getInstance("SHA-256");
```
Entonces podés iterar a través de los datos y cachear los resultados:
```java
lines.forEach(line -> //line es la clave para buscar en el map.
        dataToHash.computeIfAbsent(line, this::calculateDigest));//La operación a ejecutar si la clave está ausente.

private byte[] calculateDigest(String key) {//El helper que calculará un hash para la clave dada.
    return messageDigest.digest(key.getBytes(StandardCharsets.UTF_8));
}
```
Este patrón también es útil para Dealing conveniently con maps que almacenan múltiples valores. Si 
necesitás agregar un elemento a un Map<K, List<V>>, necesitás asegurarte de que la entrada haya sido
inicializada. Este patrón es verboso de implementar. Supone que querés construir una lista de 
películas para tu amigo Raphael:
```java
String friend = "Raphael";
List<String> movies = friendsToMovies.get(friend);
if(movies == null) {//Verifica que la lista haya sido inicializada.
    movies = new ArrayList<>();
    friendsToMovies.put(friend, movies);
}
movies.add("Star Wars");//agrega la pelicula
System.out.println(friendsToMovies);//{Raphael:[Star Wars]}
```
¿Cómo podés usar computeIfAbsent en su lugar? Devuelve el valor calculado después de agregarlo al 
Map si la clave no se encontró; de lo contrario, devuelve el valor existente. Podés usarlo de la 
siguiente manera:
```java
friendsToMovies.computeIfAbsent("Raphael", name -> new ArrayList<>())
        .add("Star Wars");//{Raphael: [Star Wars]}
```
El método computeIfPresent calcula un nuevo valor si el valor actual asociado con la clave está 
presente en el Map y es no nulo. Nota una sutileza: si la función que produce el valor devuelve 
null, la asignación actual se elimina del Map. Sin embargo, si necesitás eliminar una asignación, 
una versión sobrecargada del método remove es más adecuada para la tarea. Aprendés sobre este método
en la siguiente sección.

### 8.3.5 Patrones de remove
Ya conocés el método remove que te permite eliminar una entrada del Map para una clave dada. Desde 
Java 8, una versión sobrecargada elimina una entrada solo si la clave está asociada con un valor 
específico. Anteriormente, así era como implementabas este comportamiento (no tenemos nada en contra
de Tom Cruise, pero Jack Reacher 2 recibió críticas pobres):
```java
String key = "Raphael";
String value = "Jack Reacher 2";
if (favouriteMovies.containsKey(key) && Objects.equals(favouriteMovies.get(key), value)) {
    favouriteMovies.remove(key);
    return true;
}
else {
    return false;
}
```
Así es como podés hacer lo mismo ahora, lo cual tenés que admitir es mucho más directo:
```java
favouriteMovies.remove(key, value);
```
En la siguiente sección, aprendés sobre formas de reemplazar elementos y eliminar elementos de un 
Map.

## 8.3.6 Patrones de reemplazo
Map tiene dos nuevos métodos que te permiten reemplazar las entradas dentro de un Map:
- replaceAll—Reemplaza el valor de cada entrada con el resultado de aplicar una BiFunction. Este 
método funciona de manera similar a replaceAll en un List, que viste antes.
- replace—Te permite reemplazar un valor en el Map si una clave está presente. Una sobrecarga 
adicional reemplaza el valor solo si la clave está mapeada a un valor específico. 

Podrías formatear todos los valores en un Map de la siguiente manera:
```java
Map<String, String> favouriteMovies = new HashMap<>();//Tenemos que usar un map mutable ya que usaremos replaceAll.
favouriteMovies.put("Raphael", "Star Wars");
favouriteMovies.put("Olivia", "james bond");
favouriteMovies.replaceAll((friend, movie) -> movie.toUpperCase());
System.out.println(favouriteMovies);//{Olivia=JAMES BOND,Raphael=STAR WARS}
```
Los patrones de reemplazo que aprendiste funcionan con un solo Map. ¿Pero qué pasa si tenés que 
combinar y reemplazar valores de dos Maps? Podés usar un nuevo método merge para esa tarea.

### 8.3.7 Merge
Supone que te gustaría fusionar dos Maps intermedios, quizás dos Maps separados para dos grupos de 
contactos. Podés usar putAll de la siguiente manera:
```java
Map<String, String> family = Map.ofEntries(
        entry("Teo", "Star Wars"), entry("Cristina", "James Bond"));
Map<String, String> friends = Map.ofEntries(
        entry("Raphael", "Star Wars"));
Map<String, String> everyone = new HashMap<>(family);
everyone.putAll(friends);//Copia todas las entradas de friends into everyone.
System.out.println(everyone);//{Cristina=James Bond, Raphael=Star Wars, Teo=Star Wars}
```
Este código funciona como se espera siempre que no tengas claves duplicadas. Si necesitás más 
flexibilidad en cómo se combinan los valores, podés usar el nuevo método merge. Este método toma una
BiFunction para fusionar valores que tienen una clave duplicada. Supone que Cristina está tanto en 
los maps de family como de friends pero con diferentes películas asociadas:
```java
Map<String, String> family = Map.ofEntries(
        entry("Teo", "Star Wars"), entry("Cristina", "James Bond"));
Map<String, String> friends = Map.ofEntries(
        entry("Raphael", "Star Wars"), entry("Cristina", "Matrix"));
```
Entonces podrías usar el método merge en combinación con forEach para proporcionar una forma de 
Dealing with el conflicto. El siguiente código concatena los nombres de string de las dos películas:
```java
Map<String, String> everyone = new HashMap<>(family);
friends.forEach((k, v) -> 
        //Dada una clave duplicada, concatena los dos valores.
        everyone.merge(k, v, (movie1, movie2) -> movie1 + " & " + movie2));
System.out.println(everyone);//Outputs {Raphael=Star Wars, Cristina=James Bond & Matrix, Teo=Star Wars}
```
Tené en cuenta que el método merge tiene una forma bastante compleja de Dealing with nulos, como se
especifica en el Javadoc:

****`Si la clave especificada no está asociada con un valor o está asociada con null, merge la asocia
con el valor no nulo dado. De lo contrario, merge reemplaza el valor asociado con el resultado de la 
función de re-mapeo dada, o lo elimina si el resultado es null.`****

También podés usar merge para implementar verificaciones de inicialización. Supone que tenés un Map
para registrar cuántas veces se ve una película. Necesitás verificar que la clave que representa la 
película esté en el map antes de poder incrementar su valor:
```java
Map<String, Long> moviesToCount = new HashMap<>();
String movieName = "JamesBond";
long count = moviesToCount.get(movieName);
if(count == null) {
    moviesToCount.put(movieName, 1);
}
else{
    moviesToCount.put(moviename, count +1);
}    
```
Este código puede reescribirse como:
```java
moviesToCount.merge(movieName, 1L, (key, count) -> count + 1L);
```
El segundo argumento para merge en este caso es 1L. El Javadoc especifica que este argumento es "el 
valor no nulo que se fusionará con el valor existente asociado con la clave o, si no hay un valor 
existente o un valor nulo asociado con la clave, se asociará con la clave." Como el valor devuelto 
para esa clave es null, el valor 1 se proporciona la primera vez. La siguiente vez, como el valor 
para la clave se inicializó al valor de 1, la BiFunction se aplica para incrementar el conteo.
Aprendiste sobre las adiciones a la interfaz Map. Nuevas mejoras se agregaron a un primo: 
ConcurrentHashMap que aprenderás a continuación.

### Quiz 8.2
Descubrí qué hace el siguiente código y pensá qué operación idiomática podrías usar para simplificar
lo que está haciendo:
```java
Map<String, Integer> movies = new HashMap<>();
movies.put("JamesBond", 20);
movies.put("Matrix", 15);
movies.put("Harry Potter", 5);
Iterator<Map.Entry<String, Integer>> iterator = movies.entrySet().iterator();
while(iterator.hasNext()) {
    Map.Entry<String, Integer> entry = iterator.next();
    if(entry.getValue() < 10) {
    iterator.remove();
    }
}
System.out.println(movies);//{Matrix=15,JamesBond=20}
```
### Respuesta:
Podés usar el método removeIf en el entry set del map, que toma un predicado y elimina los elementos:
```java
movies.entrySet().removeIf(entry -> entry.getValue() < 10);
```
## 8.4 Improved ConcurrentHashMap
La clase ConcurrentHashMap fue introducida para proporcionar un HashMap más moderno, que también es 
amigable con la concurrencia. ConcurrentHashMap permite operaciones concurrentes de adición y 
actualización que bloquean solo ciertas partes de la estructura de datos interna. Por lo tanto, las 
operaciones de lectura y escritura tienen mejor rendimiento en comparación con la alternativa 
sincronizada de Hashtable. (Tené en cuenta que el HashMap estándar no está sincronizado.)

### 8.4.1 Reduce y Search
ConcurrentHashMap soporta tres nuevos tipos de operaciones, recordatorias de lo que viste con streams:
- forEach—Realiza una acción dada para cada (clave, valor)
- reduce—Combina todos los (clave, valor) dada una función de reducción en un resultado
- search—Aplica una función en cada (clave, valor) hasta que la función produzca un resultado no nulo

Cada tipo de operación soporta cuatro formas, aceptando funciones con claves, valores, Map.Entry, 
y argumentos (clave, valor):
- Opera con claves y valores (forEach, reduce, search)
- Opera con claves (forEachKey, reduceKeys, searchKeys)
- Opera con valores (forEachValue, reduceValues, searchValues)
- Opera con objetos Map.Entry (forEachEntry, reduceEntries, searchEntries)

Tené en cuenta que estas operaciones no bloquean el estado del ConcurrentHashMap; operan sobre los 
elementos a medida que avanzan. Las funciones proporcionadas a estas operaciones no deberían depender
de ningún orden ni de ningún otro objeto o valor que pueda cambiar mientras la computación está en 
progreso.
Además, necesitás especificar un umbral de paralelismo para todas estas operaciones. Las operaciones
se ejecutan secuencialmente si el tamaño actual del map es menor que el umbral dado. Un valor de 1 
habilita el paralelismo máximo usando el pool de hilos común. Un valor de umbral de Long.MAX_VALUE 
ejecuta la operación en un solo hilo. Generalmente deberías stickear a estos valores a menos que tu 
arquitectura de software tenga optimización avanzada de uso de recursos.
En este ejemplo, usás el método reduceValues para encontrar el valor máximo en el map:
```java
//Un ConcurrentHashMap, que se presume actualizado para contener varias claves y valores.
ConcurrentHashMap<String, Long> map = new ConcurrentHashMap<>();
long parallelismThreshold = 1;
Optional<Integer> maxValue =
Optional.ofNullable(map.reduceValues(parallelismThreshold, Long::max));
```
Notá las especializaciones primitivas para int, long, y double para cada operación de reduce 
(reduceValuesToInt, reduceKeysToLong, etc.), que son más eficientes ya que previenen el boxing.

### 8.4.2 Conteo
La clase ConcurrentHashMap proporciona un nuevo método llamado mappingCount, que devuelve el número 
de mapeos en el map como un long. Deberías usarlo para nuevo código en preferencia al método size, 
que devuelve un int. Hacer esto future proofs tu código para cuando el número de mapeos ya no cabe 
en un int.

### 8.4.3 Vistas de Set
La clase ConcurrentHashMap proporciona un nuevo método llamado keySet que devuelve una vista del 
ConcurrentHashMap como un Set. (Los cambios en el map se reflejan en el Set, y viceversa.) También 
podés crear un Set respaldado por un ConcurrentHashMap usando el nuevo método estático newKeySet.

### Resumen
- Java 9 soporta fábricas de colecciones, que te permiten crear pequeñas listas, sets y maps 
inmutables usando List.of, Set.of, Map.of, y Map.ofEntries.
- Los objetos devueltos por estas fábricas de colecciones son inmutables, lo que significa que no 
podés cambiar su estado después de la creación.
- La interfaz List soporta los métodos default removeIf, replaceAll, y sort.
- La interfaz Set soporta el método default removeIf.
- La interfaz Map incluye varios métodos default nuevos para patrones comunes y reduce el alcance de
bugs.
- ConcurrentHashMap soporta los nuevos métodos default heredados de Map pero proporciona 
implementaciones thread-safe para ellos.