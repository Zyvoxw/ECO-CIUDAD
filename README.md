# EcoCiudad: Recuperación Urbana

Juego educativo de escritorio (Java + Swing) para la asignatura **Programación II**.
El jugador recolecta residuos en una ciudad contaminada, los clasifica en el
contenedor correcto (azul = papel, amarillo = plástico, verde = orgánico),
gana o pierde puntos, sube de nivel y recibe consejos ecológicos.

## Requisitos

- JDK 17 o superior (probado con JDK 21).
- Sin librerías externas: solo Java estándar y Swing.

## Compilación y ejecución

Desde la carpeta raíz del proyecto (`EcoCiudad/`):

```
javac -d bin src/modelo/*.java src/recursos/*.java src/negocio/*.java src/interfaz/*.java
java -cp bin interfaz.Main
```

En Windows (PowerShell o CMD) los mismos comandos funcionan con `\` o `/`.

## Estructura del proyecto

```
src/
├── interfaz/        → capa de presentación (Swing + dibujo 2D)
│   ├── Main.java            punto de entrada
│   ├── VentanaAcceso.java   pantalla de título con registro/login (REQ-01)
│   ├── Menu.java            menú principal (Jugar, Perfil, Consejos, Salir)
│   ├── VentanaJuego.java    pantalla de juego con HUD retro (REQ-03 a REQ-08)
│   ├── VentanaPerfil.java   perfil con rango y precisión (REQ-02)
│   ├── PanelCiudad.java     panel de dibujo del mapa (paintComponent + MouseListener)
│   ├── BotonRetro.java      botón con borde pixelado 3D estilo consola
│   ├── DialogoRetro.java    ventana emergente con marco tipo RPG
│   └── BarraRetro.java      barra de limpieza segmentada estilo barra de vida
├── recursos/        → gráficos pixel art generados por código
│   ├── Paleta.java          los 24 colores del juego (estilo 16-bit)
│   ├── Sprites.java         sprites como matrices de caracteres → BufferedImage
│   └── Animacion.java       alterna frames de un sprite con javax.swing.Timer
├── modelo/          → clases del dominio
│   ├── Mostrable.java       interfaz propia
│   ├── Residuo.java         clase ABSTRACTA base
│   ├── ResiduoPapel.java / ResiduoPlastico.java / ResiduoOrganico.java
│   ├── CatalogoResiduo.java enum que crea residuos aleatorios
│   ├── Ciudad.java, Consejo.java, Contenedor.java, Jugador.java, Nivel.java
│   ├── ClasificacionInvalidaException.java   excepción propia
│   └── DatosJugadorInvalidosException.java   excepción propia
└── negocio/         → lógica del juego
    ├── GestorJuego.java     registro/login, clasificación, niveles
    └── GestorResiduos.java  generación aleatoria de residuos
