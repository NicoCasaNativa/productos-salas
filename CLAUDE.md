# productos-salas — App Productos y Salas

## Propósito
App maestra centralizada para gestionar salas Walmart y la matriz de productos Casa Nativa. **Única fuente de verdad** para toda la red Wild Labs.

## Deploy
- URL: `https://productos-salas.netlify.app`
- Repo: `nicolasonettoh-tech/productos-salas`
- Stack: Vue 3 Options API + Tailwind + Supabase JS v2 + SheetJS

## Auth
- `CN_APP_SLUG`: `productos-salas`
- `CN_SESSION_KEY`: `cn_session_productos-salas`
- Panel URL: `https://wild-apps.netlify.app`

## Tablas Supabase (todas `cn_ps_*`)

| Tabla | Descripción |
|-------|-------------|
| `cn_ps_salas` | Maestro de salas Walmart (118 filas migradas de cn_tw_salas + cn_wm_stores) |
| `cn_ps_productos` | Catálogo productos Walmart CN (11 SKUs) |
| `cn_ps_matriz` | Matriz productos × salas (475 filas, 49 salas activas × 11 SKUs) |
| `cn_ps_ranking` | Ranking de ventas externo (vacío, se carga desde Excel) |

### Campos de `cn_ps_salas`
- `id` uuid PK
- `store_nbr` integer UNIQUE — número oficial Walmart
- `nombre` text
- `tipo` enum: `hiperlider` | `lider` | `express`
- `zona` enum: `Norte` | `RM` | `Sur` | `Outlet`
- `region` text — región oficial de Chile (rellenar pendiente)
- `comuna` text
- `direccion` text
- `activa` boolean
- `notas` text

## RPCs (`cn_ps_*`)
- `_cn_ps_require_session(p_token)` → uuid (helper privado)
- `cn_ps_salas_list(p_token, p_zona?, p_tipo?, p_region?, p_activa?, p_search?)` → jsonb
- `cn_ps_sala_upsert(p_token, p_data jsonb)` → jsonb
- `cn_ps_sala_toggle(p_token, p_id, p_activa)` → void
- `cn_ps_productos_list(p_token, p_activo?)` → jsonb
- `cn_ps_producto_upsert(p_token, p_data jsonb)` → jsonb
- `cn_ps_matriz_list(p_token, p_sala_id?, p_producto_id?)` → jsonb
- `cn_ps_matriz_set_sala(p_token, p_sala_id, p_producto_ids jsonb)` → jsonb
- `cn_ps_stats(p_token)` → jsonb (cobertura completa)
- `cn_ps_ranking_list(p_token, p_periodo?, p_store_nbr?, p_sku_cn?)` → jsonb
- `cn_ps_ranking_import(p_token, p_periodo, p_fuente, p_rows jsonb)` → jsonb

## Relación con otras apps

| App | Tipo de acceso | RPCs usados |
|-----|---------------|-------------|
| `trade-walmart` | Read-only (sin edición) | `cn_tw_salas_list`, `cn_tw_productos_list`, `cn_tw_producto_sala_list` → redirigen a `cn_ps_*` |
| `oc-lider` | Read-only | `cn_wm_oc_list_stores` → redirige a `cn_ps_salas` |
| `repo-lider` | Read-only (salas asignadas) | `cn_tw_reponedor_salas.sala_id` FK apunta a `cn_ps_salas.id` |
| `stock-walmart` | Referencia | puede enricher datos de salas desde `cn_ps_salas` |

## Datos migrados
- Migración ejecutada 2026-05-19
- Fuente salas: `cn_tw_salas` (118 filas, nombres cortos) + `cn_wm_stores` (69 filas, nombres completos)
- La migración usa el nombre completo de `cn_wm_stores` cuando está disponible
- Los IDs de `cn_ps_salas` son idénticos a los de `cn_tw_salas` (mismos UUIDs) para compatibilidad con FKs existentes

## Secciones de la app
1. **Salas** — CRUD completo, filtros por tipo/zona/region/activa, búsqueda libre, paginado, exportar Excel
2. **Productos** — CRUD catálogo Walmart, filtros por categoría/estado
3. **Matriz** — 3 vistas: por sala (editar), por producto (ver distribución), grid completo (lectura)
4. **Ranking** — importar desde Excel, filtrar por período/sala/SKU, exportar
5. **Cobertura** — dashboard KPIs + barras de cobertura por sala y por SKU

## Notas
- El campo `region` está pendiente de rellenar para todas las salas (todas tienen NULL actualmente)
- El ranking se carga por usuario cuando tengan los datos de la empresa externa
- `cn_ps_ranking_import` elimina el período existente antes de insertar (reemplaza, no acumula)
