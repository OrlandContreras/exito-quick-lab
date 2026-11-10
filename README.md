# Kit de datos `exito-quick-lab` — Workshop Amazon Quick

> **Todos los datos de este kit son ficticios** y se crearon solo para el workshop de Amazon Quick con Grupo Éxito. Almacenes, productos, marcas de producto, cifras, precios y correos no corresponden a datos reales. Las marcas Éxito, Carulla, Surtimax y Super Inter se usan únicamente como contexto del caso. Los correos usan el dominio reservado `exito-lab.example`, que no recibe mensajes.

## Contexto del cliente

El caso se apoya en lo que Grupo Éxito publica sobre sí mismo en [Quiénes somos](https://www.grupoexito.com.co/es/quienes-somos) y [Negocios y marcas](https://www.grupoexito.com.co/es/negocios-marcas). Verifica las cifras públicas el día del workshop; pueden haber cambiado.

| Tema | Lo que dice el sitio del Grupo | Cómo se usa en el workshop |
| --- | --- | --- |
| Misión | "Trabajamos para que el cliente regrese" | Hilo conductor: un producto disponible en la góndola es un cliente que vuelve |
| Valores | Primero el cliente, trabajo colaborativo, innovación, consciencia social y ambiental, pasión por los resultados | Cada módulo se conecta con al menos uno (ver la lámina de cierre) |
| Presencia en Colombia | 522 almacenes y más de 34.700 empleados | Contexto de escala; el kit modela una muestra de 20 almacenes |
| Marcas en Colombia | Éxito, Carulla, Surtimax y Super Inter, entre otras | Las cuatro marcas del kit |

## Contenido

| Archivo | Contenido | Se usa en |
| --- | --- | --- |
| `ventas_resumen_chat.csv` | Resumen para el chat: ventas, unidades y días-SKU en quiebre por almacén y categoría en tres periodos de 30 días (480 filas, 60 KB) | Labs 1, 2 y 6 |
| `ventas_por_almacen.csv` | Ventas diarias por almacén y SKU, del 21-jul-2026 al 18-oct-2026 (90 días). Es demasiado grande para el chat (límite de 665.000 caracteres): úsalo solo en Quick Sight o en Excel | Opcional |
| `inventario.csv` | Movimiento diario de stock por almacén y SKU, con stock mínimo y marca de quiebre | Lab 5 (Quick Sight) |
| `almacenes.csv` | 20 almacenes ficticios Éxito, Carulla, Surtimax y Super Inter en 6 ciudades, con formato, horario y coordenadas aproximadas | Labs 5 y 6 |
| `productos.csv` | Maestro de 24 productos en 8 categorías, con precios en pesos colombianos (COP) | Lab 5 (opcional) |
| `gerentes_regionales.csv` | 3 zonas operativas con el correo del gerente regional que aprueba transferencias | Labs 6 y 7 |
| `solicitudes_reposicion.csv` | 18 solicitudes de transferencia entre almacenes de las últimas 2 semanas, con su estado (5 pendientes) | Lab 7 (referencia de la app) |
| `politicas/` | Políticas de devoluciones, cadena de frío y reposición entre almacenes (`.md` y `.docx`) | Labs 3 y 7 |
| `glosario.md` / `glosario.docx` | 15 términos de operaciones de almacenes | Lab 3 |
| `documentos_internos/` | Borrador de prioridades de operaciones 2027 (`.md` y `.docx`), para subir al Space | Labs 3 y 4 |
| `base_conocimiento/` | Calendario comercial del cuarto trimestre y condiciones del servicio a domicilio (`.md` y `.docx`), para cargar en un SharePoint, bucket de S3 o sitio de pruebas | Lab 3 |
| `lab8_apps_flows_automate/` | Laboratorio 8 de profundización: guía paso a paso para construir un Flow programado, una app y una automatización de Quick Automate con aprobación humana, más sus datos (`stock_cierre_2026-10-18.csv`, `solicitudes_lote_2026-10-19.csv`) y el documento de proceso PRO-OPS-003 (`.md` y `.docx`) | Lab 8 |

Sube al Space la versión `.docx` de cada documento. La versión `.md` tiene el mismo contenido y sirve para leerla o como respaldo.

## Columnas

**ventas_resumen_chat.csv:** `periodo` (Últimos 30 días: 19-sep a 18-oct-2026; 30 días anteriores: 20-ago a 18-sep; Primeros 30 días: 21-jul a 19-ago), `fecha_inicio`, `fecha_fin`, `id_almacen`, `nombre_almacen`, `marca`, `ciudad`, `categoria`, `unidades`, `valor_cop`, `dias_sku_en_quiebre` (combinaciones día × SKU con stock final en cero, tomadas de `inventario.csv`).

**ventas_por_almacen.csv:** `fecha`, `id_almacen`, `marca`, `ciudad`, `categoria`, `sku`, `producto`, `unidades`, `precio_unitario_cop`, `valor_cop`. Solo aparecen los días con al menos una unidad vendida.

**inventario.csv:** `fecha`, `id_almacen`, `sku`, `producto`, `categoria`, `stock_inicial`, `unidades_recibidas`, `unidades_vendidas`, `ajuste_merma`, `stock_final`, `stock_minimo`, `quiebre_stock` (Sí/No). Se cumple siempre: `stock_final = stock_inicial + unidades_recibidas − unidades_vendidas − ajuste_merma`. Hay quiebre cuando `stock_final` es 0.

**almacenes.csv:** `id_almacen`, `nombre_almacen`, `marca`, `ciudad`, `departamento`, `region` (Andina, Caribe, Pacífico), `zona_operativa` (Centro-Oriente, Antioquia-Eje Cafetero, Caribe-Pacífico), `sector`, `formato` (Grande, Mediano, Barrio), `horario`, `latitud`, `longitud`, `correo_gerente_almacen`, `correo_gerente_regional`.

**gerentes_regionales.csv:** `zona_operativa`, `id_gerente_regional`, `cargo`, `correo_gerente_regional`, `ciudades`.

**solicitudes_reposicion.csv:** `id_solicitud`, `fecha_solicitud`, `id_almacen_destino`, `ciudad`, `sku`, `producto`, `categoria`, `stock_actual`, `stock_minimo`, `cantidad_solicitada`, `id_almacen_donante`, `motivo`, `estado` (Pendiente, Aprobada, Completada, Rechazada, Escalada a centro de distribución).

**productos.csv:** `sku`, `producto`, `categoria`, `precio_cop`, `cadena_frio` (Sí/No).

## Qué va a cada lugar en Quick

| Destino | Archivos |
| --- | --- |
| Chat (subir archivo) | `ventas_resumen_chat.csv`; en el Lab 6, también `almacenes.csv` y `politicas/politica_reposicion_entre_almacenes.docx` |
| Space "Operaciones de almacenes — piloto" | Los `.docx` de `politicas/`, `glosario.docx` y `documentos_internos/estrategia_operaciones_2027.docx` |
| Base de conocimiento de pruebas (SharePoint, S3 o web) | `base_conocimiento/` |
| Conjunto de datos de Quick Sight | `inventario.csv` unido con `almacenes.csv` por `id_almacen` |
| App de reposición | Las 5 solicitudes pendientes de `solicitudes_reposicion.csv` van dentro del texto del pedido (Lab 7) |
| Flow, app y automatización del Lab 8 | `lab8_apps_flows_automate/stock_cierre_2026-10-18.csv` en los tres; `solicitudes_lote_2026-10-19.csv` y `proceso_aprobacion_transferencias.docx` en Quick Automate. Ver `lab8_apps_flows_automate/README.md` |

## Cómo se relacionan

- `ventas_por_almacen` e `inventario` se unen por `fecha` + `id_almacen` + `sku`.
- Ambos se unen con `almacenes` por `id_almacen` y con `productos` por `sku`.
- `almacenes` se une con `gerentes_regionales` por `zona_operativa`.
- `solicitudes_reposicion` se une con `almacenes` por `id_almacen_destino` o `id_almacen_donante`, y con `productos` por `sku`.
- Los precios están en pesos colombianos (COP). Carulla vende con un 8% más que Éxito; Surtimax con un 5% menos y Super Inter con un 3% menos.
- Categorías críticas para reposición: Abarrotes básicos, Desayuno y bebidas calientes, y Lácteos y refrigerados (estos últimos con cadena de frío).
