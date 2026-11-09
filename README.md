# Agenda de Contactos con Acceso Aleatorio

Aplicación de escritorio en Python que administra una agenda de contactos guardada en un archivo de **registros de tamaño fijo**. Permite llegar directamente a cualquier contacto con `seek()` sin leer el archivo completo, usa un **índice en JSON** para las búsquedas, exporta a **JSON y PDF**, verifica la **integridad con SHA-256** y crea **respaldos ZIP**.

Proyecto del **Laboratorio No. 2: Manejo de Archivos**
Universidad Rafael Landívar, Campus San Alberto Hurtado, S.J. de Quetzaltenango
Facultad de Ingeniería, Ingeniería en Informática y Sistemas

## Características

| Función | Descripción |
|---|---|
| **Contactos** | Agregar, buscar por ID y modificar contactos |
| **Registros de tamaño fijo** | Cada contacto ocupa exactamente 100 bytes |
| **Acceso aleatorio** | Lectura y escritura directa con `seek()`, mostrando la posición con `tell()` |
| **Modificación en el lugar** | Al editar solo se sobrescribe el registro; el resto del archivo no se reescribe |
| **Índice primario** | `indice.json` relaciona cada ID con su número de registro y se reconstruye solo si falta o está dañado |
| **Exportar** | Exportación a JSON y reporte PDF con fecha de generación |
| **Integridad** | Hash SHA-256 de `contactos.dat`, verificado al iniciar la aplicación |
| **Respaldo** | Archivo ZIP con los contactos y el índice, mostrando el tamaño original y el comprimido |
| **Información del archivo** | Tamaño, fecha de última modificación y permisos de lectura y escritura |
| **Manejo de errores** | La aplicación no se cierra si faltan archivos, están dañados o no hay permisos |

## Tecnologías

- Python 3.9 o superior
- [PyQt6](https://pypi.org/project/PyQt6/) para la interfaz gráfica
- [ReportLab](https://pypi.org/project/reportlab/) para el reporte PDF
- Módulos estándar: `os`, `json`, `hashlib`, `zipfile`, `datetime`

## Instalación y ejecución

1. Clona el repositorio y entra a la carpeta del proyecto:

   ```bash
   git clone <URL-DEL-REPOSITORIO>
   cd ProyectoFinal
   ```

2. (Opcional) Crea un entorno virtual:

   ```bash
   python -m venv .venv
   .venv\Scripts\activate
   ```

3. Instala las dependencias:

   ```bash
   python -m pip install PyQt6 reportlab
   ```

4. Ejecuta la aplicación:

   ```bash
   python src/main.py
   ```

Desarrollado y probado en Windows.

## Estructura del proyecto

```
ProyectoFinal/
├── src/
│   ├── main.py          Arranque de la aplicación
│   ├── interfaz.py      Ventana, menú, formularios, tablas y mensajes (PyQt6)
│   ├── contactos.py     Registros de tamaño fijo, seek() y tell()
│   ├── indice.py        Índice primario en JSON
│   ├── exportar.py      Exportación a JSON y reporte PDF (ReportLab)
│   ├── integridad.py    Hash SHA-256 y respaldos ZIP
│   ├── metadatos.py     Tamaño, fecha y permisos (os.stat, os.access)
│   └── rutas.py         Carpetas donde se guardan los archivos
├── data/                contactos.dat, indice.json, contactos.sha256
├── reports/             contactos.json, reporte_contactos.pdf, respaldo_*.zip
└── docs/                Documentación del laboratorio
```

Las carpetas `data` y `reports` se crean automáticamente si no existen.

## Estructura del registro

Cada contacto se guarda en `data/contactos.dat` como un registro de **100 bytes**, con cada campo rellenado con espacios hasta su largo máximo:

| Campo | Posición (bytes) | Largo |
|---|---|---|
| `id` | 0 a 4 | 5 |
| `nombre` | 5 a 34 | 30 |
| `teléfono` | 35 a 49 | 15 |
| `correo` | 50 a 84 | 35 |
| `ciudad` | 85 a 98 | 14 |
| salto de línea | 99 | 1 |
| **Total** | | **100** |

La posición de un registro se calcula directamente:

```
posición = número de registro × 100
```

## Cómo funciona

**Acceso aleatorio.** Para leer o modificar un contacto se hace `seek(numero * 100)` y se lee o escribe solo ese registro. `tell()` se usa antes y después de la operación, y la aplicación muestra ambas posiciones en la pantalla de Contactos.

**Índice.** `indice.json` guarda el número de registro de cada ID:

```json
{
  "40212": 0,
  "32": 1
}
```

Una búsqueda consulta primero el índice y luego hace `seek()` al registro, de modo que lee un solo registro sin importar cuántos contactos haya. Si el índice no existe, está dañado o no coincide con el archivo de contactos, se reconstruye automáticamente y se avisa al usuario.

**Integridad.** Cada vez que la aplicación guarda un cambio, calcula el SHA-256 de `contactos.dat` y lo guarda en `contactos.sha256`. Al iniciar se recalcula y se compara: si no coincide, se avisa que el archivo fue modificado fuera de la aplicación.

**Respaldo.** Crea un ZIP con `contactos.dat` e `indice.json` en la carpeta `reports`, con la fecha y hora en el nombre, y muestra el tamaño original y el comprimido.

## Decisiones de diseño

- **Los largos se miden en bytes, no en caracteres.** El archivo usa UTF-8 y letras como `ñ` o `ó` ocupan 2 bytes; medirlas en caracteres desalinearía los registros.
- **Archivo en modo binario.** En Windows el modo texto cambia `\n` por `\r\n`, lo que rompería el cálculo de posiciones.
- **Escritura con archivo temporal.** El índice, el hash y el JSON se escriben primero en un archivo `.tmp` y luego se renombran con `os.replace()`, para
