.. sectionauthor:: Роман Гайнуллов <roman.gainullov@nextgis.ru>

.. _docs_geoserv_prem_admin:

Administrator manual for NextGIS GeoServices
=====================================================

Introduction
-------------

This manual describes the process of deploying NextGIS GeoServices sowftware on-premise. Mainly it uses Docker platform and docker-compose tool. All steps are performed on Linux-based OS.

.. _nggs_prem_admin_address:

Select connection addresses
------------------------------

NextGIS GeoServices uses one HTTP (or HTTPS) entry. Users interact with the software via Web interface and API. The default value is http://server.example.com:8088 where server.example.com is the DNS name of the server where the software is deployed. If strictly necessary, the server IP address can be used instead of server.example.com.

If your IT infrastructure allows for it, it is recommended to set up a reverse proxy for TLS encryption and using HTTPS. It is especially important if the software is to be accessed not just from the local network, but also from the Internet. In that case the addresses of the entry points depend on the settings of the reverse proxy.

Contact your IT department to choose addresses you wish to use and note them down, you'll need them later. The reverse proxy is set up by the client's IT department, it is not a responsibility of NextGIS company. The required parameters are cited `below using Nginx as example <https://docs.nextgis.ru/docs_geoserv_prem/source/admin.html#nggs-prem-admin-proxy>`_.

.. _nggs_prem_admin_docker:

Install and configure Docker
---------------------------------

If the server does not yet have Docker Engine and Docker Compose installed, first you need to install them or update them to the latest versions:

* `Docker Engine <https://docs.docker.com/engine/install/>`_
* `Docker Compose <https://docs.docker.com/compose/install/linux/>`_

To get the images log in to NextGIS Container Registry with the username (example) and password (sesame) provided by NextGIS::

   $ docker login cr.nextgis.com -u example -p sesame
   Login Succeeded

If the software is deployed to a server without Internet access, contact support for a single-file image archive instead. You'll need to transfer it to the server and load the images using 'docker load' command.

.. _nggs_prem_admin_installgs:

Install NextGIS GeoServices
------------------------------

On the server where you plan to deploy GeoServices, create the ``/srv/geoservices`` directory, then go to it, download the configuration template (``docker-compose-2.16.1.tar.bz2``, where 2.16.1 is the current version) and unpack it. If the server does not have Internet access, download the file on another PC and transfer it to the server.

.. code-block::

	$ mkdir /srv/geoservices
	$ cd /srv/geoservices
	$ wget https://nextgis.com/onpremise/geoservices/docker-compose-2.16.1.tar.bz2
	$ tar jxf docker-compose-2.16.1.tar.bz2
	Edit the .env file in a text editor and enter the values for: POSTGRES_PASSWORD, DB_PASSWORD, BM_DB_PASSWORD (must have the same values), ADMIN_PASSWORD and SESSION_KEY. In the end you should get something like this:
	IMAGE_VERSION=2.16.1
	IMAGE_BASE=cr.nextgis.com/geoservices
	COMPOSE_BIND=0.0.0.0
	
	DEBUG=false
	S3_SSL=false
	EXT_SOURCES_SUPPORT=false
	POSTGRES_USER=geoservices
	SESSION_KEY=secret1
	POSTGRES_PASSWORD=secret2
	DB_PASSWORD=secret2
	BM_DB_PASSWORD=secret2
	ADMIN_PASSWORD=secret3

After that you can launch the Docker Compose stack. We recommend launching postgres service first, then after about 30 seconds launch the rest:

