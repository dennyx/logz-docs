---
title: Ship Docker logs
logo:
  logofile: docker.svg
  orientation: horizontal
shipping-summary:
  data-source: Docker container
open-source:
  - title: docker-collector-logs
    github-repo: docker-collector-logs
  - title: Logz.io Docker Logging Plugin
    github-repo: docker-logging-plugin
logzio-app-url: https://app.logz.io/#/dashboard/data-sources/Docker-Logging
contributors:
  - imnotashrimp
  - amosd92
---

<div class="branching-container">

{: .branching-tabs }
  * [docker-collector-logs <span class="sm ital">(recommended)</span>](#docker-collector-logs-config)
  * [logzio-logging-plugin](#logzio-logging-plugin-docker-logging-driver-config)

<div id="docker-collector-logs-config">

## docker-collector-logs setup

{: .tasklist .firstline-headline }
1. Pull the Docker image

    Download the logzio/docker-collector-logs image:

    ```shell
    docker pull logzio/docker-collector-logs
    ```

2. Run the Docker image

    For a complete list of options, see the parameters below the code block.👇

    ```shell
    docker run --name docker-collector-logs \
    --env LOGZIO_TOKEN="<ACCOUNT-TOKEN>" \
    --env LOGZIO_URL="<LISTENER-URL>:5015" \
    -v /var/run/docker.sock:/var/run/docker.sock:ro \
    -v /var/lib/docker/containers:/var/lib/docker/containers \
    logzio/docker-collector-logs
    ```

    Parameters
    {: .inline-header }

    LOGZIO_TOKEN <span class="required-param"></span>
    : Your Logz.io account token.
      {% include log-shipping/replace-vars.html token=true %}
      <!-- logzio-inject:account-token -->

    LOGZIO_URL <span class="required-param"></span>
    : Logz.io listener URL to ship the logs to.
      {% include log-shipping/replace-vars.html listener=true %}

    matchContainerName
    : Comma-separated list of containers you want to collect the logs from.
      If a container's name partially matches a name on the list, that container's logs are shipped.
      Otherwise, its logs are ignored. \\
      **Note**: Can't be used with skipContainerName

    skipContainerName
    : Comma-separated list of containers you want to ignore.
      If a container's name partially matches a name on the list, that container's logs are ignored.
      Otherwise, its logs are shipped. \\
      **Note**: Can't be used with matchContainerName

    <div class="info-box note">
      By default, logs from docker-collector-logs and docker-collector-metrics containers are ignored.
    </div>

3. Check Logz.io for your logs

    Spin up your Docker containers if you haven't done so already.
    Give your logs a few minutes to get from your system to ours, and then open [Kibana](https://app.logz.io/#/dashboard/kibana).

    If you still don't see your logs, see [log shipping troubleshooting]({{site.baseurl}}/user-guide/log-shipping/log-shipping-troubleshooting.html).

</div>

<div id="logzio-logging-plugin-docker-logging-driver-config">

## logzio-logging-plugin setup

**You'll need**:
Docker Engine 17.05 or later,
Docker Community Edition (Docker CE) 18.03 or later

{: .tasklist .firstline-headline }
1. Install the plugin from the Docker store

    ```shell
    docker plugin install store/logzio/logzio-logging-plugin:1.0.0 \
    --alias logzio/logzio-logging-plugin
    ```

    Check to see if logzio-logging-plugin is enabled.

    ```shell
    docker plugin ls --filter enabled=true
    ```

    If logzio-logging-plugin isn't on the list, enable it now.

    ```shell
    docker plugin enable logzio/logzio-logging-plugin
    ```

2. Configure global settings with daemon.json

    You can configure all containers with the same options using daemon.json.

    For a complete list of options, see the configuration parameters below the code sample.👇

    Code sample
    {: .inline-header }

    ```json
    {
      "log-driver": "logzio/logzio-logging-plugin",
      "log-opts": {
        "logzio-url": "<LISTENER-URL>",
        "logzio-token": "<ACCOUNT-TOKEN>",
        "logzio-dir-path": "/path/to/logs/"
      }
    }
    ```

    Parameters
    {: .inline-header}

    logzio-token <span class="required-param"></span>
    : Your Logz.io account token.
      {% include log-shipping/replace-vars.html token=true %}
      <!-- logzio-inject:account-token -->

    logzio-url	<span class="required-param"></span>
    : Listener URL and port. \\
      {% include log-shipping/replace-vars.html listener=true %}

    logzio-dir-path	<span class="required-param"></span>
    : Path of the logs to be sent to Logz.io.

    logzio-source
    : Event source.

    logzio-format <span class="default-param">`text`</span>
    : Log message format, either `json` or `text`.

    logzio-tag {% raw %} <span class="default-param">`{{.ID}}` (Container ID)</span> {% endraw %}
    : Log tag.
      For more information, see [Log tags for logging driver](https://docs.docker.com/v17.09/engine/admin/logging/log_tags/) from Docker.

    labels
    : Comma-separated list of labels to include in the log message.

    env
    :	Comma-separated list of environment variables to include in the log message.

    env-regex
    : A regular expression to match logging-related environment variables.
      Used for advanced log tag options.
      If there is collision between the `label` and `env` keys, `env` wins.
      Both options add additional fields to the attributes of a logging message.

    logzio-attributes
    : JSON-formatted metadata to include in the log message.

3. _(Optional)_ Set environment variables

    Some logzio-logging-plugin options are controlled using environment variables.
    Each of these variables has a default value, so you can skip this step if you're comfortable with the defaults.

    Environment variables
    {: .inline-header }

    LOGZIO_DRIVER_LOGS_DRAIN_TIMEOUT <span class="default-param">`5s`</span>
    : Time to wait between sending attempts.

    LOGZIO_DRIVER_DISK_THRESHOLD <span class="default-param">`70`</span>
    : Threshold, as % of disk usage, over which plugin will start dropping logs.

    LOGZIO_DRIVER_CHANNEL_SIZE <span class="default-param">`10000`</span>
    : The number of pending messages that can be in the channel before adding them to the disk queue.

    LOGZIO_MAX_MSG_BUFFER_SIZE <span class="default-param">`1048576` (1 MB)</span>
    : Appends logs that are segmented by Docker with 16kb limit.
      Specifies the biggest message, in bytes, that the system can reassemble.
      `1048576` (1 MB) maximum.

    LOGZIO_MAX_PARTIAL_BUFFER_DURATION <span class="default-param">`500ms`</span>
    : How long the buffer keeps the partial logs before flushing them.

4. _(Optional)_ Override global settings for an individual container

    You can configure the plugin separately for each container when using the `docker run` command.

    Code sample
    {: .inline-header }

    {% raw %}
    ```shell
    docker run --log-driver=logzio/logzio-logging-plugin \
    --log-opt logzio-token=<ACCOUNT-TOKEN> \
    --log-opt logzio-url=https://<LISTENER-URL>:8071 \
    --log-opt logzio-dir-path=./docker_logs \
    --log-opt logzio-tag="{{.Name}}/{{.FullID}}" \
    --log-opt labels=region \
    --log-opt env=DEV \
    --env "DEV=true" \
    --label region=us-east-1 \
    <DOCKER-IMAGE-NAME>
    ```
    {% endraw %}

    {% include log-shipping/replace-vars.html token=true listener=true %}

    For a complete list of options, see the configuration parameters in step 2.☝️

3. Check Logz.io for your logs

    Spin up your Docker containers if you haven't done so already. Give your logs a few minutes to get from your system to ours, and then open [Kibana](https://app.logz.io/#/dashboard/kibana).

    If you still don't see your logs, see [log shipping troubleshooting]({{site.baseurl}}/user-guide/log-shipping/log-shipping-troubleshooting.html).

</div>

</div>