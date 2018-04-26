---
layout: post
title: "¿Cuánto de OOP tiene Javascript?"
date: 2012-02-12 14:44
comments: true
categories: [JavaScript, OOP]
---
JavaScript se define como un lenguaje multi-paradigma, que es otra forma de decir que no es un lenguaje puro en el
sentido de seguir un único paradigma de programación. Ciertamente JavaScript es una mezcla de muchas cosas, aunque la raíz es
para mi, sin duda, la Programación Funcional (FP). Tomando como base este paradigma su creador tomo prestados
conceptos y estilos de programación de otros lenguajes.

Los lenguajes funcionales tienden a tener un estilo de programación declarativo, pero el creador de JS quiso preferenciar
la sintaxis imperativa C-like común en muchos lenguajes (incluído Java), quizás para hacer más accesible el
lenguaje, en cualquier caso, en JS es habitual ver ambos estilos mezclados. Siguiendo con esta extraña mezcla
decidió, cómo no, añadir características de programación orientada a objectos (OOP en inglés) que tan de moda estaba
en aquellos tiempos y añadió dos conceptos básicos de OOP: La definición de objeto y la herencia, y no mucho
más... y además JS lo hace de un modo muy particular, veamos cómo:

<!-- more -->

1.  **Definición de objeto.** *En JS se define un objeto como una agrupación desestructurada de propiedades/valor, nada más, un
mapa de propiedades para entendernos. Es una definición extremadamente simple y es más una estructura de datos que un
concepto OOP riguroso. No hay encapsulación. Algunos de estas propiedades serán funciones que son asignadas al objeto
dinámicamente y éstas funciones pueden usarse como métodos del objeto, en este caso la función recibe un argumento implícito "this"
cuyo valor es el objeto al que pertenece la función. Los objetos son creados utilizando, cómo no, una función constructora (que también es un objeto).*

2.  **Herencia prototípica.** *La herencia se consigue utilizando prototipos, que no son más que objetos (gran acierto)
cuyas propiedades son compartidas por una "familia" de objetos creados utilizando la misma función constructora
(otro objeto). El prototipo de un objeto no es más que una de sus propiedades (no enumerable), que además puede ser
modificada posteriormente. A su vez los prototipos, como objetos que son, tienen su propio prototipo dando lugar a lo
que se denomina cadena prototípica.*

En JavaScript las funciones son objetos y toda la semántica OOP gira entorno al concepto de objeto y al de función como objeto.
Es en esta mezcla tan acertada y equitativa entre los conceptos función y objeto, donde está la magia de JavaScript, al menos en mi
opinión. Otros lenguajes más OOP puros subordinan el concepto de función al de método del objeto, quedando así su
importancia relegada y mal aprovechada... pero esto ya va más sobre gustos que otra cosa.

Sin entrar en mucho más detalle, no pretendo que de la explicación anterior quede claro cómo hacer herencia OOP en JS
pero sí la idea de que su implementación en el lenguaje está hecha de una forma muy simple, bastante flexible y en mi
opinión (muy discutible) de una forma elegante y coherente, aunque el creador de JS descuidó, y mucho, los detalles (quizás por
las prisas), y ya se sabe que las prisas son malas y que en los detalles se esconde el diablo...

Esta simplicidad y flexibilidad de los conceptos OOP es uno de los grandes aciertos de JS pero también una de sus
desventajas, ya que no resulta nada obvio cómo programar siguiendo un estilo OOP en JavaScript... pero en parte también
esto puede ser una ventaja ya que OOP puede no ser la forma más adecuada para resolver un problema de programación. OOP no es un
estilo imperativo en JS, es una opción, casi un patrón de diseño más... y la dificultad de uso es una forma del
lenguaje, un tanto sutil, de decirle al programador que si no tiene claro como hacerlo es por que, probablemente ¡No
lo necesita! Y en muchas ocasiones esto es cierto.

Y bueno hecha la introducción veamos un par de formas para hacer OOP en Javascript.

### A) Al modo funcional:   *Un par de funciones extend() y inherits().*

{% gist 1768497 %}

**Para el 80% de los casos**, extender (extend) un objeto utilizando otro objeto, es probablemente la forma más fácil y en
muchas ocasiones la más acertada de tener “herencia” sin hacer herencia, también se denomina “mezclar con” (mixin) o
aumentar (augment) un objeto con otro objeto, está será nuestra primera opción a considerar, su uso es trivial.

