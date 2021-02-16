[![GitHub issues](https://img.shields.io/github/issues-raw/aminoz007/siteminder)](https://github.com/aminoz007/NR-Heroku-Logs/issues)
[![License](https://img.shields.io/github/license/aminoz007/NR-Heroku-Logs)](https://github.com/aminoz007/NR-Heroku-Logs/blob/master/LICENSE)
[![Badges](http://img.shields.io/:NR-Logs-ff6799.svg)](https://docs.newrelic.com/docs/logs/new-relic-logs/get-started/introduction-new-relic-logs)

# Heroku forwarder for New Relic

This is a [Heroku](https://heroku.com) application which can forward logs to New Relic, it is based on fluentd.
This application accepts logs from [HTTPS drains](https://devcenter.heroku.com/articles/log-drains#https-drains) and forward the logs to New Relic logs collector using NR logs APIs.

This application is using 3 components:
*   [Fluentd](https://fluentd.org): data collection software.
*   [Fleuntd-Heroku input plugin](https://github.com/ApplauseOSS/fluent-plugin-heroku-http): accepts and process Heroku HTTPS log drains.
*   [Fluentd-NewRelic output plugin](https://github.com/newrelic/newrelic-fluentd-output): output plugin that sends logs to New Relic.

This application uses the request PATH to specify the fluentd tag to use for routing.

## Setup

Heroku doesn't allow users to install separate processes within a single dyno. You will thus need to setup Fluentd as a separate Heroku application. This will become your central log aggregation server.

Follow the steps below to create Fluentd and NR plugin as a Heroku application.

Please make sure to use your **New Relic Insights Insert key**, which you can generate following these [steps](https://docs.newrelic.com/docs/apis/get-started/intro-apis/types-new-relic-api-keys#insights-insert-key.).

```
# Get the code
$ git clone git://github.com/aminoz007/NR-Heroku-Logs.git
$ cd NR-Heroku-Logs
$ rm -rf .git
$ git init
$ git add .
$ git commit -m 'initial commit'

# Login to Heroku, then create the app
$ heroku apps:create YOUR-LOGGING-APP-NAME (i.e NR-logs-forwarder)

# Next, configure your API key
$ heroku config:set NR_API_KEY=YOUR-NR-INSERT-API-KEY -a YOUR-LOGGING-APP-NAME

# Deploy
$ git push heroku master
```

Once the logging apllication is up and running. You can configure Heroku HTTPS log drains to publish logs to this application.

```
$ heroku drains:add https://YOUR-LOGGING-APP-NAME/newrelic.YOUR-SOURCE-APP-NAME -a YOUR-SOURCE-APP-NAME
```
**PS:** You may need to add ".herokuapp.com" to the logging app name depending on the URL of your logging app.
-   Example: 
    -   URL of logging app: ```https://NR-logs-forwarder.herokuapp.com/```
    -   App from where logs should be collected: ```my-cool-app```
    -   Command: ```$ heroku drains:add https://NR-logs-forwarder.herokuapp.com/newrelic.my-cool-app -a my-cool-app```

Logs should now be flowing from your service to New Relic :wink:.

## Troubleshoot

This application supports sending incoming events to STDOUT using a `debug`
tag prefix.
```
# debug using curl
$ curl "https://YOUR-LOGGING-APP-NAME/debug.YOUR-SOURCE-APP-NAME" -d "60 <13>1 2021-02-15T06:25:52.589365+00:00 host app web.1 - myMessage"
```
