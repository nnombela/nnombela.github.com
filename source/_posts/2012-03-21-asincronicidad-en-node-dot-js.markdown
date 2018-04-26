---
layout: post
title: "Asincronicidad en Node.js"
date: 2012-03-21 11:53
comments: true
categories: [JavaScript, Node.js]
---

El modelo de programación de Node.js es monohilo, asíncrono y dirigido por eventos. En esta primera parte del
post veremos algo de código que nos permita entender un poco mejor que significa programar en un modelo monohilo asíncrono,
en la segunda parte de post veremos un ejemplo de programación dirigida por eventos utilizando eventos propios.

Tener un sólo hilo de ejecución para todo el código tiene varias consecuencias en nuestro código de las que hay que estar alerta, estas
son:

<!-- more -->

1.  No puede haber código bloqueante o todo el servidor quedará bloqueado y esto incluye no responder a nuevas peticiones entrantes.
 Cualquier tarea que pueda dar lugar a esperas activas o simplemente su tiempo de ejecución sea demasiado grande debe ser tratado
 de manera asíncrona. Esto incluye muy especialmente
 todas las tareas que impliquen algún tipo de comunicación I/O, salida a red, a base de datos o incluso al sistema de ficheros. Si tenemos
 alguna tarea computacionalmente intensiva deberemos intentar trocearla en varios bloques o trocearla en el tiempo (time-slice) para
 darle respiro al servidor y que pueda atender otros bloques de código, afortunadamente esto último no es nada habitual.

2. La asincronicidad implica que no sabemos cuándo ni en que orden se va a ejecutar el código, generalmente esto no es importante
 pero en ocasiones sí lo es y habrá que tenerlo en cuenta. En cuanto al cuándo lo más habitual será tener que tener en cuenta
 un posible evento de `timeout`, si estamos utilizando un modulo externo será tan simple como suscribirnos al evento, si es código
 propio utilizaremos la función `setTimeout()`. En cuanto al orden de ejecución, algo también poco habitual, si tenemos que controlarlo
 utilizaremos `callbacks` anidados o la función `process.nextTick()` para retrasar la ejecución dentro de la cola de eventos.
 Luego veremos ejemplos de utilización de estas funciones.

3. En caso de error inesperado debemos capturarlo y controlar el posible estado en que haya podido quedar la ejecución
del código. Muy especialmente en caso de haya recursos que no hayan sido liberados, haya tareas
 (bloques de código o `callbacks`) dependientes de la tarea que ha generado el error o tengamos una política de reintentos.
 Aunque si nuestro código es realmente
 asíncrono y dirigido por eventos la liberación de los recursos ocurrirá cuando salte el  correspondiente evento de `timeout` asociado al
 recurso o el `timeout` asociado a la tarea dependiente.

Llegado a este punto uno puedo pensar que programar en un modelo mohilo asíncrono es mucho más difícil que programar en un modelo
multihilo sincrono más tradicional, pero no es así por que para programar BIEN en servidor en cualquier de los dos modelos un programador
tiene que tener siempre en la cabeza y tener en cuenta todos todos los aspectos anteriores, que se resumen en:

1. Esperas y posibilidad de código bloqueante
2. Eventos de `timeouts` en la comunicación I/O
3. Capturas de errores inesperados, política de reintentos y liberación de recursos

En la programación de servidor multihilo sincrona más tradicional todos estos aspectos quedan ocultos, no forman parte
 del modelo de programación, en muchas muchísimas ocasiones no se tienen en cuenta, incluso por programadores expertos.
 Erróneamente se puede llegar a pensar que la lógica que tiene que tener todo esto en cuenta no es necesaria en un primer
 momento o es una forma de optimización prematura, además no resulta sencillo implementar la lógica de manera correcta.

> Es decir, en este tipo de modelo de programación es muy fácil o casi inevitable programar MAL código de servidor.

