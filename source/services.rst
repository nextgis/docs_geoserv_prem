.. _docs_geoserv_prem_services:


Services
========

Service groups
---------------

Services can be added only to specific groups of services. Groups are created in Settings in **Service groups** tab.

.. figure:: _static/geosop_gr_serv1_en.png
   :name: geosop_gr_serv1
   :align: center
   :width: 20cm

   One of the service groups 


You can delete or edit a group using buttons.

.. figure:: _static/geosop_gr_serv2_en.png
   :name: geosop_gr_serv2
   :align: center
   :width: 20cm

   Service groups settings

To create a new group, press **Add** and enter a name for it.

.. figure:: _static/geosop_gr_serv3_en.png
   :name: geosop_gr_serv3
   :align: center
   :width: 16cm

   Adding new service group

NGW Web Maps
------------

`NextGIS Web <https://nextgis.com/nextgis-web/>`_ is a server-based geoinformation system for gathering, storing, visualising and analyzing geospacial data.

NGW Web Maps service allows to created cached tile services based on Web Maps created in NextGIS Web.

Administrator enters URL of a Web Map in NextGIS Web, service name and scale limits for caching.
After that the service will appear in the list. Service can be modified or deleted.

Working with the service does not engage NextGIS Web itself, so the service can handle high peak loads and reduce the load on NextGIS Web.

.. figure:: _static/geosop_ngw1_en.png
   :name: geosop_ngw1
   :align: center
   :width: 20cm

   Button for creating new service

.. figure:: _static/geosop_ngw2_en.png
   :name: geosop_ngw2
   :align: center
   :width: 20cm

   Parameters for the new service

.. figure:: _static/geosop_ngw3_en.png
   :name: geosop_ngw3
   :align: center
   :width: 20cm

   Newly created sevice in the group


External TMS
------------

GeoServices allows to add, cache and use external TMS.

.. figure:: _static/geosop_tms1_en.png
   :name: geosop_tms1
   :align: center
   :width: 20cm

   Button for creating new service

Enter name for the service, URL of the TMS service, select coordinate system and scale limits.
The newly created service will appear in the selected group. Service can be modified or deleted.

.. figure:: _static/geosop_tms2_en.png
   :name: geosop_tms2
   :align: center
   :width: 20cm

   Parameters for the new TMS service

.. figure:: _static/geosop_tms3_en.png
   :name: geosop_tms3
   :align: center
   :width: 20cm

   Newly created TMS sevice in the group

.. _gs_prem_seed:

Seeding
-----------

To help services work faster you can cache tiles of the area.

Press the pencil icon to enter the edit mode of the tile service.

Go to the "Seeding" tab and press **Create new task**.

.. figure:: _static/geosop_seeding_create_task_en.png
   :name: geosop_seeding_create_task_pic
   :align: center
   :width: 20cm

   "Seeding" tab

In the opened dialog configure seeding parameters.

* Cache type - Default raster (recommended) or Vector tiles;
* Task type - defines how the cache is rendered:

   * the default option is *Render only missing tiles*, the system checks already loaded tiles and loads the absent ones;
   * *Full render* - all tiles for the area will be re-loaded, it's a faster process;
   * *Delete cache* - select it if you need to clear cache and remove all previously rendered tiles;

* Zoom list - higher zoom levels need more time to process, so we recommend staying within level 12 unless necessary;

.. note:: Keep in mind that the API key also can have zoom limits. In this case there's no point to set zoom levels beyond that limit for seeding.

* Area - upload boundary as a file or draw it on the map.

.. figure:: _static/geosop_seeding_task_settings_en.png
   :name: geosop_seeding_task_settings_pic
   :align: center
   :width: 14cm

   Seeding task settings

After configuring seeding parameters press **Create**.

The task will appear on the tab. Here you can check its status: pending, in progress, finished, failed with error.

.. figure:: _static/geosop_seeding_task_status_en.png
   :name: geosop_seeding_task_status_pic
   :align: center
   :width: 20cm

   Status of the seeding task

.. important:: Seeding tasks are processed sequentially, so the new task will start only after all the previous tasks are completed.
