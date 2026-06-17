# Métricas para optimización de backtests: pre-cashflow y post-cashflow

Este documento define las métricas del workbook `metricas_optimizacion_cashflow.xlsx` con la lógica corregida. Hay exactamente dos fases:

| Fase | Regla |
|---|---|
| `Pre-cashflow` | La métrica puede calcularse antes de construir cashflow, equity, capital neto, costos monetarios o drawdown sobre equity. La excepción válida son métricas expresadas en múltiplos de R, por ejemplo `Expectancy > 0.3R`. |
| `Post-cashflow` | La métrica requiere dinero, equity, net profit, costos, sizing, leverage, drawdown o retornos de equity después de simular el cashflow. |

No existe una fase intermedia. Si una métrica puede existir en versión bruta y neta, este workbook usa la versión indicada en la fórmula y deja explícita su dependencia.

## Métricas detalladas

| Categoria | Fase | Metrica | Depende de | Formula/Calculo | Rango saludable | Uso |
|---|---|---|---|---|---|---|
| RENTABILIDAD (Performance Core) | Post-cashflow | Net PnL | Gross PnL, fees, funding, slippage y costos ejecutados | `sum(net_pnl)` o `sum(gross_pnl) - fees - funding - slippage_cost` | `> 0` y siempre validado contra drawdown | Confirmar si el edge sobrevive a fricciones reales |
| RENTABILIDAD (Performance Core) | Post-cashflow | ROI | Net PnL y capital inicial/final | `NetPnL / CapitalInicial` | Positivo y consistente con riesgo asumido | Ranking final de retorno sobre capital real |
| RENTABILIDAD (Performance Core) | Pre-cashflow | Expectancy | Resultados por trade expresados en múltiplos de R | `mean(R)` o `(WinRate * AvgWinR) - (LossRate * AvgLossR)` | `> 0.3R` | Validar edge antes de dinero, fees y equity |
| RENTABILIDAD (Performance Core) | Post-cashflow | Profit Factor | Ganancias y pérdidas monetarias después de costos | `GrossProfitNet / abs(GrossLossNet)` | `> 1.5` | Medir calidad del profit después del cashflow |
| RIESGO (Survival Metrics) | Post-cashflow | Max Drawdown (MDD) | Equity curve, picos y valles de capital | `max((Peak - Valley) / Peak)` | `< 30%` | Filtro duro de supervivencia del sistema |
| RIESGO (Survival Metrics) | Post-cashflow | Recovery Factor | Net profit y max drawdown monetario | `NetProfit / abs(MaxDrawdown)` | `> 2` | Preferir retorno eficiente por unidad de drawdown |
| RIESGO (Survival Metrics) | Post-cashflow | Risk of Ruin | Capital, sizing, win/loss profile y riesgo por trade | Modelo de ruina basado en capital real y `risk_per_trade` | Cercano a `0` | Validar si el sizing puede quebrar la cuenta |
| EFICIENCIA OPERATIVA | Pre-cashflow | Win Rate | Trades ganadores y total de trades | `Wins / TotalTrades` | No usar aislado; interpretar con R:R y expectancy | Diagnosticar perfil operativo del sistema |
| EFICIENCIA OPERATIVA | Pre-cashflow | Average R | Ganancia/pérdida por trade expresada en R | `mean(trade_R)` o `AvgWinR / abs(AvgLossR)` según análisis | Mayor es mejor si no depende de outliers | Medir recompensa promedio antes del cashflow |
| EFICIENCIA OPERATIVA | Pre-cashflow | Trade Frequency | Cantidad de trades y período evaluado | `Trades / periodo` | Suficiente muestra sin sobreoperar | Detectar baja significancia o exceso de churn |
| CALIDAD ESTADISTICA | Post-cashflow | Sharpe Ratio | Retornos de equity, risk-free y volatilidad | `(Return - Rf) / std(equity_returns)` | `> 1.5` | Evaluar retorno ajustado a volatilidad total |
| CALIDAD ESTADISTICA | Post-cashflow | Sortino Ratio | Retornos de equity y downside deviation | `(Return - Rf) / downside_std(equity_returns)` | `> 1.5` | Penalizar sólo volatilidad negativa del equity |
| CALIDAD ESTADISTICA | Post-cashflow | Standard Deviation | Retornos de equity post-cashflow | `std(equity_returns)` | Menor para igual retorno | Medir volatilidad real del capital |
| CALIDAD ESTADISTICA | Pre-cashflow | Skewness | Distribución de retornos por trade en R | `skewness(trade_R)` | Preferible positiva; negativa exige revisión | Detectar colas asimétricas antes del cashflow |
| ROBUSTEZ ESTRUCTURAL | Pre-cashflow | Largest Win / Largest Loss | Mayor trade ganador y mayor trade perdedor en R | `max(win_R) / abs(min(loss_R))` | No debe explicar todo el resultado | Detectar dependencia de eventos extremos |
| ROBUSTEZ ESTRUCTURAL | Pre-cashflow | Consecutive Losses (Max) | Secuencia de outcomes por trade antes del cashflow | `max(loss_streak)` sobre resultados de trades | Soportable por capital luego del sizing | Anticipar rachas que el sistema deberá soportar |
| ROBUSTEZ ESTRUCTURAL | Post-cashflow | Equity Curve Slope | Equity curve post-cashflow en el tiempo | Pendiente de regresión lineal sobre equity | Pendiente positiva y estable | Priorizar crecimiento consistente del capital |
| ESPECIFICAS CRYPTO | Post-cashflow | Funding Impact % | Funding pagado/cobrado y PnL monetario | `abs(Funding) / abs(GrossPnL)` | `< 15%` | Penalizar estrategias destruidas por funding |
| ESPECIFICAS CRYPTO | Post-cashflow | Fee Impact % | Fees y PnL monetario | `Fees / abs(GrossPnL)` | `< 20%` | Medir si la frecuencia queda anulada por comisiones |
| ESPECIFICAS CRYPTO | Post-cashflow | Slippage % | Precio esperado, precio ejecutado y tamaño | `abs(ExecutedPrice - ExpectedPrice) / ExpectedPrice` | Lo menor posible | Validar ejecución realista post-costos |
| ESPECIFICAS CRYPTO | Post-cashflow | Liquidation Distance % | Entry, liquidation price, leverage y modelo de margen | `abs(Entry - LiqPrice) / Entry` | Suficientemente amplio para el leverage usado | Evitar configuraciones que sobreviven sólo en papel |

