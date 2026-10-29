# 🖥️ Mesa IT · Dashboard Ejecutivo (Jira)

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-1557b0?style=for-the-badge)
![Power Query](https://img.shields.io/badge/Power%20Query%20(M)-217346?style=for-the-badge)
![Jira](https://img.shields.io/badge/Jira%20REST%20API-0052CC?style=for-the-badge&logo=jira&logoColor=white)
![Formato](https://img.shields.io/badge/formato-.pbip-555?style=for-the-badge)

> Tablero ejecutivo de la **mesa de ayuda IT** para varias plantas industriales. Toma los tickets **directo desde la API de Jira**, mide volumen, SLA, tiempos de atención y resolución, y detecta **fallas recurrentes** para atacar la causa raíz.

## 📌 Qué responde

- ¿Cuántos tickets entran y cuántos se resuelven por mes? ¿Crece o baja el backlog?
- ¿Cumplimos el **SLA** de atención y de resolución?
- ¿Cuánto tarda la **mediana** en atenderse y resolverse un ticket, por planta?
- ¿Qué tickets abiertos son los más antiguos (+30 días)?
- ¿Cuánto son **fallas** y cuánto **pedidos**? ¿Qué fallas se repiten 3 o más veces?
- ¿Qué áreas, componentes y personas generan más demanda? ¿Cuántas horas consume cada planta?

## 🔌 Extracción: Power Query contra la API de Jira

La consulta `Jira` llama a `rest/api/3/search/jql` con un **JQL** que filtra el proyecto y las plantas, y recorre todas las páginas con `List.Generate` usando el `nextPageToken`:

```m
Pages =
    List.Generate(
        () => GetPage(null),
        each _ <> null,
        each if Record.Field(_, "isLast") = true
             then null
             else GetPage(Record.Field(_, "nextPageToken")),
        each _
    ),
TodasLasIssues = List.Combine(List.Transform(Pages, each [issues]))
```

Después se expanden los campos (estado, prioridad, asignado, planta, componente, área requirente, SLA de atención y resolución, horas registradas) y se tipan las columnas.

## 📐 Medidas DAX (33)

| Grupo | Medidas |
|---|---|
| Volumen | Total Tickets, Tickets Abiertos / Cerrados, % Tickets Abiertos / Cerrados, Promedio Mensual |
| Prioridad | Tickets Alta, Tickets Bloqueantes, % Alta Cerrado, % Bloqueantes Cerrados |
| SLA | SLA Cumplidos, SLA Vencidos, **% SLA Cumplido** |
| Tiempos | Promedio y **Mediana** de Atención y Resolución, Antigüedad Abiertos, Abiertos +30 días |
| Flujo mensual | Tickets Creados (mes), Tickets Resueltos (mes), **Saldo del mes** |
| Causa raíz | Fallas, Pedidos, % Fallas, Veces Repetida, **Fallas Recurrentes** |
| Esfuerzo | Horas Trabajadas, Horas por Ticket |

```dax
-- Títulos normalizados (minúsculas, sin punto final) para agrupar tickets repetidos
Fallas Recurrentes =
COUNTROWS ( FILTER ( VALUES ( Jira[Resumen Normalizado] ), [Fallas] >= 3 ) ) + 0
```

- **Creados vs. resueltos** se comparan sobre una tabla `Meses` sin relaciones, cruzada con `TREATAS`, para que cada medida use su propia fecha (creación o resolución).
- `Estado Operativo` agrupa los estados de Jira en *Abierto / En trabajo / Cerrado*.
- `Categoría` separa **Falla** (Problema, Incidente) de **Pedido / Servicio**.

## 📄 Páginas

1. **Indicadores**: KPIs principales y SLA
2. **Performance**: tiempos de atención y resolución por planta
3. **Resumen Operativo**: estado, prioridad, backlog y abiertos más antiguos
4. **Análisis Operativo**: creados vs. resueltos, fallas vs. pedidos, fallas recurrentes, horas por planta

## 🔒 Datos

El repositorio incluye **solo el modelo, las consultas y el diseño del informe**. No incluye tickets, `.pbix` ni caché, y los nombres del equipo técnico se reemplazaron por `Técnico 1…7`. Para usarlo hace falta acceso a una instancia de Jira con credenciales propias.

## ▶️ Cómo abrirlo

1. Abrir `Mesa IT - Dashboard Ejecutivo Jira.pbip` con Power BI Desktop.
2. Cargar credenciales de Jira (cuenta + API token) cuando lo pida.
3. Actualizar.
