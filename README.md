# Crear recursos inventario/objeto

1. Crear plantilla del recurso con @export (Un gdscript que extiende de Resource y tiene un class name para que aparezca en el inspector)


2. Crear un recurso (.tres) e insertarle la plantilla, listo para usar (objeto real)


3. añadir el recurso a un nodo (@export var inventario: Resource)

>>>
- El GDScript que extiende Resource es SOLO una plantilla.
- El objeto real es el .tres creado a partir de esa plantilla.
- Ese .tres es el que se exporta y se usa en nodos/managers
>>>


# Emitir señales:
~~~~gdscript
Signal nombre_señal_crear
#crea una señal en el inspector (solo en el script en el que lo escribiste)
~~~~

~~~~gdscript
nombre_de_la_señal.emit()
#se usa en donde/cuando se va a transmitir
#si el jugador presionó una tecla:
#emitimos el mensaje
~~~~

~~~~gdscript
nombre_señal.connect(nombre_de_la_funcion)
#te conectas a la señal para poder usar ‘’nombre_de_la_funcion’’ cuando
#se haya emitido el mensaje
~~~~

# Formas de guardar un nodo para usar sus funciones
~~~~gdscript
 player = get_tree().get_first_node_in_group("player")
#usado cuando el jugador está instanciado en la escena Y NO HAY MÁS DE UN JUGADOR
~~~~

>[!IMPORTANT]
>- Asegurar que está en el GRUPO que se menciona
>- Asegurar que tildaste la palomita en la escena del objeto, no en el mapa(que son diferentes)

~~~~gdscript
if area.is_in_group("player"):
	jugador_cerca = area.get_parent
#guardamos al padre del area
~~~~
