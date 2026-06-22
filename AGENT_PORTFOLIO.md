# Agente: Portfolio Analytics — Tally Marketplace

## Propietario
Juan (Director de Estrategia) · Claude es el ejecutor responsable de los resultados.

## Objetivo
Cruzar las ventas reales de los clientes marketplace (Clientes_por_periodo.VentasList) con su retención (PeriodoRetencion) para construir una visión de **rentabilidad actual + potencial de expansión**, identificar el 20% de clientes que generan el 80% del valor, y mantener el dashboard estratégico actualizado.

---

## Fuentes de datos

| Fuente | Campo clave | Descripción |
|--------|-------------|-------------|
| `Clientes_por_periodo` | `VentasList` | Ventas MXN del cliente en ese período (col N) |
| `Clientes_por_periodo` | `PeriodoRetencion` | Retención de impuestos por período (col L) — proxy de actividad fiscal |
| `Clientes_por_periodo` | `IVA_pagar`, `ISR_pagar` | Impuestos determinados del período |
| `Clients_Load` | `Company_Id`, `ClientName`, `Owner` | Datos maestros del cliente |
| `Folder_Clientes` | `SubscriptionType` | Tipo de suscripción: Import/Export o Soft Landing |
| `Ventas_Map` | `PeriodID`, `Company_id`, `VentasList` | Hoja auxiliar cuando GViz omite col N |

**PeriodID format:** `YYYY-M_COMPANYID` → e.g. `2026-3_AZ005337`

---

## Métricas calculadas

### 1. NRR por cliente (Net Revenue Retention de ventas)
```
NRR_ventas = avg_sales_last3months / avg_sales_same3months_prev_year × 100
```
- > 100% → cliente en expansión (verde)
- 80–100% → contracción leve (amarillo)
- < 80% → contracción severa o inactivo (rojo)

### 2. Tasa de expansión actual (momentum)
```
expansion_rate = last_month_sales / avg_sales_3months_before × 100
```
Mide el impulso reciente, independiente del comparativo anual.

### 3. Revenue Score (absoluto)
```
revenue_score = sum(sales, periodos_2026)
```
Ventas totales acumuladas en 2026. Base para el ranking.

### 4. Priority Score (compuesto)
```
priority_score = revenue_score × (expansion_rate / 100)
```
Combina tamaño absoluto + momentum. Ordena la lista de atención.

### 5. Pareto / Whale Curve
- Ordenar todos los clientes por `revenue_score` DESC
- Calcular % acumulado de revenue vs % acumulado de clientes
- Threshold Pareto: primer punto donde % revenue ≥ 80%

---

## Lógica de filtros

- Incluir: **todos** los tipos (AZ*, IN*, MX*, CH*)
- Excluir: clientes sin ninguna venta en los últimos 6 meses (`revenue_score = 0`)
- Para la tabla Top 10 MoM 2026: solo meses del año 2026 (`periodKey.startsWith('2026')`)
- Para NRR: requiere al menos 3 meses de historia en 2025 Y datos en 2026

---

## Interpretación y acciones

| Cuadrante | NRR | Expansión | Acción |
|-----------|-----|-----------|--------|
| 🏆 Héroes | >100% | Alto | Upsell, caso de éxito, proteger |
| ⚠️ En riesgo | <80% | Bajo | QBR urgente, revisar operación |
| 🚀 Estrellas | >100% | Muy alto | Escalar soporte, anticipar necesidades |
| 😴 Dormidos | Variable | ~0 | Reactivación o revisión de contrato |

---

## Dashboard — secciones

| Tab | Contenido |
|-----|-----------|
| **Overview** | KPIs de salud del portafolio: clientes activos, % en expansión, revenue total 2026 |
| **NRR & Expansión** | Tabla de priorización, Top 10 MoM 2026 (multi-línea), distribución NRR |
| **Pareto + Whale Curve** | Whale curve acumulativa, tabla Pareto 20/80, lista de "héroes" |
| **Historia del Cliente** | Drill-down por cliente (mantenido del dashboard anterior) |

---

## Iteración del agente

Cuando Juan pida una actualización del dashboard:

1. Revisar si hay nuevos períodos en `Clientes_por_periodo` (meses nuevos en 2026)
2. Verificar que `Ventas_Map` esté actualizada (si GViz omite col N)
3. Recalcular las métricas con los datos nuevos
4. Actualizar la sección HTML correspondiente
5. Reportar: top 3 clientes que subieron, top 3 que bajaron vs mes anterior

---

## Historial de iteraciones

| Fecha | Cambio | Resultado |
|-------|--------|-----------|
| 2026-06-22 | v1 — Diseño inicial: NRR + Expansión + Pareto + Whale Curve | Dashboard publicado |
