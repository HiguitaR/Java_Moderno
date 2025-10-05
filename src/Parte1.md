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
---
```text
cat file1 file2 | tr "[A-Z]" "[a-z]" | sort | tail -3
```
---

