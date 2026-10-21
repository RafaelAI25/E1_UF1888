# Verificación postgreSQL

### Localizar tablas nuevas

```sql
SELECT table_name
FROM information_schema.tables
WHERE table_schema = 'public' AND table_name LIKE 'x\_%';
```

### Nuevas columnas en res_partner

```sql
SELECT column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_schema = 'public'
	AND column_name LIKE 'x\_%';
```

### Tabla correspondiente al nuevo modelo

```sql
SELECT column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_name = 'x_segmento_comercial';
```
### Verificar la relación creada

```sql
SELECT conrelid::regclass AS tabla, conname, contype,
	pg_get_constraintdef(oid) AS definicion
FROM pg_constraint
WHERE conrelid = 'x_segmento_comercial'::regclass
	OR (conrelid = 'res_partner'::regclass AND conname LIKE '%x_segmento%');
```

