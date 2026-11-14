# ¿Conviene cortar más de lo necesario?

Estudio sobre **circuit cutting** en computación cuántica, presentado al Hackathon del
Wisconsin Quantum Computing Club (pista Explorer, noviembre 2026).

**Alberto Galarza**

---

## El problema

Los procesadores cuánticos actuales tienen pocos qubits y bastante ruido. Si un circuito no
cabe en el chip, se puede **cortar** en fragmentos, ejecutarlos por separado y reconstruir el
resultado. Pero cortar cuesta: cada corte multiplica el número de subexperimentos, y con un
presupuesto fijo de mediciones, cada uno recibe menos.

La librería de referencia resuelve esto con una regla simple: *ejecuta directo si el circuito
cabe; si no cabe, corta lo mínimo indispensable*.

## La pregunta

**Si el circuito sí cabe, ¿conviene cortarlo igual?**

## El método

Cuatro familias de circuitos (QAOA/MaxCut, dinámica de espines, energía variacional y
circuitos modulares), cuatro límites de ancho de chip (2, 3, 4 y 6 qubits) y **20 semillas de
simulación** por combinación. Todo lo demás fijo: 6 qubits, ruido `mild`, 12 000 mediciones.

El error se separa en dos componentes que se comportan distinto:

- **Sesgo** — viene del ruido del hardware; no baja aunque midas más
- **Dispersión** — viene del muestreo; sí baja al medir más

## El hallazgo

Cortar más agresivamente **intercambia uno por otro**: los fragmentos son menos profundos
(menos sesgo) pero hay muchos más subexperimentos repartiéndose el presupuesto (más
dispersión). El mínimo del error total cae donde se cruzan, y ese punto **no es el mismo para
todas las familias**.

A partir de ahí se construye una regla que predice el mejor ancho **sin mirar nunca la
respuesta correcta**:

1. La varianza sale gratis — el error estándar que ya reporta el planificador correlaciona
   **0.98** con la dispersión medida
2. El sesgo se aproxima con una sola constante sobre la profundidad de dos qubits

La constante se ajustó **solo con QAOA** y se probó en las otras tres familias (prueba fuera
de muestra).

## Resultados

| Familia | La regla elige | Óptimo medido | ¿Acierta? |
|---|---|---|---|
| Espines | 3 | 4 | No — empate (0.0342 vs 0.0340) |
| Variacional | 6 | 6 | Sí |
| Modular | 6 | 3 | No — fallo real (0.0220 vs 0.0175) |

**1 de 3 aciertos exactos, pero 9.3 % menos error que la regla de referencia**, a 5.9 % del
óptimo alcanzable.

### Lo que no funcionó, y por qué importa

Al ejecutar el cuaderno en **dos máquinas distintas**, el conteo de aciertos cambió: el óptimo
de la familia de espines pasó de ancho 3 a ancho 4 por una diferencia de 0.0002 en RMSE.

Eso da la conclusión metodológica más útil del trabajo: **"¿acertó el ancho?" es una mala
métrica cuando el óptimo es plano.** Lo que importa es el error final, y esa cifra sí se
mantuvo estable entre máquinas (9.3 % contra 9.1 %).

El fallo en la familia modular está documentado en la sección 9 del cuaderno, junto con su
causa probable (el ruido entra por los SWAPs que introduce el ruteo, que la métrica usada no
cuenta) y la razón de **no haberlo corregido**: cambiar el modelo después de ver fallar el
conjunto de prueba invalidaría la prueba fuera de muestra.

## Limitaciones

- Una sola semilla de circuito y un solo tamaño (6 qubits)
- 20 réplicas: la desviación estándar estimada tiene ~16 % de error relativo
- Tres familias de prueba es una muestra minúscula
- El proxy del sesgo es deliberadamente tosco
- **Todo es simulación local con ruido sintético. No demuestra ninguna ventaja cuántica.**

## Cómo reproducirlo

Este repositorio contiene solo el trabajo propio. El material del reto lo distribuyen los
organizadores y no se redistribuye aquí.

1. Consigue `WQCC-Planner-Materials.zip` con los organizadores del hackathon
2. Descomprímelo en la raíz de este repositorio (debe quedar una carpeta `python/`)
3. Crea el entorno y ejecuta:

```bash
python3.12 -m venv .venv
.venv/bin/python -m pip install -r python/requirements.txt
.venv/bin/python python/check_environment.py   # debe dar "status": "pass"
.venv/bin/python -m jupyterlab
```

Abre `hackathon_wqcc_explorer_v2.ipynb` y ejecuta todas las celdas (~5 minutos). Las semillas
están fijas, así que los números se reproducen exactamente.

> **Nota sobre la versión de Python.** La guía recomienda 3.12. Los resultados publicados se
> generaron en 3.14, tras verificar que todas las dependencias del `requirements.txt` tienen
> wheels precompilados para esa versión en Apple Silicon.

## Contenido

| Archivo | Qué es |
|---|---|
| `hackathon_wqcc_explorer_v2.ipynb` | El cuaderno completo, ejecutado y con salidas |
| `Guia_Proyecto_Hackathon_WQCC.pdf` | Guía de estudio: explicación, glosario de términos y preguntas frecuentes |

## Uso de inteligencia artificial

Este proyecto se desarrolló con asistencia de **Claude (Anthropic)**. La declaración completa
está en la sección 13 del cuaderno. En resumen: la IA generó la mayor parte del código,
propuso el diseño experimental con réplicas y redactó las secciones explicativas. Las
decisiones sobre qué investigar, bajo qué condiciones, qué resultados reportar —incluidos los
fallos— y la decisión de no corregir el proxy tras verlo fallar fueron mías.

---

## English summary

**Does it pay to cut more than you have to?**

A study on quantum circuit cutting submitted to the WQCC Hackathon (Explorer track).

Cutting a circuit to fit a smaller chip trades **bias for variance**: fragments get shallower
(less hardware noise) but the fixed shot budget is split across far more subexperiments (more
sampling error). Across four circuit families, four width limits and 20 simulation seeds, the
crossover point differs by family.

A predictive rule combining the planner's own reported standard error (free, correlates 0.98
with measured spread) with a one-constant depth-based bias proxy was fitted on QAOA alone and
tested out-of-sample on the other three families: **1 of 3 exact width hits, but 9.3 % lower
error than the package's built-in rule**, within 5.9 % of the achievable optimum.

Running the notebook on two different machines flipped the hit count, revealing that **argmin
accuracy is a fragile metric when the optimum is flat** — the error-based score stayed stable.

All results are local simulation with synthetic noise. **No quantum advantage is claimed.**