```

## Dónde se aplica cada concepto de POO (guía para la defensa)

| Concepto | Dónde está | Detalle |
|---|---|---|
| **Clase abstracta** | `modelo/Residuo.java` | Define atributos comunes y el método abstracto `getMensajeEducativo()`. No se puede instanciar directamente. |
| **Herencia** | `ResiduoPapel`, `ResiduoPlastico`, `ResiduoOrganico` | Las tres heredan de `Residuo` con `extends` y llaman a `super(...)` en su constructor. |
| **Polimorfismo (sobreescritura)** | `getMensajeEducativo()` y `toString()` en las tres subclases | Cada subclase responde distinto al mismo mensaje. |
| **Polimorfismo (referencias de superclase)** | `CatalogoResiduo.crearResiduo()` devuelve `Residuo`; `GestorResiduos.generarResiduos()` devuelve `ArrayList<Residuo>`; `Contenedor.validar(Residuo r)` | El código trabaja con el tipo padre sin saber qué subclase concreta recibe. |
| **Interfaz propia** | `modelo/Mostrable.java` | La implementan `Residuo`, `Ciudad`, `Jugador` y `Consejo` (método `mostrar()`), usado por la GUI. |
| **Excepciones propias** | `ClasificacionInvalidaException`, `DatosJugadorInvalidosException` | Heredan de `Exception`; se lanzan con `throw` en `GestorJuego` (métodos declarados con `throws`). |
| **try / catch / finally** | `VentanaAcceso.registrar()` e `iniciarSesion()` (try/catch); `VentanaJuego.depositarEn()` (try/catch/finally); `GestorJuego.registrarJugador()` (try/catch de `NumberFormatException`) | Los mensajes se muestran con `JOptionPane`, nunca stack traces. |
| **Colecciones** | `HashMap<String, Jugador>` en `GestorJuego`; `ArrayList<Residuo>` en `GestorResiduos` | Jugadores en memoria y lista polimórfica de residuos. |
| **Enum** | `CatalogoResiduo` | Catálogo de 12 residuos con fábrica de instancias y residuo aleatorio. |

## Reglas de negocio implementadas

- **REQ-01** Registro con validación: nombre no vacío ni repetido, edad numérica entre 5 y 120, contraseña mínima de 4 caracteres. Errores → `DatosJugadorInvalidosException` mostrada en la GUI.
- **REQ-02** Rango ecológico por puntaje histórico: 0–99 Novato, 100–299 Reciclador, 300–599 Guardián Verde, 600+ Eco-Héroe (`Jugador.getRango()`).
- **REQ-03** Residuos aleatorios como botones en el mapa; la cantidad crece con el nivel (`Nivel.getCantidadResiduos()`).
- **REQ-04** Barra de progreso "Limpieza de la ciudad" y fondo del mapa que pasa de gris/marrón a verde/azul según la limpieza.
- **REQ-05** Tres contenedores con colores estándar: azul (papel), amarillo (plástico), verde (orgánico).
- **REQ-06** Acierto suma los puntos del residuo; error lanza `ClasificacionInvalidaException`, resta la penalización, ensucia la ciudad y muestra un mensaje educativo.
- **REQ-07** Cápsula educativa (consejo aleatorio de `Consejo`) al completar cada recolección y desde el menú principal.
- **REQ-08** Al alcanzar la meta del nivel (`Nivel.verificar()`), diálogo "¡Nivel Superado!" y nueva zona con más residuos y meta mayor.

## Diseño visual (pixel art generado por código)

El juego usa una estética retro 8/16-bit **sin ningún archivo de imagen
externo**: todos los gráficos se generan por código al arrancar.

- **Sprites como texto**: cada sprite es una matriz de caracteres
  (`String[]`) en `recursos/Sprites.java`, donde cada carácter es un color
  de la paleta (`'N'` = negro, `'V'` = verde, `'.'` = transparente...).
  El método `crearSprite(disenio, escala)` la convierte en un
  `BufferedImage` con transparencia (`TYPE_INT_ARGB`).
- **Escalado sin suavizado**: los sprites de 16x16 se escalan x3 o x4 con
  `RenderingHints.VALUE_INTERPOLATION_NEAREST_NEIGHBOR`, así los píxeles
  se ven nítidos como en NES/Game Boy.
- **Paleta limitada**: `recursos/Paleta.java` define los 24 colores que usa
  todo el juego, con contornos oscuros de 1 píxel en los sprites.
- **Rendimiento**: cada `BufferedImage` se construye **una sola vez**
  (constantes estáticas en `Sprites`) y se reutiliza en cada repintado.
- **Sprites incluidos**: héroe (2 frames de reposo, recogiendo y
  celebrando), residuos con versión resaltada al pasar el mouse (botella,
  papel arrugado, cáscara de banana, bolsa), contenedores en 3 estados
  (normal, feliz, error con X y humo), árbol en 3 etapas, sol, nubes,
  smog, pájaro y estrellas de rango.
- **Fondo por capas (REQ-04)**: `PanelCiudad.paintComponent()` dibuja
  cielo, edificios con ventanas, río, calle y árbol; según la limpieza la
  ciudad pasa por 3 estados: contaminada (0-39 %), en recuperación
  (40-79 %) y limpia (80-100 %, con sol y pájaro volando).
- **Control con teclado**: el héroe se mueve por la ciudad con las
  flechas o WASD (`KeyListener` + un `Timer` que aplica la velocidad) y
  con ESPACIO recoge el residuo cercano o lo suelta. Si lo suelta junto a
  un contenedor se intenta la clasificación; si no, queda en el suelo. El
  residuo que está al alcance se resalta con contorno brillante y una
  flecha señala el contenedor donde caería.
- **Animaciones**: `recursos/Animacion.java` alterna 2 frames con
  `javax.swing.Timer` (héroe respirando, pájaro); el héroe alterna dos
  frames de caminata al moverse, el residuo acertado "vuela" hasta el
  contenedor y las nubes se mueven con el tick general del juego.
- **HUD y diálogos retro**: barra de limpieza con segmentos (estilo barra
  de vida), estrellas de rango, botones con relieve 3D pixelado
  (`BotonRetro`) y ventanas emergentes con marco doble tipo RPG
  (`DialogoRetro`), que reemplazan a `JOptionPane` manteniendo los
  mensajes de validación y excepciones.

## Cómo se juega

1. Regístrate (o inicia sesión) en la ventana de acceso.
2. En el menú elige **Jugar**.
3. Mueve al Eco-Héroe con las **flechas o WASD**. Acércate a un residuo
   (se ilumina cuando está al alcance) y pulsa **ESPACIO** para recogerlo.
4. Lleva el residuo hasta el contenedor donde crees que va (una flecha
   amarilla marca el que está al alcance) y pulsa **ESPACIO** para
   depositarlo. Si lo sueltas lejos de los contenedores, queda en el suelo.
5. Acierto: sumas puntos y la ciudad se limpia. Error: pierdes puntos y la ciudad se ensucia.
6. Al llegar a la meta del nivel pasas a la siguiente zona. Tu rango y precisión se ven en **Ver perfil**.
