.. _deployment-overview:

###################
Deployment Overview
###################

.. tags:: Deployment, Infrastructure, Advanced

Up until now, the Flyte backend you've been working with has likely been accessible only on ``localhost`` and likely entirely in one Docker container. In order to handle the production load and make use of all the additional features Flyte offers, you need to replace, add, and configure certain components. This page describes at a high-level what a production-ready deployment might look like.

*******************
Usage of Helm
*******************

Flyte uses Helm to manage its deployment releases onto a K8s cluster. The chart and templates are located under the `helm folder <https://github.com/flyteorg/flyte/tree/master/charts>`__. There is a base ``values.yaml`` file but there are several files that fine tune those settings.

* ``values-eks.yaml`` should be additionally applied for AWS EKS deployments.
* ``values-gcp.yaml`` should be additionally applied for GCP GKE deployments.
* ``values-sandbox.yaml`` should be additionally applied for our sandbox install. See the :ref:`deployment-sandbox` page for more information.

.. warning:: 
   Opta is no longer actively in development. You can `migrate to terraform <https://docs.opta.dev/features/terraform/#migrate-from-opta-to-terraform>`__ to generate Terraform code from the Opta file. Briefly put, the steps would be as follows.
   
   1. Run ``opta appply`` on your Opta configuration files to migrate from Opta to Terraform and ensure that Terraform provisions your infrastructure. 
   2. To migrate from existing infrastructure state,

      1. Run ``opta generate-terraform --backend remote -c env.yaml`` for environment variable.
      2. Run ``opta generate-terraform --backend remote -c flyte.yaml`` for Flyte service.

   3. Run ``opta generate-terraform`` on every file to generate Terraform files.
   4. Save these generated Terraform files.

*********************
Relational Database
*********************

The ``FlyteAdmin`` and ``DataCatalog`` components rely on PostgreSQL to store persistent records. In the sandbox deployment, a containerized version of Postgres is included but for a proper Flyte installation, we recommend one of the cloud provided databases.  For AWS, we recommend their `RDS <https://aws.amazon.com/rds/postgresql/>`__ service, for GCP, `Cloud SQL <https://cloud.google.com/sql/docs/postgres/>`__, and Azure, `PostgreSQL <https://azure.microsoft.com/en-us/services/postgresql/>`__.

*****************************
Production Grade Object Store
*****************************

Core Flyte components such as Admin, Propeller, and DataCatalog, as well as user runtime containers rely on an Object Store to hold files. The sandbox deployment comes with a containerized Minio, which offers AWS S3 compatibility. We recommend swapping this out for `AWS S3 <https://aws.amazon.com/s3/>`__ or `GCP GCS <https://cloud.google.com/storage/>`__.

*********************
Project Configuration
*********************
As your Flyte user-base evolves, adding new projects is as simple as registering them through the command line ::

   $ flytectl create project --id myflyteproject --name "My Flyte Project" --description "My very first project onboarding onto Flyte"

A cron which runs at the cadence specified in FlyteAdmin configuration ensures that all Kubernetes resources necessary for the new project are created and new workflows can successfully
be registered and executed within it. See :std:ref:`flytectl <flytectl:flytectl_create_project>` for more information.

This project should immediately show up in the Flyte console after refreshing.

**********
Scheduling
**********
Flyte has an in-built native scheduler to provide automated periodic execution of your launch plans. See the :ref:`Scheduling Launch Plans <concepts-schedules>` page for detailed information.

*************
Notifications
*************
Users can be notified about their workflow completions via email, slack, pagerduty etc. See the :ref:`Notifications <deployment-cluster-config-notifications>` page for detailed information.

.. note::
   In AWS, `SNS <https://aws.amazon.com/sns>`_ and `SQS <https://aws.amazon.com/sqs/>`_ are used for handling notifications.

**************
Authentication
**************
Flyte ships with its own authorization server, as well as the ability to use an external authorization server if your external IDP supports it.  See the :ref:`authorization <deployment-cluster-config-auth-setup>` page for detailed configuration.
