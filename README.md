[README_1.md](https://github.com/user-attachments/files/30431479/README_1.md)
# Precios Hacienda

Proyecto Power BI (formato **.pbip / TMDL**) para el seguimiento y análisis de precios de hacienda (mercado ganadero), con series históricas por categoría de animal, precio en pesos y dólares, y variaciones en el tiempo.

## Estructura del proyecto

```
Precio Hacienda/
├── Precios Hacienda.pbip
├── Precios Hacienda.Report/          # Layout y visuales del reporte
└── Precios Hacienda.SemanticModel/   # Modelo semántico (TMDL)
    └── definition/
        ├── database.tmdl
        ├── model.tmdl
        ├── tables/
        └── relationships.tmdl
```

Al estar en formato TMDL, los cambios al modelo (tablas, medidas, relaciones) se versionan como texto plano y son visibles en `git diff`.

## Modelo de datos

### Tablas

| Tabla | Rol | Descripción |
|---|---|---|
| **vw_precio_hacienda** | Hechos | Tabla de hechos con el histórico de precios: fecha, fuente, animal, categoría, moneda, precio, y detalle de gancho/pie. |
| **Calendario** | Dimensión | Tabla de fechas con jerarquías de tiempo (Año, Trimestre, Semestre, Mes, Quincena, Semana, Día). |
| **Productos** | Dimensión | Catálogo de productos/categorías con coeficiente y operación asociada para cálculos. |
| **Bridge_ProductoCategoria** | Puente | Tabla puente entre productos y categorías (por fuente/moneda). |
| **Bridge** | Puente (oculta) | Tabla puente auxiliar de la relación Calendario. |
| **Medidas** | Tabla de medidas | Tabla contenedora de todas las medidas DAX del modelo (no contiene datos). |

### Relaciones

- `vw_precio_hacienda[fecha]` → `Calendario[Date]`
- `Calendario[Llave]` → `Bridge[BridgeKey]`
- `Productos[Llave]` → `Calendario[Llave]`

### Medidas

Las medidas están organizadas en carpetas de visualización (`Display Folder`) por categoría de animal/producto, entre ellas:

- **Novillo Exp** (Gancho / Pie)
- **Ternero AA**, **Ternera Cruza**, **Vaca con cría**
- **Vaca Manuf**, **Vaca Conserva** (Gancho / Pie)
- **Novillito 391/430**, **NOVILLOS 431/460 · 461/490 · 491/520 · +520** (Pesos / Dólares)
- **Novillito -300/390 (-350)**, **Vaquillona (270-390)**
- **Motor Genérico**: conjunto de medidas reutilizables que calculan precio base, valores de meses/trimestres/semestres/años anteriores, diferencias y variaciones % (mensual, 3M, 6M, Q1, Q2, anual), tanto en versión puntual como promedio.

#### Gancho, Pie y Pie en dólares

Varias familias de medidas (Novillo Exp, Vaca Manuf, Vaca Conserva, etc.) exponen el precio en dos variantes:

- **Gancho**: precio base, calculado como el promedio mensual de `vw_precio_hacienda[precio]`, filtrado por fuente y categoría correspondiente. Es el valor de referencia sin ajuste.
  ```DAX
  CALCULATE(
      AVERAGE(vw_precio_hacienda[precio]),
      vw_precio_hacienda[fuente] = "CCDH",
      vw_precio_hacienda[categoria] = "<categoría>"
  )
  ```
- **Pie**: resultado de multiplicar la medida **Gancho** por un coeficiente de rendimiento gancho→pie, que varía según la categoría (ej. `0.585` para Novillo Exp, `0.5` para Vaca Manuf).
  ```DAX
  [<Medida> Gancho] * <coeficiente>
  ```
- **Pie en dólares**: actualmente **no existe** como medida calculada a partir de Pie. La tabla `vw_precio_hacienda` tiene una columna `pie_dolares`, pero ninguna medida del modelo la referencia hoy. Las series en dólares que sí están calculadas (Novillito 391/430, NOVILLOS 431/460 · 461/490 · 491/520 · +520) son medidas independientes que filtran directamente `vw_precio_hacienda[moneda] = "Dolares"`, no una conversión del valor en Pie.

## Requisitos

- Power BI Desktop (versión con soporte para formato `.pbip` / TMDL).
- Acceso a la fuente de datos de `vw_precio_hacienda` para refrescar el modelo.

## Cómo trabajar con este repo

1. Abrir `Precios Hacienda.pbip` con Power BI Desktop.
2. Hacer los cambios necesarios en el modelo o el reporte.
3. Guardar desde Power BI Desktop (esto actualiza los archivos TMDL en disco).
4. Revisar los cambios antes de commitear:
   ```bash
   git status
   git diff
   ```
5. Commitear y subir:
   ```bash
   git add .
   git commit -m "Descripción del cambio"
   git push
   ```
