Lo que busco mostrar con este documento es la idea que tengo que poder seguir escribiendo software abstrayendo los problemas inherentes a entornos distribuidos y con capacidades de procesamiento escalables. 

Cuando comencé con el estudio de las capacidades que brinda la nube y por mi experiencia en sistemas on-premis, descubrí que una de las grandes cualidades de las plataformas cloud modernas es la capacidad de hacer uso de hardware bajo demanda en cualquier momento. 

Cuando teníamos problemas de falta de hardware en los sistemas ya que eran una sola pieza de software corriendo en un hardware particular, teníamos que hacer un nuevo deploy de esa pieza única de software en un hardware con mas capacidad (A lo que actualmente se llama sistemas monolíticos). Debido a esta situaciones comenzamos con la modernización de uno de los softwares en los que trabajaba para que funcionaran de otra manera. Básicamente  buscábamos que este nuevo software pudiera contar con muchas maquinas en donde el mismo software estuviera funcionando pero se comportara como un solo servicio que brindaba cierta funcionalidad (Cluster). Obviamente lo que buscamos con este tipo de solución era poder hacer crecer las capacidades del servicio en general agregando nuevo hardware pero sin la necesidad de que se tuviera que cambiar todo el existente.

Uno de los desafíos que se presentó y del cual no existían muchos antecedentes era el hecho de que ese cluster no tuviera un software del tipo master y muchos esclavos, sino que todos los nodos de ese cluster fueran pares y que cada un respondiera de la misma forma ante un estimulo (Cluster comunista... ). Esto generaba un grado de complejidad muy alto en el desarrollo ya que la interacción entre ellos debía ser muy fluida para mantener un estado global que permitiera ver al cluster como una unidad. 

El hecho de no depender de un nodo master traía otra complicación extra, el codificar entendiendo que todos los nodos deberían ser stateless individualmente, esto significa que usar la memoria o disco local para almacenar información de estado era un error ya que el conjunto completo debía verse como una sola unidad, lo que desencadeno un conjunto de desafíos extra que eran; la memoria compartida, transacciones distribuidas, manejo de errores distribuidos, etc. 

Entonces comenzamos el desarrollo de este nuevo tipo de software para una de las engranajes fundamentales de la empresa en la que trabajaba. Este software básicamente debía administrar muchos miles de conexiones tcp de bajo nivel con equipos remotos generando información continuamente, que generalmente funcionaban en redes inestables por lo que había un gran número de conexiones y desconexiones todo el tiempo. La administración de estos equipos requería la posibilidad de interactuar con los equipos por parte de los usuarios para poder enviar nuevas configuraciones.  Pensamos en comenzar por aquí debido a que mientras mas crecía el número de equipos que se conectabas se requería mas hardware para esta administración y la solución era tener muchos de estos softwares corriendo en forma independiente sin el conocimiento de que existían entre ellos.

El desarrollo comenzó y pudimos hacer que este pequeño engranaje funcione con algunas de las características que habíamos planeado, otras no fueron terminadas completamente mas allá de esto este desarrollo sigue funcionando y pudo tener una escalabilidad distinta a la del resto del software que se tenía. 


## Holanda Catalina

Con la experiencia, con el advenimiento de nuevos patrones para la arquitectura del software y la construcción de equipos para ser escalables en capacidades computacionales y en recursos humanos y la motivación de acercar la nuevas herramientas cloud a cualquier equipo de desarrollo de software, es que comencé con el a pensar en forma abstracta en cada una de las buenas prácticas asociadas a la programación orientada a objetos, para tratar de combinarlas con las nuevas tecnologías.

De esta combinación es que surge la idea de ***Holanda Catalina*** a la que defino cómo: *un conjunto de reglas y estándares que combinados generan un framework independiente de la tecnología, que brinde las siguientes características:*

 - Layers, Repositorios de implementaciones
 - Generalización del patrón [strategy](https://en.wikipedia.org/wiki/Strategy_pattern) 
 - Polimorfismo distribuido
 - Comunicación binaria y asincrónica
 - Manejo de eventos distribuidos

### Layers, Repositorio de implementaciones
La idea de este framework comienza con un concepto simple, que es muy aplicado en la programación orientada a objetos, la implementación de una [interfaz](https://en.wikipedia.org/wiki/Protocol_%28object-oriented_programming%29) y por otro lado la aplicación de un patrón del diseño orientado a objetos, el patrón [Singleton](https://en.wikipedia.org/wiki/Singleton_pattern) el cual básicamente restringe el número de instancias de una clase a uno. 
Entonces mas allá del amplio concepto de interfaces en el mundo de la programación orientada a objetos, en este caso vamos a limitar el uso de interfaces solamente para la definición de un conjunto de **métodos** agrupados por el **tipo** de la interfaz. Cada uno de estos métodos debe cumplir con las siguientes características:

 - Tiene que tener un nombre.
 - Puede o no devolver información como parte de su ejecución.
 - Recibir un conjunto ordenado de argumentos tipados que puede ser vacío.

Todos los tipo de datos, tanto para la información devuelta por un método como cada uno de los elementos del conjunto de parámetros están restringidos a los tipos de datos definidos en el estanda de json binario  [bson](http://bsonspec.org/), esta restricción cobra más importancia cuando definamos los conceptos de distribución.

Ahora, a estas interfaces limitadas por las restricciones enumeradas anteriormente la vamos a definir como **Layer Interface**, o sea que cada una de ellas describe un **tipo de layer** distinto, o sea que cada implementación nombrada de un Layer Interface es una Layer por lo que vamos a definir a un **Layer** como: *toda implementación de un Layer Interface que tenga un nombre que la identifica unívocamente dentro de un ambiente definido donde esta instancia haya sido publicada.*

Teniendo en claro el concepto de Layer, podemos incluir el concepto de [Singleton](https://en.wikipedia.org/wiki/Singleton_pattern) para definir un repositorio de layers al cual se le pueda pedir en cualquier momento un layer a partir de su nombre o un conjunto de layer mediante el uso de patrones como expresiones regulares, entonces vamos a definir el **Repositorio de Implementaciones** como: *un conjunto de layers publicadas, indexadas por su tipo y después por su nombre.*

Como se puede ver hablamos en varias ocasiones de la "Layers Publicadas" por que hay que dajar claro que el repositorio de layers debe tener un mecanismo explicito para la publicación de las layers de tal forma que se pueda controlar cual es el conjunto de implementaciones para un entorno especifico. 

### Generalización del patrón Strategy
Una de las características fundamentales que se buscan con este concepto de arquitectura es el de lograr una alta cohesión y un bajo acoplamiento entre todos estos componentes inclusive en ambientes distribuidos. El hecho de definir **Layer Interfaces**, **Layers** y **Layers Reporsitory** está totalmente apuntado a poder lograr este objetivo.

### Polimorfismo distribuido

### Comunicación binaria asincrónica

### Manejo de evento distribuidos
