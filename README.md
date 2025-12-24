# Crear recursos inventario/objeto

1. Crear plantilla del recurso con @export (Un gdscript que extiende de Resource y tiene un class name para que aparezca en el inspector)


2. Crear un recurso (.tres) e insertarle la plantilla, listo para usar (objeto real)


3. añadir el recurso a un nodo (@export var inventario: Resource)

REALIDAD:
- El GDScript que extiende Resource es SOLO una plantilla.
- El objeto real es el .tres creado a partir de esa plantilla.
- Ese .tres es el que se exporta y se usa en nodos/managers
