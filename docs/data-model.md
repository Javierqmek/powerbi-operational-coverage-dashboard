# Data Model — Operational Coverage Analytics

## Esquema

Modelo dimensional tipo **Star Schema**, con una tabla de hechos central conectada a cinco dimensiones.

```
                Dim_Date
                    │
Dim_Client ──── Fact_Coverage ──── Dim_IncidentType
                    │
             Dim_Employee
                    │
             Dim_Coordinator
```

## Tablas

### Fact_Coverage
Tabla de hechos con granularidad diaria por cliente/puesto.

| Campo | Tipo | Descripción |
|---|---|---|
| Date_Key | Fecha | Relación con Dim_Date |
| Client_Key | Texto/ID | Relación con Dim_Client |
| Employee_Key | Texto/ID | Relación con Dim_Employee (anonimizado) |
| Coordinator_Key | Texto/ID | Relación con Dim_Coordinator |
| IncidentType_Key | Texto/ID | Relación con Dim_IncidentType |
| Total_Positions_Required | Número | Puestos requeridos por el contrato |
| Covered_Positions | Número | Puestos efectivamente cubiertos |
| Uncovered_Positions | Número | Puestos no cubiertos |
| Is_Incident | Booleano | Indica si el registro corresponde a una incidencia |
| Is_Covered_Incident | Booleano | Indica si la incidencia ya fue cubierta |

### Dim_Client
Catálogo de clientes/sedes (anonimizado como "Cliente Alfa", "Cliente Beta", etc.).

| Campo | Descripción |
|---|---|
| Client_Key | Identificador único |
| Client_Name | Nombre anonimizado del cliente |
| Zone | Zona geográfica/operativa |

### Dim_Date
Calendario estándar para análisis temporal.

| Campo | Descripción |
|---|---|
| Date_Key | Fecha (clave) |
| Day, Month, Year | Componentes de fecha |
| Month_Name | Nombre del mes |
| Is_Weekend | Indicador de fin de semana |

### Dim_Employee
Catálogo de personal, anonimizado en la versión pública ("Persona 01", "Persona 02", ...).

| Campo | Descripción |
|---|---|
| Employee_Key | Identificador anonimizado |
| Fault_Count | Conteo de faltas (agregado de referencia) |

### Dim_IncidentType
Catálogo de tipos de incidencia.

| Campo | Descripción |
|---|---|
| IncidentType_Key | Identificador único |
| IncidentType_Name | Falta, Descanso Médico, Renuncia, Licencia por Fallecimiento, Descanso por Paternidad, Servicio Especial, Otros |

### Dim_Coordinator
Catálogo de coordinadores de operaciones responsables por cliente/zona.

| Campo | Descripción |
|---|---|
| Coordinator_Key | Identificador único |
| Coordinator_Name | Nombre anonimizado del coordinador |

## Medidas DAX principales

```dax
Total Incidencias =
COUNTROWS(FILTER(Fact_Coverage, Fact_Coverage[Is_Incident] = TRUE()))

Puestos No Cubiertos =
SUM(Fact_Coverage[Uncovered_Positions])

Incidencias Cubiertas =
COUNTROWS(
    FILTER(Fact_Coverage, Fact_Coverage[Is_Covered_Incident] = TRUE())
)

Coverage Rate =
DIVIDE(
    [Covered Positions],
    [Total Required Positions],
    0
)
```

## Relaciones

- `Fact_Coverage[Date_Key]` → `Dim_Date[Date_Key]` (muchos a uno)
- `Fact_Coverage[Client_Key]` → `Dim_Client[Client_Key]` (muchos a uno)
- `Fact_Coverage[Employee_Key]` → `Dim_Employee[Employee_Key]` (muchos a uno)
- `Fact_Coverage[Coordinator_Key]` → `Dim_Coordinator[Coordinator_Key]` (muchos a uno)
- `Fact_Coverage[IncidentType_Key]` → `Dim_IncidentType[IncidentType_Key]` (muchos a uno)

Todas las relaciones son de una sola dirección (single direction), siguiendo las buenas prácticas de modelado en Power BI para evitar ambigüedad de filtros.
