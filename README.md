# logzio-docker

Forward all your Docker logs to [Logz.io](http://logz.io)

![ELK Apps Docker dashboard](https://github.com/logzio/logzio-docker/blob/master/Docker-DashBoard.png)


## Usage as a Container

The simplest way to forward all your container's log to Logz.io is to
run this repository as a container, with:

```sh
docker run -d --restart=always -v /var/run/docker.sock:/var/run/docker.sock logzio/logzio-docker -t ***TOKEN*** -a env=prod
```

You can pass the `--no-stats` flag if you do not want stats to be
published to Logz.io every second. You __need this flag for Docker
version < 1.5__.

You can pass the `--no-logs` flag if you do not want logs to be published to Logz.io.

You can also pass the `-j` switch if all of the logs generated in the containers are in JSON format

You can pass the `--no-dockerEvents` flag if you do not want events to be
published to Logz.io.

The `-i/--statsinterval <STATSINTERVAL>` downsamples the logs sent to Logz.io. It collects samples and averages them before sending to Logz.io.

The `-a` allows to add more fields to the log - this can be used to tag specific application, environment etc. 

The `-z` allows you to specify the Zone of your account, us if your account is located in the USA or eu if your account is located in the EU. (set to us by default)

The `-e/--endpoint` allows controlling to which logzio endpoint the logs will be sent (will override any zone definition)

You can also filter the containers for which the logs/stats are
forwarded with:

* `--matchByName REGEXP`: forward logs/stats only for the containers whose name matches the given REGEXP.`
* `--matchByImage REGEXP`: forward logs/stats only for the containers whose image matches the given REGEXP.`
* `--skipByName REGEXP`: do not forward logs/stats for the containers whose name matches the given REGEXP.`
* `--skipByImage REGEXP`: do not forward logs/stats for the containers whose image matches the given REGEXP.`

You can also set your Logz.io token and zone with the following environment variables:

* `LOGZIO_TOKEN = <token>`
* `LOGZIO_ZONE = <zone> `

To enrich all log events published to Logz.io with labels that are set on the container use the following options:

* `--addLabels`: toggle to enable 'adding of labels' to each log entry.
* `--labelsKey <field>`: Place all label fields under this key. ie: `--labelsKey labels` would result in labels.mydockerlabel as a fieldname in Logz.io. By default the labels will be placed directly at the root of the object.
* `--labelsMatch REGEXP`: Only add labels if the label name matches the given REGEXP.

### Running container in a restricted environment.
Some environments(such as Google Compute Engine) does not allow to access the docker socket without special privileges. You will get EACCES(`Error: read EACCES`) error if you try to run the container.
To run the container in such environments add --privileged to the `docker run` command.

Example:
```sh
docker run --privileged -d --restart=always -v /var/run/docker.sock:/var/run/docker.sock logzio/logzio-docker -t ***TOKEN*** -a env=prod
docker run --privileged -d --restart=always -v /var/run/docker.sock:/var/run/docker.sock logzio/logzio-docker --token=***TOKEN*** -a env=prod
```

## How it works

This container wraps four [Docker
APIs](https://docs.docker.com/reference/api/docker_remote_api_v1.17/):

* `POST /containers/{id}/attach`, to fetch the logs
* `GET /containers/{id}/stats`, to fetch the stats of the container
* `GET /containers/json`, to detect the containers that are running when
  this module starts
* `GET /events`, to detect new containers that will start after the
  module has started

This module wraps
[docker-loghose](https://github.com/mcollina/docker-loghose) and
[docker-stats](https://github.com/pelger/docker-stats) to fetch the logs
and the stats as a never ending stream of data.
