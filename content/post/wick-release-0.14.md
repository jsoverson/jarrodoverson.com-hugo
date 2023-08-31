
+++
showonlyimage = false
draft = false
image = "/images/post/wick/wick-release-0.14.png"
date = "2023-08-28T21:11:33.488Z"
title = "Wick 0.14 release: deep invocations and packet assertions"
categories = [ "security", "wick", "WebAssembly" ]
+++

Wick 0.14 brings two major QoL improvements to wick: app invocations and a whole class of new assertions to Wick test cases.

## Deep invocations

You could always use `wick invoke` against a single component configuration.

![Alt text](/images/post/wick/wick-invoke.gif "wick invoke output")

But as components and their relationships became more elaborate it became harder to "just invoke" some components that were deeply nested inside others. It was even harder with components that were wired into an application configuration.

Now, `wick invoke` can will accept paths to an operation and invoke it from any level and will now work on applications directly.

That's right, now you can invoke any operation inside any application straight from the command line. No extra configuration needed.

That means *every operation in every application is now a command line utility.*

Need to expose functionality for another team to use? Already done.

Need to validate data in another time and setting? Use the application itself in a bash script.

Need to troubleshoot why part of an app is failing? Execute an operation from the command line and test it in isolation.

Oh and speaking of tests...

## Packet Assertions

Wick's default assertion for strict equality in test cases is fine when data was predictable and deterministic, but life isn't always predictable and deterministic.

The worst offenders are HTTP and DB operations whose output often contains timestamps, IDs, and other values that are hard to predict.

0.14 adds a new class of assertions that let you test parts of output match different criteria.

```yaml
tests:
  - name: basic_tests
    cases:

      # Old test cases still work

      - name: basic_equality
        operation: echo
        inputs:
          - name: input
            value: 'Hello, world!'
        outputs:
          - name: output
            value: 'Hello, world!'

      # Now you can specify `assertions` for each output packet.

      - name: assertions
        operation: echo
        inputs:
          - name: input
            value:
              string_value: Hello!
              number: 42
        outputs:
          - name: output
            assertions:
              - operator: Contains
                value: { string_value: Hello! }
              - operator: LessThan
                path: number
                value: 100
```

![Alt text](/images/post/wick/wick-test.gif "wick test command output")

## Release notes

Full release notes: https://github.com/candlecorp/wick/releases/tag/0.14.0

Be sure to check out [wick on github](https://github.com/candlecorp/wick), the wick documentation on [candle.dev](https://candle.dev/docs/wick/guide/), and join our [discord server](https://discord.gg/candle)!

Thanks!