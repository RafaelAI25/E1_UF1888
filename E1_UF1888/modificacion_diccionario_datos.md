# Modificación del diccionario de datos

### 1. Situación inicial
El modelo (res.patner) no permitia clasificar comercialmente a los clientes. Se modifica res.partner y se crea el modelo x_segmento_comercial.

### 2. Objetos creados
x_nivel_cliente         res.partner             selección
segmento comeercial     x_segemnto_comercial    modelo nuevo
x_name                  x_segmento_comercial    char
x_codigo                x_segemento_comercial   char
x_descripcion           x_segemento_comercial   text
x_segmento_coemrcial_id res.partner             many2one -> x_segmento_comercial

### 3. Relación y justificación

Cliente N : 1 Segmento comercial. Un cliente pertenece a un único segmento y un segmento agrupa a muchos clientes. Por eso se usa un campo many2one en res.partner, que genera la clave ajena x_segmento_comercial_id → x_segmento_comercial.id.


Segmento comercial ─1:N─ Cliente ─1:N─ Pedido de venta ─1:N─ Línea de pedido ─N:1─ Servicio

### 4. Restricciones de integridad
Restricción	Implementación	Problema que evita
Nombre y código obligatorios	Atributo Obligatorio (NOT NULL)	Segmentos sin identificar
Código único	Regla de automatización (base_automation)	Códigos duplicados y clasificación ambigua
Borrado restringido	ON DELETE RESTRICT	Clientes apuntando a segmentos inexistentes

Regla de automatización: modelo Segmento comercial, disparador Al crear y actualizar, campo Código, acción Ejecutar código:

```python
for rec in records:
    if rec.x_codigo and env['x_segmento_comercial'].search_count(
            [('x_codigo', '=ilike', rec.x_codigo.strip()), ('id', '!=', rec.id)]):
        raise UserError("Ya existe un segmento con el código '%s'." % rec.x_codigo)
```


### 5. Permisos y vista heredada

| Grupo                    | Leer | Escribir | Crear | Eliminar |
| ------------------------ | ---- | -------- | ----- | -------- |
| Administración / Ajustes | ✔    | ✔        | ✔     | ✔        |
| Usuario interno          | ✔    | ✔        | ✔     | ✘        |

Vista heredada res.partner.form.segmento: hereda de base.view_partner_form, en modo Extensión.

```xml
<data>
    <xpath expr="//notebook" position="inside">
        <page string="Clasificación comercial" name="clasificacion_comercial">
            <group>
                <field name="x_nivel_cliente"/>
                <field name="x_segmento_comercial_id"/>
            </group>
        </page>
    </xpath>
</data>
```
Acción y menú: Segmentos comerciales (list,form) en Ventas › Configuración.


### 6. Procedimiento (modo desarrollador)
Activar el modo desarrollador (/odoo?debug=1) e instalar Reglas de automatización.
Técnico › Campos: crear x_nivel_cliente en res.partner.
Técnico › Modelos: crear x_segmento_comercial y añadirle x_codigo y x_descripcion.
Técnico › Campos: crear x_segmento_comercial_id (many2one, Restringir).
Técnico › Automatización: crear la regla de código único.
Técnico › Seguridad › Derechos de acceso: crear los permisos.
Técnico › Vistas: crear la vista heredada.
Técnico › Acciones y Menús: crear el acceso a Segmentos comerciales.

### 7. Pruebas y resultado
####	Prueba	Resultado esperado	Obtenido
- P1	Abrir un cliente	Se ven los campos nuevos	[✔]
- P2	Crear segmentos	Se guardan	[✔]
- P3	Asignar nivel y segmento a un cliente	Se guarda	[✔]
- P4	Segmento sin código	No permite guardar	[✔]
- P5	Código duplicado (pyme)	Aviso de la regla	[✔]
- P6	Borrar segmento con clientes	Error, no se borra	[✔]
- P7	Borrar segmento sin clientes	Se borra	[✔]

Resultado: el modelo de clientes queda ampliado y relacionado con los segmentos, y las restricciones funcionan. La verificación en PostgreSQL está en verificacion_postgresql.md.

