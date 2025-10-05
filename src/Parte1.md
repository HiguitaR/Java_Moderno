# ***PARTE 1***
---
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
---
```text
cat file1 file2 | tr "[A-Z]" "[a-z]" | sort | tail -3
```
---
- El cual (suponiendo que file1 y file2 contienen una sola palabra por línea) imprime las tres 
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


