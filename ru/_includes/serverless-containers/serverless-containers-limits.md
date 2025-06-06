#### Квоты {#serverless-containers-quotas}

#|
|| Вид ограничения | Значение ||
|| Количество контейнеров в облаке 
`serverless.containers.count` | 10 ||
|| Количество [триггеров](../../serverless-containers/concepts/trigger/) в одном облаке^1^ 
`serverless.triggers.count` | 100 ||
|| Суммарный объем Docker-образов, которые используются в активных ревизиях контейнеров 
`serverless.containers.size` | 20 ГБ ||
|| Количество экземпляров контейнеров в каждой зоне доступности 
`serverless.containersInstances.count` | 10 ||
|| Количество одновременных запросов к контейнерам в каждой зоне доступности 
`serverless.containersRequest.count` | 10 ||
|| Суммарный объем RAM для всех запущенных экземпляров контейнеров в каждой зоне доступности 
`serverless.containersMemory.size` | 20 ГБ ||
|| Суммарное количество CPU всех запущенных экземпляров контейнеров в каждой зоне доступности 
`serverless.containersCpu.count` | 10 ||
|| Количество [подготовленных экземпляров](../../serverless-containers/concepts/container.md#provisioned-instances) контейнеров в одном облаке 
`serverless.containersWorkersProvisioned.count` | 2 ||
|#

{% include [quota-trigger-count](../quota-trigger-count.md) %}

#### Лимиты {#serverless-containers-limits}

Вид ограничения | Значение
----- | -----
Максимальный объем образа контейнера | 10 ГБ
Максимальный объем RAM для одного экземпляра контейнера | 8 ГБ
Максимальный размер HTTP-запроса к контейнеру, включая HTTP-заголовки и тело запроса | 3.5 МБ
Максимальный размер HTTP-ответа контейнера, включая HTTP-заголовки и тело ответа | 3.5 МБ
Максимальный объем временных файлов | 512 МБ
Максимальное время обработки запроса контейнером до таймаута, включая начальную инициализацию при первом запуске^1^ | 1 час
Максимальный объем переменных окружения, включая их имена^2^ | 4 КБ
Максимальное количество триггеров для одной [очереди сообщений](../../message-queue/concepts/queue.md) | 1
Максимальный размер сообщения для триггера^3^ | 230 КБ
Количество [пользовательских сетей](../../serverless-containers/concepts/networking.md#user-network) в одном облаке | 1

^1^ Таймаут больше 10 минут доступен только для [долгоживущих контейнеров](../../serverless-containers/concepts/long-lived-containers.md).
^2^ Секреты {{ lockbox-short-name }} передаются через переменные окружения и также учитываются в этом лимите.
^3^ В сообщение может быть добавлена служебная метаинформация — в таком случае максимальный размер сообщения уменьшится.