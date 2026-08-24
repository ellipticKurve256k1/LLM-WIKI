---
title: basic query
tags: [notes, references]
type: reference
priority: 2
finished: false
created_date: 2026-05-22
---

# Basic SQL Query Note 

## Abstract

Basic SQL query note

## Creating table 

### Basic table creating syntax 

```sql
create table if not exists table_name();
```

- `table_name` is mostly case **insensitive** unless defined between double-quote `"Table"`

### Defining data columns

```sql
create table if not exists table_name(
	table_id bigint generated always as identity primary key,
	name varchar(100),
	price float
);
```

- auto increment primary key: `generated always as identity`

## Enabling RLS (row level security)

```sql
alter table table_name enable row level security;
```

## Crud Table

### Creating

```sql
insert into table_name (list colums) values (
	'name', 20
);
```

### Update

```sql
update products
set price = 999
where product_id = 1;
```

### Delete

```sql
delete from products
where product_id = 1;
```




