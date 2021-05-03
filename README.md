## Contenido
- [Layers](#layers)

## Holanda Catalina

Por mi experiencia, por el nacimiento de nuevos patrones para la arquitectura del software y la construcción de equipos para ser escalables en capacidades computacionales y en recursos humanos, y la motivación de acercar la nuevas herramientas cloud a cualquier equipo de desarrollo de software. Y además convencido de que algunos de los conceptos y patrones asociados a la programación orientada a objetos logran generar soluciones basadas en muchas capas de abstracción que van ocultando los detalles de las distintas implementaciones y que uno de los objetivos principales del diseño orientado a objetos es lograr soluciones con alta cohesión y bajo acoplamiento, es que comencé a pensar la mejor forma de combinar estas buenas prácticas con el casi infinito conjunto de herramientas que provee el concepto de cloud.

De esta combinación es que surge la idea de ***Holanda Catalina*** a la que defino cómo: *un conjunto de reglas y estándares que combinados generan un framework independiente del lenguaje e implementación, que brinda las siguientes características:*

 - Layers, Repositorios de implementaciones
 - Generalización del patrón [strategy](https://en.wikipedia.org/wiki/Strategy_pattern), Servicios
 - Polimorfismo distribuido
 - Comunicación binaria y asincrónica
 - Manejo de eventos distribuidos

Entonces me gusta entender a ***Holanda Catalina***, como un proyecto de investigación basado en muchos experimentos puntuales que le dieron un marco teórico a la implementación de todas las ideas que surgieron de estos experimentos, ya que como amante de código que soy, lo primero que hice fue sentarme a codificar :(...
Con esto quiero decir que este documento creció junto con su primera implementación ([Holanda Catalina Java Framewrok](https://github.com/javaito/HolandaCatalinaFw)) en forma conjunta y cada para fue alimentado al otra continuamente.

### Layers, Repositorio de implementaciones <a name="layers"></a>
La idea de este framework comienza con un concepto simple, que es muy aplicado en la programación orientada a objetos, es la implementación de una [interfaz](https://en.wikipedia.org/wiki/Protocol_%28object-oriented_programming%29) y por otro lado la aplicación de un patrón del diseño, el patrón [Singleton](https://en.wikipedia.org/wiki/Singleton_pattern) el cual básicamente restringe el número de instancias de una clase a uno. 
Entonces mas allá del amplio concepto de interfaces en el mundo de la programación orientada a objetos, en este caso vamos a limitar el uso de interfaces solamente para la definición de un conjunto de **métodos** agrupados por el **tipo** de la interfaz. Cada uno de estos métodos debe cumplir con las siguientes características:

 - Tiene que tener un nombre.
 - Devolver un valor tipado como resultado de su ejecución, este valor puede ser vacío.
 - Recibir un conjunto ordenado de argumentos tipados que puede ser vacío.

Todos los tipo de datos, tanto para la información devuelta por un método como cada uno de los elementos del conjunto de parámetros están restringidos a los tipos de datos definidos en el estándar de json binario  [bson](http://bsonspec.org/), esta restricción cobra más importancia cuando definamos los conceptos de distribución.

Ahora, a estas interfaces definidas por las características enumeradas anteriormente la vamos a nombrar como **Layer Interface** y se define como: *Conjunto de métodos asociados a un nombre, cuyos parámetros y valor de retorno deben ser tipados y estos tipos son un subconjunto de los definidos en el estándar  [bson](http://bsonspec.org/)*. O sea que cada una de ellas describe un **tipo de layer** distinto, y cada implementación nombrada de un **Layer Interface** es una **Layer** por lo que vamos a definir a un **Layer** como: *toda implementación de un Layer Interface que tenga un nombre que la identifica unívocamente dentro de un ambiente definido donde esta instancia haya sido publicada.*

Teniendo en claro el concepto de Layer, podemos incluir el concepto de [Singleton](https://en.wikipedia.org/wiki/Singleton_pattern) para definir un repositorio de layers al cual se le pueda pedir en cualquier momento un layer a partir de su nombre o un conjunto de layer mediante el uso de patrones como expresiones regulares, entonces vamos a definir el **Repositorio de Implementaciones** como: *un conjunto de layers publicadas, indexadas por su tipo (Nombre del Layer Interface) y después por su nombre dentro de un ambiente definido.*

Como se puede ver hablamos en varias ocasiones de la "Layers Publicadas" por que hay que dajar claro que el repositorio de layers debe tener un mecanismo explicito para la publicación de las layers de tal forma que se pueda controlar cual es el conjunto de implementaciones para un entorno especifico. Habrán notado que se repite la idea de un ambiente definido, esto se debe a que necesitamos darle un marco no infinito a alcance que tiene la publicación de los layers ya sea que es para una sola computadora en donde va a funcionar el código o para un conjunto de computadoras que tendrán conciencia de que existen layers distribuidos.

### Generalización del patrón Strategy, Servicios
Una de las características fundamentales que se buscan con este concepto de arquitectura es el de lograr una alta cohesión y un bajo acoplamiento, como se mencionó anteriormente, entre todos estos componentes inclusive en ambientes distribuidos. El hecho de definir **Layer Interfaces**, **Layers** y **Layers Reporsitory** tiene como propósito poder lograr este objetivo, debido a que el concepto de layer está directamente asociado al patrón [strategy](https://en.wikipedia.org/wiki/Strategy_pattern) y no es casualidad ya que para mi es uno de los patrones que mas representan y ayudan al bajo acoplamiento entre los componentes y su alta cohesión.
En el siguiente diagrama de clases se representa la estructura del patrón strategy.

![Strategy pattern](https://github.com/javaito/HolandaCatalina/raw/main/images/Strategy.png)
En el siguiente fragmento de código podemos ver un posible uso del  patrón, el cual no es muy escalable ni de mucha utilidad, pero ejemplifica la cualidades de usar estrategias. Aquí se puede entender como usando un parámetro podríamos cambiar el resultado final del renderizado de la imagen.

``` java 
public void render(String outputType, byte[] image) {
	Renderer renderer;
	switch(outputType) {
		case "PNG": renderer = new PngRenderer(); break;
		case "TIFF" : renderer = new TiffRenderer(); break;
	}
	if(renderer != null) {
		renderer.render(image);
	}
}
```
Para dar un poco de contexto, tenemos que pensar este código dentro de un contexto que está haciendo uso de la implementación del patrón strategy.

Este patrón particularmente se caracteriza por dar una mejor escalabilidad, reutilización y mantenimiento de los algoritmos asociados a un estrategia, haciendo posible mantener una familia de algoritmos con características similares asociados a una misma interfaz. 
Entonces, el concepto de layers lleva este patrón a otro nivel, extendiendo la capacidad de escalabilidad y mantenimientos brindando la posibilidad de que cada uno de las implementaciones sean independientes del lenguaje y posiblemente estén distribuidas en distintos servicios en un mismo cluster. 
Acá se puede ver  una posible implementación del ejemplo anterior usando los componentes propuesto en donde as simple vista se ve la mejora en la implemantación ya que al usar un repositorio de layer indexados podemos hacer que esta implementación escale sin modificar el código del contexto donde se usa.
``` java 
public void render(String outputType, byte[] image) {
	//Layers es la implementación del patrón singleton donde se almacenan 
	//todos los layer publicados
	//Esta llamada pide una implementacion del layer interface llamado 
	//renderer y el nombre de la 
	//implementación es el parámetro de entrada al metodo.
	Renderer renderer = Layers.get("Renderer", outputType);
	renderer.render(image);
}
```

Debido a que los layers están definidos en términos de un estándar como los es [bson](http://bsonspec.org/), podemos pensar que hacer una llamada remota desde cualquier lenguaje a la implementación de esta en otro lenguaje debería implementarse con una comunicación TCP que permita intercambiar mensajes basados en el mismo estándar, por lo que en la siguiente sección se describe un protocolo de comunicación basado en [bson](http://bsonspec.org/) con este fin. Podemos ir un poco más allá y pensar en un mecanismo de auto-descubrimiento de estas implementaciones lo que permitiría que este esquema de renderizado, que usamos como ejemplo, crezca en formatos sin necesidad de un nuevo deploy debido a que en otras partes del cluster podrían aparecer nuevas implementaciones de la interfaz Renderer.

Habiendo definido muchos conceptos y tomando la libertad de pensar en el hecho de hacer que un layer este en forma remota podemos pensar también en el concepto de **Servicio** al que vamos a definir como: *un conjunto de layers que implementa algunas interfaces, publicadas en la red en una dirección y puertos específicos*. Está definición nos permite pensar que un servicio es un subconjunto de los layers publicados en un instancia del framework corriendo que están publicados y son accesibles mediante un protocolo para ser usados desde otra instancia del framework.

### Polimorfismo distribuido
Otro importante y casi fundamental concepto de la programación orientada a objetos es el [polimorfismo](https://en.wikipedia.org/wiki/Polymorphism_%28computer_science%29), por lo que esta claro que no lo vamos a dejar al margen en esta especificación. Para dar un poco de contexto a la explicación vamos a describir de forma simplificada el concepto de polimorfismo asociado a la programación orientada a objetos. 

Entonces el **polimorfismo** *es la propiedad de los lenguajes orientados a objetos que permite enviar mensajes, sintácticamente idénticos a [objetos](https://en.wikipedia.org/wiki/Object_%28computer_science%29)  de distintas [clase](https://en.wikipedia.org/wiki/Class_%28computer_programming%29) siempre y cuando el objeto sepa cómo responder al mensaje que se le va a enviar.*

El polimorfismo genera un gran cantidad de propiedades muy útiles a lo hora de modelar soluciones en lenguajes de programación orientada a objetos, pero en la que vamos a centrar la posibilidad que genera de tener un mismo puntero al cual podemos asignar distintos tipos de objetos, justamente este es una de las propiedades que nos permite tener una bajo acoplamiento dentro de nuestras soluciones. Se puede entender lo importante de esta propiedad en la siguiente linea del ejemplo de la sección anterior
``` java 
Renderer renderer = Layers.get("Renderer", outputType);
```
Acá se puede ver, que lo único que sabemos es que el repositorio nos va a devolver la implementación de un renderer pero no sabemos ningún detalle de su implementación por lo que es sumamente importante poder tener este tipo de punteros.

Ahora combinando, el concepto de servicio explicado anteriormente junto con el polimorfismo podemos pensar en abstraernos de saber si una variable asignada está haciendo referencia a una implementación local o remota, debido a que la interfaz que implementan es la misma, por lo que nos permite pensar en un polimorfismo distribuido. 
Se entiende lo valioso de esta herramienta ya que nos da la posibilidad de pensar soluciones completas totalmente desacopladas y sin la necesidad de preocuparnos por la posibilidad de que algunas implementaciones de la solución sean locales o remotas. Una implementación correcta de esta especificación debería proveer un repositorio de layers mientras que mantiene totalmente oculto los mecanismos y la problemática de la comunicación entre estos servicios.

Cuando hacemos referencia a una correcta implementación de este polimorfismo entendemos que todo lo referido a la comunicación subyacente necesaria para que esto se pueda lograr debe ser totalmente transparente al usuario del polimorfismo, para lograr esto se podría hacer una implementación basada en el patrón [proxy](https://en.wikipedia.org/wiki/Proxy_pattern), donde la implementación que intercepta la llamada debería ser la que hacer realmente la llamada por la red y devolver el resultado sin hacer que se note esta capa.
![Distributed Layer](https://github.com/javaito/HolandaCatalina/raw/main/images/Distributed.png)
Como se puede ver en el diagrama de secuencia tanto la llamada local como remota para el contexto donde se están usando las distintas implementaciones, son totalmente idénticas.

### Comunicación binaria asincrónica
En esta sección vamos a describir un mecanismo de comunicación basada en mensajes, binaria utilizando [bson](http://bsonspec.org/) para la serialización y desserialización del payload de cada uno de los mensajes, y de caracter asincrono.
Es muy importante entender que este protocolo de comunicación debe ser lo mas inmutable posible. Con inmutable nos referimos a que tanto la estructura de los mensajes como los mecanismos para serializar y des-serializar se mantengan en el tiempo. Obviamente esto es algo que no se puede mantener eternamente por lo que en caso de que se tenga que realizar cambios en alguna de las estructuras o en alguno de los mecanismos, es obligatorio mantener la compatibilidad hacia atrás con versiones anteriores.
Todos los detalles que tienen que ver con la necesidad de lograr un muy bajo porcentaje de cambios en el protocolo y su compatibilidad hacia atrás, tienen que ver con la alta posibilidad de que en un entorno productivo usando esta tecnología, coexistan distintas versiones de una implementación de la especificación lo que hace necesarios que el protocolo sea el mismo o mantenga la compatibilidad.

Como mencioné anteriormente la comunicación esta basada en mensajes, por lo que inicialmente vamos a definir el concepto de mensaje y cuales son los requicitos mínimos para una implementación.

 #### Mensajes
Antes de especificar el conjunto de mensajes necesarios para la comunicación entre servicios, y debido a que existen algunos atributos necesarios para todos los mensajes que se intercambien, podemos pensar en un super tipo del cual heredan cada uno de los mensajes al cual llamaremos **Message**.

Vamos a enumerar y describir los requicitos mínimos de un mensaje, siemre respetando los tipos de datos soportados el estándard [bson](http://bsonspec.org/)
 - id (UUID): Identificador único del mensaje que se ha generado, con este elemento podemos identificar al mensaje de cualquier otro mensaje y podemos mantener la trazabilidad del mismo. La única excepción, es en el caso especial de mensajes de respuesta el cual tiene el mismo id que el mensaje que origina la respuesta, esto se explica con detalle mas adelante.
 -  timestamp (entero 8 bytes): Valor del [Unix Time](https://en.wikipedia.org/wiki/Unix_time) con presición de milisegundos del momento exacto cuando se creo el mensaje.
 - originData (Document): Este documento es el payload del mensaje, donde el que origina el mensaje puede mandar un documento con toda la información necesaria para cumplir el proposito de su comunicación.
 - sessionId (UUID): Identificador de la session que genero el mensaje, al usar este campo se supone que el receptor del mensaje es capaz de reconstruir la sesion que generó el mensaje a partir de este identificador.
 - sessionBean (Document): Documento donde se serializa todos los atributos que forman parte de la sesión que geneeró el mensaje para que del lado que se recibe el mensaje no se tenga que reconstruir esta información.

Como se puede ver entre los atributos del mensaje existen; sessionId y sessionBean. La idea es que solo se envíe uno de estos dos elementos en cada mensaje pero no debería existir un mensaje que no tenga ninguno de los dos, debido a que siempre tenemos que poder identificar la sesión que ha generado el mensaje.

Como está en la descripción de la sección, la comunicación que planteamos debe ser siempre asincrónica, mas allá de que se entiende que existen muchos casos de uso en donde se requiere que la comunicación con un servicio devuelva una respuesta y sin ella no se puede continuar, o sea en forma sincrónica. Entonces vamos a emular este tipo de comportamiento al momento de hacer una llamada utilizando un esquema de listeners y timeouts para emular una respuesta sincrónica. Para poder plantear esta solución necesitamos primero definir un subtipo de mensaje que representa la respuesta a un mensaje enviado.

#### Response Message
Como ya dijimos es un subtipo de mensaje que solo agrega dos atributos a los que se mencionaron anteriormente en la definición de un mensaje, y por otro lado tiene la particularidad de que el id del mensaje de respuesta es el mismo id del mensaje que generó esa respuesta, por lo que en este único caso dos mensajes pueden tener el mismo id.

 - value (indefinido): en este campo del mensaje de respuesta se coloca la información que se quiere enviar como respuesta al mensaje que lo generó, como se puede ver el tipo de datos es indefinido, lo que significa que puede enviarse un valor con cualquiera de los tipos soportados en el estándar  [bson](http://bsonspec.org/) inclusive un valor nulo.
 - throwable (indefinido): cuando el mensaje generá un error en el servicio remoto y se quiere retornar un error como respuesta enviamos la información del error en este campo.

Entonces para emular una comunicación sincrónica para el que este tratando de enviar mensajes vamos hacer uso de una listener de la siguiente forma. 

![Asyncronous Communication](https://github.com/javaito/HolandaCatalina/raw/main/images/AsynchronousCommunication.png)
En el diagrama anterior se puede ver cómo el uso de un listener deje una espera no activa hasta que llegue otro mensaje o se cumpla el timeout, pero para el contexto donde se está usando una llamada se emula un comportamiento sincrónico, de acá se despender que la implementación para una llamada asincronica no requiere ningún tipo de listener ya que al contexto no le importa tener una respuesta de esa llamada.

#### Invocar layers remotos
Uno de los casos de uso para el que es necesario el uso de un mensaje y poder obtener una respuesta a ese mensaje, es la necesidad de realizar llamadas a Layers remotos ya que gran parte de la definición está basada en el hecho de que podemos tener Layers distribuidos.
Debido a que el caso de uso es tan importante para la implementación vamos a definir el conjunto mínimo de información necesaria para poder invocar un layer remoto.

 - layerClass (string): Nombre del layer interface que representa el tipo de layer que se quiere invocar.
 - layerName (string): Nombre de la implementacion específica a la que se va a llamar
 - methodName (string): Nombre del método especifico al que se va a llamar.
 - parameters (set): Lista de parámetros ordenados que junto al nombre, identifican unívocamente al método que se está invocando.

Queda claro que se necesita un menaje de respuesta para saber que la invocación fue correcta por mas que la implementación a la que se esta llamado no tenga una respuesta concreta, es necesario saber que la llamada se hizo.

### Manejo de evento distribuidos




