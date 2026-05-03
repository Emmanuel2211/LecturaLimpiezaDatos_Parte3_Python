# Ejercicio Evaluado — Lectura y Limpieza de Datos (Python)
### Diplomado de Ciencia de Datos — Módulo 2

Este proyecto aplica técnicas de lectura, limpieza y visualización de datos sobre el dataset
`coffee_ratings.csv` (1,339 filas × 43 columnas), que contiene evaluaciones de calidad de café
de diversas regiones del mundo. El análisis se realizó en Python con un Jupyter Notebook
(`calidad_cafe_parte3.ipynb`) y cubre 11 actividades numeradas del 0 al 10.

---

## Librerías utilizadas

| Librería | Propósito |
|----------|-----------|
| `pandas` | Manipulación de DataFrames, operaciones sobre columnas, fechas y tipos de dato |
| `numpy` | Valores NaN, operaciones numéricas |
| `re` | Expresiones regulares para extracción de patrones |
| `matplotlib.pyplot` | Creación de figuras y ejes, control de layout |
| `matplotlib.dates` | Formato de fechas en ejes de tiempo |
| `seaborn` | Visualizaciones estadísticas de alto nivel |

---

## Carga de datos (celda inicial)

```python
import pandas as pd
import numpy as np
import re
import matplotlib.pyplot as plt
import matplotlib.dates as mdates
import seaborn as sns

%matplotlib inline

datos = pd.read_csv('/home/emma/Data_Science/mod2_diplomado/coffee_ratings.csv')
```

- `import pandas as pd` — importa pandas y lo renombra `pd` para escribir menos.
- `import numpy as np` — importa numpy y lo renombra `np`; se usa principalmente para `np.nan`.
- `import re` — módulo estándar de Python para expresiones regulares.
- `import matplotlib.pyplot as plt` — interfaz principal para crear gráficas en matplotlib.
- `import matplotlib.dates as mdates` — submódulo de matplotlib para formatear ejes de fechas.
- `import seaborn as sns` — librería de visualización estadística construida sobre matplotlib.
- `%matplotlib inline` — magic command de Jupyter que hace que las gráficas aparezcan en el
  notebook en lugar de abrirse en una ventana aparte.
- `pd.read_csv(ruta)` — lee el archivo CSV y lo carga en un DataFrame de pandas. El argumento
  es la ruta al archivo en disco.

---

## Actividad 0: Análisis de valores faltantes

```python
total_filas = len(datos)
faltantes = datos.isnull().sum()
pct_faltantes = (faltantes / total_filas * 100).round(2)

resumen_na = pd.DataFrame({
    "valores_faltantes": faltantes,
    "porcentaje": pct_faltantes
})
resumen_na = resumen_na[resumen_na["valores_faltantes"] > 0].sort_values(
    "valores_faltantes", ascending=False
)
print(f"Dimensiones del dataset: {datos.shape[0]} filas x {datos.shape[1]} columnas\n")
print(resumen_na.to_string())
```

**Explicación línea por línea:**

- `len(datos)` — devuelve el número de filas del DataFrame; equivale a `datos.shape[0]`.
- `datos.isnull()` — devuelve un DataFrame booleano del mismo tamaño donde `True` indica celda
  vacía (`NaN`) y `False` indica valor presente.
- `.sum()` — al aplicarse sobre un DataFrame booleano, cuenta los `True` por columna (suma los
  valores faltantes de cada columna).
- `faltantes / total_filas * 100` — calcula el porcentaje de faltantes por columna respecto al
  total de filas.
- `.round(2)` — redondea a dos decimales.
- `pd.DataFrame({...})` — crea un nuevo DataFrame a partir de un diccionario donde las claves
  son nombres de columnas y los valores son las Series ya calculadas.
- `resumen_na[resumen_na["valores_faltantes"] > 0]` — filtra filas: sólo conserva las columnas
  que tienen al menos un valor faltante (selección booleana sobre el índice de columnas).
- `.sort_values("valores_faltantes", ascending=False)` — ordena el DataFrame por la columna
  `"valores_faltantes"` de mayor a menor.
- `datos.shape` — tupla `(filas, columnas)`. `datos.shape[0]` es número de filas, `datos.shape[1]`
  es número de columnas.
- `f"..."` — f-string de Python: permite insertar variables dentro de cadenas con `{variable}`.
- `.to_string()` — convierte el DataFrame a una cadena de texto legible sin truncar columnas.

