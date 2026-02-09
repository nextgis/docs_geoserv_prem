.. _nggs_prem_admin_upgrade:

Обновление
==========

.. note:: 

	Данный процесс описывает обновление с версии 2.16.1. Если у вас версия ниже или требуется обновление в закрытой сети - свяжитесь с нами через техподдержку support@nextgis.ru


**Шаг 1:** Ознакомьтесь с действиями и требованиями перечисленными ниже для конкретной версии:

.. contents::
   :local:
   :depth: 1

**Шаг 2:** Остановите все сервисы:


.. code-block:: shell

  $ docker compose stop

**Шаг 3:** Обновите значение ``IMAGE_VERSION`` в файле ``.env``.

**Шаг 4:** Запустите все сервисы:

.. code-block:: shell

  $ docker compose up -d

Обновление до версии 2.20.x с версии 2.19.x
-------------------------------------------

В конфигурационный файл ``docker-compose.yaml`` добавьте сервис ``node-renderer``, в сервис ``app`` добавьте переменную окружения ``NODE_RENDERER_SECRET``:

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
