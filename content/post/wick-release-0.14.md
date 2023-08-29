
+++
showonlyimage = false
draft = false
image = "/images/post/wick/wick-release-0.14.png"
date = "2023-08-28T21:11:33.488Z"
title = "Wick: An overdue introduction"
categories = [ "security", "wick", "WebAssembly" ]
+++


## Background

Two years ago, I hit a wall. I had worked in software for twenty+ years and spent seven of them helping Shape Security grow from a small startup to a unicorn.

I was ready to start something new. But I just *couldn't*.

I wasn't burnt out. I was overwhelmed. I knew what lay ahead. The same tasks that I'd performed hundreds of times. Even with new languages and frameworks, the cloud, serverless, microservices, etc. It was the same work with new names. Integrate a dozen different APIs here, make a hundred random libraries work together there, and rewrite the same logic in six new languages because that's what we do.

It wasn't that the work was difficult. It's that we shouldn't have to do it anymore.

Most of every application is software components connecting to other software components. Why can't we get the USB experience for software?

We could build real applications faster if software components connected more easily. We could automatically generate documentation and examples. We could secure them the same way, deploy them the same way, and test them the same way. If every component connected asynchronously, we could scale from a single thread to a distributed network without changing a single line of code.

None of these ideas are new. These concepts have roots in functional/reactive and flow-based programming. They've mainly applied within the boundaries of a language or runtime. But businesses don't operate within a language or runtime. Teams have different use cases, varying environments, and myriad tools to integrate with.

With advancements like WebAssembly and tools like Kubernetes and Docker, it seemed like a good time to bring these ideas to a new layer.

## Wick: An Application meta-framework

Wick is a meta-framework for building applications out of disparate components. I'm using the term "meta-framework" because it exists outside of a language and – thanks to WebAssembly – across languages.

At the core of Wick is how it defines a component. A component is a collection of operations. Each operation takes in and produces any number of streams.

Wick components are just like libraries in any programming language. They're collections of functions. The Wick difference is that the functions operate on streams and can produce multiple outputs versus a single return value.

### Wick components

Component definitions are YAML and define configuration, test cases, package metadata, operations, and an operation's inputs and outputs.

The actual logic for a component comes in four (current) flavors:

- **WebAssembly**: Wick WebAssembly components pair a definition with a WebAssembly binary. Wick delegates to a WASM function on invocation.
- **Composite**: Composite components build their logic out of others. They define a functional data pipeline through operations and expose the starting inputs and ending outputs as their signature. Think of composite component operations as similar to Unix pipes or using `map()` on arrays or streams.
- **SQL**: SQL components craft SQL queries that run on a database. Wick binds inputs to arguments for the query, and each row is output to a stream. Wick supports MS SQL Server, Postgres, and Sqlite out-of-the-box.
- **HTTP**: An HTTP component defines a base URL and operations that make requests to paths on that URL. Wick uses an operation's inputs to build the URL path, query string, and body. The operation's output is the response metadata and the body (as raw bytes or a JSON object).

The best way to understand how this all works is with examples. Building an application out of a bunch of YAML might seem nuts.

#### Quickly make a new component

The `wick new` command helps automate the boilerplate of making new components.


![Alt text](/images/post/wick/wick-new-component.gif "wick new command output")

The command above makes an example component with one operation, `echo`.

The `echo` operation pipes what comes in on a stream called `input` to a stream called `output`. Here's what it looks like:

```yaml
kind: wick/component@v1
name: my_component
component:
  kind: wick/component/composite@v1
  operations:
  - name: echo
    inputs:
    - name: input
      type: object # an "any" type
    outputs:
    - name: output
      type: object
    flow:
    - <>.input -> <>.output
```

The component configuration is always the source of truth. Even WebAssembly components – which can define their own exports – can't run in a wick context with anything but the interfaces defined in the YAML. This constraint lets tools query and assert against components without parsing myriad languages or inspecting any of a component's internals. Tools like `wick list`, for example.

The `wick list` command loads a component and reports its signature. It's simple but powerful.