**Resultado:** El dataset tiene 1,339 filas y 43 columnas; 19 columnas presentan valores
faltantes. La columna más afectada es `lot_number` (79.39 %).

---

## Actividad 1: Crear la columna `color2`

```python
COLOR_MAP = {
    "Green": "#00FF66",
    "Bluish-Green": "#CCEBC5",
    "Blue-Green": "#BFFFFF",
}

datos["color2"] = datos["color"].map(COLOR_MAP)

print(datos[["color", "color2"]].value_counts(dropna=False).to_string())
```

**Explicación línea por línea:**

- `COLOR_MAP = {...}` — diccionario Python que asocia cada nombre de color textual con su código
  hexadecimal. Las claves son los valores posibles de `color`; los valores son los hex codes.
- `datos["color"].map(COLOR_MAP)` — aplica el diccionario como función de mapeo sobre la Serie
  `color`. Cada valor de la columna se busca como clave en `COLOR_MAP` y se reemplaza por el
  valor correspondiente. Si la clave no existe (o el valor es `NaN`), el resultado es `NaN`.
- `datos["color2"] = ...` — asigna la Serie resultante como nueva columna `color2` en el
  DataFrame.
- `datos[["color", "color2"]]` — selecciona dos columnas del DataFrame (devuelve un sub-DataFrame).
  Las dobles corchetes `[[...]]` son necesarias para seleccionar múltiples columnas.
- `.value_counts(dropna=False)` — cuenta las combinaciones únicas de valores en las columnas
  seleccionadas. `dropna=False` incluye las filas con `NaN` en el conteo.

**Resultado:** 870 granos verdes (`#00FF66`), 114 verde-azulados (`#CCEBC5`), 85 verde-azul
(`#BFFFFF`), y 270 faltantes conservados como `NaN`.

---

## Actividad 2: Crear la columna numérica `bag_weight2`

```python
def extraer_numero_bag_weight(valor):
    """Extrae el primer número de bag_weight. Retorna NaN si hay ambigüedad o no hay número."""
    if pd.isna(valor):
        return np.nan
    if "," in str(valor):
        return np.nan
    numeros = re.findall(r"\d+(?:\.\d+)?", str(valor))
    return float(numeros[0]) if numeros else np.nan


datos["bag_weight2"] = datos["bag_weight"].apply(extraer_numero_bag_weight)

ambiguos_bw = datos["bag_weight"].apply(
    lambda v: not pd.isna(v) and ("," in str(v) or not re.search(r"\d", str(v)))
)
print(f"Observaciones ambiguas en bag_weight: {ambiguos_bw.sum()}")
print("Valores ambiguos únicos:", datos.loc[ambiguos_bw, "bag_weight"].unique())
```

**Explicación línea por línea:**

- `def extraer_numero_bag_weight(valor):` — define una función reutilizable que recibe un valor
  de `bag_weight` y devuelve un número o `NaN`.
- `pd.isna(valor)` — devuelve `True` si el valor es `NaN`, `None` o `pd.NA`. Permite distinguir
  valores faltantes de cadenas vacías.
- `"," in str(valor)` — convierte el valor a cadena con `str()` y verifica si contiene una coma.
  La coma indica ambigüedad (dos unidades simultáneas como `"2 kg,lbs"`).
- `re.findall(r"\d+(?:\.\d+)?", str(valor))` — busca todos los números (enteros o decimales)
  en la cadena usando regex. `\d+` coincide con uno o más dígitos; `(?:\.\d+)?` es un grupo no
  capturador opcional que coincide con la parte decimal.
- `float(numeros[0]) if numeros else np.nan` — si se encontraron números, convierte el primero
  a `float`; si la lista está vacía, devuelve `NaN`.
- `.apply(extraer_numero_bag_weight)` — aplica la función fila por fila sobre la Serie
  `bag_weight`. Es la forma idiomática de aplicar funciones personalizadas en pandas.
- `lambda v: ...` — función anónima (lambda) de una sola expresión. Se usa para lógica sencilla
  que no justifica definir una función con `def`.
- `datos.loc[ambiguos_bw, "bag_weight"]` — selección con `.loc[]` usando una máscara booleana
  para las filas y un nombre de columna para las columnas.
- `.unique()` — devuelve un array con los valores únicos de la Serie.

**Resultado:** 2 observaciones ambiguas: `'1 kg,lbs'` y `'2 kg,lbs'`.

