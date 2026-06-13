# Optimización de Tasas para Créditos Vehiculares

> Caso de Pricing Analytics aplicado a créditos vehiculares aprobados. El proyecto estima la probabilidad de aceptación de una oferta, simula distintos niveles de APR/TEA y propone una tasa óptima por perfil, conectando la decisión de pricing con una matriz accionable para CRM/campañas.

**Autor(a):** Valeria Thais Ureta Llanque  
**Stack:** Python 3.12 | pandas | numpy | statsmodels | scikit-learn | matplotlib | plotly | openpyxl

---

# Problema identificado

En créditos vehiculares, la aprobación crediticia no garantiza que el cliente acepte la oferta. Una entidad financiera puede aprobar una solicitud y ofrecer una tasa APR/TEA, pero el cliente puede rechazarla o no responder dentro del periodo de vigencia.

El problema central es que una tasa demasiado alta puede mejorar el spread financiero, pero reducir la probabilidad de aceptación. En cambio, una tasa baja puede aumentar la conversión, pero disminuir la rentabilidad por operación.

El objetivo del proyecto es construir un flujo analítico de pricing que permita estimar la probabilidad de aceptación de una oferta, simular escenarios de tasa y generar una recomendación accionable para priorizar segmentos en pilotos o campañas comerciales.

**Dataset:** `AutoLoanData.csv`
**Unidad de análisis:** solicitud de crédito vehicular aprobada
**Variable objetivo:** `Funded`

* `1`: solicitud financiada / aceptada
* `0`: solicitud no financiada / no aceptada

---

## Flujo de negocio

El caso se entiende dentro de un flujo bancario:

```text
Riesgos → Pricing → CRM / Campañas
```

* **Riesgos** aporta variables asociadas al perfil del cliente, como score, nivel de riesgo, monto y plazo aprobado.
* **Pricing** utiliza esa información para definir una tasa APR/TEA que equilibre aceptación y rentabilidad esperada aproximada.
* **CRM/canales** podrían usar el output final para priorizar pilotos, campañas o acciones diferenciadas por segmento.

Este proyecto no modela aprobación crediticia, default ni pérdida esperada. El análisis parte de solicitudes ya aprobadas y se enfoca en la aceptación de la tasa ofrecida.

---

## Estructura del proyecto

```text
.
├── optimizacion_tasas.ipynb                    # Notebook principal: EDA, modelo de propensión y optimización de tasa
├── AutoLoanData.csv                            # Dataset original
├── output_pricing.xlsx                         # Output accionable para Pricing y CRM
├── requirements.txt
├── README.md
└── assets/
    └── matriz_visual_pricing.png                 # Imagen de la matriz visual de priorización
```

---

## Metodología

El proyecto se desarrolló en cuatro etapas:

1. **EDA orientado a pricing**

   * Revisión de la tasa de aceptación base.
   * Análisis de variables como APR, PrimeRate, Tier, FICO, monto, plazo, tipo de vehículo y canal/origen.
   * Identificación del problema de negocio: no toda solicitud aprobada termina en financiamiento.

2. **Modelo de propensión a la aceptación**

   * Entrenamiento de una regresión logística para estimar la probabilidad de aceptación de una oferta.
   * Interpretación del modelo como probabilidad de adquisición, no como modelo de riesgo crediticio.
   * Evaluación de la capacidad del modelo para ordenar solicitudes según propensión.

3. **Simulación y optimización de tasa**

   * Simulación de distintos niveles de APR/TEA.
   * Cálculo de una rentabilidad esperada aproximada usando spread financiero y probabilidad de aceptación.
   * Estimación de una tasa óptima por perfil.

4. **Output accionable para negocio**

   * Construcción de una matriz visual de priorización.
   * Generación de KPIs por segmento.
   * Exportación de un archivo Excel para facilitar el uso de resultados por Pricing y CRM/canales.

---

## Output accionable para Pricing y CRM

Se generó un archivo Excel con tres hojas:

| Hoja                  | Descripción                                                                                                                    |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `matriz_visual`       | Matriz 2x2 de priorización según probabilidad de aceptación y profit esperado aproximado.                                      |
| `matriz_segmentos`    | KPIs agregados por segmento, incluyendo APR actual, APR óptima, brecha APR, probabilidad esperada, spread y profit aproximado. |
| `detalle_solicitudes` | Resultado a nivel de solicitud, con tasa actual, tasa óptima, probabilidad de aceptación y acción sugerida.                    |

La matriz visual permite clasificar segmentos en cuatro cuadrantes:

| Cuadrante                       | Interpretación                                              | Acción sugerida           |
| ------------------------------- | ----------------------------------------------------------- | ------------------------- |
| Alta probabilidad + alto profit | Segmentos atractivos por conversión y rentabilidad esperada | Priorizar campaña         |
| Alta probabilidad + bajo profit | Buena conversión, pero margen limitado                      | Mantener / revisar margen |
| Baja probabilidad + alto profit | Buen margen potencial, pero baja aceptación                 | Testear ajuste de tasa    |
| Baja probabilidad + bajo profit | Bajo atractivo comercial                                    | No priorizar              |

### Matriz visual de priorización

> Inserta aquí la imagen exportada de tu Excel o una captura de la matriz visual.

```markdown
![Matriz visual de priorización](assets/matriz_visual_pricing.png)
```

---

## Principales hallazgos

* La base contiene **152,963 solicitudes aprobadas**, pero solo cerca del **17.2%** termina en financiamiento, evidenciando que la aprobación crediticia no garantiza adquisición.
* La tasa APR/TEA cumple un rol clave en la conversión: tasas más altas pueden mejorar el spread, pero tienden a reducir la probabilidad de aceptación.
* El modelo de propensión permite ordenar solicitudes según probabilidad estimada de adquisición, lo que facilita la simulación de tasas por perfil.
* La tasa óptima no es única para todos los clientes; depende del perfil, plazo, tipo de vehículo, nivel de riesgo y contexto de fondeo.
* El output final traduce el análisis en una matriz accionable para priorizar segmentos y diseñar pilotos o campañas desde Pricing hacia CRM/canales.

---

## Limitaciones

* El profit utilizado es una aproximación basada en spread financiero y probabilidad de aceptación.
* No se incorporan pérdida esperada, costo operativo, costo de capital, mora, recuperaciones ni prepagos.
* El modelo estima aceptación de oferta, no aprobación crediticia ni probabilidad de default.
* La simulación de tasa aprende patrones históricos y no prueba causalidad.
* Las recomendaciones deberían validarse mediante pilotos/control antes de escalarse comercialmente.

---

## Métricas del modelo

| Métrica                 |  Valor |
| ----------------------- | -----: |
| Tasa base de aceptación |  17.2% |
| AUC en test             | 0.8731 |
| Accuracy en test        | 0.8885 |
| Precision en test       | 0.7664 |
| Recall en test          | 0.5164 |

El modelo no se evalúa solo por accuracy, ya que la base presenta desbalance entre solicitudes aceptadas y no aceptadas. Para pricing, el valor principal está en ordenar clientes por probabilidad de aceptación y alimentar la simulación de tasas.

---

## Conclusión

El proyecto conecta analítica, pricing y negocio mediante un flujo completo: análisis de solicitudes aprobadas, estimación de propensión a la aceptación, simulación de tasas y generación de un output accionable para Pricing y CRM. La principal contribución es mostrar cómo una decisión de tasa puede evaluarse considerando tanto conversión esperada como rentabilidad aproximada.