.. code-block::

	$ docker compose up -d postgres && sleep 30      
	[+] Running 3/3
	 ✔ Network geoservices_default       Created         0.0s 
	 ✔ Volume "geoservices_postgres"     Created         0.1s 
	 ✔ Container geoservices-postgres-1  Started         4.2s
	
	$ docker compose up -d
	[+] Running 8/8
	 ✔ Volume "geoservices_s3"           Created         0.0s 
	 ✔ Volume "geoservices_secret"       Created         0.1s 
	 ✔ Volume "geoservices_data"         Created         0.0s 
	 ✔ Volume "geoservices_redis"        Created         0.1s 
	 ✔ Container geoservices-postgres-1  Running         0.0s 
	 ✔ Container geoservices-redis-1     Started         7.3s 
	 ✔ Container geoservices-s3-1        Started         7.5s 
	 ✔ Container geoservices-app-1       Started         5.9s

This completes the installation. If you use HTTPS, next `configure the reverse proxy server <https://docs.nextgis.ru/docs_geoserv_prem/source/admin.html#nggs-prem-admin-proxy>`_. Otherwise proceed to `operability check <https://docs.nextgis.ru/docs_geoserv_prem/source/admin.html#nggs-prem-admin-check>`_.





.. _nggs_prem_admin_proxy:

Recommendations for reverse proxy setup
---------------------------------------------------

To use HTTPS encryption we recommend setting up a reverse proxy server based on Nginx. For reference here's a fragment of the configuration file for geoservices.example.com:

.. code-block::

	server {
	    server_name geoservices.example.com;
	    # Server directives: listen, ssl_* etc
	
	    location / {
	        client_max_body_size 2G;
	
	        proxy_http_version 1.1;
	        proxy_pass http://127.0.0.1:8088;
	        proxy_set_header Host $http_host;
	        proxy_set_header Upgrade $http_upgrade;
	        proxy_set_header Connection $proxy_connection;
	        proxy_set_header X-Forwarded-Proto $scheme;
	        proxy_set_header X-Forwarded-For $remote_addr;
	    }
	}

The client_max_body_size directive defines the max size of the upload file (2 GiB in our example).



.. _nggs_prem_admin_check:

Operability check
-----------------------------

In a Web browser open the Web interface of NextGIS GeoServices using the URL you've chosen.

A sign-in form should appear. Enter the username 'admin' and the password that you set in the ADMIN_PASSWORD variable.

Go to the About page, it must look like this:

.. figure:: _static/geosop_set_about_en.png
   :name: geosop_set_about_pic
   :align: center
   :width: 16cm

   About page


.. _docs_geoserv_prem_admin_var:

Complete list of environment variables for NextGIS GeoServices
-------------------------------------------------------------------

For each variable the table provides the following info: required or not, default value, short description.

