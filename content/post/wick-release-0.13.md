
+++
showonlyimage = false
draft = false
image = "/images/post/wick/wick-release-0.13.png"
date = "2023-08-20T21:11:33.488Z"
title = "Wick 0.13 release: audit & lockdown"
categories = [ "security", "wick", "WebAssembly" ]
+++

Wick 0.13 is out! This release adds two huge new features to improve security in your applications, `wick config audit` to audit the resources your apps use and Lockdown configurations to restrict access to those resources.

## Audit reports

![Alt text](/images/post/wick/wick-config-audit.gif "wick config audit output")

The `wick config audit` command outputs a list of all resources used by an application in one fell swoop. This report lists all the URLs, files, directories, and ports that your application wants to access and you can get it without running the application at all.

DevOps or security teams can use these reports to audit and validate an application before deployment. Like with any other command, you can use the `--json` flag to get machine-readable output and integrate the output with other tools.

## Lockdown configuration

Seeing a list of resources is great, but visually assessing a list of URLs and ports for red flags shouldn't be part of anyone's daily workflow. That's where lockdown configurations come in. Lockdowns are a new type of Wick configuration that define restrictions for application. Lockdowns can restrict access to URLs, files, directories, and ports on a *component by component basis*.

```yaml
kind: wick/lockdown@v1
resources:
- kind: wick/resource/tcpport@v1
  address: 0.0.0.0
  port: '8999'
  components: [__local__] # __local__ is the root app context

- kind: wick/resource/url@v1
  allow: https://login.microsoftonline.com/organizations/oauth2/v2.0/token
  components: [oauth]

- kind: wick/resource/url@v1
  allow: http://localhost:5173/
  components: [__local__]

- kind: wick/resource/url@v1
  allow: mssql://account.database.windows.net:1433/database
  components: [reports, users]
```

Let’s say you want to integrate a dependency that makes requests from a new URL or needs file system access. You shouldn’t have to grant that access to your whole application, just the component that needs it.

The wick config audit conveniently accepts a --lockdown flag and will happily generate a (passing) lockdown configuration from an audit report.

![Alt text](/images/post/wick/wick-config-lockdown.gif "wick config lockdown output")

Security teams can use that lockdown configuration as a starting point to configure what an application is allowed to do.

## Also in 0.13

We redid how logging, tracing, and telemetry is managed so you have greater control over what is logged and where it goes.

Full release notes: https://github.com/candlecorp/wick/releases/tag/0.13.0

If you're interested in learning more, check out [wick on github](https://github.com/candlecorp/wick), the wick documentation here on [candle.dev](https://candle.dev/docs/wick/guide/), and join our [discord server](https://discord.gg/candle)!

Thanks!