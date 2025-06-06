## Терминология {{ interconnect-name }}

* **CIC** — сокращенное название услуги {{ interconnect-full-name }}.
* **ТП** — [точки присутствия](../../interconnect/concepts/pops.md). Коммерческие узлы связи, на которых размещается сетевое оборудование третьих лиц, в том числе оборудование {{ yandex-cloud }}.
* **Кроссировка** (cross-connect, XC) — два оптических волокна, которые прокладываются в точке присутствия для подключения физического порта оборудования клиента (оператора связи) к физическому порту оборудования {{ yandex-cloud }}.
* [Трансивер](../../interconnect/concepts/transceivers.md) — оптический модуль, который устанавливается в порт оборудования клиента и в порт оборудования {{ yandex-cloud }}. К трансиверу подключается кроссировка.
* **Оператор связи** — организация, которая предоставляет телекоммуникационные услуги и имеет собственное оборудование на точке присутствия. В общем случае подключение к оборудованию {{ yandex-cloud }} через оператора связи требует организации кроссировки.
* [Партнер CIC](../../interconnect/concepts/partners.md) — оператор связи, который предоставляет расширенный набор опций по подключению к оборудованию {{ yandex-cloud }}. Партнер предоставляет клиенту `транковое подключение через партнера`, организация которого требует меньше времени и эксплуатационных затрат со стороны клиента, чем организация прямого подключения.
* [Транковое подключение](../../interconnect/concepts/trunk.md) (транк) — физическое подключение на точке присутствия между оборудованием {{ yandex-cloud }} с одной стороны и оборудованием клиента (оператора связи) с другой стороны.
* [Приватное соединение](../../interconnect/concepts/priv-con.md) (Private Connection, PRC) — соединение площадки клиента (On-Prem) с [Routing Instance](../../cloud-router/concepts/routing-instance.md).
* [Публичное соединение](../../interconnect/concepts/pub-con.md) — логическое соединение площадки клиента (On-Prem) с [сервисами](../../interconnect/concepts/pub-con.md#svc-list) {{ yandex-cloud }} без использования `VPC`.
* [Поддержка]({{ link-console-support }}) — команда технической поддержки {{ yandex-cloud }}, которая обрабатывает обращения клиентов.
