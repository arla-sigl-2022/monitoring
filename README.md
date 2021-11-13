# Monitoring Workshop

This workshop is for student of EPITA SIGL 2022.

You will add some logging (not login!) to your garlaxy web api.
Those logs to an ELK (Elastic, Logstash, Kibana) stack.

Here are differents the different tools in actions:

![garlaxy logging workshop](docs/garlaxy-logging-workshop.png)

You will deploy all those tools on your localhost, and try out to visualize logs on Kibana,
base on the usage of Garlaxy on your localhost.

## Step 1: Setup a local ELK stack

In this step, you start an ELK stack:
- a container for ElasticSearch database to hold your logs
- a container for Logstash to ingest logs from your API
- a container to expose Kibana on your localhost

We found a nice all-in-one github repository that launches the latest ELK stack using a single docker-compose file.

To run ELK on your local machine:

1. Clone [arla-sigl-2022/elk](https://github.com/arla-sigl-2022/elk) (generated from [deviantony/docker-elk](https://github.com/deviantony/docker-elk) on your computer (doesn't matter where).
1. run the docker compose file in daemon mode: 
```sh
# from elk/
docker-compose up -d
```
> This may take a while to get all docker images (some hundreds of megabytes).

Once your container are running, give Kibana a minute to start, and check if you can reach kibana from your browser on http://localhost:5601

Credentials are:
- user: `elastic`
- password: `changeme`

Please don't perform any action yet, as we didn't send any logs.

Once we send the first logs, kibana setup will be easier.

## Step 2: Configure logs for Garlaxy web API

Let's add a new logger, connected to your local logstash.

### Install new dependencies

You need to install the following node modules in your web API: 
1. `log4js` to have logging tagged with specific names. And different level of loggings (`trace`, `debug`, `info`, `warn` and `error`)
1. `log4js-logstash` an adapter for `log4js` to send logs to a logstash service

To do so, from your garlaxy's `backend/` folder, install modules:
```sh
# from backend/
nvm use v16
npm i --save log4js log4js-logstash
```

### Log some information

Let's configure a new logger for your web api.

First, you need to configure your logger to connect to your logstash.

From your group's repository:
- [Adapt your backend/src/server.js](https://github.com/arla-sigl-2022/groupe-13/pull/8/commits/cca302de49ae14e8e31686080c8f61e1623645af#diff-36e2c2dd1e67a7419cef780285f514e743e48ac994a01526288acd31707e09ae): adds the logger configuration to your web API.

This configure a new logger that will output logs on:
- your console, thanks to the console appender
- your logstrash instance running on your localhost, thanks to log4js-logstash appender

You have set in `categories` default behaviour to send all log's **level** to both appenders.

> Note:
> Your differents log's **level** are:
> - `trace`: when you need to use it to track a request path, this is for very specific debug sessions
> - `debug`: when you want to debug some values in some functions
> - `info`: when you want to logs some information to help you understand usage of your app
> - `warning`: when something is wrong, but not critical for the user
> - `error`: when an error is thrown, and it's a bug.
> 
> When you set your logging level to `trace` in your `log4js` config options, it means that **ALL** log levels from trace to error will be send to both `console` and `logstash` > appenders.
> 
> If you set your log level to `error` in the `categories.default.level` config, only logs with `error` level will be sent.

Let's add some logger and logging to your `backend/src/server.js` services.
- [Adapt backend/src/server.js](https://github.com/arla-sigl-2022/groupe-13/pull/8/commits/ec30914a8a6f151709563e0be500a143cccb64fd#diff-36e2c2dd1e67a7419cef780285f514e743e48ac994a01526288acd31707e09ae): Creates `ressourceLogger` and `contractorLogger` and logs all kind of possible scenarios for usage of the service (failure and success).

This will help you to roughly know how many failed vs success requests you have on your contractor API.

### Step 3: Produce some logs

Run your frontend and your api on your local machine.

- Run frontend in one shell:
```sh 
# from frontend/
nvm use v16
npm start
```
- Run api in another shell:
```sh
# from backend/
nvm use v16
node src/server.js
```
- Start your `docker-compose` from the database workshop, to have your Postgres instance running.

Now, navigate to your app and load some contractor's comment.

If you see some `INFO` or `ERROR` logs on your stdout of `backend/` shell, you can proceed to the next step.

### Step 4: Configure Kibana to read from elasticSearch

Go to kibana dashboard on http://localhost:5301

Login using `elasctic/changeme` as credentials.

You should see a welcome screen like:
![welcome](docs/welcome-elastic.png)

Click on `Add data`

Then on the sidebar menu icon, select the discover section:

![discover](docs/discover-menu.png)

Then, you should see that kibana noticed you have some logs in your elasticsearch, but you don't have any indexes yet to search on those logs.

Let's create them!

Select `Create index` button:
![create-index](docs/create-index-pattern.png)

For step 1 of index creation, enter `logstash` as index pattern name, and click `Next step`:
![step-1](docs/step-1-logstash.png)

For step 2 of index creation, you need to select the field that reprensents the log timestamp. Select @timestamp form the filed selector:
![step-2](docs/step-2-timestamp.png)

Now, click on `create index pattern` after selecting timestamp field.

Go back to the sidebar menu > discover view.

You should see your logs from previous step:
![discover-table](docs/discover-table-log.png)

By default, it shows logs from latest 15 minutes, you can increase the time range to find back your logs if it was more that 15 minutes ago.

Congratulation!! You've just setup a whole ELK stack and connect logs from your API.

Now, you can produce more logs of different level, and try to query them in the discover section, create dashboard with custom visualizations and more.

