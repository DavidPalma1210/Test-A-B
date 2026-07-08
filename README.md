# Test-A-B
Proyecto Final: Análisis del Test A/B


# Análisis de Test A/B — Sistema de recomendaciones

Evaluación de un experimento A/B (`recommender_system_test`) que probó un nuevo sistema de recomendaciones frente al sistema actual, midiendo el impacto en la conversión a lo largo del embudo `login → product_page → product_cart → purchase`.

## Objetivo

Determinar si el grupo de prueba (B, con el nuevo sistema de recomendaciones) mejora la conversión respecto al grupo de control (A), con una mejora esperada de al menos **10%** en las etapas clave del embudo, y validar si el experimento fue ejecutado correctamente antes de tomar esa decisión.

| Parámetro | Valor esperado |
|---|---|
| Región objetivo | EU |
| Periodo de registro de usuarios | 2020-12-07 a 2020-12-21 |
| Periodo de eventos | 2020-12-07 a 2021-01-01 |
| Ventana de análisis por usuario | 14 días desde el registro |
| Participantes esperados | 6,000 (15% de nuevos usuarios EU) |
| Nivel de significancia | α = 0.05 |

## Datos

| Archivo | Descripción |
|---|---|
| `ab_project_marketing_events_us.csv` | Calendario de campañas de marketing 2020 |
| `final_ab_new_users_upd_us.csv` | Usuarios registrados entre el 7 y el 21 de diciembre de 2020 |
| `final_ab_events_upd_us.csv` | Eventos de los nuevos usuarios (7 dic 2020 – 1 ene 2021) |
| `final_ab_participants_upd_us.csv` | Usuarios participantes en los tests A/B |

## Metodología

1. Validación del diseño experimental (usuarios en más de un grupo/test, tamaño de muestra, balance A/B).
2. Revisión de campañas de marketing activas durante el periodo del test.
3. Filtrado de usuarios y eventos válidos (región, fechas, ventana de 14 días).
4. Análisis exploratorio: usuarios por grupo, eventos por tipo, eventos por día, **eventos por usuario entre grupos**.
5. Construcción del embudo de conversión por grupo.
6. Prueba Z de proporciones por etapa del embudo, con corrección de Bonferroni por comparaciones múltiples.
7. Prueba de Mann-Whitney U para comparar la distribución de eventos por usuario entre grupos.

## Hallazgos clave

**Calidad del experimento — con problemas serios:**
- Se esperaban 6,000 participantes; se obtuvieron **3,481 usuarios válidos** (58% de lo planeado, 8.82% de los nuevos usuarios EU vs. el 15% esperado).
- Grupos desbalanceados: A = 2,604 vs. B = 877 (≈75/25 en vez de 50/50).
- **887 usuarios** participaron simultáneamente en más de un test A/B (riesgo de contaminación cruzada).
- La campaña *"Christmas & New Year Promo"* (EU) se traslapa con el tramo final del periodo del experimento.
- 0 usuarios quedaron asignados a más de un grupo dentro del test objetivo (correcto).

**Eventos por usuario — no distribuidos equitativamente:**
- Mediana de 6 eventos/usuario en A vs. 4 en B (medias: 6.62 vs. 5.38).
- Mann-Whitney U: diferencia altamente significativa (p ≈ 1.03e-20).

**Embudo de conversión — B por debajo de A en las 4 etapas:**

| Etapa | Conversión A | Conversión B | Lift B vs A |
|---|---:|---:|---:|
| login | 100.00% | 99.89% | -0.11% |
| product_page | 64.71% | 56.21% | **-13.13%** |
| product_cart | 30.03% | 27.82% | -7.35% |
| purchase | 31.99% | 28.39% | -11.24% |

**Prueba Z de proporciones (α = 0.05):**
- `product_page`: diferencia significativa (p = 0.000007), se mantiene tras corrección de Bonferroni.
- `purchase`: significativa sin corrección (p = 0.0465), pero **no** tras Bonferroni (α ajustado = 0.0125).
- `login` (p = 0.0848) y `product_cart` (p = 0.2147): sin diferencia significativa.

## Conclusión

El test **no fue exitoso**: el grupo B no mejora la conversión en ninguna etapa del embudo y empeora de forma significativa en `product_page`. Además, la calidad del experimento (muestra insuficiente, grupos desbalanceados, usuarios en múltiples tests, campaña de marketing solapada) limita la confianza en cualquier resultado, positivo o negativo, que hubiera arrojado.

## Recomendación de negocio

1. **No liberar el cambio a producción** — el riesgo de empeorar la conversión ya se observó en los datos.
2. **Repetir el test A/B** corrigiendo el diseño: grupos balanceados 50/50, exclusión de usuarios en tests simultáneos, y evitar ventanas con campañas de marketing activas en la región objetivo.
3. Si el resultado se repite con un diseño correcto, el sistema de recomendaciones necesita un rediseño antes de una nueva prueba.

## Stack técnico

Python · pandas · NumPy · SciPy (prueba Z, Mann-Whitney U) · Matplotlib

## Contenido del repositorio

```
├── notebooks/
│   └── 01_planificacion_test_ab.ipynb   # Notebook completo: planificación, EDA, embudo, pruebas estadísticas
├── presentation/
│   └── ab_test_conclusiones.pdf         # Resumen ejecutivo (10 slides)
├── data/
│   └── README.md                        # Origen y descripción de los datos (no incluidos por tamaño)
└── requirements.txt
```

> Los archivos de datos originales no se incluyen en este repositorio por su tamaño; ver `data/README.md` para su origen.