Чтобы настроить время технического обслуживания (в т. ч. для выключенных кластеров), добавьте к описанию кластера блок `maintenance_window`:

```hcl
resource "yandex_spark_cluster" "<имя_кластера>" {
  ...
  maintenance_window {
    type = <тип_технического_обслуживания>
    day  = <день_недели>
    hour = <час_дня>
  }
  ...
}
```

{% include [Maintenance window description](../../terraform/maintenance-window-description.md) %}