![Alt text](/images/post/wick/wick-list.gif "wick list command output")

All `wick` commands accept a `--json` flag to produce output that's easy to parse and integrate with other tools. Since component definitions support descriptive fields on many elements, you can use this output to generate documentation and examples, like we do on [our docs site](https://candle.dev/docs/wick/guide/components/sql/):

![Alt text](/images/post/wick/wick-docs.png "wick docs screenshot")

#### Running components from the command line

![Alt text](/images/post/wick/wick-invoke.gif "wick invoke command output")

Since every wick component follows the same general API, the `wick invoke` command can invoke *any* operation in *any* component ***or*** *any* application.

That's right, you can invoke a function deep inside an application straight from the command line. No extra configuration needed. That means *every operation in every application is now a command line utility.*

Need to expose functionality for another team to use? Already done.

Need to validate data in another time and setting? Use the application itself in a bash script.

Need to troubleshoot why part of an app is failing? Execute an operation from the command line and test it in isolation.

Speaking of tests...

#### Testing components

While you're free to write unit tests in the language you compile to WebAssembly, `wick` comes with its own test framework to make adding tests trivial.

Define wick tests in separate YAML files or embed them directly in a component definition. The test cases define what to pass to an operation and what to assert against in the output.

```yaml
tests:
  - name: basic_tests
    cases:
      - name: basic_equality
        operation: echo
        inputs:
          - name: input
            value: 'Hello, world!'
        outputs:
          - name: output
            value: 'Hello, world!'
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

Wick prints results in the TAP – Test Anything Protocol – format. Many CI/CD tools support TAP output out of the box.

#### Taking "secure by default" to a new level

I am as passionate about application security as I am flabbergasted we are still falling victim to vulnerabilities we understood in the 90s.

I don't know about you, but I never want to hear about another SQL injection vulnerability again. I never want to hear about a new compromised dependency. I never want to hear about backdoors left open in applications.

Wick's not a silver bullet, but we've spent hundreds of hours crafting solutions that expose all the power developers want while maintaining safeguards and features to secure applications the first time.

Features like:

- Developers must explicitly configure resources like base URLs, directories, files, ports, etc. Wick can understand and validate resources before granting components access.
- Wick individually sandboxes WebAssembly components, each with separate permissions and access.
- We designed SQL components so that SQL injection is impossible and queries are auditable without running or parsing any code.
- All HTTP requests are auditable by the base URL.
- Only the root application can access ENV variables. Components can not access the environment and require it to bubble up through configuration.

#### Auditing applications safely

DevOps and security teams can independently audit and validate an application without running any component code by using the `wick config audit` command.

The `audit` command outputs a list of all resources used by an application in one fell swoop.

![Alt text](/images/post/wick/wick-config-audit.gif "wick config audit command output")

#### Locking down applications

An audit lets you see what an application wants to access. Lockdown configurations define what an application is allowed to access.

Do you want to integrate a dependency that makes requests from a new URL or needs file system access? Don't expose that access to your whole application, just the component that needs it.

The `wick config audit` conveniently accepts a `--lockdown` flag and will happily generate a (passing) lockdown configuration from an audit report. Teams can use it as a starting point to secure an application before deployment.

![Alt text](/images/post/wick/wick-config-lockdown.gif "wick run --lockdown command output")

## So much more

There's so much more to talk about with `wick` with features like:

- Distributed components
- `wick install` to install components locally
- Wick triggers for HTTP servers, CLI commands, and cron jobs
- How to automatically generate artifacts like OpenAPI specifications.
- Candle's cloud service & registry
- Logging and traces
- Components to automatically add features like OAuth, Permissions, and Payments
- Substreams!
- And more!

We've been deploying Wick apps to production for months now. There's still a lot in the roadmap, and I'm excited for what's to come.

If you're interested in learning more, check out [wick on github](https://github.com/candlecorp/wick), the wick documentation here on [candle.dev](https://candle.dev/docs/wick/guide/), and join our [discord server](https://discord.gg/candle)!

Thanks!