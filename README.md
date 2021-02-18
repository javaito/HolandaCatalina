Lo que busco mostrar con este documento es la idea que tengo que poder seguir escribiendo software abstrayendo los problemas inherentes a entornos distribuidos y con capacidades de procesamiento escalables. 

Cuando comencé con el estudio de las capacidades que brinda la nube y por mi experiencia en sistemas on-premis, descubrí que una de las grandes cualidades de las plataformas cloud modernas es la capacidad de hacer uso de hardware bajo demanda en cualquier momento. 

Cuando teníamos problemas de falta de hardware en los sistemas ya que eran una sola pieza de software corriendo en un hardware particular, teníamos que hacer un nuevo deploy de esa pieza única de software en un hardware con mas capacidad (A lo que actualmente se llama sistemas monolíticos). Debido a esta situaciones comenzamos con la modernización de uno de los softwares en los que trabajaba para que funcionaran de otra manera. Básicamente  buscábamos que este nuevo software pudiera contar con muchas maquinas en donde el mismo software estuviera funcionando pero se comportara como un solo servicio que brindaba cierta funcionalidad (Cluster). Obviamente lo que buscamos con este tipo de solución era poder hacer crecer las capacidades del servicio en general agregando nuevo hardware pero sin la necesidad de que se tuviera que cambiar todo el existente.

Uno de los desafíos que se presentó y del cual no existían muchos antecedentes era el hecho de que ese cluster no tuviera un software del tipo master y muchos esclavos, sino que todos los nodos de ese cluster fueran pares y que cada un respondiera de la misma forma ante un estimulo (Cluster comunista... ). Esto generaba un grado de complejidad muy alto en el desarrollo ya que la interacción entre ellos debía ser muy fluida para mantener un estado global que permitiera ver al cluster como una unidad. 

El hecho de no depender de un nodo master traía otra complicación extra, el codificar entendiendo que todos los nodos deberían ser stateless individualmente, esto significa que usar la memoria o disco local para almacenar información de estado era un error ya que el conjunto completo debía verse como una sola unidad, lo que desencadeno un conjunto de desafíos extra que eran; la memoria compartida, transacciones distribuidas, manejo de errores distribuidos, etc. 

Entonces comenzamos el desarrollo de este nuevo tipo de software para una de las engranajes fundamentales de la empresa en la que trabajaba. Este software básicamente debía administrar muchos miles de conexiones tcp de bajo nivel con equipos remotos generando información continuamente, que generalmente funcionaban en redes inestables por lo que había un gran número de conexiones y desconexiones todo el tiempo. La administración de estos equipos requería la posibilidad de interactuar con los equipos por parte de los usuarios para poder enviar nuevas configuraciones.  Pensamos en comenzar por aquí debido a que mientras mas crecía el número de equipos que se conectabas se requería mas hardware para esta administración y la solución era tener muchos de estos softwares corriendo en forma independiente sin el conocimiento de que existían entre ellos.

El desarrollo comenzó y pudimos hacer que este pequeño engranaje funcione con algunas de las características que habíamos planeado, otras no fueron terminadas completamente mas allá de esto este desarrollo sigue funcionando y pudo tener una escalabilidad distinta a la del resto del software que se tenía. 


## Holanda Catalina

Con la experiencia, con el nacimiento de nuevos patrones para la arquitectura del software y la construcción de equipos para ser escalables en capacidades computacionales y en recursos humanos, y la motivación de acercar la nuevas herramientas cloud a cualquier equipo de desarrollo de software, es que comencé con el a pensar en forma abstracta en cada una de las buenas prácticas asociadas a la programación orientada a objetos, para tratar de combinarlas con las nuevas tecnologías.

