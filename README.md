# README TÉCNICO

**Stored Procedure:** `[dbo].[UP_MAN_CF_LOTETICKET]`  
**Autor:** Julio Albert Bendezú Gutiérrez  
**Fecha:** 2025-12-05

## Descripción
Este documento resume el análisis técnico-funcional del Stored Procedure `UP_MAN_CF_LOTETICKET`. Su objetivo es proporcionar una referencia rápida sobre propósito, uso, reglas de negocio, parámetros y consideraciones para desarrolladores que integren o mantengan esta lógica en .NET, SQL Server u otras plataformas.

## Tabla de Contenidos
- [Resumen Ejecutivo](#resumen-ejecutivo)
- [Objetivo Funcional](#objetivo-funcional)
- [Parámetros Principales](#parámetros-principales)
- [Tablas Involucradas](#tablas-involucradas)
- [Reglas de Negocio](#reglas-de-negocio)
- [Flujo General por Acción](#flujo-general-por-acción)
- [Manejo de Errores y Transacciones](#manejo-de-errores-y-transacciones)
- [Mejoras Técnicas](#mejoras-técnicas)
- [Recomendaciones de Integración con .NET](#recomendaciones-de-integración-con-net)
- [Conclusión](#conclusión)

## Resumen Ejecutivo
`UP_MAN_CF_LOTETICKET` gestiona la creación, actualización y eliminación de lotes de tickets dentro de un flujo productivo industrial (costura, proveedores, familias de ítems, órdenes de compra y producción). Soporta las acciones:
- `I` — Insert
- `U` — Update
- `D` — Delete

Cada acción se ejecuta con validaciones para garantizar integridad, consistencia y cumplimiento de reglas de negocio.

## Objetivo Funcional
El Stored Procedure asegura que:
- Los registros creados o modificados correspondan a entidades válidas (proveedores, familias, OP, OC, sectores, líneas, centros de costo).
- No existan duplicados por fecha, operación y usuario.
- No se modifiquen/eliminen lotes con tickets asociados.
- Se mantenga consistencia transaccional mediante `COMMIT`/`ROLLBACK`.
- Se registre auditoría cuando corresponda.

## Parámetros Principales

| Parámetro                        | Descripción                                                  |
|----------------------------------|--------------------------------------------------------------|
| `@ACCION`                        | `I` (Insert), `U` (Update), `D` (Delete).                    |
| `@CATEGORIA_MOVIM_DESTINO`       | Define tipo de movimiento; determina validaciones.          |
| `@COD_OPERACION_COSTURA`         | Código de operación productiva asociada al lote.            |
| `@COD_PROVEEDOR_DESTINO`         | Identificador del proveedor.                                |
| `@COD_FAMITEM_DESTINO`           | Familia o categoría del ítem.                               |
| `@COD_SEC_COSTURA_DESTINO`       | Sector de costura.                                          |
| `@COD_LINEA_COSTURA_DESTINO`     | Línea de producción.                                        |
| `@COD_CENCOS_DESTINO`            | Centro de costos.                                           |
| `@ORDEN_COMPRA_DESTINO`          | Orden de compra asociada.                                   |
| `@ORDEN_PRODUCCION_DESTINO`      | Orden de producción asociada.                               |
| `@FECHA_DOCUMENTO`               | Fecha del lote/documento.                                   |
| `@USUARIO`                       | Usuario ejecutor de la operación.                           |
| `@NUM_LOTE_DESDE / @NUM_LOTE`    | Número o rango de lote.                                     |

> Nota: Documentar tipos de datos y longitudes en la especificación técnica del procedimiento (si no está ya).

## Tablas Principales Involucradas
- `CF_LOTETICKET` — Tabla principal del lote.
- `CF_LECTURA_TICKETS` — Auditoría de lecturas/tickets.
- `PROVEEDOR`, `ITEM_FAMILIA`, `OPCAB`, `OCPEDCAB` — Tablas maestras y de órdenes.
- Tablas relacionadas a `SECTORES`, `LINEAS`, `CENTROS_DE_COSTO`.

## Reglas de Negocio

### Reglas generales
- Un lote no puede modificarse o eliminarse si tiene tickets asociados.
- Los lotes deben ser únicos por combinación de fecha, operación y usuario.
- Validaciones específicas según la categoría de movimiento.
- Control de permisos para inserciones y cambios.
- Integridad entre proveedor, familia, OP, OC y destino del lote.

### Validaciones por categoría (resumen)
- Categoría 1: Validación de tipo de movimiento.
- Categorías 2 y 4: Validación de sector, línea y asociaciones.
- Categoría 3: Validaciones complejas (proveedor, familia, OP, OC, consistencia destino).
- Categoría 7: Validación de centros de costo.

## Flujo General por Acción

### INSERT (`I`)
- Verificar permisos de inserción.
- Comprobar duplicados (fecha/operación/usuario).
- Generar número de lote (según reglas internas).
- Insertar registro en `CF_LOTETICKET`.
- Registrar auditoría en `CF_LECTURA_TICKETS` (si aplica).

### UPDATE (`U`)
- Validar existencia del lote.
- Rechazar si existen tickets asociados.
- Revalidar entidades relacionadas (proveedor, familia, OP, OC, sector, línea, centro de costo).
- Actualizar datos del lote.
- Registrar auditoría de cambios (si aplica).

### DELETE (`D`)
- Rechazar si existen tickets asociados.
- Eliminar el lote (o marcar como inactivo según política).
- Registrar auditoría de eliminación (si aplica).

## Manejo de Errores y Transacciones
- Inicio con `BEGIN TRAN`.
- `ROLLBACK` ante cualquier error o validación fallida.
- `COMMIT` si la operación concluye correctamente.
- Mensajes descriptivos para:
  - Entidades inexistentes.
  - Inconsistencias proveedor–familia–lote.
  - Lote en uso (no modificable).
  - Falta de permisos.
  - Parámetros obligatorios faltantes.

## Posibles Mejoras Técnicas
- Modularizar validaciones en funciones o SPs auxiliares.
- Normalizar nombres de variables y parámetros.
- Centralizar mensajes de error en una tabla o catálogo (para i18n y mantenibilidad).
- Crear vistas o UDFs para validaciones recurrentes.
- Añadir pruebas unitarias/procedimientos de validación en un entorno de desarrollo.

## Recomendaciones de Integración con .NET
- Implementar una capa de acceso a datos (DAL) para invocar el SP con `SqlCommand` parametrizado.
- Usar DTOs para representar input/output del SP.
- Exponer operaciones mediante una API REST en ASP.NET Core si se requiere acceso desde front-end/Apps móviles.
- Para cargas masivas, usar procedimientos parametrizados y batching controlado.
- Manejar transacciones desde la capa .NET sólo si se requiere coordinación con otras operaciones.

## Conclusión
`UP_MAN_CF_LOTETICKET` es un SP crítico que centraliza la lógica de gestión de lotes de tickets en un entorno productivo. Garantiza validaciones empresariales, integridad transaccional y registros de auditoría. La documentación y las sugerencias incluidas facilitan mantenimiento, refactorización e integración con aplicaciones modernas.

## Contacto / Autor
- **Autor:** Julio Albert Bendezú Gutiérrez  
- **Fecha del documento:** 2025-12-05