---
layout: post
title: "Groovy closures explicados con Java"
date: 2012-02-14 11:26
comments: true
categories: Groovy, Java
---

Llevando 3 años usando Grails casi de continuo, en mi trabajo es habitual que me pregunten cuánto tiempo le llevaría a un javero de toda la vida
ponerse las pilas y coger ritmo en un proyecto en Grails. Yo siempre respondo lo mismo, Grails es un framework para hacer aplicaciones
web, el javero, a veces muy a su pesar, está muy curtido en esto de probar y utilizar nuevos frameworks, de hecho encontrará Grails
especialmente sencillo y fácil de utilizar, y si es de los que ha bregado a fondo con frameworks del universo (por extenso) Spring incluso
te dirá que de sencillo que es no le parece un framework serio.

Pero el mayor obstáculo y la verdadera dificultad de trabajar con Grails está en el lenguaje que se utiliza: Groovy. Siendo un
lenguaje, Groovy te obliga a cambiar la forma de hacer las cosas y de pensar en ellas.
Afortunadamente la curva de entrada de Groovy para un javero es muy suave, no es una renuncia de Java, es casi una evolución,
al principio es habitual seguir programando igual que en Java y poco a poco ir incorparando la nueva sintáxis y la nueva forma de hacer las cosas.
El tiempo para una adaptación plena varia con la persona, pero típicamente lleva varias semanas o incluso meses, depende de las ganas que le ponga.

<!-- more -->

Y de todas las dificultades la mayor que se va a encontrar, y probablemente la última en superar, será entender y usar adecuadamente el concepto de
`Closure` en Groovy. Personalmente (es lo que a mi me ocurrió) creo que la mayor dificultad para entenderlo es que normalmente se explica
en lenguaje convencional o natural, en Inglés o en Castellano, y dicho así no se pilla.

Un ejercicio bastante didáctico, y que espero ayude a captar el concepto `Closure` más rápidamente a un javero, es intentar explicarlo en Java...

Vamos a ello, para modelar los closures vamos a utilizar dos aspectos del lenguaje Java poco usados que son: las clases anonimas y las clases internas.

De momento un `Closure` se puede modelar en Java como una interfaz, con un único método `execute()` que acepta un mapa de argumentos
y devuelve un resultado.

{% codeblock lang:java %}
interface Closure {
    Object execute(Map<String, Object> args);
}
{% endcodeblock %}


Una implementación del método `forEach()` utilizando closures sería esta

{% codeblock lang:java %}
static Object forEach(Collection <Object> collection, Closure closure) {
    Object[] array = collection.toArray();
    for (int i = 0; i <array.length; ++i) {
        Map<String, Object> args = new HashMap<String, Object>();
        args.put("index", i);
        args.put("it", array[i]);

        closure.execute(args);
    }
    return collection;
}
{% endcodeblock %}

Y un ejemplo de uso en Java del anterior código sería el siguiente

{% codeblock lang:java %}
    // First we build the a collection
    Collection<Object> collection = new ArrayList<Object>();
    collection.add("Hola");
    collection.add("Adios");

    // Then we traverse the collection
    forEach(collection, new Closure() {
        public Object execute(Map<String, Object> args) {
            System.out.println(args.get("index") + ": " + args.get("it") + " Caracola");
            return null;
        }
    });
{% endcodeblock %}

En java usamos clases anónimas para crear el closure, pero la sintaxis como puede observarse no es muy intuitiva.
En groovy el código anterior quedaría reducido a

{% codeblock lang:java %}
['Hola', 'Adios'].eachWithIndex { it, index ->
    println "$index : $it Caracola"
}
{% endcodeblock %}

> Nota: El método `eachWithIndex()` debería de llamarse `each()` a secas, ver [remove eachWithIndex](http://jira.codehaus.org/browse/GROOVY-1182)

Si esto fuese todo, los closures no serían complicados de entender pero la cosa se complica cuando tenemos en cuenta el
aspecto más interesante de los closures:
**los closures tienen acceso a las variables del contexto donde fueron creados**. Para modelar esto en Java es necesario
introducir una nueva entidad `Context`.
En java existe algo parecido a los closures en el aspecto de llevar consigo el contexto donde fueron creados, que son las inner clases, y de hecho se metieron en la especificación
final de Java con esta idea y con muchas dudas (que el tiempo a confirmado fundadas) sobre su utilidad.

Vamos a modelar el contexto con un clase `Context` (qué basicamente es un mapa de variables) y el closure con una clase interna (inner class)
que está vez no será una interfaz si no una clase abstracta. Los argumentos del método `execute()` serán en vez de un mapa un `Context`
y podemos crear los argumentos usando el método `createArgs()` del objeto `Closure`. El objeto `Context` puede tener a su vez un
contexto "padre", pudiendo formar una cadena de contextos.

{% codeblock lang:java %}
class Context {
    Context parent = null;
    Map<String, Object> variables = new HashMap<String, Object>();

    public Context(Context parent) {
        this.parent = parent;
    }

    public Context add(String name, Object object) {
        variables.put(name, object);
        return this;
    }

    // Traverse context chain
    public Object get(String name) {
        return variables.containsKey(name)? variables.get(name) :
            parent != null? parent.get(name) : null;
    }

    public abstract class Closure {
        public abstract Object execute(Context args);

        public Context createArgs() {
            return new Context(Context.this);
        }
    }
}
{% endcodeblock %}


Como podéis ver evidentemente la cosa se ha complidado.... veamos el método `forEach()` y el ejemplo de uso que prácticamente no han cambiado.

{% codeblock lang:java %}
static Object forEach(Collection <Object> collection, Context.Closure closure) {
    Object[] array = collection.toArray();
    for (int i = 0; i <array.length; ++i) {
        Context args = closure.createArgs().
            add("index", i).add("it", array[i]);

        closure.execute(args);
    }
    return collection;
}
{% endcodeblock %}

Y el ejemplo de uso en Java

{% codeblock lang:java %}
Collection<Object> collection = new ArrayList<Object>();
collection.add("Hola");
collection.add("Adios");

Context cxt = new Context(null); // No parent context
cxt.add("name", "Caracola");

forEach(collection, cxt.new Closure() {
    public Object execute(Context args) {
        System.out.println(args.get("index") + ": " +args.get("it") + " " + args.get("name"));
        return null;
    }
});
{% endcodeblock %}

Puede que la sintaxis de creación de las clases anónimas dentro de clases internas `cxt.new Closure()` no te sea familiar, no te preocupes no estás sólo,
yo tuve que mirarlo de nuevo para escribir el post...

En groovy el código anterior quedaría reducido a

{% codeblock lang:java %}
def name = 'Caracola'

['Hola', 'Adios'].eachWithIndex { it, index ->
    println "$index : $it $name"
}
{% endcodeblock %}


Espero que el post te haya servido para entender un poco mejor que son los closures de Groovy, de manera que la próxima
vez que veas un closure (recuerda que en Groovy podemos crear uno con la sintaxis simplificada de llaves cerradas `{ }`),
te preguntes ¿Cuáles son sus argumentos? y ¿Cuál es su contexto? Y sobre todo aunque se parecen mucho no lo confundas con
un método. El método tiene como contexto el objecto al que pertenece, el `Closure` tiene como contexto el ámbito donde fue creado.