.. list-table::
   :header-rows: 1

   * - **Variable**
     - **Required**
     - **Default value**
     - **Description**
   * - DEBUG
     - no
     - true
     - Enable SQL debugging
   * - ADMIN_PASSWORD
     - yes
     - admin
     - Pre-set administrator password
   * - SESSION_KEY
     - no
     - secret
     - Session key - random text
   * - GIN_MODE
     - no
     - release
     - Controls debugging of the gin library and web application diagnostic messages
   * - INCLUDE_ORIGIN_SUFFIXES
     - no
     - "nextgis.com", "nextgis.ru"
     - Origin array that is added to those specified in the API key
   * - TOKEN_CACHE_SIZE
     - no
     - 1024
     - Max number of authorization tokens in cache
   * - TIMEOUT
     - no
     - 180
     - Network request timeout
   * - FILE_TIMEOUT
     - no
     - 1800
     - Timeout for file downloads
   * - SESSION_MAX_AGE
     - no
     - 259200
     - How long a web application session lasts
   * - HTTP_SKIP_SSL_VERIFY
     - no
     - false
     - Do not check https certificates
   * - LDAP_LOGIN
     - no
     - false
     - | Authentication via LDAP
       | Keep default
   * - LDAP_TLS
     - no
     - no
     - Use TLS
   * - LDAP_URL
     - no
     - “”
     - LDAP server address
   * - LDAP_USER_FILTER
     - no
     - (objectClass=posixAccount)
     - User search filter
   * - LDAP_USER_ATTR
     - no
     - uid
     - User attribute
   * - LDAP_GROUP_FILTER
     - no
     - cn=geoservices
     - Group search filter
   * - LDAP_GROUP_ATTR
     - no
     - memberUid
     - Group attribute
   * - LDAP_DEFAULT_GROUP_ID
     - no
     - 0
     - Default group for LDAP users - 0 group assignment disabled
   * - LDAP_UPDATE_GROUPS
     - no
     - false
     - Update inclusion to groups for users
   * - OAUTH2_LOGIN
     - no
     - false
     - Enable authentication via OAuth2
   * - OAUTH2_CLIENT_ID
     - no
     - 
     - OAuth2 client ID
   * - OAUTH2_CLIENT_SECRET
     - no
     - 
     - OAuth2 client secret
   * - OAUTH2_REDIRECT_URI
     - no
     - 
     - OAuth2 redirect URI
   * - OAUTH2_ENDPOINT
     - no
     - https://my.nextgis.com
     - Endpoint
   * - OAUTH2_SCOPE
     - no
     - user_info.read
     - Scope
   * - OAUTH2_TYPE
     - no
     - 1
     - Authorization type: 1 -NextGIS ID, 2 - Keycloak, 3 - custom, 4 - Blitz
   * - OAUTH2_TOKEN_ENDPOINT
     - no
     - https://my.nextgis.com/oauth2/token
     - Token endpoint
   * - OAUTH2_AUTH_ENDPOINT
     - no
     - https://my.nextgis.com/oauth2/authorize
     - Authorization endpoint
   * - OAUTH2_USERINFO_ENDPOINT
     - no
     - https://my.nextgis.com/api/v1/user_info
     - Endpoint for user info (not needed for JWT)
   * - OAUTH2_INTROSPECTION_ENDPOINT
     - no
     - https://my.nextgis.com/oauth2/introspect
     - Introspection endpoint
   * - OAUTH2_PROFILE_SUBJ_ATTR
     - no
     - nextgis_guid
     - Field for getting user ID (subject)
   * - OAUTH2_PROFILE_KEYNAME_ATTR
     - no
     - username
     - Field for getting username
   * - OAUTH2_PROFILE_FIRSTNAME_ATTR
     - no
     - first_name
     - Field for user's first name
   * - OAUTH2_PROFILE_LASTNAME_ATTR
     - no
     - last_name
     - Field for user's last name
   * - OAUTH2_USER_AUTOCREATE
     - no
     - true
     - Creates user on first enter
   * - OAUTH2_VALIDATE_KEY
     - no
     - “”
     - Key to verify JWT signature
   * - OAUTH2_CREATE_GROUPS
     - no
     - false
     - Create groups based on user roles
   * - OAUTH2_UPDATE_GROUPS
     - no
     - false
     - Update user inclusion into groups based on user roles
   * - OAUTH2_TOKEN_CACHE_TTL
     - no
     - 3600
     - Default token lifetime unless another is not returned by server
   * - OAUTH2_LOGOUT_ENDPOINT
     - no
     - “”
     - Logout endpoint
   * - OAUTH2_GROUPS_JWT_KEY
     - no
     - resource_access/{client_id}/roles
     - Path for role/group search in JWT token
   * - LOCAL_LOGIN
     - no
     - true
     - Allow local user accounts
   * - DEFAULT_LANGUAGE
     - no
     - en
     - Default language
   * - LOG
     - no
     - false
     - stdout messages in structured format
   * - LOG_ONLY_EDITS
     - no
     - false
     - stdout only contains message on data modifications
   * - CLOUD_MODE
     - no
     - false
     - Cloud launch mode
   * - MAX_AGE
     - no
     - 43200
     - Time the tiles are stored in user's browser - 12 hrs
   * - EXT_TMS_SUPPORT
     - no
     - false
     - Enable external TMS service support
   * - 
     - no
     - https://geoservices.nextgis.com
     - URL for integration with public cadaster map (PKK)
   * - PKK_EXTERNAL_APIKEY
     - no
     - “”
     - APIKey for integration with PKK
   * - PKK_TILES_URL
     - no
     - “”
     - URL of the local serveer for PKK integration
   * - PKK_FEATURES_URL
     - no
     - “”
     - URL of the local serveer for PKK integration
   * - PKK_MIN_ZOOM
     - no
     - 3
     - Min level of PKK tile zoom
   * - PKK_MAX_ZOOM
     - no
     - 18
     - Max level of PKK tile zoom
   * - PKK_REGION
     - no
     - | MULTIPOLYGON (((-168.4 84,-168.4 50,-179.999999 50,-179.9999999 84,-168.4 84)),
       | ((130 40,130 84,179.999999 84,179.999999 40,130 40)),
       | ((129.999999 84,129.999999 47,82.5 47,82.5 84,129.999999 84)),
       | ((82.4999999 50,50 50,50 84,82.4999999 84,82.4999999 50)),
       | ((20 84,49.999999 84,49.999999 40,20 40,20 84)))
     - Are for PKK tiles query
   * - DB_TYPE
     - yes
     - sqlite3
     - DB type - sqlite3, postgres, mysql
   * - DB_USER
     - no
     - geoservices
     - User account used to access DB
   * - DB_PASSWORD
     - yes
     - 
     - DB password
   * - DB_HOST
     - yes
     - localhost
     - DB address
   * - DB_PORT
     - yes
     - 5432
     - DB port
   * - DB_NAME
     - no
     - geoservices
     - DB name
   * - DB_MAXCONN
     - no
     - 50
     - Max number of connections
   * - DB_MAXIDLECONN
     - no
     - 10
     - Max number of idle connections
   * - DB_SSL_MODE
     - no
     - 
     - | disable - I don't care about security, and I don't want to pay the overhead of encryption.
       | allow - I don't care about security, but I will pay the overhead of encryption if the server insists on it.
       | prefer - I don't care about encryption, but I wish to pay the overhead of encryption if the server supports it.
       | require - I want my data to be encrypted, and I accept the overhead. I trust that the network will make sure I always connect to the server I want.
       | verify-ca - I want my data encrypted, and I accept the overhead. I want to be sure that I connect to a server that I trust.
       | verify-full - I want my data encrypted, and I accept the overhead. I want to be sure that I connect to a server I trust, and that it's the one I specify.
   * - DB_SSL_CERT
     - no
     - 
     - Path to certificate file
   * - DB_SSL_KEY
     - no
     - 
     - Path to key file
   * - DB_SSL_ROOT_CERT
     - no
     - 
     - Path to root certificate
   * - REDIS_ENDPOINT
     - yes
     - localhost:6379
     - Redis service address
   * - REDIS_MAX_IDLE
     - no
     - 100
     - Max time before pool connection is closed
   * - REDIS_MAX_ACTIVE
     - no
     - 1000
     - Max number of active connections in the pool
   * - REDIS_IDLE_TIMEOUT
     - no
     - 60
     - time before pool connection is closed
   * - REDIS_CLUSTER
     - no
     - false
     - Connect to Redis cluster
   * - REDIS_NODES
     - no
     - "localhost:6379 localhost:7001 localhost:7002 localhost:7003 localhost:7004 localhost:7004"
     - Redis cluster node (only used if REDIS_CLUSTER == true)
   * - REDIS_KEY_PREFIX
     - no
     - “”
     - Prefix for Redis keys generated by the app
   * - REDIS_USER
     - no
     - geoservices
     - Redis user login
   * - REDIS_DATABASE
     - no
     - 0
     - Redis data base
   * - REDIS_SSL
     - no
     - false
     - Connection using SSL/TLS
   * - REDIS_INSECURE_SSL
     - no
     - false
     - Do not validate SSL/TLS
   * - S3_ACCESS_KEY
     - yes
     - Q3AM3UQ867SPQQA43P2F
     - Access key for S3
   * - S3_SECRET_KEY
     - yes
     - zuf+tfteSlswRu7BJ86wekitnifILbZam1KYY3TG
     - Secret access key for S3
   * - S3_ENDPOINT
     - yes
     - play.min.io
     - S3 server address
   * - S3_SSL
     - no
     - true
     - Use encryption
   * - S3_INSECURE_SSL
     - no
     - false
     - Do not check SSL certificates
   * - S3_DEFAULT_STORAGE_CLASS
     - no
     - REDUCED_REDUNDANCY
     - Storage method: REDUCED_REDUNDANCY or STANDARD
   * - S3_BUCKET_NAME
     - no
     - geoservices
     - Bucket name
   * - S3_KEY_PREFIX
     - no
     - “”
     - Prefix for S3 keys generated by the app
   * - S3_NO_OBJECT_TAGGING
     - no
     - false
     - Do not dedupe or apply expiration time if S3 does not support tags
   * - RASTER_MAX_ZOOM
     - no
     - 20
     - Max zoom for raster tiles
   * - VECTOR_MAX_ZOOM
     - no
     - 14
     - Max zoom for vector tiles
   * - EXPIRE_TILES_MIN_ZOOM
     - no
     - 7
     - Min zoom for tile expiration control
   * - EXPIRE_TILES_MAX_ZOOM
     - no
     - 16
     - Max zoom for tile expiration control
   * - NET_MAX_RETRY_COUNT
     - no
     - 5
     - Number of attempts for iterative queries
   * - LONG_REQUEST_MIN_TIME
     - no
     - 0
     - Only log long queries - 0 disabled
   * - NGW_URL
     - no
     - https://sandbox.nextgis.com
     - Address of associated NextGIS Web (to create cache from basemaps)
   * - NGW_LOGIN
     - no
     - administrator
     - Login for NextGIS Web - needed to render tile while seeding
   * - NGW_APIKEY
     - no
     - admin
     - Password for NextGIS Web - needed to render tile while seeding
   * - NGW_FEATURE_LIMIT
     - no
     - 256
     - Number of entries in page mode
   * - USERS_MAINTANCE_SCHEDULE
     - no
     - @every 9m1s
     - Schedules user cache clearing
   * - SERVICE_MAINTANCE_SCHEDULE
     - no
     - @every 10m4s
     - Schedules service cache clearing
   * - SERVICE_HOUSEKEEPING_SCHEDULE
     - no
     - @every 25h30m10s
     - Schedules system clearing
   * - DATA_STORE
     - no
     - /data
     - | Path to data necessary for service functioning
       | Keep default
   * - FILE_STORE
     - no
     - /work
     - Path to the working directory. This is the folder for downloading files, performing operations, creating temporary files.
   * - BM_DB_HOST
     - no
     - localhost
     - | Host with PostGIS DB. Upon starting web application checks for DB connection and necessary extensions 
       | If connection fails or extensions are not found, basemap section is disabled 
   * - BM_DB_PORT
     - no
     - 5432
     - Port for PostGIS DB
   * - BM_DB_NAME
     - no
     - basemap
     - DB name for OSM dump import
   * - BM_DB_USER
     - no
     - geoservices
     - User account used to access basemap DB
   * - BM_DB_PASSWORD
     - yes
     - 
     - Password for basemap DB access
   * - BM_DB_SSL_MODE
     - no
     - 
     - | disable - I don't care about security, and I don't want to pay the overhead of encryption.
       | allow - I don't care about security, but I will pay the overhead of encryption if the server insists on it.
       | prefer - I don't care about encryption, but I wish to pay the overhead of encryption if the server supports it.
       | require - I want my data to be encrypted, and I accept the overhead. I trust that the network will make sure I always connect to the server I want.
       | verify-ca - I want my data encrypted, and I accept the overhead. I want to be sure that I connect to a server that I trust.
       | verify-full - I want my data encrypted, and I accept the overhead. I want to be sure that I connect to a server I trust, and that it's the one I specify.
   * - BM_DB_SSL_CERT
     - no
     - 
     - Path to certificate file
   * - BM_DB_SSL_KEY
     - no
     - 
     - Path to key file
   * - BM_DB_SSL_ROOT_CERT
     - no
     - 
     - Path to root certificate
   * - BM_DB_PARALLEL_SQL
     - no
     - true
     - Perform parallel DB queries for vector tiles
   * - BM_DIFF_URL
     - no
     - 
     - Address to download OSM delta files (only if EXT_SOURCES_SUPPORT == true)
   * - BM_EXPIRE_TILES_MIN_ZOOM
     - no
     - 7
     - Min zoom to log invalid tiles
   * - BM_EXPIRE_TILES_MAX_ZOOM
     - no
     - 16
     - Max zoom to log invalid tiles
   * - EXT_SOURCES_SUPPORT
     - no
     - false
     - | Allow/forbid getting files from Internet. For example, to initialize DB by downloading a dump from Internet or getting diff regularly. 
       | Keep default
   * - EXT_RASTER_RESAMPLING
     - no
     - bilinear
     - | Raster interpolation. Supported methods:
       | near: nearest neighbour resampling (default, fastest algorithm, worst interpolation quality).
       | bilinear: bilinear resampling.
       | cubic: cubic resampling.
       | cubicspline: cubic spline resampling.
       | lanczos: Lanczos windowed sinc resampling.
       | average: average resampling, computes the weighted average of all non-NODATA contributing pixels.
       | rms root mean square / quadratic mean of all non-NODATA contributing pixels (GDAL >= 3.3)
       | mode: mode resampling, selects the value which appears most often of all the sampled points. 
       | In the case of ties, the first value identified as the mode will be selected.
       | max: maximum resampling, selects the maximum value from all non-NODATA contributing pixels.
       | min: minimum resampling, selects the minimum value from all non-NODATA contributing pixels.
       | med: median resampling, selects the median value of all non-NODATA contributing pixels.
       | q1: first quartile resampling, selects the first quartile value of all non-NODATA contributing pixels.
       | q3: third quartile resampling, selects the third quartile value of all non-NODATA contributing pixels.
       | sum: compute the weighted sum of all non-NODATA contributing pixels (since GDAL 3.1)
   * - EXT_ZEROBLOCKHTTPCODES
     - no
     - "204,404"
     - Codes of HTTP responses for white tiles
   * - LOCALES
     - no
     - “ru en”
     - List of user interface languages
   * - OUTDATED_STAT_TABLE_ROWS
     - no
     - 2*365*24*time.Hour
     - Delete log entries from before 2 years
   * - ENABLE_SWAGGER
     - no
     - true
     - Enable web interface for swagger
   * - SSL_CERT_FILE
     - no
     - 
     - | To override path to certificate
       | https://stackoverflow.com/a/67622500/2901140
       | 
       | You can also add certificates using following paths (depends on the platform):
       | 
       | "/etc/ssl/certs/ca-certificates.crt", 
       | // Debian/Ubuntu/Gentoo etc. "/etc/pki/tls/certs/ca-bundle.crt", 
       | // Fedora/RHEL 6 "/etc/ssl/ca-bundle.pem", 
       | // OpenSUSE "/etc/pki/tls/cacert.pem", 
       | // OpenELEC "/etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem", 
       | // CentOS/RHEL 7 "/etc/ssl/cert.pem", 
       | // Alpine Linux
       | 
       | 
       | https://stackoverflow.com/a/40051432/2901140
   * - DEFAULT_KEY_EXPIRE
     - no
     - 7 days
     - TTL for tiles of external services


