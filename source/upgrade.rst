.. _nggs_prem_admin_upgrade:

Upgrade
==========


The version number for NextGIS GeoServices On-Premise contains three parts in ``A.B.C`` pattern, where ``A.B`` is the main version number and ``C`` is a patch version.
Updates between main versions must be performed step by step, without skipping a version. So if your current version is ``2.18.0`` and you wish to update to ``2.20.0``, you need to update to ``2.19.0`` first, and only then to ``2.20.0``. If there are patched versions (e.g. ``2.18.1``, ``2.18.2`` and so on), upgrade to the latest patched version of the main version.

.. important:: All steps in this section must be performed by the ``root`` user. If you use ``sudo`` to avoid mixing up the commands, we recommend first running ``sudo -i`` to get a fully functional root user session.

To upgrade from one version to another follow these steps:

**Step 1:** Review the actions and requirements listed below for the specific version:

.. contents::
   :local:
   :depth: 1 

**Step 2:** Stop all the services:


.. code-block:: shell

  $ docker compose stop

**Step 3:** In the file ``.env`` change the value of ``IMAGE_VERSION`` to the code of the version you're upgrading to.

If you haven't modified the configuration, you can instead `download the current config template <https://docs.nextgis.com/docs_geoserv_prem/source/admin.html#nextgis-geoservices>`_ and unpack it. If the server does not have Internet access, download the file on another PC and transfer it to the server.

**Step 4:** Restart all the services:

.. code-block:: shell

  $ docker compose up -d



Upgrade to 2.25.x from 2.22.x
-------------------------------

If you set up an OAuth server with GeoServices, change path in the Redirect URI from ``/oauth2/callback`` to ``/oauth``.

Upgrade to 2.22.x from 2.20.x
-------------------------------------------

No additional steps needed, proceed with the standard steps described above.

Upgrade to 2.20.x from 2.19.x
-------------------------------

If you've modified the configuration, then in order to preserve them, do the following:

In the  ``docker-compose.yaml`` file add ``node-renderer``, in the ``app`` service add environment variable ``NODE_RENDERER_SECRET``:

.. code-block:: yaml  
   
  app:
    ...
    environment:
      NODE_RENDERER_SECRET: node-renderer-secret
  ...
  node-renderer:
    image: ${IMAGE_BASE}/node-renderer:${IMAGE_VERSION}
    restart: unless-stopped
    environment:
      SECRET: node-renderer-secret

After that perform the upgrade as described above.

If no modifications were made, you can instead download a current template at Step 3 and unpack it.

Upgrade to 2.17.x - 2.19.x
----------------------------

No additional steps needed, proceed with the standard steps described above.

For versions 2.16.1 and below
------------------------------

If you have GeoServices 2.16.0 or earlier, contact us at support@nextgis.com.


.. note:: If you need to perform an upgrade in a closed network, contact us at support@nextgis.com