## Rangos de filtro

La hoja `Rangos` contiene sólo estos 10 filtros resumen.

| Metrica | Rango saludable | Fase | Uso |
|---|---|---|---|
| Profit Factor | `> 1.5` | Post-cashflow | Calidad del profit neto después de costos |
| Expectancy | `> 0.3R` | Pre-cashflow | Edge expresado en múltiplos de R antes del cashflow |
| Max DD | `< 30%` | Post-cashflow | Supervivencia del equity |
| Sharpe | `> 1.5` | Post-cashflow | Retorno ajustado a volatilidad |
| Recovery Factor | `> 2` | Post-cashflow | Retorno por unidad de drawdown |
| Fee Impact | `< 20%` | Post-cashflow | Control de comisiones |
| Consecutive Losses | Soportable por capital | Pre-cashflow | Rachas esperadas antes de aplicar sizing |
| Sortino | `> 1.5` | Post-cashflow | Retorno ajustado a downside risk |
| Funding Impact | `< 15%` | Post-cashflow | Control del costo de mantener posiciones |
| Liquidation Distance | Suficientemente amplio | Post-cashflow | Margen de seguridad contra liquidación |

## Por qué `avg` no alcanza

Para análisis estadístico serio, una media simple puede ocultar outliers, colas pesadas y regímenes distintos. Cada métrica crítica debería guardar una distribución resumida con:

| Estadistico | Uso |
|---|---|
| `mean` | Centro promedio, útil pero sensible a outliers |
| `median` | Caso típico robusto |
| `std` | Dispersión general |
| `downside_std` | Dispersión negativa para Sortino |
| `p05` | Percentil bajo para stress testing |
| `p25` | Cuartil inferior |
| `p75` | Cuartil superior |
| `p95` | Percentil alto |
| `min` | Peor observación |
| `max` | Mejor observación |
| `skewness` | Asimetría de la distribución |
| `kurtosis` | Peso de colas y extremos |
| `n` | Tamaño de muestra |

Una estrategia con `Profit Factor` medio de `1.8` pero `p25 = 0.9` puede ser más frágil que otra con media `1.5` y `p25 = 1.3`. Por eso los filtros no deberían depender sólo de `avg`.

## Orden de categorías

El workbook mantiene este orden fijo:

1. RENTABILIDAD (Performance Core)
2. RIESGO (Survival Metrics)
3. EFICIENCIA OPERATIVA
4. CALIDAD ESTADISTICA
5. ROBUSTEZ ESTRUCTURAL
6. ESPECIFICAS CRYPTO