---

## Actividad 3: Crear las columnas `method1` y `method2`

```python
separador_pm = r"\s*/\s*"

split_pm = datos["processing_method"].str.split(separador_pm, n=1, expand=True)
datos["method1"] = split_pm[0].str.strip()
datos["method2"] = split_pm[1].str.strip() if 1 in split_pm.columns else np.nan

ambiguos_pm = datos["processing_method"].apply(
    lambda v: pd.isna(v) or "/" not in str(v)
)
print(f"Observaciones ambiguas en processing_method: {ambiguos_pm.sum()}")
print("Valores ambiguos únicos:", datos.loc[ambiguos_pm, "processing_method"].unique())
```

**Explicación línea por línea:**

- `r"\s*/\s*"` — cadena raw de Python (el prefijo `r` impide que `\` sea interpretado como
  secuencia de escape). El patrón regex coincide con `"/"` rodeado de cero o más espacios
  (`\s*` = cero o más espacios en blanco).
- `.str.split(separador_pm, n=1, expand=True)` — método de pandas para dividir cadenas.
  - `.str` — accesor de string de pandas, activa métodos de cadena vectorizados.
  - `n=1` — realiza máximo una división, produciendo a lo más 2 partes.
  - `expand=True` — en lugar de devolver una Serie de listas, devuelve un DataFrame con una
    columna por cada parte resultante (columna `0` y columna `1`).
- `split_pm[0]` — primera parte del split (antes del `"/"`). Es la columna `0` del DataFrame
  resultado.
- `.str.strip()` — elimina espacios en blanco al inicio y al final de cada cadena en la Serie.
- `if 1 in split_pm.columns` — comprobación defensiva: si ninguna fila tenía `"/"`, el DataFrame
  resultado sólo tiene la columna `0` y no existe la columna `1`.
- `pd.isna(v) or "/" not in str(v)` — una fila es ambigua si su valor es `NaN` (no se puede
  dividir) o si no contiene el separador `"/"` (no se puede separar en dos métodos).

**Resultado:** 196 observaciones ambiguas: 170 con `NaN` + 26 con valor `"Other"` sin separador.

---

## Actividad 4: Crear columnas de fecha de expiración

```python
patron_exp = r"^(\w+)\s+(\d{1,2})(?:st|nd|rd|th),\s+(\d{4})$"

exp_parsed = datos["expiration"].str.extract(patron_exp)
exp_parsed.columns = ["expiration_month", "expiration_day_str", "expiration_year_str"]

datos["expiration_day"]   = pd.to_numeric(exp_parsed["expiration_day_str"],  errors="coerce").astype("Int64")
datos["expiration_month"] = exp_parsed["expiration_month"]
datos["expiration_year"]  = pd.to_numeric(exp_parsed["expiration_year_str"], errors="coerce").astype("Int64")

ambiguos_exp = exp_parsed["expiration_month"].isna() & datos["expiration"].notna()
print(f"Observaciones ambiguas en expiration: {ambiguos_exp.sum()}")
print(datos[["expiration", "expiration_day", "expiration_month", "expiration_year"]].head(8).to_string())
```

**Explicación línea por línea:**

- `r"^(\w+)\s+(\d{1,2})(?:st|nd|rd|th),\s+(\d{4})$"` — patrón regex con 3 grupos de captura:
  - `^` y `$` — anclas de inicio y fin de cadena.
  - `(\w+)` — grupo 1: captura el nombre del mes (una o más letras/dígitos).
  - `\s+` — uno o más espacios en blanco.
  - `(\d{1,2})` — grupo 2: captura el día (1 o 2 dígitos).
  - `(?:st|nd|rd|th)` — sufijo ordinal inglés (grupo no capturador: no se guarda, pero debe
    estar presente).
  - `,\s+` — coma seguida de uno o más espacios.
  - `(\d{4})` — grupo 3: captura el año (exactamente 4 dígitos).
- `.str.extract(patron_exp)` — aplica el regex y devuelve un DataFrame con una columna por cada
  grupo de captura. Las filas donde el regex no coincide quedan como `NaN`.
- `exp_parsed.columns = [...]` — renombra las columnas del DataFrame resultado para mayor
  claridad.
- `pd.to_numeric(..., errors="coerce")` — convierte una Serie a tipo numérico. Con
  `errors="coerce"`, los valores que no se pueden convertir (incluido `NaN`) se reemplazan por
  `NaN` en lugar de lanzar un error.
- `.astype("Int64")` — convierte al tipo entero nullable de pandas (`Int64` con mayúscula).
  A diferencia del `int64` estándar de NumPy, `Int64` puede contener `NaN`.
- `exp_parsed["expiration_month"].isna() & datos["expiration"].notna()` — máscara booleana
  combinada con `&` (AND elemento a elemento): identifica filas donde el regex no capturó nada
  pero el valor original existe (verdaderas ambigüedades).

**Resultado:** 0 observaciones ambiguas. El formato de `expiration` es completamente consistente.

---

## Actividad 5: Crear columnas de año y mes de cosecha

```python
split_hy = datos["harvest_year"].str.split(r"\s*/\s*|\s*-\s*(?=[0-9])", n=1, expand=True)

datos["harvest_mes"]  = split_hy[0].str.strip()
datos["harvest_anio"] = split_hy[1].str.strip() if 1 in split_hy.columns else np.nan

ambiguos_hy = datos["harvest_anio"].isna()
print(f"Observaciones ambiguas en harvest_year: {ambiguos_hy.sum()}")
print(f"  · NaN originales en harvest_year:         {datos['harvest_year'].isna().sum()}")
print(f"  · Valores sin separador divisible:         {(ambiguos_hy & datos['harvest_year'].notna()).sum()}")
```

**Explicación línea por línea:**

- `r"\s*/\s*|\s*-\s*(?=[0-9])"` — patrón regex con dos alternativas separadas por `|`:
  - `\s*/\s*` — barra `/` con cero o más espacios a cada lado.
  - `\s*-\s*(?=[0-9])` — guión `-` con cero o más espacios, **seguido de un dígito** (lookahead
    positivo `(?=[0-9])`). El lookahead `(?=...)` no consume caracteres; sólo comprueba que lo
    que sigue es un dígito. Esto evita dividir palabras con guión como `"Washed-Dry"`.
- `n=1` — como en la Actividad 3, realiza máximo una división.
- `split_hy[1]` — la segunda parte del split (lo que viene después del separador). Si no hubo
  separador, esta columna contiene `NaN` para esas filas.
- `datos["harvest_anio"].isna()` — máscara booleana que identifica filas donde `harvest_anio`
  es `NaN`, es decir, donde no se pudo dividir.
- `ambiguos_hy & datos["harvest_year"].notna()` — intersección de dos condiciones: filas sin
  año y filas con valor original no nulo. Así se distinguen los `NaN` originales de los que
  resultan de la falta de separador.

**Resultado:** 1,196 observaciones ambiguas (47 `NaN` originales + 1,149 sin separador divisible,
incluyendo años simples como `"2014"` y formatos no estándar como `"23 July 2010"`).

---

## Actividad 6: Scatter de `acidity` vs `total_cup_points` coloreado por `color2`

```python
sns.set_theme(style="whitegrid", font_scale=1.2)

df6 = datos.dropna(subset=["color2", "total_cup_points", "acidity"])

color_labels = {
    "#00FF66": "Green",
    "#CCEBC5": "Bluish-Green",
    "#BFFFFF": "Blue-Green",
}

fig, ax = plt.subplots(figsize=(9, 6))

for hex_color, label in color_labels.items():
    subset = df6[df6["color2"] == hex_color]
    ax.scatter(
        subset["acidity"],
        subset["total_cup_points"],
        color=hex_color,
        label=label,
        alpha=0.75,
        edgecolors="dimgrey",
        linewidths=0.4,
        s=55,
    )

ax.set_xlabel("Acidez (Acidity)", fontsize=13)
ax.set_ylabel("Puntuación Total (Total Cup Points)", fontsize=13)
ax.set_title("Relación entre Acidez y Puntuación Total del Café\nsegún Color del Grano",
             fontsize=14, fontweight="bold")
ax.legend(title="Color del grano", framealpha=0.9, edgecolor="grey")
sns.despine()
plt.tight_layout()
plt.savefig("act6_color_acidity.png", dpi=150, bbox_inches="tight")
plt.show()
```

**Explicación línea por línea:**

- `sns.set_theme(style="whitegrid", font_scale=1.2)` — configura el tema global de seaborn:
  `"whitegrid"` aplica fondo blanco con rejilla gris, y `font_scale=1.2` aumenta el tamaño de
  todos los textos un 20%.
- `.dropna(subset=[...])` — elimina las filas donde al menos una de las columnas indicadas
  tiene `NaN`. Sólo actúa sobre el subconjunto especificado, no sobre todo el DataFrame.
- `color_labels = {...}` — diccionario inverso al de la Actividad 1: mapea hex code → nombre
  legible, para construir la leyenda.
- `fig, ax = plt.subplots(figsize=(9, 6))` — crea una figura y un único eje (subplot).
  `fig` es el objeto figura global; `ax` es el área de graficación. `figsize` especifica el
  tamaño en pulgadas `(ancho, alto)`.
- `for hex_color, label in color_labels.items():` — itera sobre el diccionario. En cada
  iteración, `hex_color` es la clave (código hex) y `label` es el valor (nombre del color).
- `df6[df6["color2"] == hex_color]` — filtra las filas cuyo `color2` coincide con el hex
  actual; permite dibujar cada grupo de puntos con su color real.
- `ax.scatter(x, y, ...)` — dibuja puntos en el eje `ax`.
  - `color=hex_color` — aplica el código hexadecimal directamente como color de los puntos.
  - `alpha=0.75` — transparencia del 75% (0 = invisible, 1 = opaco).
  - `edgecolors` y `linewidths` — color y grosor del borde de cada punto.
  - `s=55` — tamaño de cada punto en puntos² (unidad matplotlib).
- `ax.set_xlabel/ylabel/title` — establece etiquetas de ejes y título del gráfico.
- `ax.legend(...)` — añade una leyenda. `framealpha` controla la opacidad del fondo del
  recuadro; `edgecolor` controla el color del borde.
- `sns.despine()` — elimina los bordes superior y derecho del gráfico para un estilo más limpio.
- `plt.tight_layout()` — ajusta automáticamente los márgenes para que nada quede cortado.
- `plt.savefig("...", dpi=150, bbox_inches="tight")` — guarda la figura en disco. `dpi=150`
  controla la resolución (puntos por pulgada); `bbox_inches="tight"` recorta márgenes blancos
  sobrantes.
- `plt.show()` — muestra la figura en el notebook.

**Resultado:** No hay diferencia marcada en puntuación entre colores. Se observa correlación
positiva moderada entre acidez y puntuación.

---

## Actividad 7: Densidad de `bag_weight2` por especie

```python
sns.set_theme(style="whitegrid", font_scale=1.2)

df7 = datos.dropna(subset=["bag_weight2", "species"]).copy()
df7 = df7[df7["bag_weight2"] < 2000]

fig, ax = plt.subplots(figsize=(9, 6))

sns.kdeplot(
    data=df7,
    x="bag_weight2",
    hue="species",
    fill=True,
    alpha=0.35,
    linewidth=2.5,
    palette="Set1",
    ax=ax,
)

ax.set_xlabel("Peso del Saco (kg)", fontsize=13)
ax.set_ylabel("Densidad", fontsize=13)
ax.set_title("Distribución del Peso de los Sacos de Café por Especie",
             fontsize=14, fontweight="bold")
sns.despine()
plt.tight_layout()
plt.savefig("act7_density_bagweight.png", dpi=150, bbox_inches="tight")
plt.show()
```

**Explicación línea por línea:**

- `.copy()` — crea una copia independiente del sub-DataFrame. Es buena práctica para evitar la
  advertencia `SettingWithCopyWarning` de pandas cuando se modificará el subconjunto.
- `df7[df7["bag_weight2"] < 2000]` — filtra valores extremos (outliers). Pesos mayores a 2,000 kg
  son atípicos y distorsionarían la escala del eje X.
- `sns.kdeplot(...)` — dibuja una estimación de densidad kernel (KDE). El KDE es una versión
  suavizada del histograma que estima la función de densidad de probabilidad de los datos.
  - `x="bag_weight2"` — variable a graficar en el eje horizontal.
  - `hue="species"` — divide el gráfico por grupos (uno por especie), cada uno con color propio.
  - `fill=True` — rellena el área bajo la curva de densidad.
  - `alpha=0.35` — transparencia del relleno, permite ver superposición entre grupos.
  - `linewidth=2.5` — grosor de la línea de contorno.
  - `palette="Set1"` — paleta de colores predefinida de seaborn (colores vivos y distinguibles).
  - `ax=ax` — especifica el eje donde se dibuja (necesario cuando se usa `plt.subplots()`).

**Resultado:** Arabica tiene distribución concentrada en 0–100 kg; Robusta muestra patrón
diferenciado con menor amplitud.

---

## Actividad 8: Puntuación total vs fecha de expiración (4 países)

```python
paises = ["Mexico", "Brazil", "Colombia", "Guatemala"]
month_num_map = {
    "January": 1, "February": 2, "March": 3, "April": 4,
    "May": 5, "June": 6, "July": 7, "August": 8,
    "September": 9, "October": 10, "November": 11, "December": 12,
}

df8 = datos[datos["country_of_origin"].isin(paises)].copy()
df8 = df8.dropna(subset=["expiration_year", "expiration_month", "total_cup_points"])
df8["month_num"] = df8["expiration_month"].map(month_num_map)

df8["fecha_exp"] = pd.to_datetime(
    df8[["expiration_year", "month_num"]]
      .assign(day=1)
      .rename(columns={"expiration_year": "year", "month_num": "month"}),
    errors="coerce",
)

df8_grouped = (
    df8.groupby(["fecha_exp", "country_of_origin"], as_index=False)["total_cup_points"]
    .mean()
    .sort_values("fecha_exp")
)

palette = {"Mexico": "#E63946", "Brazil": "#457B9D", "Colombia": "#2A9D8F", "Guatemala": "#E9C46A"}

fig, ax = plt.subplots(figsize=(12, 6))

for pais in paises:
    sub = df8_grouped[df8_grouped["country_of_origin"] == pais]
    ax.plot(sub["fecha_exp"], sub["total_cup_points"],
            marker="o", linewidth=2, markersize=7, label=pais, color=palette[pais])

ax.xaxis.set_major_formatter(mdates.DateFormatter("%b %Y"))
ax.xaxis.set_major_locator(mdates.MonthLocator(interval=3))
plt.xticks(rotation=45, ha="right")
ax.set_xlabel("Fecha de Expiración (Mes/Año)", fontsize=12)
ax.set_ylabel("Puntuación Total Promedio", fontsize=12)
ax.set_title("Puntuación Total del Café por Fecha de Expiración\n(México, Brasil, Colombia y Guatemala)",
             fontsize=14, fontweight="bold")
ax.legend(title="País de Origen", framealpha=0.9)
sns.despine()
plt.tight_layout()
plt.savefig("act8_expiration_cups.png", dpi=150, bbox_inches="tight")
plt.show()
```

**Explicación línea por línea:**

- `.isin(paises)` — método de pandas que devuelve una máscara booleana `True` donde el valor
  de la Serie está dentro de la lista proporcionada. Equivale a múltiples condiciones `==` con
  `|` (OR).
- `df8["expiration_month"].map(month_num_map)` — convierte el nombre del mes en inglés a número
  (1–12) usando el diccionario. Necesario para construir una fecha numérica.
- `pd.to_datetime(...)` — convierte un DataFrame o Serie al tipo `datetime64` de pandas.
  Cuando se le pasa un DataFrame con columnas `"year"`, `"month"`, `"day"`, construye fechas
  directamente. `errors="coerce"` convierte valores inválidos a `NaT` (Not a Time).
- `.assign(day=1)` — añade una columna temporal `day=1` al DataFrame, necesaria para
  `pd.to_datetime()` que requiere día, mes y año.
- `.rename(columns={...})` — renombra columnas del DataFrame para que coincidan con los nombres
  esperados por `pd.to_datetime()` (`"year"`, `"month"`, `"day"`).
- `.groupby(["fecha_exp", "country_of_origin"], as_index=False)` — agrupa el DataFrame por
  combinaciones únicas de fecha y país. `as_index=False` conserva las columnas de agrupamiento
  como columnas normales en lugar de moverlas al índice.
- `["total_cup_points"].mean()` — sobre cada grupo, calcula la media de `total_cup_points`.
- `ax.plot(x, y, marker="o", ...)` — dibuja una línea con marcadores circulares en cada punto.
- `mdates.DateFormatter("%b %Y")` — formatea fechas en el eje X como `"Jan 2016"`, `"Apr 2017"`,
  etc. `%b` = mes abreviado en inglés; `%Y` = año de 4 dígitos.
- `mdates.MonthLocator(interval=3)` — coloca marcas en el eje X cada 3 meses, evitando que el
  eje quede congestionado.
- `plt.xticks(rotation=45, ha="right")` — rota las etiquetas del eje X 45° y las alinea a la
  derecha para evitar solapamiento.

**Resultado:** Se aprecian tendencias diferenciadas por país. Algunos países sólo tienen datos
en ciertos periodos según la disponibilidad de lotes certificados.

---

## Actividad 9: Altitudes por mes de expiración (2016–2017)

```python
paises = ["Mexico", "Brazil", "Colombia", "Guatemala"]
month_order = ["January","February","March","April","May","June",
               "July","August","September","October","November","December"]

df9 = datos[
    datos["country_of_origin"].isin(paises) &
    datos["expiration_year"].isin([2016, 2017])
].copy()

df9 = df9.dropna(subset=["expiration_month",
                          "altitude_low_meters",
                          "altitude_mean_meters",
                          "altitude_high_meters"])

df9["expiration_month"] = pd.Categorical(
    df9["expiration_month"], categories=month_order, ordered=True
)

df9_long = df9.melt(
    id_vars=["expiration_month", "expiration_year"],
    value_vars=["altitude_low_meters", "altitude_mean_meters", "altitude_high_meters"],
    var_name="tipo_altitud",
    value_name="altitud_metros",
)
alt_labels = {
    "altitude_low_meters":  "Altitud Mínima",
    "altitude_mean_meters": "Altitud Media",
    "altitude_high_meters": "Altitud Máxima",
}
df9_long["tipo_altitud"] = df9_long["tipo_altitud"].map(alt_labels)

fig, axes = plt.subplots(1, 2, figsize=(14, 6), sharey=True)

for i, year in enumerate([2016, 2017]):
    sub = df9_long[df9_long["expiration_year"] == year]
    sns.lineplot(
        data=sub,
        x="expiration_month",
        y="altitud_metros",
        hue="tipo_altitud",
        estimator="mean",
        errorbar=("ci", 95),
        marker="o",
        linewidth=2,
        palette="Dark2",
        ax=axes[i],
    )
    axes[i].set_title(f"Año de Expiración: {year}", fontsize=13, fontweight="bold")
    axes[i].tick_params(axis="x", rotation=45)

fig.suptitle("Altitud de Cultivo por Mes de Expiración\n(México, Brasil, Colombia y Guatemala)",
             fontsize=14, fontweight="bold", y=1.02)
sns.despine()
plt.tight_layout()
plt.savefig("act9_altitude_expiration.png", dpi=150, bbox_inches="tight")
plt.show()
```

**Explicación línea por línea:**

- `datos["country_of_origin"].isin(paises) & datos["expiration_year"].isin([2016, 2017])` —
  dos condiciones combinadas con `&` (AND). Filtra sólo las filas que cumplen ambas condiciones
  simultáneamente.
- `pd.Categorical(df9["expiration_month"], categories=month_order, ordered=True)` — convierte
  la columna de meses a tipo `Categorical` ordenado. Esto permite que pandas y seaborn respeten
  el orden cronológico (enero→diciembre) en el eje X en lugar de ordenar alfabéticamente.
- `df9.melt(...)` — transforma el DataFrame de formato ancho (*wide*) a formato largo (*long/tidy*).
  - `id_vars` — columnas que se mantienen como identificadores (no se transforman).
  - `value_vars` — columnas que se "derriten" en una sola columna de valores.
  - `var_name` — nombre de la nueva columna que contiene los nombres de las columnas originales.
  - `value_name` — nombre de la nueva columna que contiene los valores.
  El resultado triplica las filas: cada fila original genera 3 filas (una por tipo de altitud).
- `fig, axes = plt.subplots(1, 2, ...)` — crea 1 fila y 2 columnas de subplots. `axes` es
  un array de 2 ejes.
- `sharey=True` — los dos subplots comparten la misma escala en el eje Y, facilitando la
  comparación directa entre 2016 y 2017.
- `sns.lineplot(estimator="mean", errorbar=("ci", 95), ...)` — dibuja la media de los valores
  en cada punto X con un intervalo de confianza del 95% sombreado alrededor.
- `enumerate([2016, 2017])` — itera produciendo tuplas `(índice, valor)`: `(0, 2016)` y
  `(1, 2017)`. El índice `i` se usa para seleccionar el eje `axes[i]`.
- `axes[i].tick_params(axis="x", rotation=45)` — rota las etiquetas del eje X del subplot `i`.
- `fig.suptitle(...)` — título global de la figura (sobre todos los subplots). `y=1.02` lo
  desplaza ligeramente hacia arriba para no solaparse con los títulos individuales.

**Resultado:** La altitud máxima registra mayor variabilidad. Las altitudes mínima y media
siguen patrones similares a lo largo del año.

---

## Actividad 10: Pairplot de `aftertaste`, `acidity`, `body` por especie

```python
sns.set_theme(style="ticks", font_scale=1.1)

df10 = datos[["aftertaste", "acidity", "body", "species"]].dropna()

var_labels = {"aftertaste": "Aftertaste", "acidity": "Acidez", "body": "Cuerpo"}

g = sns.pairplot(
    df10,
    hue="species",
    vars=["aftertaste", "acidity", "body"],
    diag_kind="kde",
    plot_kws={"alpha": 0.45, "s": 35, "edgecolor": None},
    diag_kws={"fill": True, "alpha": 0.45, "linewidth": 2},
    palette="Set1",
    height=3.2,
    aspect=1,
)

for ax in g.axes.flatten():
    if ax is None:
        continue
    if ax.get_xlabel() in var_labels:
        ax.set_xlabel(var_labels[ax.get_xlabel()], fontsize=11)
    if ax.get_ylabel() in var_labels:
        ax.set_ylabel(var_labels[ax.get_ylabel()], fontsize=11)

g.figure.suptitle(
    "Relaciones entre Aftertaste, Acidez y Cuerpo del Café por Especie",
    y=1.03, fontsize=14, fontweight="bold",
)
g.legend.set_title("Especie")
plt.tight_layout()
plt.savefig("act10_pairplot_species.png", dpi=150, bbox_inches="tight")
plt.show()
```

**Explicación línea por línea:**

- `datos[["aftertaste", "acidity", "body", "species"]].dropna()` — selecciona sólo las cuatro
  columnas relevantes y elimina filas con cualquier `NaN` en ellas. Reduce el DataFrame para
  trabajar sólo con lo necesario.
- `sns.pairplot(...)` — crea una matriz de gráficas N×N donde N es el número de variables.
  Muestra todos los pares posibles de variables:
  - La **diagonal** muestra la distribución de cada variable individualmente (KDE en este caso).
  - Los **paneles fuera de la diagonal** muestran dispersogramas de cada par de variables.
  - `hue="species"` — colorea los puntos y curvas según la especie.
  - `vars=[...]` — especifica las variables a incluir (en lugar de usar todas las columnas).
  - `diag_kind="kde"` — tipo de gráfica en la diagonal: estimación de densidad kernel.
  - `plot_kws={...}` — argumentos adicionales para los dispersogramas fuera de la diagonal.
    `alpha=0.45` = transparencia; `s=35` = tamaño de puntos; `edgecolor=None` = sin borde.
  - `diag_kws={...}` — argumentos adicionales para las gráficas de la diagonal.
    `fill=True` = rellena el área bajo la curva KDE.
  - `height=3.2` — altura (en pulgadas) de cada celda de la matriz.
  - `aspect=1` — relación de aspecto de cada celda (1 = cuadrada).
- `g.axes.flatten()` — `g.axes` es una matriz 2D de ejes. `.flatten()` la convierte en un
  array 1D para poder iterar sobre todos los ejes con un solo `for`.
- `ax.get_xlabel()` — devuelve la etiqueta actual del eje X de ese subplot. Se usa para
  detectar qué variable se está graficando y reemplazar el nombre en inglés por el español.
- `g.figure` — accede a la figura `matplotlib.Figure` subyacente del pairplot.
- `g.legend.set_title("Especie")` — accede directamente al objeto de leyenda del pairplot y
  cambia su título.

**Resultado:** Las tres variables están positivamente correlacionadas. Arabica muestra mayor
dispersión y puntajes altos; Robusta valores más concentrados y diferenciados.

---

## Archivos generados

| Archivo | Descripción |
|---------|-------------|
| `calidad_cafe_parte3.ipynb` | Notebook principal con todo el análisis |
| `coffee_ratings.csv` | Dataset original (1,339 × 43) |
| `act6_color_acidity.png` | Scatter de acidez vs puntuación por color del grano |
| `act7_density_bagweight.png` | KDE del peso de sacos por especie |
| `act8_expiration_cups.png` | Series de tiempo de puntuación por fecha de expiración |
| `act9_altitude_expiration.png` | Altitudes por mes de expiración (2016–2017) |
| `act10_pairplot_species.png` | Pairplot de métricas de sabor por especie |