En cambio en la programación de servidor monohilo asíncrono dirigida por eventos todos estos aspectos están presentes de serie
 desde el princio, forman parte del modelo de programación, son mucho más evidentes y si no se tienen en cuenta el posible problema no pasa inadvertido y desde el principio
 se manifiesta con resultados a veces catastróficos como dejar el servidor bloqueado. Ademas la programación asíncrona por eventos
 resulta la manera más sencilla, correcta y desacoplada de tratar con todos los aspectos anteriores.

> Es decir, en este tipo de modelo de programación es difícil o fácilmente evitable programar MAL código de servidor.


Ahora vamos a ver un ejemplo sencillo de código bloqueante, de esperas, de la cola de eventos y de timeouts. Veamos
primero un ejemplo de código bloqueante, un "Hello World" con un bloque de código bloqueante.

{% codeblock lang:javascript %}
var http = require("http");
var util = require("util");

function writeResponse(response) {
    response.writeHead(200, {"Content-Type": "text/plain"});
    response.write("Hello World");
    response.end();
    util.log("OnRequest ended");
}

function sleepSynch(seconds, response) {
    var startTime = new Date().getTime();
    while (new Date().getTime() < startTime + 1000 * seconds) {
        // do nothing
    }
    writeResponse(response);
}

http.createServer(function(request, response) {
    util.log("OnRequest started");
    sleepSynch(10, response);   // bloque síncrono

    // sleepAsynch(10, response)   // bloque asíncrono
    // sleepAsynchWithNextTick(10, response); // bloque asíncrono con nextTick
}).listen(8888);

console.log("Listening on port 8888");
{% endcodeblock %}

`sleepSynch` es un bloque de código síncrono, espera activamente durante 10 segundos sin hacer nada, es un simple bucle
`while` que comprueba el tiempo transcurrido. Pudiera ser un bloque que realiza una tarea computacionalmente intesiva o una lectura
o escritura en disco de un fichero de gran tamaño o cualquier otra tarea bloqueante, lo importante del ejemplo es que si
lo ejecutamos veremos como nuestro servidor de "Hola Mundo después de 10 segundos" queda bloqueado y es más bien un servidor
de "Hola Mundo después de 10 segundos o mucho más". Atiende las peticiones en serie una detrás de otra.

Vamos a ver ahora este mismo ejemplo pero utilizando código asíncrono no bloqueante, todo queda igual excepto el bloque de
código bloqueante que ahora sería asíncrono

{% codeblock lang:javascript %}
function sleepAsynch(seconds, response) {
    setTimeout(function() {
        doResponse(response);
    }, 1000 * seconds);
}
{% endcodeblock %}

La función `setTimeout()` es un función nativa asíncrona, muy frecuéntemente utilizada en Node.js para controlar
tiempos de ejecución de código. Lo importante es ver que ahora nuestro servidor de "Hola Mundo después de 10 segundos" queda
desbloqueado, ahora ya es capaz de responder a todas las peticiones de forma paralela (al menos de forma efectiva)

Por último vamos a ver otra forma de conseguir la asincronicidad de forma menos óptima utizando la función `process.nextTick()`. Esta
función acepta un bloque de código o `callback` que será ejecutado una vez vaciada la cola de eventos actual, mediante esta técnica
conseguimos que el servidor no quede bloqueado y que el resto de tareas tengan oportunidad de ejecutarse.

{% codeblock lang:javascript %}
function sleepAsynchWithNextTick(seconds, response) {
    var endTime = new Date().getTime() + 1000 * seconds;

    (function check() {
        process.nextTick(function() {
            if (new Date().getTime() < endTime) {
                check();
            } else {
                doResponse(response);
            }
        });
    })()
}
{% endcodeblock %}


La función `process.nextTick()` es un técnica sencilla de conseguir asincronicidad y evitar el bloqueo del servidor, otra forma
hubiese sido utilizar un `setTimeout(0, ..)` con tiempo 0, otra forma posible forma
hubiese sido emitir un evento propio y escuchar por ese evento, aunque no está nada claro el orden de ejecución de los eventos
dentro de la cola de eventos, no he encontrado documentación al respecto...

Espero que este ejemplo simple sirva para entender un poco mejor el modelo de programación de Node.js, en la segunda parte de este
post veremos un ejemplo de programación dirigida por eventos propios






