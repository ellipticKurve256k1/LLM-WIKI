---
title: postgresql access note
tags: [note, reference]
type: reference
priority: 3
finished: false
created_date: 2026-05-25
---

# Postgresql note - access 

## Abstract

Brief note of postgresql note about accessing

## access permission

- Postgresql has two types of permissions
	1. Table permission (CRUD)
	2. Data permission (RLS)

- `service_role` default to have bypassing RLS policy.

### Table Permission

- giving all CRUD permission to `service role`.

```sql
GRANT SELECT, INSERT, UPDATE, DELETE ON public.table TO service_role;
```

### Data Permission (RLS)

1. Give table a RLS enabled first

```sql
alter table table_name enable row level security; 
```

2. Create RLS policy