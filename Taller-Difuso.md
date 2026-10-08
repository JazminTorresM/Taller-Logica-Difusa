# Taller práctico: Lógica Difusa en Python

![Python](https://img.shields.io/badge/Python-3.x-blue) ![Colab](https://img.shields.io/badge/Google-Colab-orange) ![Nivel](https://img.shields.io/badge/Nivel-b%C3%A1sico-green) ![Duraci%C3%B3n](https://img.shields.io/badge/Duraci%C3%B3n-20%20min-lightgrey)

Material de apoyo de la exposición **Fuzzy Logic** (*Fundamentos de IA*).

> [!NOTE]
> Este taller es una **demostración**: el equipo ejecuta el código y los demás solo observan cómo funciona. Todo lo marcado como *opcional* es para quien quiera probarlo por su cuenta.
>
> Tampoco se trabajan funciones de membresía: ya se explicaron en la presentación y aquí vienen definidas en el código.

## Contenido

1. [¿Qué vamos a hacer?](#qué-vamos-a-hacer)
2. [Antes de empezar](#antes-de-empezar)
3. [Ejercicio 1: operaciones difusas](#ejercicio-1-operaciones-difusas)
4. [Ejercicio 2: ¿cuánta propina dejo?](#ejercicio-2-cuánta-propina-dejo)
5. [Reto opcional](#reto-opcional)
6. [Código completo](#código-completo)
7. [Problemas frecuentes](#problemas-frecuentes)

---

## Qué vamos a hacer

| | Ejercicio | Tema de la presentación | Tiempo |
|---|---|---|---|
| 1 | Unión, intersección y complemento con 3 líneas de Python | Operaciones difusas | 5 min |
| 2 | Un sistema que decide el porcentaje de propina | Inferencia difusa (4 fases) | 15 min |

---

## Antes de empezar

> [!IMPORTANT]
> Esta sección es **opcional**: solo es necesaria si quieres ejecutar el código en tu propia computadora. Para seguir la demostración no tienes que instalar nada.

Elige **un** entorno y marca cada paso conforme lo termines.

<details>
<summary><b>Opción A: Google Colab (no se instala nada)</b></summary>

- [ ] Abre [colab.research.google.com](https://colab.research.google.com) e inicia sesión con tu cuenta de Google.
- [ ] Clic en **Nuevo cuaderno**.
- [ ] En la primera celda escribe esto y presiona ▶:
  ```python
  !pip install scikit-fuzzy
  ```
- [ ] Espera a que termine (unos segundos).

Cada bloque de código del taller va en una **celda nueva** (botón **+ Código**).

</details>

<details>
<summary><b>Opción B: Terminal de Linux (Ubuntu/Debian)</b></summary>

- [ ] Instala Python y dependencias:
  ```bash
  sudo apt update
  sudo apt install python3 python3-pip python3-venv python3-tk
  ```
- [ ] Crea una carpeta y un entorno virtual:
  ```bash
  mkdir taller-fuzzy && cd taller-fuzzy
  python3 -m venv entorno
  source entorno/bin/activate
  ```
  Debe aparecer `(entorno)` al inicio de la línea.
- [ ] Instala las librerías:
  ```bash
  pip install scikit-fuzzy numpy scipy networkx matplotlib
  ```
- [ ] Crea el archivo donde pegarás el código:
  ```bash
  nano taller.py
  ```
  Guardar: `Ctrl+O`, `Enter`. Salir: `Ctrl+X`.

Al terminar los ejercicios se ejecuta con `python taller.py`.

</details>

---

## Ejercicio 1: operaciones difusas

Aquí no hace falta ninguna librería. En lógica difusa el **OR** es el máximo, el **AND** es el mínimo y el **NOT** es `1 - valor`.

**Paso 1.1.** Copia y ejecuta:

```python
# Grados de pertenencia (entre 0 y 1)
alto = 0.8
pesado = 0.4

print("OR  (unión):        ", max(alto, pesado))
print("AND (intersección): ", min(alto, pesado))
print("NOT (complemento):  ", round(1 - alto, 2))
```

**Resultado esperado:**

```
OR  (unión):         0.8
AND (intersección):  0.4
NOT (complemento):   0.2
```

**Paso 1.2 (opcional).** Si quieres practicar, cambia los valores de `alto` y `pesado` y adivina el resultado antes de ejecutar.

---

## Ejercicio 2: ¿cuánta propina dejo?

Un restaurante quiere sugerir el porcentaje de propina según dos datos:

- **Entradas:** calidad de la comida y del servicio, de 0 a 10.
- **Salida:** propina, de 0 a 25 %.

### Paso 2.1: importar las librerías

```python
import numpy as np
import skfuzzy as fuzz
from skfuzzy import control as ctrl
import matplotlib.pyplot as plt
```

### Paso 2.2: crear las variables lingüísticas

Aquí se pasa de números a palabras (*mala, regular, buena*). Las etiquetas de entrada se generan automáticamente con `automf`.

```python
comida   = ctrl.Antecedent(np.arange(0, 11, 1), 'comida')
servicio = ctrl.Antecedent(np.arange(0, 11, 1), 'servicio')
propina  = ctrl.Consequent(np.arange(0, 26, 1), 'propina')

comida.automf(names=['mala', 'regular', 'buena'])
servicio.automf(names=['mala', 'regular', 'buena'])

propina['baja']  = fuzz.trimf(propina.universe, [0, 0, 13])
propina['media'] = fuzz.trimf(propina.universe, [0, 13, 25])
propina['alta']  = fuzz.trimf(propina.universe, [13, 25, 25])
```

### Paso 2.3: escribir las reglas SI-ENTONCES

```python
r1 = ctrl.Rule(servicio['mala'] | comida['mala'], propina['baja'])
r2 = ctrl.Rule(servicio['regular'], propina['media'])
r3 = ctrl.Rule(servicio['buena'] | comida['buena'], propina['alta'])
```

| Regla | Se lee como |
|---|---|
| `r1` | SI el servicio es malo **O** la comida es mala, ENTONCES propina baja |
| `r2` | SI el servicio es regular, ENTONCES propina media |
| `r3` | SI el servicio es bueno **O** la comida es buena, ENTONCES propina alta |

El `|` es el OR del Ejercicio 1.

### Paso 2.4: probar con un caso

```python
sistema = ctrl.ControlSystemSimulation(ctrl.ControlSystem([r1, r2, r3]))

sistema.input['comida'] = 6.5
sistema.input['servicio'] = 9.8
sistema.compute()

print("Propina sugerida:", round(sistema.output['propina'], 2), "%")
```

**Resultado esperado:**

```
Propina sugerida: 19.85 %
```

### Paso 2.5: ver cómo decidió

```python
propina.view(sim=sistema)
plt.savefig("propina.png")
plt.show()
```

El área sombreada es la combinación de las reglas activadas, y la línea roja es el número final.

### ¿Dónde ocurre cada fase de la presentación?

| Fase | En el código |
|---|---|
| 1. Fuzzificación | Se asignan los valores en `sistema.input[...]` |
| 2. Evaluación | Se aplican las reglas `r1`, `r2`, `r3` |
| 3. Agregación | Ocurre dentro de `sistema.compute()` |
| 4. Defuzzificación | Se obtiene `sistema.output['propina']` |

---

## Reto opcional

Solo para quien quiera practicar por su cuenta. Cambia los números del Paso 2.4, ejecuta de nuevo y completa la tabla:

| Comida | Servicio | Tu resultado | Referencia |
|:---:|:---:|:---:|:---:|
| 2 | 3 | | 10.84 % |
| 5 | 5 | | 12.67 % |
| 9 | 9 | | 16.81 % |
| 2 | 9 | | 13.06 % |

**Pregunta para pensar:** en el último caso la comida es mala y el servicio excelente. ¿Por qué el sistema no da ni la propina más baja ni la más alta?

**Extra (opcional):** agrega una regla nueva, por ejemplo `ctrl.Rule(comida['regular'] & servicio['regular'], propina['media'])`. El `&` es el AND del Ejercicio 1.

---

## Código completo

Todo el taller junto en un solo bloque. En Colab va en una sola celda (después de `!pip install scikit-fuzzy`); en Linux se guarda como `taller.py`.

```python
import numpy as np
import skfuzzy as fuzz
from skfuzzy import control as ctrl
import matplotlib.pyplot as plt

# ---------- EJERCICIO 1: Operaciones difusas ----------
alto = 0.8
pesado = 0.4

print("OR  (unión):        ", max(alto, pesado))
print("AND (intersección): ", min(alto, pesado))
print("NOT (complemento):  ", round(1 - alto, 2))

# ---------- EJERCICIO 2: Inferencia difusa (propina) ----------
# Variables lingüísticas
comida   = ctrl.Antecedent(np.arange(0, 11, 1), 'comida')
servicio = ctrl.Antecedent(np.arange(0, 11, 1), 'servicio')
propina  = ctrl.Consequent(np.arange(0, 26, 1), 'propina')

comida.automf(names=['mala', 'regular', 'buena'])
servicio.automf(names=['mala', 'regular', 'buena'])

propina['baja']  = fuzz.trimf(propina.universe, [0, 0, 13])
propina['media'] = fuzz.trimf(propina.universe, [0, 13, 25])
propina['alta']  = fuzz.trimf(propina.universe, [13, 25, 25])

# Reglas SI-ENTONCES
r1 = ctrl.Rule(servicio['mala'] | comida['mala'], propina['baja'])
r2 = ctrl.Rule(servicio['regular'], propina['media'])
r3 = ctrl.Rule(servicio['buena'] | comida['buena'], propina['alta'])

sistema = ctrl.ControlSystemSimulation(ctrl.ControlSystem([r1, r2, r3]))

# Caso principal (cámbienlo para experimentar)
sistema.input['comida'] = 6.5
sistema.input['servicio'] = 9.8
sistema.compute()
print("Propina sugerida:", round(sistema.output['propina'], 2), "%")

# Gráfica del resultado
propina.view(sim=sistema)
plt.savefig("propina.png")
plt.show()

# ---------- RETO: los 4 casos de la tabla ----------
print("\nComida | Servicio | Propina")
for c, s in [(2, 3), (5, 5), (9, 9), (2, 9)]:
    prueba = ctrl.ControlSystemSimulation(ctrl.ControlSystem([r1, r2, r3]))
    prueba.input['comida'] = c
    prueba.input['servicio'] = s
    prueba.compute()
    print(f"{c:^6} | {s:^8} | {prueba.output['propina']:.2f} %")
```

**Salida esperada en consola:**

```
OR  (unión):         0.8
AND (intersección):  0.4
NOT (complemento):   0.2
Propina sugerida: 19.85 %

Comida | Servicio | Propina
  2    |    3     | 10.84 %
  5    |    5     | 12.67 %
  9    |    9     | 16.81 %
  2    |    9     | 13.06 %
```

> [!TIP]
> En la terminal, la ventana de la gráfica detiene el programa hasta que la cierres. Los 4 casos del reto se imprimen después de cerrarla.

---

## Problemas frecuentes

<details>
<summary><code>ModuleNotFoundError: No module named 'skfuzzy'</code></summary>

No se instaló la librería, o en Linux no está activado el entorno virtual.
- Colab: ejecutar `!pip install scikit-fuzzy`.
- Linux: `source entorno/bin/activate` y luego `pip install scikit-fuzzy numpy scipy networkx matplotlib`.

</details>

<details>
<summary><code>ModuleNotFoundError: No module named 'scipy'</code> (o <code>networkx</code>)</summary>

`scikit-fuzzy` necesita esas dos librerías y en un entorno virtual nuevo no siempre se instalan solas. Con el entorno activado:
```bash
pip install scipy networkx
```

</details>

<details>
<summary><code>error: externally-managed-environment</code> (Linux)</summary>

Las versiones recientes de Ubuntu/Debian no permiten `pip install` fuera de un entorno virtual. Crea y activa uno como se indica en la Opción B.

</details>

<details>
<summary>La gráfica no se abre en la terminal</summary>

Puede faltar `python3-tk` o no haber entorno gráfico (máquina virtual sin pantalla, SSH). El archivo `propina.png` se genera igual en la carpeta del taller y se puede abrir desde el explorador de archivos.

</details>

<details>
<summary><code>NameError: name 'ctrl' is not defined</code></summary>

Se saltó el Paso 2.1 o se ejecutaron las celdas fuera de orden. En Colab: **Entorno de ejecución → Ejecutar todas**.

</details>
