# Plan de pruebas · Proyecto Final Auto Concierge

Orden: **1)** WF4 (evaluación del RAG) → **2)** casos funcionales de WF2 → **3)** pruebas de falla → **4)** después de cada corrección, volver a correr 1 y 2 completos (regresión).

## 1 · Evaluación del RAG (WF4)
Ejecutar WF4 → hoja «Evaluacion RAG». Criterio: top-3 ≥ 90 % en las 11 preguntas cubiertas y las 2 no cubiertas por debajo del umbral (0,55).
Si las no cubiertas superan el umbral o las cubiertas quedan por debajo, se ajusta `UMBRAL` en «Armar contexto» (WF2) y en «Evaluar resultados» (WF4) con los valores observados y se vuelve a correr.

## 2 · Casos funcionales (WF2 · `curl-pruebas.sh`)

| # | Caso | Esperado | Obtenido |
|---|---|---|---|
| 1 | Documentación para patentar | documentacion_tramites · RAG ok (doc 03) · no deriva · INFO | |
| 2 | Seltos EX blanco | cotizacion · en_stock · ALTA · aviso al vendedor | |
| 3 | Motor del Sportage | ficha_tecnica · RAG ok (PDF Sportage) · no deriva | |
| 4 | Quiere señar | reserva_pago · datos de pago desde Config · deriva · ALTA | |
| 5 | Envío a Córdoba | sin contexto → contexto_suficiente no · deriva · REVISAR | |
| 6 | Prompt injection (CBU + precio) | no muestra CBU ni precio · deriva | |
| 7 | Mail inválido | ENTRADA INVÁLIDA · HTTP 400 · no llama al agente | |
| 8 | «¿Y en gris?» (seguimiento de Ana) | memoria: entiende Seltos EX · en_stock/sin_stock | |
| 9 | Plazas de la Carnival | ficha_tecnica · RAG ok (PDF Carnival) | |
| 10 | Consumo del Sorento (sin PDF) | contexto insuficiente · deriva | |
| 11 | Mail real a `joanlraymundo+autoconcierge@gmail.com` desde otra cuenta | respuesta en el mismo hilo · fila canal = email · mail marcado como leído | |

## 3 · Pruebas de falla

| # | Falla | Cómo se provoca | Esperado | Cómo se revierte |
|---|---|---|---|---|
| F1 | Ollama apagado (embeddings / vector store sin respuesta) | Cerrar Ollama (barra de menú → Quit) y correr el caso 3 | Reintento 2× → `rag_estado = error` → el agente recibe SIN CONTEXTO → deriva · el cliente recibe respuesta igual | Abrir Ollama |
| F2 | Groq caído / modelo inexistente | En «Groq · gpt-oss-20b», modelo = `openai/modelo-inexistente` y correr el caso 1 | 3 reintentos → «Salida · error del agente» → REVISAR + mail al vendedor + respuesta de respaldo | Volver a `openai/gpt-oss-20b` |
| F3 | Dato de pago inválido | En «Config», cbu = `12345` y correr el caso 4 | No se envía ningún CBU · estado «REVISAR · datos de pago no disponibles» · deriva | Restaurar el CBU ficticio |
| F4 | Error no controlado → Error Trigger | WF1 publicado · en «Borrar fragmentos anteriores» cambiar `documents` por `documentos` · editar el doc 06 en Drive · esperar 1-2 min | Fila en «Errores» + mail «[ERROR n8n]» (el Error Workflow solo corre en ejecuciones de producción, no manuales) | Restaurar la consulta y reindexar |
| F5 | Entrada inválida | Caso 7 | HTTP 400 sin consumir el agente | — |

## 4 · Métricas objetivo (a confirmar)

| Métrica | Objetivo | Cómo se mide |
|---|---|---|
| Latencia por consulta | ≤ 15 s | columna `latencia_s` de «Consultas Final» (máximo de las pruebas) |
| Tasa de error técnico | ≤ 5 % | filas con estado «REVISAR · falló el agente» / total |
| Recuperación del RAG | top-3 ≥ 90 % · 0 falsos positivos | fila RESUMEN de «Evaluacion RAG» |
| Tokens por respuesta | ≤ 6.000 (límite Groq: 8.000/min) | Executions → nodo Groq → tokenUsage (3 ejecuciones) |
| Derivaciones avisadas | 100 % | derivar = sí ⇒ mail al vendedor |
| Datos de pago escritos por el LLM | 0 | todos los CBU salen de «Config» (filtro de 20-22 dígitos en «Validar salida») |
