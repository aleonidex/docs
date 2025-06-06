* `security_group_ids`: List of [security group](../../../../managed-airflow/concepts/network.md#security-groups) IDs.
* `webserver`, `scheduler`, `worker`, `triggerer`: {{ maf-name }} [component](../../../../managed-airflow//concepts/index.md#components) configuration:

    * `count`: Number of instances in the cluster for the web server, scheduler, and Triggerer.
    * `min_count`, `max_count`: Minimum and maximum number of instances in the cluster for the worker.
    * `resource_preset_id`: ID of the computing resources of the web server, scheduler, worker, and Triggerer. The possible values are:

        * `c1-m2`: 1 vCPU, 2 GB RAM
        * `c1-m4`: 1 vCPU, 4 GB RAM
        * `c2-m4`: 2 vCPUs, 4 GB RAM
        * `c2-m8`: 2 vCPUs, 8 GB RAM
        * `c4-m8`: 4 vCPUs, 8 GB RAM
        * `c4-m16`: 4 vCPUs, 16 GB RAM
        * `c8-m16`: 8 vCPUs, 16 GB RAM
        * `c8-m32`: 8 vCPUs, 32 GB RAM

* `deb_packages`, `pip_packages`: Lists of deb and pip packages enabling you to install additional libraries and applications in the cluster for running DAG files:

    If required, you can set version restrictions for the installed packages, for example:

    ```hcl
    pip_packages = ["pandas==2.0.2","scikit-learn>=1.0.0","clickhouse-driver~=0.2.0"]
    ```

    The package name format and version are defined by the install command: `pip install` for pip packages and `apt install` for deb packages.

* `code_sync.s3.bucket`: Name of the bucket to store DAG files in.
* `deletion_protection`: Enables cluster protection against accidental deletion. The possible values are `true` or `false`.

    Even if it is enabled, one can still connect to the cluster manually and delete it.

* `lockbox_secrets_backend.enabled`: Enables using secrets in [{{ lockbox-full-name }}](../../../../lockbox/concepts/index.md) to [store {{ AF }} configuration data, variables, and connection parameters](../../../../managed-airflow/concepts/impersonation.md#lockbox-integration). The possible values are `true` or `false`.
* `airflow_config`: [{{ AF }}](https://airflow.apache.org/docs/apache-airflow/2.2.4/configurations-ref.html) additional properties, e.g., `core` for configuration section, `load_examples` for key, and `False` for value.
* `logging`: Logging parameters:

    * `enabled`: Enables logging. Logs generated by {{ AF }} components will be sent to {{ cloud-logging-full-name }}. The possible values are `true` or `false`.
    * `folder_id`: Folder ID. Logs will be written to the default [log group](../../../../logging/concepts/log-group.md) for this folder.
    * `log_group_id`: Custom log group ID. Logs will be written to this group.

      Specify one of the two parameters: `folder_id` or `log_group_id`.

    * `min_level`: Minimum logging level. Possible values: `TRACE`, `DEBUG`, `INFO` (default), `WARN`, `ERROR`, and `FATAL`.

    You can specify only one of the parameters: `folder_id` or `log_group_id`.
