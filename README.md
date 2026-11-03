# 🐶 EdadCanina · v2

Aplicación Android (Java) que calcula la edad "humana" de un perro multiplicando su edad por 7.

Esta es la **versión 2**: valida la entrada y controla los errores sin que la app se cierre.

---

## Objetivos

Al terminar sabrás:

- Detectar por qué una app se cierra (`NumberFormatException`) y evitarlo con `try/catch`.
- Validar datos de entrada: campo vacío, texto no numérico y rango de valores.
- Mostrar errores al usuario con `EditText.setError()`.
- Sustituir "números mágicos" por constantes.
- Llevar los textos a `strings.xml` y usar formatos con varios parámetros.
- Trabajar en una rama de Git distinta de `main`.

---

## Paso 0 · Prepara el proyecto

1. Hice un **fork** del repositorio y clone mi fork:

```bash
git clone https://github.com/JorgeMorenoRivero/edadCaninav1.git
```

2. Lo abrí en Android Studio para ejecutarlo en el emulador.

## Paso 1 · Rompe la app (a propósito)

Resultados de la v1 con cada entrada:

| Entrada | ¿Qué ocurre? |
|---|---|
| *(campo vacío)* | La app se cierra. `NumberFormatException: For input string: ""` |
| `abc` | La app se cierra. `NumberFormatException` |
| `3.5` | La app se cierra. `NumberFormatException` (no es un entero) |
| `0` | No se cierra, pero muestra 0 años (resultado absurdo) |
| `-4` | No se cierra, pero muestra -28 años (resultado absurdo) |
| `99999999999` | La app se cierra. `NumberFormatException` (no cabe en un `int`) |
| `400000000` | No se cierra, pero muestra -1494967296 años (desbordamiento) |

**Respuesta:** el método que lanza la excepción es `Integer.parseInt()`. El último caso no cierra la app porque `400000000` sí cabe en un `int`; lo que se desborda es la multiplicación `× 7` (2.800.000.000 supera el máximo de 2.147.483.647). Java no lanza excepción en un desbordamiento de `int`, solo da la vuelta y devuelve un número negativo.

## Paso 2 · Crea la rama de trabajo

```bash
git checkout -b v2-validacion
```

Todo el trabajo va en esta rama; `main` se queda como está.

## Paso 3 · Corrige los fallos que ya trae la v1

1. **`activity_main.xml`**: quitado el `>` que sobraba en `</TextView>>`.
2. **`strings.xml`**: corregida la falta de ortografía en `tu_edad_format` ("fuero" → "fuera").
3. **Texto a fuego**: el `android:hint="Ejemplo: 6"` ahora está en `strings.xml` como `hint_edad` y se usa con `@string/hint_edad`.
4. **`MainActivity.java`**: eliminados los tres `import` que no se usaban (`Insets`, `ViewCompat`, `WindowInsetsCompat`).

## Paso 4 · Añade los mensajes de error

```xml
<string name="error_vacio">Introduce la edad de tu perro</string>
<string name="error_no_numero">Introduce un número entero válido</string>
<string name="error_rango">La edad debe estar entre %1$d y %2$d años</string>
```

**Respuesta:** `%1$d` y `%2$d` son huecos para números enteros. El número indica el orden de los argumentos: `%1$d` se rellena con el primero (`EDAD_MINIMA`) y `%2$d` con el segundo (`EDAD_MAXIMA`).

## Paso 5 · Restringe la entrada desde el layout

Añadido al `EditText` (`@+id/age_input`):

```xml
android:inputType="number"
android:maxLength="2"
android:gravity="center"
android:importantForAutofill="no"
```

| Atributo | Para qué sirve |
|---|---|
| `inputType="number"` | Muestra el teclado numérico: sin letras, sin signo menos ni decimales. |
| `maxLength="2"` | No deja escribir más de 2 cifras. |
| `gravity="center"` | Centra el número en el campo. |
| `importantForAutofill="no"` | Evita que el sistema ofrezca autocompletar. |

Esto no sustituye a la validación en Java: el campo puede quedarse vacío y no conviene fiarse solo de la interfaz.

## Paso 6 · Sustituye los números mágicos por constantes

```java
private static final int FACTOR_CANINO = 7;
private static final int EDAD_MINIMA = 1;
private static final int EDAD_MAXIMA = 30; // ningún perro supera esta edad
```

## Paso 7 · Convierte las vistas en atributos de la clase

```java
private EditText ageEdit;
private TextView resultText;
```

En `onCreate()` se asignan sin volver a escribir el tipo:

```java
ageEdit = findViewById(R.id.age_input);
resultText = findViewById(R.id.result_text);
```

**Respuesta:** si dejara `EditText ageEdit = findViewById(...)` dentro de `onCreate()`, crearía una variable local nueva que tapa al atributo. El atributo se quedaría en `null` y `calcularEdad()` lanzaría un `NullPointerException`.

## Paso 8 · Saca la lógica del botón a un método

```java
button1.setOnClickListener(view -> calcularEdad());
```

Toda la lógica está ahora en `private void calcularEdad()`.

## Paso 9 · Crea un método para mostrar errores

```java
private void mostrarError(String mensaje) {
    ageEdit.setError(mensaje);
    ageEdit.requestFocus();
    resultText.setText("");
}
```

## Paso 10 · Valida la entrada en `calcularEdad()`

Orden de las comprobaciones:

1. Leer el texto con `trim()`.
2. Si está vacío → `error_vacio`.
3. Convertir a `int` dentro de un `try/catch (NumberFormatException e)` → `error_no_numero`.
4. Si `age < EDAD_MINIMA || age > EDAD_MAXIMA` → `error_rango`.
5. Si todo es correcto: `setError(null)`, calcular `age * FACTOR_CANINO` y mostrar el resultado.

**Respuesta:** con la edad limitada a 30, el resultado máximo es 30 × 7 = 210, muy lejos del máximo de un `int` (2.147.483.647). Así nunca se puede producir el desbordamiento.

## Paso 11 · Vuelve a probar

| Entrada | Resultado |
|---|---|
| *(vacío)* | Error: "Introduce la edad de tu perro" |
| `0` | Error: "La edad debe estar entre 1 y 30 años" |
| `31` | Error de rango |
| `1` | "Si tu perro fuera humano tendría 7 años" |
| `30` | "Si tu perro fuera humano tendría 210 años" |

**Respuesta:** para comprobar que el `try/catch` funciona, quitaría `inputType` del `EditText` un momento, escribiría `abc` y vería que aparece "Introduce un número entero válido" en vez de cerrarse la app.

## Paso 12 · Guarda y sube tu trabajo

```bash
git add .
git commit -m "Validación de la entrada y control de errores aritméticos"
git push -u origin v2-validacion
```

---

## ✅ Lista de comprobación

- [x] La app no se cierra con ninguna entrada.
- [x] Todos los textos están en `strings.xml` (ningún aviso *Hardcoded string*).
- [x] No quedan números mágicos en `MainActivity`.
- [x] `onCreate()` solo prepara la pantalla; la lógica está en `calcularEdad()`.
- [x] El trabajo está en la rama `v2-validacion`, no en `main`.
