.. _deployment-gcp:

Deploying to GCP
----------------

Hosting Dagit on GCE
~~~~~~~~~~~~~~~~~~~~

To host dagit on a bare VM or in Docker on GCE, see `Running Dagit as a service <local.html>`_.


Using Cloud SQL for run and event log storage
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We recommend launching a Cloud SQL PostgreSQL instance for run and events data. You can configure
Dagit to use Cloud SQL to run and events data by setting blocks in your
``$DAGSTER_HOME/dagster.yaml`` appropriately:

.. literalinclude:: dagster-pg.yaml
  :caption: dagster.yaml
  :language: YAML

In this case, you'll want to ensure you provide the right connection strings for your Cloud SQL
instance, and that the node or container hosting Dagit is able to connect to Cloud SQL.

Be sure that this file is present, and `DAGSTER_HOME` is set, on the node where Dagit is running.

Note that using Cloud SQL for run and event log storage does not require that Dagit be running in
the cloud. If you are connecting a local Dagit instance to a remote Cloud SQL storage, double check
that your local node is able to connect to Cloud SQL.

Using GCS for intermediates storage
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You'll probably also want to configure a GCS bucket to store intermediates. This enables
reexecution, review and audit of intermediate results, and cross-node cooperation (e.g., with the
multiprocessing or Dagster celery executors).

You'll first need to add GCS storage to your :py:class:`ModeDefinition`:

.. code-block:: python

    from dagster_gcp.gcs.resources import gcs_resource
    from dagster_gcp.gcs.system_storage import gcs_plus_default_storage_defs
    from dagster import ModeDefinition

    prod_mode = ModeDefinition(
        name='prod',
        system_storage_defs=gcs_plus_default_storage_defs,
        resource_defs={'gcs': gcs_resource}
    )


Then, just add the following YAML to your pipeline config:

.. code-block:: yaml

    storage:
      gcs:
        config:
          gcs_bucket: your-gcs-bucket-name

With this in place, your pipeline runs will store intermediates on GCS in the location
``gs://<bucket>/dagster/storage/<pipeline run id>/intermediates/<solid name>.compute``
