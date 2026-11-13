# Semana 4 — Actividad 1: Recuperación con `try/except`

**Modalidad:** Grupal (3 o 4 personas). Un solo integrante sube el trabajo.

## Objetivo

Asegurar la robustez del sistema implementando `try/except` en la función de carga de datos,
para detectar, aislar y recuperarse de errores (montos negativos, formatos incorrectos) sin
detener la ejecución del programa.

## Archivos

| Archivo | Para qué sirve |
|---------|----------------|
| [`robustez_try_except.py`](./robustez_try_except.py) | Lectura tolerante a fallos: procesa un archivo con datos corruptos sin detenerse. |
| [`transacciones_corruptas.txt`](./transacciones_corruptas.txt) | 7 registros, **3 con errores intencionales**. |

## El problema

En la Semana 3 la clase `Transaccion` se protegía a sí misma lanzando `ValueError` en el
*setter* del monto. Pero si **nadie atrapaba** ese error, el programa entero se detenía en el
primer registro corrupto: si el registro #50 de 100 fallaba, se perdían los 50 siguientes.

El archivo `transacciones_corruptas.txt` tiene 3 errores a propósito:

| Línea | Dato | Error que provoca |
|------|------|-------------------|
| C003 | `DEBITO,texto_invalido` | **ValueError** — no se puede convertir `"texto_invalido"` a número |
| C004 | `CREDITO,-200000` | **ValueError** — el *setter* rechaza montos negativos (validación de la Semana 3) |
| C006 | `DEBITO` (falta el monto) | **TypeError** — al constructor le faltan argumentos |

## Solución

`cargar_transacciones()` envuelve la creación del objeto en un `try` y atrapa cada fallo con un
`except` específico (`ValueError` / `TypeError`). Imprime el error con el número de línea
(estrategia de recuperación pasiva: *log* + `continue`) y el bucle **continúa** con el siguiente
registro, sin detener el programa.

Para cubrir **todos** los posibles errores (no solo los dos esperados por el enunciado) y
apuntar al nivel más alto de la rúbrica, se añadieron tres refuerzos de robustez:

- **`except Exception`** al final de los `except` específicos, como red de seguridad que
  atrapa cualquier error no previsto en una línea, sin dejar que nada detenga la carga.
- **`try/except FileNotFoundError` / `OSError`** alrededor de la apertura del archivo, para que
  un archivo faltante o sin permisos tampoco tumbe el programa.
- **`finally`** con un contador real de líneas leídas (se ejecuta siempre, haya error o no), a
  modo de auditoría de cuántos registros se procesaron en total.

### Salida esperada

```
  [Linea 3] ValueError: invalid literal for int() with base 10: 'texto_invalido'  ->  se ignora: C003,DEBITO,texto_invalido
  [Linea 4] ValueError: el monto no puede ser negativo  ->  se ignora: C004,CREDITO,-200000
  [Linea 6] TypeError: datos insuficientes  ->  se ignora: C006,DEBITO

  Lineas de datos leidas en total: 7
  Lineas con error (registradas en el log): 3

Se procesaron 4 transacciones validas:
   C001 | DEBITO | $150000
   C002 | CREDITO | $500000
   C005 | DEBITO | $45000
   C007 | CREDITO | $10000
```

> 💡 El programa lee un archivo **lleno de datos corruptos** y aun así procesa **todos los
> datos válidos**, dejando un registro (log) de los errores, sin detenerse. Eso es un sistema
> **tolerante a fallos**.

## Cómo ejecutar

```bash
python robustez_try_except.py
```

## Entregable

Carpeta con `robustez_try_except.py` y `transacciones_corruptas.txt` — código fuente
ejecutable (verificado, coincide con la salida esperada del enunciado).
