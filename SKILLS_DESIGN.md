# SKILLS_DESIGN.md

Diseño de las dos skills personalizadas del agente Pixel.

---

## daily-log 📓

**1. ¿Qué hace?**

Convierte lo que le cuento de mi día en una entrada ordenada dentro de mi diario
de trabajo en Google Docs.

**2. ¿Qué input necesita?**

Le hablo por el chat y le cuento lo que hice, tal cual me sale, sin ordenarlo ni
darle ninguna estructura. Si no le doy suficientes detalles, me hace una única
pregunta para saber qué hice, dónde me atasqué y qué me queda pendiente.

Todo lo demás ya lo sabe por su configuración, así que no se lo repito:

- Dónde guardarlo: la carpeta de Drive donde trabaja está definida en TOOLS.md
- Con qué fecha y hora: mi zona horaria está en USER.md
- Con qué tono escribir: está en IDENTITY.md
- Cómo pensar la entrada: por SOUL.md sabe que tiene que anotar el motivo real
  de un problema y no solo lo que se vio por fuera, y que debe quedarse con lo
  importante en vez de copiar todo lo que le dije
- Qué no debe escribir nunca: contraseñas, claves ni dónde están guardadas

**3. ¿Cómo es un buen output?**

Escribe una sección por día, encabezada con la fecha, dividida en tres partes:
lo que hice, dónde me atasqué y qué sigue. No más de ocho puntos entre las tres.

El texto se añade al final del documento "Diario de trabajo", que vive en mi
carpeta de Drive. No crea un documento nuevo cada día: va escribiendo debajo,
así que el diario crece hacia abajo y puedo leer la semana seguida.

Sé que ha funcionado cuando:

- El documento aparece en mi carpeta y no suelto en cualquier sitio del Drive
- Respeta las tres secciones y no se pasa de ocho puntos
- En la parte de atascos escribe el motivo de fondo y no lo primero que se vio.
  En la prueba real le conté que me había salido un error 401 y anotó que la
  causa era que la clave de la API nunca se había llegado a guardar, en lugar
  de limitarse a repetir el número del error
- Resume de verdad: le solté más de diez cosas seguidas y me devolvió una
  entrada corta y ordenada por importancia

Sin toda esa configuración detrás, con el mismo mensaje me habría devuelto una
lista plana repitiendo lo que le dije, en un documento perdido en el Drive.

---

## repo-review 🔍

**1. ¿Qué hace?**

Revisa uno de mis repositorios de GitHub y me dice qué problemas tiene, con
especial atención a datos míos que no deberían estar publicados.

**2. ¿Qué input necesita?**

Solo el nombre del repositorio o su enlace. Nada más.

Lo que ya sabe sin que se lo tenga que decir:

- Que únicamente puede mirar: no cambia archivos, no crea ramas y no abre nada
  por su cuenta
- Qué cuenta como un riesgo: por su configuración sabe que el nombre de mi
  servidor, mis identificadores y las rutas donde guardo claves no pintan nada
  en un repositorio público
- Cómo contármelo: tiene que separar lo que ha comprobado de lo que se está
  imaginando, y no opinar sobre archivos que no ha abierto

**3. ¿Cómo es un buen output?**

Me responde en el chat con tres apartados: lo que está bien (como mucho tres
cosas, y solo las que no saltan a la vista), los problemas, y los riesgos. Cada
problema viene acompañado de qué hacer para arreglarlo.

La respuesta se queda en el chat y no genera ningún documento. Una revisión se
lee una vez y se actúa; si la guardara, tendría el Drive lleno de informes que
dejan de ser ciertos en cuanto toco el repositorio.

Sé que ha funcionado cuando:

- Cada problema trae su solución, y no solo el diagnóstico
- Encuentra cosas que yo no había visto. En la prueba real se dio cuenta de que
  un archivo seguía publicado con el nombre de mi servidor y mi identificador de
  Telegram, y de que otro tenía anotado el puerto por el que entro por SSH
- No toca nada por su cuenta: me ofreció preparar una limpieza y se quedó
  esperando a que yo se lo pidiera
- Cuando no encuentra riesgos, lo dice claramente en lugar de dejar el apartado
  en blanco, que parecería que no ha mirado