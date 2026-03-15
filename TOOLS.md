ARCHIVO: TOOLS.md

# Instrucciones de contenido:
A continuación se mapean las APIs de Django REST Framework que el agente puede consumir.

## guardar_snippet_codigo

**Descripción:** Guarda un fragmento de código útil para que el estudiante lo reutilice en sus proyectos.

**Datos que recibe:**
- `titulo` (string): Nombre para identificar el código (ej. "Conexión a PostgreSQL").
- `lenguaje` (string): Lenguaje de programación (ej. "Python").
- `codigo` (text): El bloque de código en sí.
- `materia_id` (int): El ID de la materia relacionada (opcional).

**Qué hace:**
Recibe los datos, los pasa por el serializer de Django y crea un nuevo registro en la base de datos vinculado al estudiante.

**Límites de seguridad:**
- Validar que el campo `codigo` no contenga scripts maliciosos.
- Estricto: El agente solo debe almacenar el código como texto, **NUNCA ejecutarlo**.

---

## gestionar_tarea_escolar

**Descripción:** Crea o actualiza una tarea, proyecto o examen en la agenda del estudiante.

**Datos que recibe:**
- `id` (int, opcional): El ID de la tarea si se va a actualizar. Si no se envía, crea una nueva.
- `titulo` (string): De qué trata la tarea.
- `estado` (string): "Pendiente", "En progreso" o "Completada".
- `fecha_entrega` (datetime): Cuándo se debe entregar.

**Qué hace:**
Si no recibe `id`, hace un POST para crear la tarea. Si recibe un `id`, hace un PATCH o PUT para actualizar el estado o la fecha de la tarea en la base de datos.

**Límites de seguridad:**
- **No eliminar la tarea de la base de datos sin confirmar explícitamente con el usuario.** Si el usuario dice "borra la tarea", el agente debe preguntar "¿Estás seguro de que deseas eliminar la tarea X?".

---

## generar_pregunta_repaso

**Descripción:** Inicia una sesión de estudio generando una pregunta basada en los temas de la escuela o el código guardado.

**Datos que recibe:**
- `materia_id` (int): El ID de la clase que se quiere repasar (ej. 4 para "Bases de Datos").
- `dificultad` (string): "Fácil", "Medio" o "Difícil".

**Qué hace:**
El agente consulta la API (mediante un GET) para obtener los temas, conceptos y snippets de código asociados a ese `materia_id`. Luego, formula una pregunta de repaso para el estudiante.

**Límites de seguridad:**
- Límite de tasa (Rate Limiting): No generar más de 15 preguntas por sesión para evitar saturar el servidor y al estudiante.

---

## evaluar_respuesta_repaso

**Descripción:** Recibe la respuesta del estudiante a una pregunta de repaso y la califica.

**Datos que recibe:**
- `pregunta_id` (int): El ID único de la pregunta que se está contestando.
- `respuesta_usuario` (string): La respuesta que escribió el estudiante.

**Qué hace:**
Envía la respuesta al backend. La API compara la respuesta del usuario con la respuesta correcta guardada en la base de datos y devuelve retroalimentación (Correcto/Incorrecto y el porqué). Actualiza el puntaje de estudio del alumno.

**Límites de seguridad:**
- Acción de Solo Lectura y Actualización de puntaje. No debe modificar la pregunta original ni borrar el historial de respuestas anteriores.
