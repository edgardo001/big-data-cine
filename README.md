# El mundo del Big Data a través del cine

<img
  src="dcc-ec-logo.jfif"
  align="right"
  width="180"
  alt="Imagen"
/>

En este taller usaremos **Apache Spark (PySpark)** para explorar los datos públicos de IMDb desde nuestro propio computador. Veremos una demostración durante el taller, pero opcionalmente, al seguir estas instrucciones (idealmente antes del taller, o en caso contrario cuando puedas), también puedes correr todo localmente desde tu computador.

Las instrucciones sirven para **Windows, macOS y Linux**.

Estimo que se necesitará en el orden de **20 GB de espacio libre en disco** (idealmente más).

## 1. Descargar los datos de IMDb

Descarga los siguientes siete archivos desde <https://datasets.imdbws.com/> (en total, unos **2 GB** comprimidos). IMDb actualiza los archivos a diario, así que el conteo de filas puede variar levemente (el conteo indicado es del 2026-10-21 e incluye el encabezado):

- `name.basics.tsv.gz` (15.751.277 filas)
- `title.akas.tsv.gz` (59.861.137 filas)
- `title.basics.tsv.gz` (12.882.528 filas)
- `title.crew.tsv.gz` (12.882.528 filas)
- `title.episode.tsv.gz` (9.963.742 filas)
- `title.principals.tsv.gz` (102.413.746 filas)
- `title.ratings.tsv.gz` (1.723.303 filas)

Guárdalos juntos en una carpeta, por ejemplo, `big-data-cine/data/` (esta carpeta está en `.gitignore`, para no subir los datos al repositorio por error).

**No es necesario descomprimirlos:** Spark puede leer directamente los archivos `.tsv.gz`. Eso sí, un archivo `.gz` no se puede dividir, por lo que Spark lee cada archivo con una sola tarea: la primera lectura de los archivos grandes puede tardar algo más.

Los archivos contienen datos en formato **TSV** (*tab-separated values*, similar a CSV, pero con tabs como separadores). La descripción de cada archivo está en la [documentación de los conjuntos de datos no comerciales de IMDb](https://data.imdb.com/non-commercial-datasets/). (Solo se pueden usar estos datos para fines no comerciales.)

**En la carpeta [`muestra/`](https://github.com/aidhog/big-data-cine/tree/main/muestra) de este repositorio, dejamos muestras de las primeras 1.000 filas de los archivos** (solo para ilustrar cómo se ven: mejor no trabajar con estos datos, pues los identificadores de un archivo pueden no aparecer en otro).

## 2. Comprobar Java

Necesitamos **Java 17 o 21** (versiones LTS soportadas por Spark 4). Abre una terminal (en Windows, **PowerShell**, o en adelante también "_terminal_" cuando no importe si usas Windows, macOS o Linux) y ejecuta:

```bash
java -version
```

Deberías ver algo parecido a:

```text
openjdk version "17.0.x" ...
```

Si aparece la versión 17 o 21, puedes continuar. Si Java no está instalado o la versión es anterior, instala un JDK desde [Eclipse Temurin (Adoptium)](https://adoptium.net/).

En Windows, durante la instalación, habilita la opción **“Set or override JAVA_HOME variable”** y, si está disponible, la opción para agregar Java a `PATH`. Cierra y vuelve a abrir PowerShell antes de repetir `java -version`.

Si todavía hay problemas, comprueba `JAVA_HOME`:

**Windows (PowerShell):**

```powershell
$env:JAVA_HOME
```

**macOS o Linux (Terminal):**

```bash
printenv JAVA_HOME
```

La ruta debería apuntar a la carpeta del JDK, por ejemplo, `C:\Program Files\Eclipse Adoptium\jdk-17...` en Windows. La ubicación exacta depende del sistema, pero debería ser una carpeta con el nombre `jdk-XX...` donde `XX` es la versión (17 o 21).

Verifica de nuevo con `java -version`.

## 3. Comprobar Python

Necesitamos **Python 3.11 o posterior**.

Prueba uno de estos comandos en la terminal:

```bash
python --version
```

```bash
python3 --version
```

```powershell
py --version
```

Si aparece una versión adecuada (por ejemplo, `Python 3.12.x`), continúa. Si no, instala una versión estable desde <https://www.python.org/downloads/>.

> **Importante:** en los siguientes pasos usamos `python` como ejemplo. Si en tu computador funciona `python3` o `py` (con versión 3.11 o posterior), reemplaza `python` por ese comando.

## 4. Instalar PySpark y JupyterLab

En la terminal, ejecuta:

```bash
python -m pip install pyspark==4.2.0 jupyterlab
```

La instalación puede tardar algunos minutos.

Si aparece una advertencia similar a esta, **puedes ignorarla para este taller**:

```text
WARNING: The script ... is installed in ... which is not on PATH.
```
(Otras advertencias o errores, sí podrían requerir atención en el contexto de tu sistema operativo o tu computador. Ojalá no haya.)

> **Nota:** si tu sistema no permite instalar paquetes en el entorno global de Python, utiliza un entorno virtual (`venv`).

## 5. Iniciar JupyterLab

Desde la terminal, ejecuta:

```bash
python -m jupyterlab
```

JupyterLab debería abrirse automáticamente en tu navegador. Si no ocurre, copia en el navegador la dirección local que aparezca en la terminal (normalmente comienza con `http://localhost:8888/`). **No cierres la terminal** mientras estés usando JupyterLab.

## 6. Comprobar que Spark funciona

En JupyterLab, selecciona **Notebook → Python 3** (o el kernel de Python disponible).

Copia el siguiente código en la primera celda:

```python
from pyspark.sql import SparkSession

spark = (
    SparkSession.builder
    .master("local[*]")
    .appName("big-data-cine")
    .config("spark.driver.memory", "4g")
    .getOrCreate()
)

spark.range(2).count()
```

(La línea `spark.driver.memory` le da 4 GB de memoria a Spark; el valor por defecto, 1 GB, no alcanza para los archivos más grandes. Si tu computador tiene poca RAM, puedes probar con `"2g"`.)

Haz clic en el botón **▶** o presiona **Shift + Enter**.

Espera unos segundos (la primera ejecución puede tardar un poco más). Debajo de la celda debería aparecer el resultado:

```text
2
```

**¡Eso!** Si aparece `2`, ya tienes PySpark y JupyterLab funcionando y queda todo listo para el taller.

## ¿Problemas?

- **`python` no se reconoce:** prueba `python3` o, en Windows, `py`.
- **`No module named pip` o `No module named jupyterlab`:** revisa que estés usando el mismo intérprete de Python para instalar y ejecutar JupyterLab.
- **`java` no se reconoce:** revisa la instalación de Java, `PATH` y `JAVA_HOME`; después abre una terminal nueva.
- **Advertencia `Did not find winutils.exe` / `HADOOP_HOME and hadoop.home.dir are unset` (Windows):** puedes ignorarla; Spark lee los datos sin problema.
- **Advertencia `PySpark does not yet fully support pandas >= 3.0.0`:** puedes ignorarla para este taller.
- **`OutOfMemoryError` o `Java heap space`:** aumenta `spark.driver.memory` (por ejemplo, a `"6g"`), reinicia el kernel y vuelve a ejecutar.
- **¿Otros?** Al usar nuestros propios computadores, es difícil controlar todas las permutaciones de sistemas operativos, software ya instalado, recursos disponibles, etc. Puedes intentar Googlear o (con cuidado) usar la IA para resolver el problema. Pero es opcional: durante el taller vamos a ver una demostración.
