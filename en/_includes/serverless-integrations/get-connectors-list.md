Get a list of connectors:

```bash
yc serverless eventrouter connector list
```

Result:

```text
+----------------------+-----------------------+----------------------+----------------------+---------+---------------------+
|          ID          |         NAME          |        BUS ID        |      FOLDER ID       | STATUS  | DELETION PROTECTION |
+----------------------+-----------------------+----------------------+----------------------+---------+---------------------+
| f66g4h59ih2g******** | connector-my-ydb-flow | f66qn4p7uk6p******** | b1g681qpemb4******** | STOPPED | true                |
| f66g6jcnc5no******** | yds-via-cli           | f66qn4p7uk6p******** | b1g681qpemb4******** | RUNNING | false               |
| f66jfhcg1u71******** | mq-via-terraform      | f66qn4p7uk6p******** | b1g681qpemb4******** | RUNNING | false               |
+----------------------+-----------------------+----------------------+----------------------+---------+---------------------+
```