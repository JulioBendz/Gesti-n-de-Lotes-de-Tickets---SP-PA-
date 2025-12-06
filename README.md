README TÉCNICO

Stored Procedure: [dbo].[UP_MAN_CF_LOTETICKET]
Autor: Bendezú Gutiérrez, Julio Albert
Fecha: 12/05/2025

1. Propósito del Documento

Este README resume el análisis técnico-funcional del Stored Procedure UP_MAN_CF_LOTETICKET. Su objetivo es brindar una referencia rápida sobre su propósito, uso, reglas de negocio, parámetros y consideraciones para desarrolladores que integren o mantengan esta lógica en sistemas basados en .NET, SQL Server u otras plataformas.

2. Resumen Ejecutivo

UP_MAN_CF_LOTETICKET gestiona la creación, actualización y eliminación de lotes de tickets dentro de un flujo productivo industrial (costura, proveedor, familia de ítems, órdenes de compra y producción).

Las operaciones soportadas incluyen:

I – Insert

U – Update

D – Delete

Cada acción se ejecuta bajo un conjunto de validaciones orientadas a garantizar integridad, consistencia y cumplimiento de reglas de negocio.

3. Objetivo Funcional del SP

El Stored Procedure asegura que:

Los registros creados o modificados correspondan a entidades válidas (proveedores, familias, OP, OC, sectores, líneas, centros de costo).

No existan duplicados por fecha, operación o usuario.

No se modifiquen/eliminan lotes con tickets asociados.

Se mantenga consistencia transaccional mediante COMMIT/ROLLBACK.

Se registre auditoría cuando corresponda.

En la operación general del sistema, funciona como el núcleo de gestión de movimientos y lotes de producción.

4. Parámetros Principales
| Parámetro                       | Descripción                                        |
| ------------------------------- | -------------------------------------------------- |
| **@ACCION**                     | I (Insert), U (Update), D (Delete).                |
| **@CATEGORIA_MOVIM_DESTINO**    | Define tipo de movimiento; determina validaciones. |
| **@COD_OPERACION_COSTURA**      | Operación productiva asociada al lote.             |
| **@COD_PROVEEDOR_DESTINO**      | Identificador del proveedor.                       |
| **@COD_FAMITEM_DESTINO**        | Familia o categoría del ítem.                      |
| **@COD_SEC_COSTURA_DESTINO**    | Sector de costura.                                 |
| **@COD_LINEA_COSTURA_DESTINO**  | Línea de producción.                               |
| **@COD_CENCOS_DESTINO**         | Centro de costos.                                  |
| **@ORDEN_COMPRA_DESTINO**       | Orden de compra asociada.                          |
| **@ORDEN_PRODUCCION_DESTINO**   | Orden de producción asociada.                      |
| **@FECHA_DOCUMENTO**            | Fecha del lote/documento.                          |
| **@USUARIO**                    | Usuario ejecutor de la operación.                  |
| **@NUM_LOTE_DESDE / @NUM_LOTE** | Número o rango de lote.                            |

5. Tablas Principales Involucradas

CF_LOTETICKET (tabla principal del lote)

CF_LECTURA_TICKETS (auditoría de lectura)

PROVEEDOR, ITEM_FAMILIA, OPCAB, OCPEDCAB

Tablas de sectores, líneas, centros de costo

6. Reglas de Negocio Detectadas
6.1 Reglas Generales

Un lote no puede modificarse o eliminarse si tiene tickets asociados.

Los lotes deben ser únicos por fecha, operación y usuario.

Validaciones específicas por categoría de movimiento.

Control de permisos para inserciones.

Integridad entre proveedor, familia, OP, OC y lote destino.

6.2 Validaciones por Categoría (Resumen)

Categoría 1: Validación de tipo de movimiento.

Categorías 2 y 4: Validación de sector, línea y asociaciones.

Categoría 3: Validaciones complejas (proveedor, familia, OP, OC, consistencia destino).

Categoría 7: Validación de centros de costo.

7. Flujo General del SP
7.1 INSERT (I)

Validación de permisos.

Verificación de duplicados.

Generación del número de lote.

Inserción y registro de auditoría.

7.2 UPDATE (U)

Validación de existencia del lote.

Bloqueo si hay tickets asociados.

Revalidación completa de entidades.

Actualización del lote.

7.3 DELETE (D)

Rechazo si existen tickets.

Eliminación del lote.

8. Manejo de Errores

El SP utiliza mensajes descriptivos sobre:

Entidades inexistentes.

Inconsistencias proveedor–familia–lote.

Lote utilizado/no modificable.

Falta de permisos.

Parámetros obligatorios faltantes.

9. Transacciones

BEGIN TRAN al iniciar la operación.

ROLLBACK ante cualquier error.

COMMIT si la operación concluye correctamente.

Asegura integridad y evita operaciones parciales.

10. Posibles Mejoras Técnicas

Modularizar validaciones en funciones o SPs auxiliares.

Normalizar nombres de variables.

Centralizar mensajes de error en una tabla o catálogo.

Crear vistas para validaciones recurrentes.

11. Recomendaciones de Integración con .NET

Crear una capa de acceso a datos (DAL) centralizada para invocar el SP.

Utilizar SqlCommand con parámetros tipados para evitar inyecciones.

Implementar un Data Transfer Object (DTO) para las operaciones del lote.

Consumir el SP desde una API REST en ASP.NET Core si se requiere exponerlo a aplicaciones móviles o front-end modernos.

Utilizar procedimientos almacenados parametrizados para cargas masivas o automatizaciones.

12. Conclusión

UP_MAN_CF_LOTETICKET es un SP crítico que centraliza la lógica de negocio para la gestión de lotes de tickets en un entorno industrial. Su estructura cumple roles de validación, integridad transaccional y consistencia operativa. La documentación asociada (flujo, pseudocódigo y análisis técnico) permite comprender su funcionamiento y facilita su mantenimiento, refactorización y futura integración con aplicaciones en .NET, web o móviles.