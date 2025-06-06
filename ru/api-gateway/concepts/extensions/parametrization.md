# Параметризация спецификации

Параметризация спецификации позволяет изменять поведение {{ api-gw-short-name }}, указывая разные значения для отдельных переменных, а не всей спецификации. Для этого используется расширение `x-yc-apigateway:variables`.

## Расширение x-yc-apigateway:variables

Расширение `x-yc-apigateway:variables` позволяет объявить переменные, которые используются для параметризации спецификации.

### VariableObject

`VariableObject` — набор параметров спецификации OpеnAPI, которые определяют переменную и ее возможные значения.

#### Параметры

В таблице ниже перечислены параметры объекта `VariableObject`.

Параметр   | Тип                                     | Параметр обязателен   | Описание
-----------|-----------------------------------------|-----------------------|---------
`default`    | `string`, `int`, `double`, `boolean`         |Да                     | Значение переменной по умолчанию. Используется, если не указан параметр `enum`.
`enum`      | `string[]`, `int[]`, `double[]`, `boolean[]` |Нет                    | Список значений, допустимых для данной переменной. Если не указан, переменная может принимать любое значение.
`description` | `string`                                  |Нет                    | Текстовое описание переменной.

### Использование

После объявления переменной ниже в спецификации можно использовать конструкцию вида `${var.name}`, где `name` — название переменной. При необходимости {{ api-gw-short-name }} подставит вместо названия переменной значение, заданное при создании или обновлении API-шлюза.

### Спецификация расширения

Пример спецификации расширения:
```yaml
openapi: 3.0.0
info:
  title: Sample API
  version: 1.0.0

x-yc-apigateway:
  variables:
    environment:
      default: "prod"
      enum:
        - "prod"
        - "testing"
        - "dev"
      description: "API Gateway installation"

paths:
  /env/info:
    get:
      x-yc-apigateway-integration:
        type: dummy
        content:
          '*': It is ${var.environment} environment!
        http_code: 200
        http_headers:
          Content-Type: text/plain
```

## Примеры использования {#examples}

* [{#T}](../../tutorials/canary-release.md)
