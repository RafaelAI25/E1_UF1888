# Analisis del modelo de datos

## Identificación de modelos y tablas

### res.partner
- nombre funcional: Contacto
- modelo de Odoo: res.partner
- tabla PostgreSQL asociada: res_partner
- campos principales: company_name, email, name
- clave primaria: id
- principales claves ajenas: parent_id, company_id, country_id, user_id
- relaciones con otros objetos: country_id -> res.country
  
### product.product

- nombre funcional: Variante de producto
- modelo de Odoo: product.product
- tabla PostgreSQL asociada: product_product
- campos principales: barcode, active
- clave primaria: id
- principales claves ajenas: product_tmpl_id
- relaciones con otros objetos: product.template

### sale.order

- nombre funcional: Pedido de venta
- modelo de Odoo: sale.order
- tabla PostgreSQL asociada: sale_order
- campos principales: name, date_order
- clave primaria: id
- principales claves ajenas: partner_invoice_id, user_id
- relaciones con otros objetos: res.user

### sale.order.line

- nombre funcional: Línea Pedido de venta
- modelo de Odoo: sale.order.line
- tabla PostgreSQL asociada: sale_order_line
- campos principales: name, secuence
- clave primaria: id
- principales claves ajenas: order_id, salesman_id
- relaciones con otros objetos: sale.order


### Tablas de los modelos
```sql
SELECT table_name
FROM information_schema.tables
WHERE table_schema ='public'
	AND table_name IN ('res_partner', 'sale_order', 'sale_order_line', 'product_template', 'product_product')
ORDER BY table_name;
```

### Columnas principales
```sql
SELECT column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_schema ='public'
	AND table_name = 'sale_order_line'
ORDER BY ordinal_position;
```

### Claves primarias
```sql
SELECT conrelid::regclass AS tabla, conname, pg_get_constraintdef(oid) AS definicion
FROM pg_constraint
WHERE contype = 'p'
	AND conrelid IN ('res_partner'::regclass, 'sale_order'::regclass,
				'sale_order_line'::regclass, 'product_template'::regclass,
				'product_product'::regclass);
```

## Explicar la estructura del modelo

- Qué es un modelo en Odoo: Es una clase Python (ej. res.partner)
- Qué realción existe entre modelo y la tabla PostgreSQL: ORM crea y mantiene automaticamente una tabla por cada modelo. (res.partner -> res_partner)
- Función de una clave primaria: id -> identifica cada registro de forma única.
- Función de una clave ajena: puede ser una columna many2one, que en la base de datos es una columna *_id.
- Concepto integridad referencial: garantizar que una clave foranea apunte a un registro que existe.
```
Segmento comercial (x_segmento_comercial)
 | 1:N [res.partner.x_segmento_comercial_id]
  -- Cliente (res_partner)
      | 1:N [sale_order.partner_id]
       -- Pedido de venta (sale_order)
           |  1:N [sale_order_line.order_id]
            -- Línea de pedido (sale_order_line)
                 -- N:1 [product_id] -- Servicio (product_product
                                   -- N:1 [product_tmpl_id] --
                        product_template, type:'service')
                 
```

## Explicar herramientas utilizadas
- ORM de Odoo: traduce modelo de Python a tablas
- Modelos y campos del diccionario de Odoo: guardan como registros los modelos y campos del sistema
- Interfaz técnica de Odoo: editar el diccionario: modelos, campos, vistas.
- PostgreSQL 16: sistema gestor de bases de datos.
- pgAdmin4: cliente gráfico de postgreSQL
-