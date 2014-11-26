RunbookWraps
============

**RunbookWraps** is a set of simple python scripts that can be used to interact with [Runbook.io](https://runbook.io). This repo contains two scripts, `monitor.py` and `reactor.py`.

## Monitor.py

`monitor.py` is used to run commands or scripts from the shell. If the command exits with an exit code of `0` the `monitor.py` process will make a webhook call to Runbook setting the monitor as **Healthy**. Any other exit code will result in a webhook call setting the monitor as **Failed**. 

### Calling Monitor.py

To run `monitor.py` you can run it from the command line passing a `YAML` configuration file.

    $ monitor.py -c /path/to/config.yml

When defining a configuration file the first key is `monitors` which contains a dictionary of monitors.

    monitors:

The second key is a human friendly monitor name which also contains a YAML dictionary.

    monitors:
      Monitors Human Name:

The individual monitors dictionary requires at least 3 keys.

* `url` 

This is a unique URL that you can get from Runbook.io when creating a webhook monitor

* `check_key` 

This is a unique token that you can get from the same location as the unique URL

* `cmd` 

This is the command that the monitor.py wrapper script should execute.

* `args` 

Optional: This can be used to pass arguments to the specified command. You are free to not use this option and simply specify all arguments via the `cmd` key.

When put all together the configuration looks like the following.

    monitors:
      Check nginx:
        url: unique_url
        check_key: unique_check_key
        cmd: /bin/ps -elf | /bin/grep -q nginx

Given the above configuration when called the `monitor.py` will execute `/bin/ps -elf | grep -q nginx` and based on the success or failure of the command send an appropriate webhook to Runbook via the `url` specified, using the `check_key` token for authorization. 

## Reactor.py

Where `monitor.py` is used to run a check script/command and notify Runbook; `reactor.py` is used to check the status of a monitor from Runbook and execute appropriate "reactions". Using the above monitor as an example a reaction would be to execute `service nginx restart`.

### Calling Reactor.py

To run `reactor.py` you can also execute it from the command line passing a `YAML` configuration file.

    $ reactor.py -c /path/to/config.yml

The configuration for `reactor.py` follows the same syntax as the `monitor.py` configuration file. In fact you can utilize the same `YAML` config file for both `monitor.py` and `reactor.py` (you can see this in `config/nginx.yml.sample`). To start a `reactor.py` configuration, you must first have a monitor defined. However; this monitor configuration only requires the `url` and `check_key` keys.

    monitors:
      Check nginx:
        url: unique_url
        check_key: unique_check_key

After specifying a monitor, reactions for that monitor can be specified by adding a `reactions` key.

    monitors:
      Check nginx:
        url: unique_url
        check_key: unique_check_key
        reactions:

The `reactions` key is a dictionary that can contain multiple reactions. This is similar to the `monitors` dictionary.

    monitors:
      Check nginx:
        url: unique_url
        check_key: unique_check_key
        reactions:
          reaction number 1:

The individual reaction requires 3 keys:

* `trigger` 

This is used to specify the number of failed or healthy monitors must be performed before the reaction is executed. This uses the `failcount` key in the JSON reply from Runbook.io's webhook.

* `callon` 

This key can specify either `healthy` or `failed` and is used to specify when the reaction should be performed.

* `cmd` 

This key is similar to the `monitor.py`'s key in that it is the comand to execute as the reaction

* `args` 

Optional: This key is available for passing arguments to the commands specified in the `cmd` key. Again, you can simply specify the full command in the `cmd` key, if desired.

Put together a full configuration for nginx would look like the following.

    monitors:
      Check nginx:
        url: unique_url
        check_key: unique_check_key
        cmd: /bin/ps -elf | /bin/grep -q nginx
        reactions:
          restart nginx:
            trigger: 5
            callon: failed
            cmd: /etc/init.d/nginx restart

Checkout the `config/nginx.yml.sample` sample configuration file for more advanced configurations.
