# Using Buildarr

Apart from the configuration, most of the interactions with Buildarr are done via the command line.

There are three major operating modes for Buildarr:

* `buildarr run` - Manually perform an update run on one or more instances and exit
* `buildarr daemon` - Run Buildarr forever: perform an initial update run, and then
  schedule periodic updates
* `buildarr <plugin-name> <command...>` - Ad-hoc commands defined by any loaded plugins

!!! note
    Every time Buildarr performs an update run, a metadata file called `secrets.json` is generated to store secrets information for every configured instance in the current directory. Ensure that you are running Buildarr in a folder that is not world-viewable, to avoid exposing secrets.

    If you are using the Docker image, ensure that the folder mounted as `/config` into the container has appropriately secure permissions.

The verbosity of Buildarr logging output can be adjusted using the `--log-level` option.
This option can also be set using the `$BUILDARR_LOG_LEVEL` environment variable.

For more interactive documentation, you can pass `--help` to any individual command.

```text
$ buildarr --help
Usage: buildarr [OPTIONS] COMMAND [ARGS]...

  Construct and configure Arr PVR stacks.

  Can be run as a daemon or as an ad-hoc command.

  Supports external plugins to allow for adding support for multiple types of
  instances.

Options:
  -l, --log-level [CRITICAL|ERROR|WARNING|INFO|DEBUG|NOTSET]
                                  Buildarr logging system log level. Can also
                                  be set using the `$BUILDARR_LOG_LEVEL'
                                  environment variable.  [default: INFO]
  --help                          Show this message and exit.

Commands:
  daemon  Run as a daemon and periodically update defined instances.
  run     Configure instances defined in the Buildarr config file, and exit.
  sonarr  Sonarr instance ad-hoc commands.
