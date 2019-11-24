![](https://img.shields.io/npm/v/loadflux?color=green&style=flat-square)
![](https://img.shields.io/github/package-json/v/chaoyangnz/loadflux?color=blue&style=flat-square)
![](https://img.shields.io/github/license/chaoyangnz/loadflux?color=orange&style=flat-square)
![](https://github.com/chaoyangnz/loadflux/workflows/build/badge.svg)

# LOADFLUX

## What is it?

Loadflux is a simplistic tool for load testing.
You just need to focus more on how to design your scenario, and the scripting is just taking several minutes.
The infrastructure requirement is pretty low, you can simulate hundreds of virtual users to send requests to your application in your local machine.
It support different strategy to ramp the load, by default in this library, we provide:

- sustain a constant load, for example, you want to keep 200 users in your web application to do different actions. 
When one user exits, another user will be arriving to sustain the load
- ramp up the load until you know the extrame capability of your infrastructure and user experience, for example, 
you can ramp up 50 new users per second, then sometime your application will crash and have no response.

### Scenario and Action logs 

![](screenshot.gif)

### Metrics

We integrate [InfluxDB](https://www.influxdata.com/) and send real-time measurements to the cloud, so we can do analytics later. 
Some basic metrics like requests/success/failure count, mean response time, RPS, active virtual users etc can be monitored in its platform.
We choose InfluxDB as it has an intuitive UI and easy to integrate. But we will add more integrations to other monitoring platforms in the future.

![](https://i.imgur.com/yEh9lpc.gif)

## Concepts

- `Virtual User`: a.k.a `VU`, represent a user which is supposed to execute your scenarios.
- `Scenario`: a scenarios is a list of actions a normal user can take in your applications.
- `Action`: an operation in your application, typically including requests (APIs, static assets etc), think time, logging.
- `Runner`: a runner is the pilot who controls everything of your testing. When a new virtual user arrives, 
the runner will first checkin at candidates registry, and choose a scenario probabilistically as per the scenario weight. The last step is to assign 
the virtual user to that scenario and run it until the scenario is completed. You can decide how new virtual users arrive in a constant number or a ramp-up rate.

## Usage

### Write you scenarios

You can refer to the `samples` folder, there are some examples. If you read them, actually it is easy to write and you just need to take advantage of 
your NodeJS/Javascript knowledge.

### Run you code

Just like any other NodeJS application, run it using `node` command line. `Loadflux` is just a _library_, **NOT** a framework or CLI.

`Loadflux` has some environment variables worth noticing. Either create a `.env` file or specify them when you run your application 
(e.g. in Dockerfile, docker-composer file, k8s deployment.yaml, CloudFormation, etc).

Environment Variables:
- General
    - `LOADFLUX_VU_POOL_SIZE`: define how many virtual users (`VU`) how many users can be available at maximum. These users are volunteers, which does
    not necessarily mean all of them will participate the execution of your scenarios. You decide how many virtual users are planned and checked in/out by your testing strategy.
    - `LOADFLUX_DURATION`: how long you plan to run your load testing.
    - `LOADFLUX_BASE_URL`: the base url of your application. If you don't set, you have to use absolute URL in your request action.
    - `LOADFLUX_TEST_ID`: the test id (16 chars at most) used for different iteration of your testing. Default: current timestamp since unix epoch.
- Metrics
    - `LOADFLUX_INFLUXDB_API`: `InfluxDB` v2 API endpoint, typically like `https://us-west-2-1.aws.cloud2.influxdata.com/api/v2`. Refer to `InfluxDB` documentation.
    - `LOADFLUX_INFLUXDB_TOKEN`: `InfluxDB` token, which can be found in InfluxDB cloud admin console.
    - `LOADFLUX_INFLUXDB_ORG`: `InfluxDB` organization, which be can be found in the URL of InfluxDB cloud admin console. e.g. `https://us-west-2-1.aws.cloud2.influxdata.com/orgs/<orgID>`.
    - `LOADFLUX_VERBOSE_METRICS`: in terms of InfluxDB cost, text fields will be sent when it is set as `true`, otherwise only numeric fields are sent. Default: `false`.
    As we know, InfluxDB Cloud free plan has a rate limit of 10kb/s write.
- Logging
    - `DEBUG`: we are use `debug` as our underlying logger, so you can enable a namespace. e.g. `DEBUG=loadflux:*`

### Report and logs

Normally, the report logs around scenarios and actions can be seen in console and it is written to `stderr`. 
So if you are running in a container environment or cloud platform (e.g. GCP), not surprised about the log severity is `error` level, which is intentional.

Other than the report logs, all rest logs are going to `stdout`, further redirected to a log file in current directory: `loadflux.log`.
You can run the command line: `tail -f loadflux.log` to keep the file open to display updated changes to console.

### Monitor metrics and build your dashboard

All the data is collected in `InfluxDB` if you set up the account beforehand. Then you can filter and aggregate time-series data, and end up with fancy charts.
If you like, build a dashboard and keep monitoring. All these things are depending on your Data processing skills.


## Roadmap

- collect statistical data to help analytics and build metrics to visualise
- real-time charts and 360° view of virtual users activities
- complex load phases: multiple phases for different load strategy and duration
- [`loadflux recorder`](https://github.com/StuffNZ/loadflux-recorder) Chrome extension to record the scenario and generate scenario file automatically 
- distributed load testing

## Story behind it

The original intention is we have to develop an ad-hoc solution for our cases of load testing. We evaluated different load 
testing tools: 
- [`Cypress`](https://github.com/StuffNZ/stuff-composer-load-test): generate load with e2e testing and run in a browser for each virtual user, but it's slow and prone to crash sometimes
- [`Artillery`](https://github.com/StuffNZ/stuff-composer-load-test-artillery): ramp up users per second, but not support the case - sustain a constant load
- `K6`: customised runtime, not compatible with existing nodejs modules and 3rd-party library
- `Gatling`: we do use it to do the performance testing for our Java application, but we think the scripting in Scala is not our flavour.

## Last words

UNIT TESTS ARE COMING!! NO HURRY. LOADFLUX IS STILL A BABY.

