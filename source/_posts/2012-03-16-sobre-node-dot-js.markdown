---
layout: post
title: "Sobre Node.js"
date: 2012-03-16 12:12
comments: true
categories: [JavaScript, Node.js]
---

En el [post](http://nnombela.com/blog/2012/02/17/eventual-consistency/) anterior comenté que [Node.js](htt://nodejs.org) era la tecnología
que más rápidamente he visto crecer en poco tiempo. Realmente
a poco que uno lo siga, el fenómeno es abrumador. El sistema de paquetes de Node [npm](http://npmjs.org/) tiene cerca
de [8000 paquetes](http://search.npmjs.org/)! A pesar del poco tiempo muchos de ellos ya están siendo utilizados en producción en proyectos
reales y por empresas muy serias como Amazon, Linkedlin y Microsoft. Actualmente es el tecnología que más actividad
y crecimiento tiene en GitHub.

Un ejemplo real que pone de manifiesto este fenómeno sería [ldapjs](http://ldapjs.org/), es un LDAP hecho con Node.js en 4 días
prácticamente desde cero! A 24 horas de su publicación en twitter ya había un proyecto utilizándolo y en una semana ya
estaba integrado en varios proyectos importantes. En comparación un proyecto similar hecho en
Java [OpenDS](http://www.opends.org/) ha tardado 1 año en sacar la primera versión estable (y no es mucho tiempo) y apenas tiene
actividad. Estamos hablando de una ratio de 1 a 100 en tiempos de desarrollo, es una salvajada, desde luego es un caso
extremo y no creo que esa sea la norma, pero da una idea del dinamismo y de la velocidad a la que se está moviendo el
fenómeno Node.js que desde luego está atrapando a desarrolladores con mucho talento.

<!-- more -->

Resulta curioso que el fenómeno que protagoniza Node.js en el lado del servidor es en realidad una cara de un fenómeno
más amplio y que tiene como movimiento recíproco lo que está ocurriendo en el lado del cliente con JQuery y frameworks
MVC que tradicionalmente estaban en el servidor como Backbone.js. La capa cliente se hace cada vez más pesada
pareciéndose más a la capa servidor y por el otro lado la capa servidor se hace cada vez más ligera pareciéndose
más a la capa cliente. Curioso.

De hecho más que hablar de capas es más correcto hablar de nodos. Donde antes había dos lados (cliente y servidor)
completamente distintos en cuanto a lenguaje, lógica, modelo de programación y peso, ahora empieza a vislumbrarse
lo contrario, dos nodos, cliente y servidor muy parecidos (lenguaje, modelo de programación, tipo de lógica) y con
un peso también parecido (mayor peso en el lado cliente y menor en el lado servidor).

> Tanto es el parecido entre el nodo cliente y el nodo servidor que ya hay interesantes
iniciativas en [npm](http://npmjs.org/) como [nowjs](http://nowjs.com/) para que el acoplamiento y comunicación entre ambos nodos sea aún más directo,
transparente y sencillo.

Vamos a repasar las características de Node.js para entender un poco mejor este nuevo
modelo de programación en servidor (nuevo en cuanto a popularidad). Empecemos con la característica más sorprendente:

* **Single Threaded**. *Todo tu código (no todo el código pero sí el tuyo) corre en un único hilo de ejecución. No se vosotros
pero la primera vez que vi esto no encontré ningún aspecto positivo, después de conocerlo mejor tengo que decir que
estaba equivocado y que de hecho puede ser muy positivo en aspectos en los que la programación web más tradicional
falla continuamente, como es la complejidad de la programación multi-hilo o en la ejecución inadvertida de código
bloqueante, para entender cómo un defecto puede convertirse en virtud hay que explicar la siguiente característica.*

* **Asynchronous Event Driven**. *Se utiliza un modelo de programación asíncrona dirigida por eventos.
Este es el modelo que permite que la ejecución en un único hilo sea posible. La asincronicidad es una característica
no ideal en cualquier modelo de programación, una perdida de información, se pierden dos cosas: el intervalo de
tiempo y el orden de ejecución del código. Es el precio que hay que pagar a cambio de ganar en otros aspectos*

Veamos cuáles son estos aspectos:

1.  El código asíncrono dirigido por eventos es, en principio, código no bloqueante, entendido bloqueante como aquel
en el que el código (la CPU) no esta haciendo nada constructivo excepto esperar por alguna condición externa, típicamente
algún tipo de comunicación I/O. En realidad el
código en Node.js puede ser igualmente bloqueante, pero resulta primero, más difícil hacerlo inadvertidamente, y
segundo, resulta más catastrófico (recuerda que sólo hay un único hilo de ejecución). El programador es mucho más
consciente del problema, su estilo de programación tiende a ser completamente asíncrono y cuando no lo es en seguida
advierte el problema, no queda oculto como ocurre en la programación multi-hilo.

    He tenido la oportunidad de ver y incluso auditar bastantes aplicaciones web con problemas de rendimiento hechas en Java,
en el 100% de los casos Java como lenguaje no tenía nada que ver con el problema. El problema estaba en muchos casos
en código bloqueante: Acceso a otros Servicios Web, acceso a BBDD o acceso al sistema de ficheros, en ese orden.

2.  El código asíncrono no sólo es idealmente no bloqueante sino que como consecuencia de ello es más
eficiente (no consume tiempo de esperas) y por lo tanto su rendimiento y su eficiencia son mayorres. Además como
todo tu código corre en un único hilo no existe el “overhead” asociado al cambio de hilo.

3.  Como el código corre en un único hilo no existen, en principio, los problemas asociados al código multihilo, que
tan difíciles son de detectar y de solucionar (aunque de hecho puede haberlos igualmente). Hay menos código y es más sencillo y por lo tanto es más mantenible

4.  La programación dirigida por eventos permite una programación mucho más modular, ya no es necesario diseñar
cuidadosamente las dependencias entre módulos.  En la programación por eventos la comunicación se realiza emitiendo
eventos y registrando `listeners` de esos eventos, es un mecanismo totalmente desacoplado del diseño. El resultado es
código mucho más sencillo, más desacoplado, más modular y sobre todo con mucha más flexibilidad respecto a los
cambios de diseño, aunque también tiene sus aspecto negativos como la dificultad en la depuración.

En cierta manera el modelo de programación asíncrono, mono-hilo y dirigido por eventos sería equivalente a un modelo
altamente multi-hilo, donde cada uno de los trozos de código asíncronos o `callbacks` es un hilo de ejecución por
software o `green thread` y los eventos son mensajes que se envían y reciben por los hilos siguiendo un patrón parecido
al `Actor Model` , carece de la sofisticación del modelo implementado en Erlang o Scala, pero el principio es básicamente
el mismo, en mi opinión, y al mismo tiempo es más simple de utilizar. Este tipo de modelos están demostrando tener grandes ventajas
en el lado del servidor.

La programación asíncrona mono-hilo y dirigida por eventos no es una gran desconocida para el programador web, al
menos no para aquel que no trabaja exclusivamente en código de servidor, ya que es el modelo que se utiliza en la
programación cliente con el código JavaScript que se ejecuta dentro del navegador. Y lo ha hecho con enorme éxito y
esa es una de las razones que ha hecho este mismo modelo se haya llevado al servidor en Node.js

En el post anterior [Eventual Consistency](http://nnombela.com/blog/2012/02/17/eventual-consistency/) comentaba que el
modelo de programación de Node.js es uno de los que mejor
encajan con el modelo de datos con consistencia eventual, ahora ya podemos ver por qué es así, la consistencia eventual
implica asincronicidad y el modelo de programación asíncrono usado en Node.js es uno de los más sencillos.

Por último comentar un detalle importante sobre Node.js, utiliza JavaScript como lenguaje de programación (aunque hay
una tendencia cada vez más fuerte a utilizar CoffeScript). Existen frameworks similares en
Python [Twisted](http://twistedmatrix.com/trac/) y Ruby [Event Machine](http://rubyeventmachine.com/) y el
modelo de programación web
dirigido por eventos no es ni siquiera novedoso, pero una de las claves de la explosión de Node.js ha sido la decisión
muy acertada de su creador [Ryan Dahl](http://www.youtube.com/watch?v=jo_B4LTHi3I) de usar JavasScript como lenguaje y el
motor V8 de Chrome.
