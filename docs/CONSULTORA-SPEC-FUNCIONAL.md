# Especificación Funcional del Sistema (POS + Inventario + Compras + Reportes)
Fecha: 2026-03-27 16:14:58  
Versión: 1.0  
Estado: Listo para Diseño (SDD)  
Alcance: MVP completo

## 1. Resumen ejecutivo
Sistema web para operación de punto de venta (POS), gestión de productos y precios, control de inventario por almacén, compras (pedidos y recepciones), seguridad (usuarios/roles/perfiles) y reportes (compras, ventas, ganancias, usuarios).  
Incluye emisión de comprobantes tipo Ticket/Boleta/Factura **básica** (sin integración con entidad tributaria).

## 2. Roles del sistema (RBAC)
Roles mínimos (se pueden ampliar):
- ADMIN: control total.
- CAJERO: POS, emisión comprobante, devoluciones, consulta catálogo.
- COMPRAS: pedidos, recepciones, almacenes, consulta productos.
- GERENCIA: reportes.
- AUDITOR: lectura + reportes.

Regla: toda acción de creación/edición/anulación requiere permiso explícito.

## 3. Look & Feel (estándar consultora)
- Web responsive (desktop-first).
- Sidebar por módulos + topbar con búsqueda rápida.
- POS en layout de 2 columnas: carrito + totales/cobro.
- Tablas: filtros, orden, paginación, export CSV.
- Estados UX obligatorios: loading, empty, error, confirm modal.
- Accesibilidad: contraste AA, focus visible.

## 4. Módulos y alcance funcional (determinista)

### 4.1 Seguridad
#### 4.1.1 Login
Campos:
- usuario_o_email (req, 3–120)
- password (req, 8–72)

Reglas:
- 5 intentos fallidos => bloqueo 15 min (configurable)
- usuario Inactivo/Bloqueado => denegar acceso
- auditoría: last_login_at, ip, user-agent

Errores (mensaje único): "Credenciales inválidas."

#### 4.1.2 Recuperar contraseña
- Solicitud por email (respuesta no revela existencia).
- Token expira 30 min.
- Nueva contraseña: 8–72, 1 mayús, 1 minús, 1 número, 1 símbolo; confirmación igual.

#### 4.1.3 Gestión de usuarios
Campos:
- username (req, 3–30, regex ^[a-zA-Z0-9._-]+$, único)
- nombres (req, 2–80)
- apellidos (req, 2–80)
- email (req, único, formato)
- telefono (opc, ^\+?[0-9\s-]{7,20}$)
- estado (Activo/Inactivo/Bloqueado)
- roles (req, min 1)
- perfil (req)

Reglas:
- No desactivar último ADMIN activo.
- Soft delete (inactivar).
- Auditoría obligatoria para cambios de estado/roles.

#### 4.1.4 Roles
- CRUD roles (nombre único 3–50).
- Matriz de permisos por módulo/acción (ver sección 7).

#### 4.1.5 Perfiles (configuración de operación)
Campos:
- nombre_perfil (req, único)
- tema_ui (Light/Dark)
- pagina_inicio (Dashboard/POS/Reportes/Productos)
- almacen_por_defecto (req)
- impresora_por_defecto (opc)
- zona_horaria (req)

---

### 4.2 Producto
#### 4.2.1 Categorías
- nombre (req, único 2–60)
- estado Activo/Inactivo
Regla: no inactivar categoría con productos activos (o confirmar con impacto).

#### 4.2.2 Productos
Campos core:
- sku (req, único 2–40)
- nombre (req 2–120)
- categoria (req)
- codigo_barra (opc, único si se usa)
- unidad_medida (req)
- impuesto_aplica (req bool)
- tasa_impuesto (req si aplica, 0–100)
- estado (req)
- stock_minimo (opc >=0)
- imagen (opc jpg/png/webp <=2MB)

Reglas:
- Producto inactivo no se vende ni se compra.
- Stock controlado por almacén.
- No permitir stock negativo (por defecto: NO; configurable por ADR).

#### 4.2.3 Precios (vigencias)
Campos:
- producto_id (req)
- tipo_precio (Público/Mayorista/Promoción; configurable)
- precio (req >=0.01)
- fecha_inicio (req)
- fecha_fin (opc >= inicio)
Reglas:
- No solapamiento por (producto_id, tipo_precio) con estado Activo.
- POS toma precio vigente; si no hay, usa precio_venta_base.

---

### 4.3 Ventas
#### 4.3.1 Punto de venta (POS)
Componentes:
- Buscador (SKU/nombre/código barra)
- Carrito (líneas)
- Totales (subtotal, impuesto, total)
- Botón Cobrar

Reglas:
- Solo productos activos.
- Cantidad por línea: int >=1.
- Validar stock disponible del **almacén por defecto del usuario**.
- No permitir cobrar con carrito vacío.

#### 4.3.2 Generar comprobante (Ticket/Boleta/Factura básica)
Tipos:
- Ticket (sin datos fiscales obligatorios)
- Boleta (datos mínimos opcionales)
- Factura básica (requiere datos fiscales, pero sin envío a tributar)

