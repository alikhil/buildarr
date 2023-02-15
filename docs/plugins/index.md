# Plugins

Buildarr supports external plugins to allow support for additional *Arr stack applications to be added.

As Buildarr is still under heavy development, however, to allow for easier iteration of both the plugin and the API, the currently supported plugins are vendored into the core Buildarr package, and can be used without installing any additional packages.

Successfully installed plugins will be loaded when Buildarr is run. Configured plugins will be listed under `Using plugins`.

```text
$ buildarr run
2023-02-11 14:50:49,356 buildarr:53612 buildarr.main [INFO] Buildarr version 0.1.0 (log level: INFO)
2023-02-11 14:50:49,357 buildarr:53612 buildarr.main [INFO] Loading configuration file '/config/buildarr.yml'
2023-02-11 14:50:49,374 buildarr:53612 buildarr.main [INFO] Finished loading configuration file
2023-02-11 14:50:49,378 buildarr:53612 buildarr.main [INFO] Plugins loaded: sonarr
2023-02-11 14:50:49,380 buildarr:53612 buildarr.main [INFO] Using plugins: sonarr
...
```

## Supported plugins

At the time of this release the following plugins are available:

* [buildarr-sonarr](sonarr/index.md) - [Sonarr](https://sonarr.tv) PVR for TV shows (V3 only)