```

## Manual runs

Buildarr is capable of executing individual update runs, optionally passing
an arbitrary configuration file to parse.
This is useful to test configuration files before formally deploying Buildarr.

```bash
$ buildarr run [/path/to/config.yml]
```

If using Docker, the command would look something like this:
```bash
$ docker run --rm -v /path/to/config:/config -e PUID=<PUID> -e PGID=<PGID> callum027/buildarr:latest run [/config/buildarr.yml]
```

Executing `buildarr run` will result in something resembling the following output.

```text
$ buildarr run
2023-02-11 14:50:49,356 buildarr:53612 buildarr.main [INFO] Buildarr version 0.1.0 (log level: INFO)
2023-02-11 14:50:49,357 buildarr:53612 buildarr.main [INFO] Loading configuration file '/config/buildarr.yml'
2023-02-11 14:50:49,374 buildarr:53612 buildarr.main [INFO] Finished loading configuration file
2023-02-11 14:50:49,378 buildarr:53612 buildarr.main [INFO] Plugins loaded: sonarr
2023-02-11 14:50:49,380 buildarr:53612 buildarr.main [INFO] Using plugins: sonarr
2023-02-11 14:50:49,381 buildarr:53612 buildarr.main [INFO] Loading secrets file from 'secrets.json'
2023-02-11 14:50:49,383 buildarr:53612 buildarr.main [INFO] Finished loading secrets file
2023-02-11 14:50:49,384 buildarr:53612 buildarr.plugins.sonarr default [INFO] Checking and fetching secrets
2023-02-11 14:50:49,384 buildarr:53612 buildarr.plugins.sonarr default [INFO] Finished checking and fetching secrets
2023-02-11 14:50:49,385 buildarr:53612 buildarr.main [INFO] Saving updated secrets file to 'secrets.json'
2023-02-11 14:50:49,386 buildarr:53612 buildarr.main [INFO] Finished saving updated secrets file
2023-02-11 14:50:49,388 buildarr:53612 buildarr.plugins.sonarr default [INFO] Getting remote configuration
2023-02-11 14:50:49,886 buildarr:53612 buildarr.plugins.sonarr default [INFO] Finished getting remote configuration
2023-02-11 14:50:49,931 buildarr:53612 buildarr.plugins.sonarr default [INFO] Updating remote configuration
2023-02-11 14:50:50,321 buildarr:53612 buildarr.plugins.sonarr default [INFO] sonarr.settings.general.host.instance_name: 'Sonarr' -> 'Sonarr (Example)'
2023-02-11 14:50:50,417 buildarr:53612 buildarr.plugins.sonarr default [INFO] Remote configuration successfully updated
2023-02-11 14:50:50,417 buildarr:53612 buildarr.plugins.sonarr default [INFO] Finished updating remote configuration
```

Of note in particular is the following line:

```text
2023-02-11 14:50:50,321 buildarr:53612 buildarr.plugins.sonarr default [INFO] sonarr.settings.general.host.instance_name: 'Sonarr' -> 'Sonarr (Example)'
```

When Buildarr detects that the remote configuration differents from the locally defined configuration, the remote configuration will be updated. In this case, Buildarr detected that on the `default` instance configured in the Sonarr plugin, the configured GUI instance name is different from the locally defined value, so it updated the Sonarr instance to reflect the change.

If the run fails for one reason or another, an error message (or exception traceback, depending on the error) will be logged and Buildarr with exit with a non-zero status.

## As a service (daemon mode)

This is the mode in which that Buildarr is intended to run in most cases.

Buildarr will run as a daemon in the foreground. First, an initial update run is performed, similar to `buildarr run`. If the initial run is successful, Buildarr will then schedule updates to run at specific times according to the configuration,
logging when the next scheduled run will occur.

This is intended to be used to keep a *Arr stack continually up to date, particularly if it has TRaSH-Guides metadata configured. Every scheduled run, the TRaSH-Guides metadata is updated, ensuring that any instances using it will always be using the most up to date profiles.

There are several options for changing how Buildarr daemon runs in the [Buildarr configuration](configuration.md#buildarr-settings).

```text
2023-02-11 13:43:48,890 buildarr:4308 buildarr.main [INFO] Buildarr version 0.1.0 (log level: INFO)
2023-02-11 13:43:48,891 buildarr:4308 buildarr.main [INFO] Loading configuration file '/config/buildarr.yml'
2023-02-11 13:43:48,898 buildarr:4308 buildarr.main [INFO] Finished loading configuration file
2023-02-11 13:43:48,900 buildarr:4308 buildarr.main [INFO] Daemon configuration:
2023-02-11 13:43:48,900 buildarr:4308 buildarr.main [INFO]  - Watch configuration files: Yes
2023-02-11 13:43:48,900 buildarr:4308 buildarr.main [INFO]  - Configuration files to watch:
2023-02-11 13:43:48,900 buildarr:4308 buildarr.main [INFO]    - /config/buildarr.yml
2023-02-11 13:43:48,901 buildarr:4308 buildarr.main [INFO]  - Update at:
2023-02-11 13:43:48,901 buildarr:4308 buildarr.main [INFO]    - Monday 03:00
2023-02-11 13:43:48,901 buildarr:4308 buildarr.main [INFO]    - Tuesday 03:00
2023-02-11 13:43:48,901 buildarr:4308 buildarr.main [INFO]    - Wednesday 03:00
2023-02-11 13:43:48,901 buildarr:4308 buildarr.main [INFO]    - Thursday 03:00
2023-02-11 13:43:48,901 buildarr:4308 buildarr.main [INFO]    - Friday 03:00
2023-02-11 13:43:48,901 buildarr:4308 buildarr.main [INFO]    - Saturday 03:00
2023-02-11 13:43:48,902 buildarr:4308 buildarr.main [INFO]    - Sunday 03:00
2023-02-11 13:43:48,902 buildarr:4308 buildarr.main [INFO] Applying initial configuration
2023-02-11 13:43:48,904 buildarr:4308 buildarr.main [INFO] Plugins loaded: sonarr
2023-02-11 13:43:48,906 buildarr:4308 buildarr.main [INFO] Using plugins: sonarr
2023-02-11 13:43:48,907 buildarr:4308 buildarr.main [INFO] Loading secrets file from 'secrets.json'
2023-02-11 13:43:48,909 buildarr:4308 buildarr.main [INFO] Finished loading secrets file
2023-02-11 13:43:48,910 buildarr:4308 buildarr.plugins.sonarr default [INFO] Checking and fetching secrets
2023-02-11 13:43:48,910 buildarr:4308 buildarr.plugins.sonarr default [INFO] Finished checking and fetching secrets
2023-02-11 13:43:48,910 buildarr:4308 buildarr.main [INFO] Saving updated secrets file to 'secrets.json'
2023-02-11 13:43:48,911 buildarr:4308 buildarr.main [INFO] Finished saving updated secrets file
2023-02-11 13:43:48,914 buildarr:4308 buildarr.plugins.sonarr default [INFO] Getting remote configuration
2023-02-11 13:43:49,870 buildarr:4308 buildarr.plugins.sonarr default [INFO] Finished getting remote configuration
2023-02-11 13:43:49,914 buildarr:4308 buildarr.plugins.sonarr default [INFO] Updating remote configuration
2023-02-11 13:43:49,954 buildarr:4308 buildarr.plugins.sonarr default [INFO] sonarr.settings.general.host.instance_name: 'Sonarr' -> 'Sonarr (Buildarr Example)'
2023-02-11 13:43:50,177 buildarr:4308 buildarr.plugins.sonarr default [INFO] Remote configuration successfully updated
2023-02-11 13:43:50,177 buildarr:4308 buildarr.plugins.sonarr default [INFO] Finished updating remote configuration
2023-02-11 13:43:50,178 buildarr:4308 buildarr.main [INFO] Finished applying initial configuration
2023-02-11 13:43:50,179 buildarr:4308 buildarr.main [INFO] Scheduling update jobs
2023-02-11 13:43:50,180 buildarr:4308 buildarr.main [INFO] Finished scheduling update jobs
2023-02-11 13:43:50,180 buildarr:4308 buildarr.main [INFO] The next run will be at 2023-02-12 03:00
2023-02-11 13:43:50,181 buildarr:4308 buildarr.main [INFO] Setting up config file monitoring
2023-02-11 13:43:50,183 buildarr:4308 buildarr.main [INFO] Finished setting up config file monitoring
2023-02-11 13:43:50,183 buildarr:4308 buildarr.main [INFO] Setting up signal handlers
2023-02-11 13:43:50,183 buildarr:4308 buildarr.main [INFO] Finished setting up signal handlers
2023-02-11 13:43:50,184 buildarr:4308 buildarr.main [INFO] Buildarr ready.
```

Buildarr daemon supports the following signal types:

* `SIGTERM`/`SIGINT` - Gracefully shutdown the Buildarr daemon.
* `SIGHUP` - Reload the Buildarr configuration file and perform an update run
  (the same action taken as when the `watch_config` option is enabled and Buildarr detects configuration changes).
  Not supported on Windows.

## Plugin-specific commands

Plugins can implement their own ad-hoc commands. These are mainly used for things such as dumping configuration from running instances.

For more information, refer to the user guide for the respective plugin.