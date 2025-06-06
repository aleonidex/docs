# Storage in {{ dataproc-name }}

{{ dataproc-name }} enables you to use network storage drives to store subclusters. Network drives are based on network blocks, which are virtual disks in the {{ yandex-cloud }} infrastructure.

When creating a cluster, you can choose between the following storage types:

* Network HDD storage (`network-hdd`): Most cost-effective option for clusters that do not require high read/write performance.
* Network SSD storage (`network-ssd`): Balanced solution. Such disks are slower than non-replicated SSDs, but, unlike non-replicated disks, they ensure data integrity in case {{ yandex-cloud }} hardware fails.
* Non-replicated SSD storage (`network-ssd-nonreplicated`): Network SSDs with enhanced performance achieved by eliminating redundancy.

   The storage size can only be increased in 93 GB increments.

   For more information about this disk type, see the [{{ compute-full-name }} documentation](../../compute/concepts/disk.md#nr-disks).

{% include [available-storage](../../_includes/mdb/available-storage.md) %} 

## Specifics of non-replicated SSD storage {#network-nrd-storage-features}

Non-replicated SSD storage does not provide fault tolerance; this means, if a disk fails, the data is permanently lost.
