---
layout: post
title: "Eventual Consistency, Parte II"
date: 2012-05-15 14:07
comments: true
categories: [Arquitectura, JavaScript, Eventual Consistency]
---

En la primera parte sobre [Consistencia Eventual (EC)](http://nnombela.com/blog/2012/02/17/eventual-consistency/), un modelo de consistencia de datos para una arquitectura distribuida,
deje sin contar dos de los aspectos más importantes de la misma: **la política de resolución de conflictos**, y también, **la política de propagación de los cambios**.

Antes de empezar, para concretar y generar un poco más de interés pongamos ejemplos de sistemas distribuidos que utilizan alguna forma de EC:

* Los sistemas de control de versiones distribuidos y muy en concreto [Git](http://git-scm.com/) que es un caso paradigmático.
* Los sistemas de almacenamiento en la nube, por ejemplo [Dropbox](http://www.dropbox.com).
* Los juegos multiusuario en tiempo real, casi todos ellos.
* Los motores de búsqueda, cualquiera de ellos.
* Algunas Aplicaciones Web como las redes sociales, pongamos [Facebook](http://facebook.com) o [Twitter](http://twitter.com)

<!-- more -->

En todos estos sistemas dos observadores del mismo pueden ver, en un momento dado, cosas distintas
pero eventualmente (en el tiempo) todos los observadores verán lo mismo (mejor dicho verán los mismos cambios).

En todos estos sistemas por tanto puede haber conflictos durante la propagación de los cambios,
y en todos existe una política de resolución de conflictos que resuelve mediante la aplicación de reglas establecidas dicho conflicto.

El modo de propagación de cambios se refiere a cómo los diferentes nodos sincronizan los datos entre si, en este punto hay mucha libertad,
la propagación puede ser de tipo push/pull a petición del usuario (como el que utiliza git), de tipo periódico (sincronización automática cada cierto intervalo de tiempo)
o puede ser dirigida por eventos como las de tipo pub/sub. Los nodos pueden tener una conexión permanente o establecer conexiones puntuales.
Las conexiones pueden ser bidireccionales o en un sólo sentido (típica arquitectura web), etc...

El modo de propagación de cambios no es importante en EC siempre y cuando se respete el principio de consistencia eventual:
**La consistencia de datos se alcanza en algún intervalo de tiempo (eventualmente)**, es decir tiene que existir propagación de cambios.

Lo que sí es importante para garantizar la consistencia de datos es **la política de resolución de conflictos**, porque la posibilidad de conflicto es el precio que hay que pagar en EC.
Un precio barato como ya comenté en la primera parte del [post](http://nnombela.com/blog/2012/02/17/eventual-consistency/), ya que es a cambio de un montón de cosas estupendas: escalabilidad, tolerancia a fallos, posibilidad de modo offline, etc...

En el caso concreto de los sistemas distribuidos de control de versiones, que probablemente todos hemos utilizado,
todos sabemos que es un conflicto y cómo se resuelve, se resuelve a *manini* y en origen por el usuario, mezclando los cambios, quizás con alguna herramienta de mezclado.

En el caso de los juegos multiusuario también puede haber conflictos, por ejemplo dos jugadores se disparan casi a la vez pero... *only one can win, my friend!*
En estos casos puede ocurrir que alguno de los jugadores vea por un momento a su oponente muerto y un instante después el que está mordiendo
el polvo sea el mismo y su oponente saliendo de rositas... *damn it!*

En el caso de las redes sociales también puede haber conflictos, podemos ver cómo después de haber comentado una noticia y
creer que somos los primeros en hacerlo y de hecho verlo de esa manera, vemos posteriormente que no así y que alguien se nos ha adelantado con la misma ocurrencia...

En el caso de los motores de búsqueda, pues ya se sabe, más que conflictos lo que puede haber es diferentes resultados para la misma búsqueda dependiendo de dónde geográficamente se realice.

En realidad todos estos casos suelen ser casos extremos, casos que no son lo habitual, esa es una de las bazas de EC,
los conflictos son excepcionales y no interfieren en el normal funcionamiento del sistema.

Con riesgo de ser pesado, repito una vez más, la existencia de un conflicto durante la propagación de cambios no implica inconsistencia de datos
ya que existe una política que determina sin ambigüedad cómo se resuelve el conflicto y
cómo se propaga esta resolución (otra cosa es que esta política sea la más acertada para el sistema en concreto)

Para quien piense que no todos los sistemas pueden admitir EC y posibilidad de conflicto, sólo un dato,
no se me ocurre nada más sensible que el código de una aplicación (pensemos en una aplicación militar o de un banco como casos extremos) y
ya hace tiempo que la política de bloqueo de recursos en los sistemas de control de versiones cayó en desgracia...

Un sistema muy sensible a la inconsistencia de datos tendrá mecanismos de control, testeo unitario, de integración, etc..
En el caso de un banco, doble y triple contabilidad, etc.. Los tendrá no por que utilice EC si no por que incluso utilizando otros
modelos de consistencia “más garantistas” necesita, aún con todo, de estos mecanismos de control de la consistencia

Veamos entonces algunas posibles políticas de resolución de conflictos:

1) **Mezclado automático, versionado y marcado como conflicto.** Resolución en cliente.
Este es el más habitual en los sistemas de control de versiones y en la mayor parte de los sistemas de almacenamiento distribuido.
Los cambios se versionan y se intentan mezclar de forma automática y si no se consigue se marcan como *en conflicto*, se guardan ambas versiones.
Al nodo origen se le avisa de la situación y se espera que de forma activa la solucionen y propaguen el cambio.

2) **Timestamp en origen.** Este es el más habitual en los juegos multijugador y uno de los más sencillos.
Todos los cambios se propagan con un timestamp en origen (que tiene que estar sincronizado en todos los nodos),
cuando existe un conflicto prevalece el cambio con mayor o menor timestamp (al gusto), se resuelve en servidor y la resolución se propaga de vuelta a los nodos origen.

3) **Prioridad en origen.** a los nodos origen (donde se originan los cambios) se les da una prioridad distinta,
en caso de que exista conflicto durante la propagación prevalece el cambio del nodo con mayor prioridad, la resolución se propaga de vuelta.

Hay muchos otras y en realidad cada sistema distribuido que quiera utilizar alguna forma de EC deberá diseñar su propia política de resolución específica de la aplicación
y que mejor resultado pueda dar. Esta política es absolutamente clave para el éxito de este tipo de modelo de consistencia.

Una cosa importante es que los conflictos se pueden resolver en todos los nodos (que tengan estado): en origen, en nodos intermedios o en los nodos de datos finales
(la consistencia se alcanza en todos los nodos), y su resolución también se propaga como otro cambio. La política de resolución y propagación puede ser distinta
dependiendo del tipo de nodo pero su aplicación tiene que garantizar la consistencia, y aquí es donde hay que ser muy cuidadosos o podemos encontrarnos con una situación de tipo ping-pong.

Lo mejor es que sea bien el nodo origen (caso de los sistemas de control de versiones) o bien el nodo final (casos de los juegos multiusuarios) quien
resuelva el conflicto lo antes posible y en el menor tiempo posible y propague dicho cambio.

Bueno, con esta segunda parte cierro este tema de momento. Es un tema que da mucho de si especialmente en estos momentos en
los que este modelo de consistencia de datos está llegando a aplicaciones modestas y con bajo presupuesto pero que tienen vocación de crecer y hacerlo muy rápidamente.
Además se empiezan a ofrecerse servicios distribuidos en la nube (también económicamente elásticos) con arquitecturas que utilizan estos
principios de consistencia (muy en particular los que ofrece Amazon y Windows Azure, en esto Google y especialmente Apple van por detŕas)

Quizás en algún momento me anime hacer algún prototipo de aplicación web con este tipo de arquitectura y con todos los elementos clave:
nodo cliente (con estado), nodo servicio (sin estado) y nodo de datos (con estado distribuido) y política de resolución de conflictos en nodo cliente por el usuario (que es la que más me gusta).

Como adelanto, hoy por hoy y para un prototipo utilizaría: para el nodo cliente utilizaría [Backbone.js](http://documentcloud.github.com/backbone/), para el servicio sin estado [Node.js](http://nodejs.org/)
(y algún módulo para servicios REST como [Restify](http://mcavage.github.com/node-restify/)) y para la base de datos [CouchDB](http://couchdb.apache.org/)
 con alguna vista Map/Reduce chula y quizás sincronización
automática de datos con los nodos cliente (hasta donde sé CouchDB es el único que ofrece esto con cosas como [PouchDB](https://github.com/mikeal/pouchdb) para el navegador y
[TouchDB-iOS](https://github.com/couchbaselabs/TouchDB-iOS) y [TouchDB-Android](https://github.com/couchbaselabs/TouchDB-Android) para dispositivos móviles).
En cualquier caso como veis todo estaría con JavaScript y JSON, y ojo al dato, serían todas tecnologías que aún no están maduras...
Esta tercera parte la dejo muy pendiente.