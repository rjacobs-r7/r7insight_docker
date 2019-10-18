# r7insight_docker

Forward all your logs to [Rapid7 InsightOps](https://www.rapid7.com/products/insightops/), like a breeze.

![InsightOps dashboard](https://raw.githubusercontent.com/rapid7/r7insight_docker/master/dashboard.png)

You can download the community pack created by InsightOps at [Docker pack](https://insightops.help.rapid7.com/docs/docker). The community pack comes with pre-defined out-of-the-box alerts and widgets to get you started.

## Usage as a Container

The simplest way to forward all your container's log to Rapid7 InsightOps is to
run this repository as a container, with:

```sh
docker run -v /var/run/docker.sock:/var/run/docker.sock rapid7/r7insight_docker -t <TOKEN> -r <REGION> -j -a host=`uname -n`
```

You can also use different tokens for logging, stats and events:
```sh
docker run -v /var/run/docker.sock:/var/run/docker.sock rapid7/r7insight_docker -l <LOGSTOKEN> -k <STATSTOKEN> -e <EVENTSTOKEN> -r <REGION> -j -a host=`uname -n`
```

You can pass the `--no-stats` flag if you do not want stats to be
published to Rapid7 InsightOps every second. You __need this flag for Docker
version < 1.5__.

You can pass the `--no-logs` flag if you do not want logs to be published to Rapid7 InsightOps.

You can pass the `--no-dockerEvents` flag if you do not want events to be
published to Rapid7 InsightOps.

The `-i/--statsinterval <STATSINTERVAL>` downsamples the logs sent to Rapid7 InsightOps. It collects samples and averages them before sending to Rapid7 InsightOps.

If you don't use `-a` a default ``host=`uname -n` `` value will be added.

You can also filter the containers for which the logs/stats are
forwarded with:

* `--matchByName REGEXP`: forward logs/stats only for the containers whose name matches the given REGEXP.
* `--matchByImage REGEXP`: forward logs/stats only for the containers whose image matches the given REGEXP.
* `--skipByName REGEXP`: do not forward logs/stats for the containers whose name matches the given REGEXP.
* `--skipByImage REGEXP`: do not forward logs/stats for the containers whose image matches the given REGEXP.

### Running container in a restricted environment.
Some environments(such as Google Compute Engine) does not allow to access the Docker socket without special privileges. You will get EACCES(`Error: read EACCES`) error if you try to run the container.
To run the container in such environments add --privileged to the `docker run` command.

Example:
```sh
docker run --privileged -v /var/run/docker.sock:/var/run/docker.sock rapid7/r7insight_docker -t <TOKEN> -r <REGION> -j -a host=`uname -n`
```

## Usage as a CLI

1. `npm install r7insight_docker -g`
2. `r7insight_docker -t TOKEN -r REGION -a host=\`uname -n\``


You have to specify TOKEN by passing `-t TOKEN`

You have to specify REGION by passing `-r REGION`. Region is mandatory.

You can also pass the `-j` switch if you log in JSON format, like
[bunyan](http://npm.im/bunyan).

You can pass the `--no-stats` flag if you do not want stats to be
published to Rapid7 InsightOps every second.

You can pass the `--no-logs` flag if you do not want logs to be published to Rapid7 InsightOps.

You can pass the `--no-dockerEvents` flag if you do not want events to be
published to Rapid7 InsightOps.

The `-a/--add` flag allows to add fixed values to the data being
published. This follows the format 'name=value'.

The `-i/--statsinterval` downsamples the logs sent to Rapid7 InsightOps. It collects samples and averages them before sending to Rapid7 InsightOps.

You can also filter the containers for which the logs/stats are
forwarded with:

* `--matchByName REGEXP`: forward logs/stats only for the containers whose name matches the given REGEXP.
* `--matchByImage REGEXP`: forward logs/stats only for the containers whose image matches the given REGEXP.
* `--skipByName REGEXP`: do not forward logs/stats for the containers whose name matches the given REGEXP.
* `--skipByImage REGEXP`: do not forward logs/stats for the containers whose image matches the given REGEXP.

## Embedded usage

Install it with: `npm install r7insight_docker --save`

Then, in your JS file:

```
var insightops = require('r7insight_docker')({
  json: false, // or true to parse lines as JSON
  secure: true, // or false to connect over plain TCP
  token: process.env.TOKEN, // insightops TOKEN
  newline: true, // Split on newline delimited entries
  stats: true, // disable stats if false
  add: null, // an object whose properties will be added

  // the following options limit the containers being matched
  // so we can avoid catching logs for unwanted containers
  matchByName: /hello/, // optional
  matchByImage: /matteocollina/, //optional
  skipByName: /.*pasteur.*/, //optional
  skipByImage: /.*dockerfile.*/ //optional
})

// insightops is the source stream with all the
// log lines

setTimeout(function() {
  insightops.destroy()
}, 5000)
```

## Building a Docker repo from this repository

### Using the plain Docker file
First clone this repository, then:

```bash
docker build -t r7insight_docker .
docker run -v /var/run/docker.sock:/var/run/docker.sock r7insight_docker -t <TOKEN> -r <REGION> -j -a host=`uname -n`
```
### Using Make - the official nodejs onbuild image
```bash
export BUILD_TYPE=node-onbuild
make build
make test
make tag
```

### Using Make - the alpine linux build (~42Mb)
```bash
export BUILD_TYPE=alpine-node
make build
make test
make tag
```

### Pushing to your own Docker repository
After you've build, tested, tagged it locally
```bash
export DOCKER_REGISTRY_PREFIX=<your-dockerhub-user>/<your-image-name>
make push
```

### Publishing your own node package
- Update **package.json** depending on your requirements
- `make publish`

## How it works

This module wraps four [Docker
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

All the originating requests are wrapped in a
[never-ending-stream](https://github.com/mcollina/never-ending-stream).

## License

MIT


## Contact Support

Please email our support team at support@rapid7.com if you need any help.
