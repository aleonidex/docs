---
editable: false
sourcePath: ru/_api-ref/datalens/function-ref/FLOOR.md
---

# FLOOR



#### Синтаксис {#syntax}


```
FLOOR( number )
```

#### Описание {#description}
Округляет значение до ближайшего целого числа в меньшую сторону.

**Типы аргументов:**
- `number` — `Дробное число | Целое число`


**Возвращаемый тип**: `Дробное число`

#### Пример {#examples}



| **[value]**   | **FLOOR([value])**   |
|:--------------|:---------------------|
| `1.00`        | `1.00`               |
| `0.10`        | `0.00`               |
| `-2.00`       | `-2.00`              |
| `50.50`       | `50.00`              |
| `0.00`        | `0.00`               |
| `-3.50`       | `-4.00`              |




#### Поддержка источников данных {#data-source-support}

`ClickHouse 21.8`, `Файлы`, `Google Sheets`, `Microsoft SQL Server 2017 (14.0)`, `MySQL 5.7`, `Oracle Database 12c (12.1)`, `PostgreSQL 9.3`, `Яндекс Документы`, `YDB`.
