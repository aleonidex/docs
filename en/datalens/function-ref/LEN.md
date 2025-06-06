---
editable: false
sourcePath: en/_api-ref/datalens/function-ref/LEN.md
---

# LEN

_Function `LEN` is also found in the following categories: [Array functions](LEN_ARRAY.md)._

#### Syntax {#syntax}


```
LEN( value )
```

#### Description {#description}
Returns the number of characters in the string or items in array `value`.

**Argument types:**
- `value` — `Array of fractional numbers | Array of integers | Array of strings | String`


**Return type**: `Integer`

#### Example {#examples}

```
LEN("Computer") = 8
```


#### Data source support {#data-source-support}

`ClickHouse 21.8`, `Files`, `Google Sheets`, `Microsoft SQL Server 2017 (14.0)`, `MySQL 5.7`, `Oracle Database 12c (12.1)`, `PostgreSQL 9.3`, `Yandex Documents`, `YDB`.
