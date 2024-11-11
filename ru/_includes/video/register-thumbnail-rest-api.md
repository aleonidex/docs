```bash
curl \
  --request POST \
  --url 'https://video.{{ api-host }}/video/v1/thumbnails' \
  --header 'Authorization: Bearer <IAM-токен>' \
  --header 'Content-Type: application/json' \
  --data '{
    "channelId": "<идентификатор_канала>"
  }'
```

Где:
* `<IAM-токен>` — полученный перед началом работы IAM-токен.
* `<идентификатор_канала>` — сохраненный ранее идентификатор канала.

Результат:

```text
{
 "done": true,
 "metadata": {
  "@type": "type.googleapis.com/yandex.cloud.video.v1.CreateThumbnailMetadata",
  "thumbnailId": "vpltaurfr4pr********"
 },
 "response": {
  "@type": "type.googleapis.com/yandex.cloud.video.v1.Thumbnail",
  "id": "vpltaurfr4pr********",
  "channelId": "vplcdyphvqik********",
  "createdAt": "2024-11-02T16:56:19.296797Z"
 },
 "id": "vplpgbyqopdr********",
 "description": "Thumbnail create",
 "createdAt": "2024-11-02T16:56:19.301776Z",
 "createdBy": "ajeol2afu1js********",
 "modifiedAt": "2024-11-02T16:56:19.301776Z"
}
```