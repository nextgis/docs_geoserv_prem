Changelog
================

**2.26 released on 18.08.2026**

- Add languages: French, Spanish.
- MapLibre for Node.js upgraded to 6.4.1.
- Add OpenType fonts support.
- Improved initial map extent on service preview.
- Minor interface improvements.

**2.25 released on 29.06.2026**

- Support tileset create from MBTiles.
- Service preview without browser cache.
- Release info on "About" page.

**2.24 released on 26.05.2026**

- NextGIS Web service default extent set from constraining web map extent instead of initial.
- External service tile size setting has been removed. Only 256x256 px tiles are supported.
- Minor stability and error handling improvements.

**2.23 released on 27.04.2026**

- New service type - Tileset.
- Create API key when registering OAuth user.
- Select API key for URL templates.
- Fixed default maximum zoom in API key create dialog.
- Fix error when logging in using OAuth.
- Fix user profile edit error.

**2.22 released on 31.03.2026**

- WebP format support for external services.
- Fix seeding of external services with CRS EPSG:3857.
- Fix tiles deletion from Redis during seeding.
- Simplify service pages paths.
- Clear service permissions for deleted users and groups.
- Improved internal database integrity.

**2.21 released on 17.12.2025**

- Fix missing space symbol in font glyphs.
- Replace unsupported text font with Open Sans.
- Support HTTP(S) proxy.
- Healthcheck NGW connection.

**2.20 released on 1.11.2025**

- Add storage estimation feature.
- Make name_{lang} MVT attribute contain a certain language.
- Show zoom on preview page.
- Add status and OSM PBF hints.
- Remove legacy renderer.
- Fixed getting OAuth groups from token.
- Dropped LDAP login support.

**2.19.0 released on 21.07.2025**


- Show seeding services at overview page.
- Delete tile cache on service delete.
- Fix tile rendering of basemap and NGW services on certain scales.
- Fix raster cache invalidation.
- Fix label layers of default styles.
- Fix creation of NGW services from web map with empty extent.
- Fix delete basemap data button.

**2.18.0 released on 14.05.2025**


- Add picker with 3 default styles for basemap service.
- Fix authorization and permissions check.
- Reduce MVT languages to "name", "name:en" and "name:ru".
- Improve MVT generation performance.
- Allow CORS for maplibre.org by default.
- Improve seeding stability.
- Fix metatile locking, better cache errors handling.
- Experimental NGW layers support for basemap service.

**2.17.0 released on 10.03.2025**

- Add "landcover" basemap layer.
- Add OpenSans font.
- PBF glyphs of all system fonts are available via API.
- Basemap auto update disabled by default.
- Fix edge basemap tiles rendering.
- Fix NGW service scales vision.
- Fix API-key creating with expiration date.
- A lot of fixes for seeding.
- Own Redis container based on Ubuntu 24.04.
- Experimental Maplibre GL renderer support.
