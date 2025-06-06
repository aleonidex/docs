#### What is the backup procedure for a {{ CH }} database? {#backup-overview}

Backups are created every 24 hours and stored for seven days after being created. You can restore data only as of the backup creation time.

#### Is database host backup enabled by default? {#default-backup}

Yes, backup is enabled by default. For {{ CH }}, a full backup is performed once a day with the possibility to restore it to any saved backup.

#### When are backups performed? Is a database cluster available during backup? {#backup-window}

When [creating](../../managed-clickhouse/operations/cluster-create.md) or [updating](../../managed-clickhouse/operations/update.md#change-additional-settings) a cluster, you can set the time interval during which the backup will start. Default time: `22:00 - 23:00` UTC (Coordinated Universal Time).

Clusters remain fully accessible during the backup window.

#### How many backups are stored in {{ mch-name }}? For how long? {#how-many-backups}

The size and amount of backups are not limited. Automatically created backups are stored for seven days, while manually created ones are stored indefinitely.

#### Can I delete a backup? {#delete-backup}

Yes, if you created it manually. To delete a backup, [follow this guide](../../managed-clickhouse/operations/cluster-backups.md#delete-backup).

#### Can I change the automatic backup retention period? {#backup-retain-days}

You can set the retention period for automatic backups when [creating](../../managed-clickhouse/operations/cluster-create.md) or [modifying](../../managed-clickhouse/operations/update.md#change-additional-settings) a cluster.

#### What does a daily backup include? {#daily-backups}

Backup data is only stored for the `MergeTree` engine family. For other engines, backups only store table schemas. For more information, see [Backups](../../managed-clickhouse/concepts/backup.md).

#### Why does it take a long time to restore a cluster from a backup? {#long-restore}

The approximate average speed of restoring a cluster from a backup is 100 Mbps. The recovery time may vary significantly depending on host class and the nature of data in the DB.

#### How do I move an existing {{ CH }} cluster to {{ yandex-cloud }}? {#migration}

Use [{{ data-transfer-full-name }}](../../data-transfer/quickstart.md).

#### Can I restore a shard from a backup into a shard in an existing cluster? {#shard-to-shard}

This option is not currently supported.

However, you can [restore your shard](../../managed-clickhouse/operations/cluster-backups.md#restore) from a backup to a new {{ CH }} cluster and transfer data from that cluster into an existing one. To do this, explore the below options:

* [{{ data-transfer-name }}](../../managed-clickhouse/tutorials/ch-to-mch-migration.md). You can use this service to transfer a database or separate tables to a {{ mch-name }} cluster.
* [{{ CH }}'s built-in `remote`](../../managed-clickhouse/tutorials/data-migration.md#transfer-remote) function. Use it to transfer separate tables to a {{ mch-name }} cluster.
* [{{ CH }} `BACKUP` and `RESTORE`](../../managed-clickhouse/tutorials/data-migration.md#backup-objstorage) commands. These will help you to back up a database or single table to a [{{ objstorage-full-name}}](../../storage) bucket and then restore your data from the bucket to a {{ mch-name }} cluster.
