.. sectionauthor:: Роман Гайнуллов <roman.gainullov@nextgis.ru>

.. _docs_geoserv_prem_admin:

Инструкция администратора для NextGIS GeoServices
=====================================================

Введение
---------

В настоящей инструкции приведен порядок развертывания ПО NextGIS GeoServices. В качестве преимущественного способа развертывания и работы ПО предполагается использование технологии Docker и средство docker-compose. Все шаги необходимо выполнять в ОС на базе Linux.

.. _nggs_prem_admin_address:

Выбор адресов подключения
------------------------------

NextGIS GeoServices использует одну точку подключения по HTTP (или HTTPS). Веб-интерфейс NextGIS GeoServices и API - основной адрес по которому пользователи взаимодействуют с продуктом. Значение по умолчанию -  http://server.example.com:8088, где server.example.com DNS-имя сервера, на котором развернут продукт. В крайнем случае вместо server.example.com можно использовать IP-адрес сервера.

Если ваша ИТ-инфраструктура позволяет, рекомендуется настроить обратный прокси-сервер (reverse proxy) для обеспечения TLS-шифрования и работы через HTTPS. Это особенно важно, если доступ к продукту будет осуществляться не только из локальной сети, но и через Интернет. В этом случае адреса точек подключения зависят от настроек обратного прокси-сервера.

Совместно с вашим ИТ-департаментом и выберите адреса, которые вы хотите использовать и запишите их, они потребуются в дальнейшем. Настройка обратного прокси-сервера должна выполняется силами ИТ-департамента клиента и не относится к области ответственности компании NextGIS. Необходимые параметры приведены в этом документе ниже на примере Nginx.

.. _nggs_prem_admin_docker:

Установка и настройка Docker
---------------------------------

Если на сервере еще не установлен Docker Engine и Docker Compose их нужно установить или обновить до актуальных версий:

* `Docker Engine <https://docs.docker.com/engine/install/>`_
* `Docker Compose <https://docs.docker.com/compose/install/linux/>`_

Для получения образов необходимо выполнить авторизацию в NextGIS Container Registry c использованием имени пользователя (example) и пароля (sesame), предоставленного компанией NextGIS::

   $ docker login cr.nextgis.com -u example -p sesame
   Login Succeeded

В случае если развертывание осуществляется на сервере без доступа к Интернет, то вместо этого шага свяжитесь со службой поддержки для получения архива образов в виде одного файла. Его нужно будет перенести на сервер и загрузить образы командой docker load.

.. _nggs_prem_admin_installgs:

Установка NextGIS GeoServices
------------------------------

На сервере, где планируется развернуть GeoServices, создайте директорию ``/srv/geoservices`` и перейдите в нее, скачайте шаблон конфигурации (``docker-compose-2.16.1.tar.bz2``, где 2.16.1 - текущая версия) и распакуйте его. Если установка производится на сервере без доступа в Интернет, скачайте файл на другом ПК и перенесите его на сервер.

.. code-block::

	$ mkdir /srv/geoservices
	$ cd /srv/geoservices
	$ wget https://nextgis.com/onpremise/geoservices/docker-compose-2.16.1.tar.bz2
	$ tar jxf docker-compose-2.16.1.tar.bz2
	Отредактируйте файл .env в текстовом редакторе заполнив значения переменных: POSTGRES_PASSWORD, DB_PASSWORD, BM_DB_PASSWORD (должны иметь одинаковые значения), ADMIN_PASSWORD и SESSION_KEY. В итоге должно получится приблизительно следующее:
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

После этого можно запускать стек Docker Compose, вначале рекомендуется запустить сервис postgres, подождать полминуты и затем уже запустить остальное:

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

На этом установка завершена, если используется HTTPS, то на этом этапе выполните настройку обратного прокси-сервера. Если нет, то сразу приступайте к проверке работоспособности.




.. _nggs_prem_admin_proxy:

Рекомендации по настройке обратного прокси-сервера
---------------------------------------------------

Для обеспечения HTTPS шифрования мы рекомендуем использовать обратный прокси-сервер на базе Nginx, для справки приведем пример фрагмента конфигурационного файла для geoservices.example.com:

.. code-block::

	server {
	    server_name geoservices.example.com;
	    # Директивы сервера: listen, ssl_* и пр.
	
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

Директива client_max_body_size определяет максимальный размер загружаемого файла (в примере 2 GiB).



.. _docs_geoserv_prem_admin_launch:

Запуск
---------

Для запуска и проверки корректности работы стэка необходимо выполнить:

.. code-block::

	docker-compose up

Для запуска в режиме сервиса:

.. code-block::

	docker-compose up -d

После запуска в браузере открыть адрес сервиса с указанием порта 8088. Например,

.. code-block::

	http://localhost:8088

.. _nggs_prem_admin_check:

Проверка работоспособности
-----------------------------

Откройте в браузере веб-интерфейс NextGIS GeoServices по адресу, который вы выбрали.

Должна открыться страница ввода имени пользователя и пароля. Введите имя пользователя admin и пароль, который вы указали в переменной ADMIN_PASSWORD.

Если перейти на страницу *О проекте*, то страница должна выглядеть следующим образом.

.. figure:: _static/geosop_set_about_ru.png
   :name: geosop_set_about_pic
   :align: center
   :width: 16cm

   Раздел "О проекте"


.. _docs_geoserv_prem_admin_var:

Приложение: полный список переменных окружения NextGIS GeoServices
-------------------------------------------------------------------

Для каждой переменной указано, обязательная ли она, значение по умолчанию, а также дано краткое описание.

.. list-table::
   :width: 80%
   :header-rows: 1

   * - **Переменная**
     - **Обязательная**
     - **Значение по умолчанию**
     - **Описание**
   * - DEBUG
     - нет
     - true
     - Включить режим отладки SQL запросов
   * - ADMIN_PASSWORD
     - да
     - admin
     - Изначальный пароль администратора
   * - SESSION_KEY
     - нет
     - secret
     - Ключ для сессии - случайная строка
   * - GIN_MODE
     - нет
     - release
     - Параметр для управления отладкой библиотеки gin и вывода диагностических сообщений веб приложения
   * - INCLUDE_ORIGIN_SUFFIXES
     - нет
     - "nextgis.com", "nextgis.ru"
     - Массив origin который добавляется к тем что указаны в API ключе
   * - TOKEN_CACHE_SIZE
     - нет
     - 1024
     - Максимальное количество токенов авторизации в кэше
   * - TIMEOUT
     - нет
     - 180
     - Таймаут сетевых запросов
   * - FILE_TIMEOUT
     - нет
     - 1800
     - Таймаут на скачивание файла
   * - SESSION_MAX_AGE
     - нет
     - 259200
     - Время жизни сессии веб приложения
   * - HTTP_SKIP_SSL_VERIFY
     - нет
     - false
     - Не проверять https сертификаты
   * - LDAP_LOGIN
     - нет
     - false
     - | Вход через ldap
       | Оставить по умолчанию
   * - LDAP_TLS
     - нет
     - no
     - Использовать TLS
   * - LDAP_URL
     - нет
     - “”
     - Адрес сервера LDAP
   * - LDAP_USER_FILTER
     - нет
     - (objectClass=posixAccount)
     - Фильтр поиска пользователей
   * - LDAP_USER_ATTR
     - нет
     - uid
     - Атрибут пользователя
   * - LDAP_GROUP_FILTER
     - нет
     - cn=geoservices
     - Фильтр поиск групп
   * - LDAP_GROUP_ATTR
     - нет
     - memberUid
     - Атрибут группы
   * - LDAP_DEFAULT_GROUP_ID
     - нет
     - 0
     - Группа по умолчания для LDAP пользователей - 0 выключено назначение групп
   * - LDAP_UPDATE_GROUPS
     - нет
     - false
     - Обновлять принадлежность пользователя к группе
   * - OAUTH2_LOGIN
     - нет
     - false
     - Разрешить вход через OAuth2
   * - OAUTH2_CLIENT_ID
     - нет
     - 
     - OAuth2 client ID
   * - OAUTH2_CLIENT_SECRET
     - нет
     - 
     - OAuth2 client secret
   * - OAUTH2_REDIRECT_URI
     - нет
     - 
     - OAuth2 redirect URI
   * - OAUTH2_ENDPOINT
     - нет
     - https://my.nextgis.com
     - Оконечная точка
   * - OAUTH2_SCOPE
     - нет
     - user_info.read
     - Скоуп
   * - OAUTH2_TYPE
     - нет
     - 1
     - Тип авторизации: 1-NextGIS ID, 2 - Keycloak, 3 - custom, 4 - Blitz
   * - OAUTH2_TOKEN_ENDPOINT
     - нет
     - https://my.nextgis.com/oauth2/token
     - Оконечная точка получения токена
   * - OAUTH2_AUTH_ENDPOINT
     - нет
     - https://my.nextgis.com/oauth2/authorize
     - Оконечная точка авторизации
   * - OAUTH2_USERINFO_ENDPOINT
     - нет
     - https://my.nextgis.com/api/v1/user_info
     - Оконечная точка получения информации о пользователе (в случае JWT не требуется)
   * - OAUTH2_INTROSPECTION_ENDPOINT
     - нет
     - https://my.nextgis.com/oauth2/introspect
     - Оконечная точка для интроспекции
   * - OAUTH2_PROFILE_SUBJ_ATTR
     - нет
     - nextgis_guid
     - Поле для получения идентификатора пользователя (subject)
   * - OAUTH2_PROFILE_KEYNAME_ATTR
     - нет
     - username
     - Поле для получения логина пользователя
   * - OAUTH2_PROFILE_FIRSTNAME_ATTR
     - нет
     - first_name
     - Поле для получения имени пользователя
   * - OAUTH2_PROFILE_LASTNAME_ATTR
     - нет
     - last_name
     - Поле для получения фамилии пользователя
   * - OAUTH2_USER_AUTOCREATE
     - нет
     - true
     - При первом входе создает пользователя
   * - OAUTH2_VALIDATE_KEY
     - нет
     - “”
     - Ключ для проверки подписи JWT
   * - OAUTH2_CREATE_GROUPS
     - нет
     - false
     - Создавать группы на основе ролей пользователей
   * - OAUTH2_UPDATE_GROUPS
     - нет
     - false
     - Обновлять состав групп на основе ролей пользователей
   * - OAUTH2_TOKEN_CACHE_TTL
     - нет
     - 3600
     - Время жизни токена по умолчанию в сек, если иное не возвращено сервером
   * - OAUTH2_LOGOUT_ENDPOINT
     - нет
     - “”
     - Оконечная точка для разлогина
   * - OAUTH2_GROUPS_JWT_KEY
     - нет
     - resource_access/{client_id}/roles
     - Путь поиска ролей/групп в JWT токене
   * - LOCAL_LOGIN
     - нет
     - true
     - Разрешить локальные учетные записи
   * - DEFAULT_LANGUAGE
     - нет
     - en
     - Язык по умолчанию
   * - LOG
     - нет
     - false
     - Выводить в stdout сообщения в структурированном формате
   * - LOG_ONLY_EDITS
     - нет
     - false
     - Выводить в stdout сообщения только при модификации данных
   * - CLOUD_MODE
     - нет
     - false
     - Режим запуска в облаке
   * - MAX_AGE
     - нет
     - 43200
     - Длительность хранения тайлов в браузере пользователя - 12 часов
   * - EXT_TMS_SUPPORT
     - нет
     - false
     - Включить поддержку внешних сервисов TMS
   * - 
     - нет
     - https://geoservices.nextgis.com
     - Адрес сервиса интеграции с ПКК
   * - PKK_EXTERNAL_APIKEY
     - нет
     - “”
     - APIKey для взаимодействия с сервисом интеграции с ПКК
   * - PKK_TILES_URL
     - нет
     - “”
     - Адрес локального сервиса интеграции с ПКК
   * - PKK_FEATURES_URL
     - нет
     - “”
     - Адрес локального сервиса интеграции с ПКК
   * - PKK_MIN_ZOOM
     - нет
     - 3
     - Минимальный уровень увеличения тайлов ПКК
   * - PKK_MAX_ZOOM
     - нет
     - 18
     - Максимальный уровень увеличения тайлов ПКК
   * - PKK_REGION
     - нет
     - | MULTIPOLYGON (((-168.4 84,-168.4 50,-179.999999 50,-179.9999999 84,-168.4 84)),
       | ((130 40,130 84,179.999999 84,179.999999 40,130 40)),
       | ((129.999999 84,129.999999 47,82.5 47,82.5 84,129.999999 84)),
       | ((82.4999999 50,50 50,50 84,82.4999999 84,82.4999999 50)),
       | ((20 84,49.999999 84,49.999999 40,20 40,20 84)))
     - Территория для запросов тайлов ПКК
   * - DB_TYPE
     - да
     - sqlite3
     - Тип БД - sqlite3, postgres, mysql
   * - DB_USER
     - нет
     - geoservices
     - Пользователь из под которого будет выполнено подключение к БД
   * - DB_PASSWORD
     - да
     - 
     - Пароль от БД
   * - DB_HOST
     - да
     - localhost
     - Адрес БД
   * - DB_PORT
     - да
     - 5432
     - Порт БД
   * - DB_NAME
     - нет
     - geoservices
     - Имя БД
   * - DB_MAXCONN
     - нет
     - 50
     - Максимальное количество подключений
   * - DB_MAXIDLECONN
     - нет
     - 10
     - Максимальное количество свободных подключений
   * - DB_SSL_MODE
     - нет
     - 
     - | disable - I don't care about security, and I don't want to pay the overhead of encryption.
       | allow - I don't care about security, but I will pay the overhead of encryption if the server insists on it.
       | prefer - I don't care about encryption, but I wish to pay the overhead of encryption if the server supports it.
       | require - I want my data to be encrypted, and I accept the overhead. I trust that the network will make sure I always connect to the server I want.
       | verify-ca - I want my data encrypted, and I accept the overhead. I want to be sure that I connect to a server that I trust.
       | verify-full - I want my data encrypted, and I accept the overhead. I want to be sure that I connect to a server I trust, and that it's the one I specify.
   * - DB_SSL_CERT
     - нет
     - 
     - Путь до файла сертификата
   * - DB_SSL_KEY
     - нет
     - 
     - Путь до файла ключа
   * - DB_SSL_ROOT_CERT
     - нет
     - 
     - Путь до корневого сертификата
   * - REDIS_ENDPOINT
     - да
     - localhost:6379
     - Адрес сервиса Redis
   * - REDIS_MAX_IDLE
     - нет
     - 100
     - Максимальная длительность когда подключение в пуле не будет закрыто
   * - REDIS_MAX_ACTIVE
     - нет
     - 1000
     - Максимальное количество активных подключений в пуле
   * - REDIS_IDLE_TIMEOUT
     - нет
     - 60
     - длительность когда подключение в пуле не будет закрыто
   * - REDIS_CLUSTER
     - нет
     - false
     - Подключаться к кластеру Redis
   * - REDIS_NODES
     - нет
     - "localhost:6379 localhost:7001 localhost:7002 localhost:7003 localhost:7004 localhost:7004"
     - Ноды Redis cluster (применяется только при REDIS_CLUSTER == true)
   * - REDIS_KEY_PREFIX
     - нет
     - “”
     - Префикс ключей Redis которые создаются приложением
   * - REDIS_USER
     - нет
     - geoservices
     - Логин пользователя Redis
   * - REDIS_DATABASE
     - нет
     - 0
     - База Redis
   * - REDIS_SSL
     - нет
     - false
     - Соединение с использованием SSL/TLS
   * - REDIS_INSECURE_SSL
     - нет
     - false
     - Не валидировать SSL/TLS
   * - S3_ACCESS_KEY
     - да
     - Q3AM3UQ867SPQQA43P2F
     - Ключ доступа к S3
   * - S3_SECRET_KEY
     - да
     - zuf+tfteSlswRu7BJ86wekitnifILbZam1KYY3TG
     - Секретный ключ доступа к S3
   * - S3_ENDPOINT
     - да
     - play.min.io
     - Адрес сервера S3
   * - S3_SSL
     - нет
     - true
     - Использовать шифрование
   * - S3_INSECURE_SSL
     - нет
     - false
     - Не проверять сертификаты SSL
   * - S3_DEFAULT_STORAGE_CLASS
     - нет
     - REDUCED_REDUNDANCY
     - Способ хранения REDUCED_REDUNDANCY или STANDARD
   * - S3_BUCKET_NAME
     - нет
     - geoservices
     - Имя бакета
   * - S3_KEY_PREFIX
     - нет
     - “”
     - Префикс ключей S3 которые создаются приложением
   * - S3_NO_OBJECT_TAGGING
     - нет
     - false
     - Не использовать дедупликацию и время истечения если S3 не поддерживает тэги
   * - RASTER_MAX_ZOOM
     - нет
     - 20
     - Максимальный уровень увеличения растровых тайлов
   * - VECTOR_MAX_ZOOM
     - нет
     - 14
     - Максимальный уровень увеличения векторных тайлов
   * - EXPIRE_TILES_MIN_ZOOM
     - нет
     - 7
     - Минимальный уровень увеличения контроля устаревания тайлов
   * - EXPIRE_TILES_MAX_ZOOM
     - нет
     - 16
     - Максимальный уровень увеличения контроля устаревания тайлов
   * - NET_MAX_RETRY_COUNT
     - нет
     - 5
     - Количество попыток перезапроса сетевых запросов
   * - LONG_REQUEST_MIN_TIME
     - нет
     - 0
     - Выводить в журнал длительные запросы - 0 отключено
   * - NGW_URL
     - нет
     - https://sandbox.nextgis.com
     - Адрес ассоциированного NextGIS Web (для создания кэшей на основе веб-карт)
   * - NGW_LOGIN
     - нет
     - administrator
     - Логин для доступа к NextGIS Web - нужен для отрисовки тайлов при сидировании
   * - NGW_APIKEY
     - нет
     - admin
     - Пароль доступа к NextGIS Web - нужен для отрисовки тайлов при сидировании
   * - NGW_FEATURE_LIMIT
     - нет
     - 256
     - Количество записей при постраничном запросе
   * - USERS_MAINTANCE_SCHEDULE
     - нет
     - @every 9m1s
     - Планировщик очистки кэшей пользователей
   * - SERVICE_MAINTANCE_SCHEDULE
     - нет
     - @every 10m4s
     - Планировщик очистки кэшей сервисов
   * - SERVICE_HOUSEKEEPING_SCHEDULE
     - нет
     - @every 25h30m10s
     - Планировщик очистки системы
   * - DATA_STORE
     - нет
     - /data
     - | Путь до данных необходимых для функционирования сервисов
       | Оставить по умолчанию
   * - FILE_STORE
     - нет
     - /work
     - Путь до рабочей папки. Сюда загружаются файлы, выполняются операции, создаются временные файлы.
   * - BM_DB_HOST
     - нет
     - localhost
     - | Хост с БД PostGIS. При старте веб приложения выполняется проверка подключения к БД и наличия всех расширений, необходимых для работы. 
       | Если подключение неуспешно или нет расширений, то раздел базовых карт отключается. 
   * - BM_DB_PORT
     - нет
     - 5432
     - Порт до БД PostGIS
   * - BM_DB_NAME
     - нет
     - basemap
     - Имя БД для импорта дампа OSM
   * - BM_DB_USER
     - нет
     - geoservices
     - Пользователь из под которого будет выполнено подключение к БД базовой картографической подложки
   * - BM_DB_PASSWORD
     - да
     - 
     - Пароль от БД данных базовой картографической подложки
   * - BM_DB_SSL_MODE
     - нет
     - 
     - | disable - I don't care about security, and I don't want to pay the overhead of encryption.
       | allow - I don't care about security, but I will pay the overhead of encryption if the server insists on it.
       | prefer - I don't care about encryption, but I wish to pay the overhead of encryption if the server supports it.
       | require - I want my data to be encrypted, and I accept the overhead. I trust that the network will make sure I always connect to the server I want.
       | verify-ca - I want my data encrypted, and I accept the overhead. I want to be sure that I connect to a server that I trust.
       | verify-full - I want my data encrypted, and I accept the overhead. I want to be sure that I connect to a server I trust, and that it's the one I specify.
   * - BM_DB_SSL_CERT
     - нет
     - 
     - Путь до файла сертификата
   * - BM_DB_SSL_KEY
     - нет
     - 
     - Путь до файла ключа
   * - BM_DB_SSL_ROOT_CERT
     - нет
     - 
     - Путь до корневого сертификата
   * - BM_DB_PARALLEL_SQL
     - нет
     - true
     - Выполнять запросы к БД на получение векторных тайлов параллельно
   * - BM_DIFF_URL
     - нет
     - 
     - Адрес скачивания дельт ОСМ (имеет смысл только если EXT_SOURCES_SUPPORT == true
   * - BM_EXPIRE_TILES_MIN_ZOOM
     - нет
     - 7
     - Минимальный уровень увеличения для учета инвалидации тайлов
   * - BM_EXPIRE_TILES_MAX_ZOOM
     - нет
     - 16
     - Максимальный уровень увеличения для учета инвалидации тайлов
   * - EXT_SOURCES_SUPPORT
     - нет
     - false
     - | Флаг для разрешения/запрета получения файлов из Интернета. Например для инициализации БД через загрузку дампа из интернета или периодического получения diff. 
       | Оставить по умолчанию
   * - EXT_RASTER_RESAMPLING
     - нет
     - bilinear
     - | Сглаживание растра. Поддерживаемые методы:
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
     - нет
     - "204,404"
     - Коды ответов HTTP для белых тайлов
   * - LOCALES
     - нет
     - “ru en”
     - Список языков пользовательского интерфейса
   * - OUTDATED_STAT_TABLE_ROWS
     - нет
     - 2*365*24*time.Hour
     - Удалять записи лога старше 2х лет
   * - ENABLE_SWAGGER
     - нет
     - true
     - Включить веб интерфейс для swagger
   * - SSL_CERT_FILE
     - нет
     - 
     - | Для переопределения пути до сертификата
       | https://stackoverflow.com/a/67622500/2901140
       | 
       | Также можно примонтировать файлы с сертификатами по следующим путям (в зависимости от платформы):
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
     - нет
     - 7 дней
     - TTL тайлов сервисов external