Datos fiscales (Factura básica) requeridos:
- tipo_documento (RUC/NIT/ID_FISCAL) [configurable]
- numero_documento (req, regex configurable por país; default: alfanumérico 8–15)
- razon_social (req 2–120)
- direccion_fiscal (opc 0–200)
- email (opc, formato)

Cobro:
- metodo_pago: Efectivo/Tarjeta/Transferencia/Mixto
- monto_recibido: req si incluye efectivo, >= total
- referencia_pago: req si tarjeta/transferencia (4–40)

Reglas:
- Numeración correlativa por tipo y serie.
- Generar representación imprimible (PDF/HTML print).
- Registrar venta y descontar stock al confirmar.

Acciones:
- Reimprimir comprobante.
- Anular (solo permisos) con motivo req 5–200.
- Nota: anulación revierte stock (sí) y revierte impacto en reportes.

#### 4.3.3 Devoluciones
Flujo:
- Seleccionar comprobante -> seleccionar ítems y cantidades a devolver -> motivo -> confirmar
Validaciones:
- cantidad devuelta acumulada <= cantidad vendida
- ventana devolución: 30 días (configurable)
Efectos:
- Incrementa stock en almacén de la venta (por defecto) o almacén seleccionado con permiso
- Impacta reportes: resta ventas netas y ganancia

---

### 4.4 Compras
#### 4.4.1 Almacenes
Campos:
- nombre (req, único)
- direccion (opc)
- responsable (opc)
- estado (req)
Reglas:
- No inactivar almacén por defecto asignado a usuarios sin reasignar.

#### 4.4.2 Pedidos de compra + Recepción
Estados:
- Borrador -> Aprobado -> Recibido Parcial -> Recibido Total -> Cerrado
- Cancelado (desde Borrador/Aprobado, con motivo)

Pedido (campos):
- proveedor_nombre (req 2–120) [MVP sin catálogo proveedor]
- almacen_id (req)
- fecha_pedido (req)
- items[] (req min 1):
  - producto_id (req)
  - cantidad (req int >=1)
  - costo_unitario (req decimal >=0)

Recepción:
- registrar cantidad recibida por item (<= pendiente)
- al recepcionar: aumenta stock del almacén
- opcional: actualizar costo_referencial o costo_promedio (decisión en ADR)

---

### 4.5 Reportes
Filtros comunes:
- fecha_inicio/fecha_fin (req)
- almacén (opc)
- export CSV (req)

#### 4.5.1 Reporte de compras
- Totales por período
- Detalle por pedido (estado, total)
- Compras por producto/categoría (opc)

#### 4.5.2 Reporte de ventas
- Ventas netas (incluye devoluciones por defecto)
- Top productos
- Ventas por cajero/usuario
- Tickets promedio

#### 4.5.3 Reporte de ganancias (determinista)
Definición:
- Ganancia = Ventas netas - Costo de productos vendidos (COGS)
COGS (MVP):
- Se calcula usando el último costo_unitario recepcionado por producto (fallback a costo_referencial si no existe).
Regla:
- si no existe costo para un producto, marcarlo como “Costo no definido” y excluirlo del margen % (pero sí mostrarlo en reporte con alerta).

#### 4.5.4 Reporte de usuarios
- Listado con roles, estado, último login
- Métricas: #por rol, #bloqueados, actividad

## 5. Validaciones transversales (resumen)
- Strings: trim, no permitir solo espacios.
- Números:
  - int: >=0 o >=1 según campo
  - decimal: 2 decimales; redondeo estándar (half-up) definido en ADR
- Fechas: fecha_fin >= fecha_inicio
- Unicidad: username/email/sku/categoría/almacén
- Auditoría obligatoria para acciones críticas.

## 6. Inventario por almacén (reglas deterministas)
- Stock se maneja por (producto_id, almacen_id).
- Operaciones que afectan stock:
  - Recepción compra: +stock
  - Venta confirmada: -stock
  - Devolución confirmada: +stock
  - Anulación venta: +stock (si estaba confirmada)
- Stock mínimo: si stock < stock_minimo, marcar en UI con alerta.

## 7. Permisos mínimos (matriz resumida)
- Seguridad:
  - ADMIN: todo
- Ventas:
  - CAJERO: POS, emitir comprobante, reimprimir; devolución (si permitido)
  - GERENCIA: ver ventas (no operar)
- Compras:
  - COMPRAS: pedidos, recepciones, almacenes
- Producto:
  - ADMIN/COMPRAS: gestionar productos, categorías, precios
- Reportes:
  - GERENCIA/AUDITOR: ver/exportar

(La matriz final detallada se implementa en docs/RBAC.md)

## 8. No funcionales (MVP)
- Disponibilidad objetivo: 99.5% (MVP) / 99.9% (prod)
- Latencia: p95 < 300ms en operaciones lectura; p95 < 600ms en cobro/confirmación
- Seguridad: hash de password (bcrypt/argon2), secretos en GitHub Secrets
- Observabilidad: logs estructurados + auditoría

## 9. Entregables de consultora
- Documento funcional (este)
- Prototipo UI de POS y listados (wireframes)
- Matriz RBAC detallada
- SDD + ADRs
- Backlog inicial con criterios de aceptación
