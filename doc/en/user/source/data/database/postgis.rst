.. _data_postgis: 

PostGIS
=======

`PostGIS <http://postgis.net>`_ is an open source spatial database based on `PostgreSQL <http://postgresql.com/>`_, and is currently one of the most popular open source spatial databases today.

Adding a PostGIS database
-------------------------

As with all formats, adding a shapefile to GeoServer involves adding a new store to the existing :ref:`webadmin_stores`  through the :ref:`web_admin`.

Using default connection
````````````````````````

To begin, navigate to :menuselection:`Stores --> Add a new store --> PostGIS NG`.

.. figure:: images/postgis.png
   :align: center

   *Adding a PostGIS database*

.. list-table::
   :widths: 20 80

   * - **Option**
     - **Description**
   * - :guilabel:`Workspace`
     - Name of the workspace to contain the database.  This will also be the prefix of any layer names created from tables in the database.
   * - :guilabel:`Data Source Name`
     - Name of the database.  This can be different from the name as known to PostgreSQL/PostGIS.
   * - :guilabel:`Description`
     - Description of the database/store. 
   * - :guilabel:`Enabled`
     - Enables the store.  If disabled, no data in the database will be served.
   * - :guilabel:`dbtype`
     - Type of database.  Leave this value as the default.
   * - :guilabel:`host`
     - Host name where the database exists.
   * - :guilabel:`port`
     - Port number to connect to the above host.
   * - :guilabel:`database`
     - Name of the database as known on the host.
   * - :guilabel:`schema`
     - Schema in the above database.
   * - :guilabel:`user`
     - User name to connect to the database.
   * - :guilabel:`passwd`
     - Password associated with the above user.
   * - :guilabel:`namespace`
     - Namespace to be associated with the database.  This field is altered by changing the workspace name.
   * - :guilabel:`max connections`
     - Maximum amount of open connections to the database. 
   * - :guilabel:`min connections`
     - Minimum number of pooled connections.
   * - :guilabel:`fetch size`
     - Number of records read with each interaction with the database.
   * - :guilabel:`Connection timeout`
     - Time (in seconds) the connection pool will wait before timing out.
   * - :guilabel:`validate connections`
     - Checks the connection is alive before using it.
   * - :guilabel:`Loose bbox`
     - Performs only the primary filter on the bounding box.  See the section on :ref:`postgis_loose_bbox` for details.
   * - :guilabel:`preparedStatements`
     - Enables prepared statements.

When finished, click :guilabel:`Save`.

Using JNDI
``````````

GeoServer can also connect to a PostGIS database using `JNDI <http://java.sun.com/products/jndi/>`_ (Java Naming and Directory Interface).

To begin, navigate to :menuselection:`Stores --> Add a new store --> PostGIS NG (JNDI)`.

.. figure:: images/postgisjndi.png
   :align: center

   *Adding a PostGIS database (using JNDI)*

.. list-table::
   :widths: 20 80

   * - **Option**
     - **Description**
   * - :guilabel:`Workspace`
     - Name of the workspace to contain the store.  This will also be the prefix of all of the layer names created from the store.
   * - :guilabel:`Data Source Name`
     - Name of the database.  This can be different from the name as known to PostgreSQL/PostGIS.
   * - :guilabel:`Description`
     - Description of the database/store. 
   * - :guilabel:`Enabled`
     - Enables the store.  If disabled, no data in the database will be served.
   * - :guilabel:`dbtype`
     - Type of database.  Leave this value as the default.
   * - :guilabel:`jndiReferenceName`
     - JNDI path to the database.
   * - :guilabel:`schema`
     - Schema for the above database.
   * - :guilabel:`namespace`
     - Namespace to be associated with the database.  This field is altered by changing the workspace name.

When finished, click :guilabel:`Save`.

Configuring PostGIS layers
--------------------------

When properly loaded, all tables in the database will be visible to GeoServer, but they will need to be individually configured before being served by GeoServer.  See the section on :ref:`webadmin_layers` for how to add and edit new layers.

.. _postgis_loose_bbox:

Using loose bounding box
------------------------

When the option :guilabel:`loose bbox` is enabled, only the bounding box of a geometry is used.  This can result in a significant performance gain, but at the expense of total accuracy; some geometries may be considered inside of a bounding box when they are technically not.

If primarily connecting to this data via WMS, this flag can be set safely since a loss of some accuracy is usually acceptable. However, if using WFS and especially if making use of BBOX filtering capabilities, this flag should not be set.

Publishing a PostGIS view
-------------------------

Publishing a view follows the same process as publishing a table. The only additional step is to manually ensure that the view has an entry in the ``geometry_columns`` table. 

For example consider a table with the schema::

  my_table( id int PRIMARY KEY, name VARCHAR, the_geom GEOMETRY )

Consider also the following view::

  CREATE VIEW my_view as SELECT id, the_geom FROM my_table;

Before this view can be served by GeoServer, the following step is necessary to manually create the ``geometry_columns`` entry::

  INSERT INTO geometry_columns VALUES ('','public','my_view','my_geom', 2, 4326, 'POINT' );

Performance considerations
--------------------------

GEOS
````

`GEOS <http://trac.osgeo.org/geos/>`_ (Geometry Engine, Open Source) is an optional component of a PostGIS installation.  It is recommended that GEOS be installed with any PostGIS instance used by GeoServer, as this allows GeoServer to make use of its functionality when doing spatial operations.  When GEOS is not available, these operations are performed internally which can result in degraded performance.

Spatial indexing
````````````````

It is strongly recommended to create a spatial index on tables with a spatial component (i.e. containing a geometry column).  Any table of which does not have a spatial index will likely respond slowly to queries.

Common problems
---------------

Primary keys
````````````

In order to enable transactional extensions on a table (for transactional WFS), the table must have a primary key.  A table without a primary key is considered read-only to GeoServer.

GeoServer has an option to expose primary key values (to make filters easier). Please keep in mind that these values are only exposed for your convenience - any attempted to modify these values using WFS-T update will be silently ignored. This restriction is in place as the primary key value is used to define the FeatureId. If you must change the FeatureId you can use WFS-T delete and add in a single Transaction request to define a replacement feature. 

Multi-line
``````````

To insert multi-line text (for use with labeling) remember to use escaped text::
   
   INSERT INTO place VALUES (ST_GeomFromText('POINT(-71.060316 48.432044)', 4326), E'Westfield\nTower');

   
   
