# Sistema de diálogos dinámicos en Godot
Leer https://github.com/nathanhoad/godot_dialogue_manager para reforzar este sistema

Tipo de dialogo para este sistema:
World-bound dialogue

(diálogo ligado al mundo)

- El texto pertenece al espacio del juego

- Puede haber movimiento

- Puede haber cámara libre

- El diálogo convive con el gameplay

- NO ES DIALOGOS VISUAL NOVEL

---

> Documento pensado para **GitHub**, para que **yo del futuro** y **una IA** puedan entender rápidamente qué se hizo, por qué funciona y cómo reutilizarlo sin volver al infierno.

---

## 🎯 Objetivo del sistema

Implementar diálogos **en tiempo real** donde:

- El globo **no está fijo** en el centro.
- El globo **sigue al personaje** (Marker2D).
- El diálogo puede:
  - avanzar **manual**
  - avanzar **automáticamente** (`[next=5]`, `[next=auto]`).
- La **flecha de continuar**:
  - aparece solo cuando **se espera input**
  - **no aparece** en diálogos automáticos.
- El sistema funciona aunque:
  - el personaje se mueva
  - haya cámara
  - el texto esté tipeándose.

---

## 🧩 Piezas del sistema (arquitectura mental)

### 📄 Datos (qué se dice)

- Archivos `.dialogue`.
- `DialogueLine`.
- Definen texto, personaje, respuestas y auto-avance.
- **NO controlan la UI**.

### 🧠 Lógica (cuándo pasa)

- `DialogueManager` (autoload del plugin).
- Emite señales (`got_dialogue`, etc.).
- Decide qué línea va después.

### 🎨 Presentación (cómo se ve)

- **Copia** de `ExampleBalloon` (nunca el original del plugin).
- Vive en un `CanvasLayer`.
- Controla tamaño, posición, flecha y comportamiento visual.

---

## 📜 Regla clave #1 – El `.dialogue` NO controla la UI

El `.dialogue` solo describe **contenido y flujo**:

- texto
- personaje (`character`)
- auto-avance (`[next=5]`, `[next=auto]`)
- respuestas

👉 **Cómo se ve** eso en pantalla lo decide el **Balloon**, no el `.dialogue`.

---

## ⏱️ Auto-avance – cómo detectarlo de verdad

### ❌ Incorrecto

Buscar `[next=5]` en el texto.

### ✅ Correcto (regla real del plugin)

```
dialogue_line.time != ""  → auto-avance
dialogue_line.time == ""  → espera input
```

Esto aplica tanto a `[next=5]` como a `[next=auto]`.

---

## ➡️ La flecha de avanzar (`progress`)

### Problema

Aunque se hacía `progress.hide()`, la flecha reaparecía.

### Causa real

El `_process()` del Balloon **fuerza su visibilidad cada frame**.

👉 Cualquier `hide()` fuera de `_process()` es pisado.

---

## 📐 Regla clave #2 – La flecha se decide en `_process()`

La flecha solo debe verse cuando:

- el texto **ya terminó** de tipear
- **no hay** respuestas
- **no hay** voice
- **no hay** auto-avance

Regla lógica:

```
progress.visible =
    not dialogue_label.is_typing
    and dialogue_line.responses.size() == 0
    and not dialogue_line.has_tag("voice")
    and dialogue_line.time == "" <---- añadir
```

👉 Este fue el **fix definitivo**.

---

## 🎯 Globo que sigue al personaje

### Problema

El globo no seguía bien al personaje o se descolocaba.

### Causa

El globo está en un `CanvasLayer`:

- el `CanvasLayer` **no usa coordenadas del mundo**
- usa coordenadas de **pantalla**

---

## 🌍 Regla clave #3 – Convertir mundo → pantalla

### ❌ Incorrecto

```
mark2D.global_position
```

### ✅ Correcto

```
viewport.get_canvas_transform() * mark2D.global_position
```

Esto:

- respeta cámara
- respeta zoom
- mantiene el globo sobre el Marker2D siempre

---

## 🔔 Señales y errores comunes

### Problema

`dialogo_activo` nunca pasaba a `true`.

### Causa

La señal enviaba parámetros, pero la función no los recibía.

👉 **Regla**: la función debe añadir "(\_dialogue)"

---

## 🔄 Cuándo recalcular la posición del globo

### ❌ Incorrecto

Solo mover el globo cuando llega una línea nueva.

### ✅ Correcto

- `_cuando_haya_una_nueva_linea_de_dialogo_nueva(linea)` → decidir **quién habla**
- `_process()` → mover el globo **cada frame mientras el diálogo esté activo**

---

## 🗣️ Estado del hablante (patrón usado)

El personaje que está hablando NO se mueve directamente
cuando llega una línea nueva.

Patrón usado:
- `_cuando_haya_una_nueva_linea_de_dialogo_nueva(linea)` solo decide QUIÉN habla
- El personaje actual se guarda en una variable
- `_process()` usa ese estado para mover el globo cada frame

Esto permite que:
- el personaje se mueva
- la cámara se mueva
- el globo siga correctamente

---


## 🧱 UI – por qué no mover todo “a ojo”

Mover UI manualmente:

- funciona una vez
- se rompe con resoluciones, escalado y textos largos

### Regla clave #4 – Mandan los Containers

Usar:

- `PanelContainer`
- `MarginContainer`
- `VBoxContainer`
- `Custom Minimum Size` <---- este para cambiar el tamaño
- `Theme Overrides`

👉 La UI se adapta sola, sin posiciones mágicas.

---

## 🚫 Qué NO tocar / Qué SÍ tocar

### ❌ NO tocar

- `DialogueManager`
- el plugin original
- el `.dialogue` para lógica visual

### ✅ SÍ tocar

- copia de `ExampleBalloon`
- `_process()`
- `apply_dialogue_line()`
- estructura interna de UI

---

## 🧠 Reglas mentales rápidas

- El diálogo dice **qué pasa**, el Balloon decide **cómo se ve**.
- `CanvasLayer ≠ mundo`.
- Si algo reaparece, `_process()` manda.
- Auto-avance = `dialogue_line.time != ""`.
- Flecha solo cuando espera input.
- La UI se **estructura**, no se arrastra.

