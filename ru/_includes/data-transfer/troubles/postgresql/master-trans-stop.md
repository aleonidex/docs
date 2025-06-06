### Остановка сессии мастер-транзакции трансфера {#master-trans-stop}

Текст ошибки:

```text
Cannot set transaction snapshot:
ERROR: invalid snapshot identifier: "<идентификатор_снапшота>" (SQLSTATE 22023).
```

Возможные причины:

* На источнике работает cron-задание или другое приложение, которое периодически завершает слишком длительные сессии.
* Кто-то вручную завершил мастер-транзакцию.
* Ресурсов CPU на источнике не хватает для выполнения запроса.
* В настройке кластера {{ PG }} [Session duration timeout](../../../../managed-postgresql/concepts/settings-list#setting-session-duration-timeout) установлено ограничение на время жизни активной сессии.

**Решение:** отключите такое cron-задание, добавьте дополнительные ресурсы для CPU на источник, а также установите значение параметра **Session duration timeout** равным `0` на время трансфера. После внесения изменений [активируйте](../../../../data-transfer/operations/transfer.md#activate) трансфер повторно.