**Para el 20% de los restantes casos**, si pensándolo bien finalmente decidimos que lo que queremos es herencia pura y dura,
utilizando la función anterior lo haríamos así:

{% gist 1789034 %}

Lo que me gusta de esta sintaxis es que es muy limpia e intuitiva, se ve claramente lo que estás haciendo y ves al
mismo tiempo y de forma agrupada la función constructora, las propiedades compartidas y los métodos.

Especial atención a cómo se accede a los métodos y propiedades de la “clase padre” desde la “clase hija” .

        Child.parent.prototype.property;
        Child.parent.prototype.functionName.call(this, args..);

Podríamos haber utilizado perfectamente (y quizás más correctamente) una versión más corta

        Parent.prototype.property;
        Parent.prototype.functionName.call(this, args..);

Pero he preferido utilizar la propiedad añadida (no enumerable) `parent` de la función constructora por que es habitual
añadir los métodos posteriormente, una vez hecha la definición, y en estos casos es conveniente no tener que recordar y hacer mención
explicita a la función padre,  de cualquier forma la sintaxis es (discutiblemente) engorrosa, en la siguiente sección veremos una posible alternativa
más sencilla y un poco más OOP (pero con inconvenientes que explicaré).
___
<br/>

### B) A un modo más OOP:  *Utilizando modularización y encapsulación.*
Esta opción es una evolución de la anterior pero con un enfoque más orientado a objectos, en vez de utilizar funciones
haremos que las las funciones constructoras extiendan de un objeto padre (OOP) que tiene como métodos
el método `extend()` para herencia y el método `augment()` para extender (ya se que el cruce de nombres es un poco confuso
pero las terminologías FP y la OOP se solapan y tienen estas cosas, lo siento). El código además se presenta como una
librería modularizada (con soporte CommonJS)

{% gist 1768435 %}

Como ejemplo de uso de la librería anterior tenemos.

{% gist 1792721 %}

El ejemplo es parecido al anterior en A) pero en vez de funciones utilizamos métodos (funciones de un objeto) y además he añadido una forma un
poco más amigable de poder llamar a los metodos de la “clase padre” (prototipo de la función constructora), de manera
que en vez de

        ColorPoint.parent.prototype.constructor.call(this, x, y);
        ColorPoint.parent.prototype.toString.call(this);

Podemos, si queremos, usar de manera equivalente:

        this.parent('constructor', x, y);
        this.parent('toString');

Esta segunda posibilidad aunque más intuitiva y sencilla tiene un inconveniente importante, las funciones método sólo
podrán ser utilizadas ligándolas a objetos del
"mismo tipo" (misma función constructora) al objeto al que pertenecen, es decir no podrán utilizarse como funciones desligadas
con objetos que en su cadena prototípica no tengan la función `parent()` con la misma semántica, esto nos aleja de una programación funcional
y nos acerca más a la programación OOP, pero bueno es opcional y dependiendo de nuestro diseño no tiene por qué ser negativo.

Para facilitar la operación de ligar una funcion con el objeto al que pertenece he includo un método no enumerado
dentro de la cadena prototípica que se llama `method()`, podéis ver su uso en el ejemplo para generar una función
callback como argumento de un `setTimeout()`. Todo el tema de funciones, métodos y funciones ligadas es muy importante
en JS y uno de los que más confusión genera especialmente al principio si se viene de programar en Java donde esta
problemática no existe, como este post ya es bastante largo lo dejo para uno futuro.

> **Un aviso,** la herencia es probablemente el recurso más incorrectamente utilizado en programación OOP, especialmente
en aquellos lenguajes más OOP puros que incitan a utilizarlo. Los diseños evolutivos son los que más padecen el uso
o abuso de este recurso ya que la herencia es la decisión que más rigidez y difulcultad de cambio aportan a un diseño
(si eres de los que ha utilizado herencia con profusión en un modelo de datos sabrás de lo que estoy
hablando). Java se intentó proteger de este abuso no permitiendo la herencia múltiple y favoreciendo más el uso de
interfaces, aún con todo incluso en las librerías estándar, especialmente en las más antiguas, es fácil ver este
problema. Ver [Inheritance as antipattern](http://asserttrue.blogspot.com/2009/02/inheritance-as-antipattern.html), asi que... <br/>
**Use it at your own risk!**

JavaScript es un lenguaje muy incomprendido... espero que este post os haya servido para entenderlo un poco mejor o
al menos haber despertado vuestro interés. Intentaré seguir en ello.