De esta combinación es que surge la idea de ***Holanda Catalina*** a la que defino cómo: *un conjunto de reglas y estándares que combinados generan un framework independiente del lenguaje e implementación, que brinda las siguientes características:*

 - Layers, Repositorios de implementaciones
 - Generalización del patrón [strategy](https://en.wikipedia.org/wiki/Strategy_pattern), Servicios
 - Polimorfismo distribuido
 - Comunicación binaria y asincrónica
 - Manejo de eventos distribuidos

### Layers, Repositorio de implementaciones
La idea de este framework comienza con un concepto simple, que es muy aplicado en la programación orientada a objetos como es la implementación de una [interfaz](https://en.wikipedia.org/wiki/Protocol_%28object-oriented_programming%29) y por otro lado la aplicación de un patrón del diseño orientado a objetos, el patrón [Singleton](https://en.wikipedia.org/wiki/Singleton_pattern) el cual básicamente restringe el número de instancias de una clase a uno. 
Entonces mas allá del amplio concepto de interfaces en el mundo de la programación orientada a objetos, en este caso vamos a limitar el uso de interfaces solamente para la definición de un conjunto de **métodos** agrupados por el **tipo** de la interfaz. Cada uno de estos métodos debe cumplir con las siguientes características:

 - Tiene que tener un nombre.
 - Devolver un valor tipado como resultado de su ejecución, este valor puede ser vacío.
 - Recibir un conjunto ordenado de argumentos tipados que puede ser vacío.

Todos los tipo de datos, tanto para la información devuelta por un método como cada uno de los elementos del conjunto de parámetros están restringidos a los tipos de datos definidos en el estándar de json binario  [bson](http://bsonspec.org/), esta restricción cobra más importancia cuando definamos los conceptos de distribución.

Ahora, a estas interfaces limitadas por las restricciones enumeradas anteriormente la vamos a definir como **Layer Interface**, o sea que cada una de ellas describe un **tipo de layer** distinto, por lo tanto cada implementación nombrada de un Layer Interface es una Layer por lo que vamos a definir a un **Layer** como: *toda implementación de un Layer Interface que tenga un nombre que la identifica unívocamente dentro de un ambiente definido donde esta instancia haya sido publicada.*

Teniendo en claro el concepto de Layer, podemos incluir el concepto de [Singleton](https://en.wikipedia.org/wiki/Singleton_pattern) para definir un repositorio de layers al cual se le pueda pedir en cualquier momento un layer a partir de su nombre o un conjunto de layer mediante el uso de patrones como expresiones regulares, entonces vamos a definir el **Repositorio de Implementaciones** como: *un conjunto de layers publicadas, indexadas por su tipo y después por su nombre dentro de un ambiente definido.*

Como se puede ver hablamos en varias ocasiones de la "Layers Publicadas" por que hay que dajar claro que el repositorio de layers debe tener un mecanismo explicito para la publicación de las layers de tal forma que se pueda controlar cual es el conjunto de implementaciones para un entorno especifico. Habrán notado que se repite la idea de un ambiente definido, esto se debe a que necesitamos darle un marco no infinito a alcance que tiene la publicación de los layers ya sea que es para una sola computadora en donde va a funcionar el código o para un conjunto de computadoras que tendrán conciencia de que existen layers distribuidos.

### Generalización del patrón Strategy, Servicios
Una de las características fundamentales que se buscan con este concepto de arquitectura es el de lograr una alta cohesión y un bajo acoplamiento entre todos estos componentes inclusive en ambientes distribuidos. El hecho de definir **Layer Interfaces**, **Layers** y **Layers Reporsitory** está totalmente apuntado a poder lograr este objetivo, debido a que el concepto de layer está directamente asociado al patrón [strategy](https://en.wikipedia.org/wiki/Strategy_pattern)

![Strategy pattern](https://raw.githubusercontent.com/javaito/HolandaCatalina/main/images/Strategy.svg)

Este patrón particularmente se caracteriza por dar una mejor escalabilidad, reutilización y mantenimiento de los algoritmos asociados a un estrategia, haciendo posible mantener una familia de algoritmos con características similares a una misma interfaz. 
Entonces, el concepto de layers lleva este patrón a otro nivel, extendiendo la capacidad de escalabilidad y mantenimientos brindando la posibilidad de que cada uno de las implementaciones sean independientes del lenguaje y posiblemente estén distribuidas en distintos servicios en un mismo cluster. Debido a que los layers están definidos en términos de un estándar como los es [bson](http://bsonspec.org/), podemos pensar que hacer una llamada remota desde cualquier lenguaje a la implementación de esta en otro lenguaje debería implementarse con una comunicación TCP que permita intercambiar mensajes basados en el mismo estándar, por lo que en la siguiente sección se describe un protocolo de comunicación basado en [bson](http://bsonspec.org/) con este fin.
Habiendo definido muchos conceptos y tomando la libertad de pensar en el hecho de hacer que un layer este en forma remota podemos pensar también en el concepto de **Servicio** al que vamos a definir como: *un conjunto de layers que implementa algunas interfaces, publicadas en la red en una dirección y puertos específicos*. Está definición nos permite pensar que un servicio es un subconjunto de los layers publicados en un instancia del framework corriendo que están publicados y son accesibles mediante un protocolo para ser usados desde otra instancia del framework.

### Polimorfismo distribuido
Otro importante y casi fundamental concepto de la programación orientada a objetos es el [polimorfismo](https://en.wikipedia.org/wiki/Polymorphism_%28computer_science%29), por lo que esta claro que no lo vamos a dejar al margen en esta especificación. Para dar un poco de contexto a la explicación vamos a describir de forma simplificada el concepto de polimorfismo asociado a la programación orientada a objetos. 
Entonces el polimorfismo *es la propiedad de los lenguajes orientados a objetos que permite enviar mensajes, sintácticamente idénticos a [objetos](https://en.wikipedia.org/wiki/Object_%28computer_science%29)  de distintas [clase](https://en.wikipedia.org/wiki/Class_%28computer_programming%29) siempre y cuando el objeto sepa cómo responder al mensaje que se le va a enviar.*
El polimorfismo genera un gran cantidad de propiedades muy útiles a lo hora de modelar soluciones en lenguajes de programación orientada a objetos, pero en la que vamos a centrar la posibilidad que genera de tener un mismo puntero al cual podemos asignar distintos tipos de objetos, justamente este es una de las propiedades que nos permite tener una bajo acoplamiento dentro de nuestras soluciones. 
Ahora combinando, el concepto de servicio explicado anteriormente junto con el polimorfismo podemos pensar en abstraernos de saber una variable asignada está haciendo referencia a una implementación local o remota debido a que la interfaz que implementan es la misma, por lo que nos permite pensar en un polimorfismo distribuido. Personalmente creo que es muy valiosa esta herramienta ya que nos da la posibilidad de pensar soluciones completas totalmente desacopladas y sin la necesidad de pensar si cada una de las partes de la solución son locales o remotas y si la implementación está correctamente desarrollada debería ser ajena al problema que queremos solucionar la problemática de la comunicación entre estos servicios.
Cuando hacemos referencia a una buena implementación de este polimorfismo distribuido es por que se entiende que todo lo referido a la comunicación subyacente necesaria para que esto se pueda lograr debe ser totalmente transparente al usuario del polimorfismo, para lograr esto se podría hacer una implementación basada en el patrón [proxy](https://en.wikipedia.org/wiki/Proxy_pattern), donde la implementación que intercepta la llamada debería ser la que hacer realmente la llamada por la red y devolver el resultado sin hacer que se note esta capa.



### Comunicación binaria asincrónica

### Manejo de evento distribuidos
