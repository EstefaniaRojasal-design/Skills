---
name: aa-python-script-generator
description: Genera scripts Python listos para ejecutarse desde Automation Anywhere (A360), cumpliendo el contrato de una función principal con un único argumento string y retorno JSON con status success/error. Úsalo siempre que el usuario solicite código Python, scripts, funciones, lectura/escritura de archivos o automatizaciones para Automation Anywhere, AA, bots o RPA.
---

# Generador de scripts Python para Automation Anywhere

Genera scripts Python listos para ejecutarse desde Automation Anywhere (A360, paquete Python Script / Execute function). Automation Anywhere invoca una función de un archivo `.py`, le pasa un único argumento string y espera un string de retorno. Todo script generado para AA debe respetar ese contrato, sea para Excel, APIs, PDF, archivos, correos, bases de datos u otras tareas.

## Reglas obligatorias para todo script

1. **Una función de entrada principal:** por defecto `main`. El flujo de la tarea vive en ella. Se permiten helpers privados (por ejemplo, `_parse_args`), pero AA solo llama a la función principal. Indica explícitamente al usuario el nombre de la función que debe configurar en AA.
2. **Un único argumento string:** usar `def main(param="")`. Los parámetros de negocio se envían dentro de ese string y se parsean internamente.
3. **Aceptar formatos de parámetros:** JSON, diccionario en texto estilo Python y valores separados por comas, tanto `clave=valor` como posicionales según `PARAM_KEYS`.
4. **Todo dentro de `try/except`:** incluye parseo y lógica. La función nunca debe propagar excepciones a AA; convertir todos los errores a respuesta JSON `status: error`.
5. **Retornar siempre un string JSON:** usar `json.dumps(response, ensure_ascii=False, default=str)`. `default=str` evita fallos con fechas y `Decimal`.
6. **Estados exactos:** usar exclusivamente `success` y `error`.
   - Éxito: `{"status": "success", ...datos relevantes}`.
   - Error: `{"status": "error", "message": "...", "error_type": "..."}`.
7. **No usar `print`, `input()` ni `sys.exit()` para comunicar resultados.** AA recibe el `return`; `input()` puede dejar el bot bloqueado.

## Flujo de trabajo

1. Entiende la tarea y determina los parámetros necesarios. Si falta información crítica (ruta de archivo, nombre de hoja, endpoint), pregunta una sola vez y brevemente. Si es razonable asumir, hazlo y dilo.
2. Si el repositorio del skill incluye `assets/aa_template.py`, lee y reutiliza esa plantilla. Ajusta `PARAM_KEYS` al orden natural de los argumentos, valida parámetros obligatorios dentro del `try` y sustituye la lógica de ejemplo.
3. Devuelve datos útiles como campos del JSON (por ejemplo `"filas": 120`, `"ruta_salida": "..."`, `"data": [...]`) en lugar de mensajes largos, para que AA pueda consumirlos como JSON/diccionario.
4. Si el entorno de ejecución está disponible, guarda el script en `/mnt/user-data/outputs/` y valida con:
   `python <ruta-del-skill>/scripts/validate_aa_script.py <script.py> main`
   Corrige cualquier FAIL. Si los archivos o servicios requeridos no existen en el entorno, valida al menos el contrato: parseo, manejo de excepciones y JSON con `status`.
5. Entrega el archivo `.py` adjunto si la plataforma tiene herramientas para crear y presentar archivos. No afirmes que lo validaste o adjuntaste si no fue posible.

## Manejo de errores y parámetros

- Valida de forma explícita los errores previsibles: columna inexistente, archivo no encontrado, hoja inexistente, argumentos faltantes o valores inválidos. Usa mensajes accionables, como `ValueError` o `FileNotFoundError` que indiquen qué falló y con qué valor.
- Mantén un `except Exception` general como red de seguridad, y devuelve `str(e)` junto con `type(e).__name__`.
- En valores recibidos como texto, convierte explícitamente: `int(args["n"])`; booleanos con `str(value).lower() in ("true", "1", "si", "sí")`.
- Normaliza rutas con `os.path.normpath`; para rutas Windows usa strings raw o `os.path.join`, sin asumir separadores `/`.
- En JSON, `\t`, `\n` y `\b` se interpretan como caracteres de control y pueden corromper rutas. Al explicar rutas recomienda `/` (p. ej. `C:/datos/a.xlsx`) o backslashes escapados (p. ej. `C:\\datos\\a.xlsx`). En formato separado por comas no es necesario escapar backslashes.
- Comas dentro de valores rompen el formato separado por comas: en ese caso recomienda JSON.
- Cierra archivos, conexiones y libros de Excel usando `with` o `finally`, para evitar recursos bloqueados entre ejecuciones del bot.
- No hardcodees credenciales. Solicítalas como parámetro (preferiblemente desde el Credential Vault de AA) y nunca las incluyas en el JSON de respuesta.
- Si el resultado es grande (miles de filas), guárdalo en archivo y devuelve la ruta en el JSON, en lugar de devolver un string enorme.

## Formato de respuesta final

Mantén la respuesta breve e incluye solo:

- **Función a llamar en AA:** `main` (o el nombre elegido).
- **Parámetros:** cómo enviarlos, con un ejemplo concreto de string para esa tarea; preferir JSON y mencionar formato por comas si aplica.
- **Retornos:** ejemplos de éxito y error pertinentes a la tarea.
- **Dependencias:** comandos `pip install ...` para librerías externas que deben existir en el intérprete Python configurado por AA. Prefiere biblioteca estándar; si se necesita `openpyxl`, `pandas`, `requests` u otra, dilo explícitamente.
- Buenas prácticas específicas de AA si son relevantes para la tarea.

## Ejemplo de contrato

Petición: “Script que lea un Excel y me diga cuántas filas tiene una hoja.”

```python
# FUNCION A LLAMAR DESDE AUTOMATION ANYWHERE: main
import json
import ast

PARAM_KEYS = ["ruta", "hoja"]

def _parse_args(raw, keys=None):
    # Implementar el parseo de JSON, diccionario de texto y valores separados por comas.
    # Puede reutilizarse assets/aa_template.py cuando esté disponible.
    ...

def main(param=""):
    try:
        args = _parse_args(param, PARAM_KEYS)
        if not args.get("ruta"):
            raise ValueError("Falta el parámetro requerido: ruta")

        from openpyxl import load_workbook
        wb = load_workbook(args["ruta"], read_only=True, data_only=True)
        try:
            ws = wb[args["hoja"]] if args.get("hoja") else wb.active
            filas = ws.max_row
            nombre_hoja = ws.title
        finally:
            wb.close()

        response = {"status": "success", "hoja": nombre_hoja, "filas": filas}
    except Exception as e:
        response = {"status": "error", "message": str(e), "error_type": type(e).__name__}
    return json.dumps(response, ensure_ascii=False, default=str)
```

Parámetros desde AA (JSON): `{"ruta": "C:/datos/ventas.xlsx", "hoja": "Q1"}`.

Retorno de éxito: `{"status": "success", "hoja": "Q1", "filas": 120}`.

Retorno de error: `{"status": "error", "message": "...", "error_type": "FileNotFoundError"}`.

La plantilla y el validador mencionados en el flujo se usan solo si existen en el repositorio/entorno; no inventes su ruta ni afirmes haber ejecutado una validación que no se realizó.
