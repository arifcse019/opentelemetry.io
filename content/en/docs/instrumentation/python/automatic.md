---
title: Automatic Instrumentation
linkTitle: Automatic
weight: 30
spelling: cSpell:ignore distro mkdir uninstrumented virtualenv
---

One of the best ways to instrument Python applications is to use OpenTelemetry
automatic instrumentation (auto-instrumentation). This approach is simple, easy,
and doesn’t require many code changes. You only need to install a few Python
packages to successfully instrument your application’s code.

## Overview

This example demonstrates how to use auto-instrumentation in OpenTelemetry. The
example is based on an [OpenTracing example][]. You can download or view the
[source files][] used in this page from the `opentelemetry-python` repo.

This example uses two different scripts. The main difference between them is
whether or not they’re instrumented manually:


1. `server_instrumented.py` - instrumented manually
2. `server_uninstrumented.py` - not instrumented manually

Run the first script without the automatic instrumentation agent and the second
with the agent. They should both produce the same results, demonstrating that
the automatic instrumentation agent does exactly the same thing as manual
instrumentation.

To better understand auto-instrumentation, see the relevant part of both
scripts:

### Manually instrumented server

`server_instrumented.py`

```python
@app.route("/server_request")
def server_request():
    with tracer.start_as_current_span(
        "server_request",
        context=extract(request.headers),
        kind=trace.SpanKind.SERVER,
        attributes=collect_request_attributes(request.environ),
    ):
        print(request.args.get("param"))
        return "served"
```

### Server not instrumented manually

`server_uninstrumented.py`

```python
@app.route("/server_request")
def server_request():
    print(request.args.get("param"))
    return "served"
```

## Prepare

Execute the following example in a separate virtual environment. Run the
following commands to prepare for auto-instrumentation:

```console
$ mkdir auto_instrumentation
$ virtualenv auto_instrumentation
$ source auto_instrumentation/bin/activate
```

## Install

Run the following commands to install the appropriate packages. The
`opentelemetry-distro` package depends on a few others, like `opentelemetry-sdk`
for custom instrumentation of your own code and `opentelemetry-instrumentation`
which provides several commands that help automatically instrument a program.

```console
$ pip install opentelemetry-distro
$ pip install opentelemetry-instrumentation-flask
$ pip install flask
$ pip install requests
```

The examples that follow send instrumentation results to the console. Learn more
about installing and configuring the [OpenTelemetry Distro](../distro) to send
telemetry to other destinations, like an OpenTelemetry Collector.

## Execute

This section guides you through the manual process of instrumenting a server as
well as the process of executing an automatically instrumented server.

### Execute a manually instrumented server

Execute the server in two separate consoles, one to run each of the scripts that
make up this example:

```console
$ source auto_instrumentation/bin/activate
$ python server_instrumented.py
```

```console
$ source auto_instrumentation/bin/activate
$ python client.py testing
```

The console running `server_instrumented.py` will display the spans generated by
instrumentation as JSON. The spans should appear similar to the following
example:

```json
{
    "name": "server_request",
    "context": {
        "trace_id": "0xfa002aad260b5f7110db674a9ddfcd23",
        "span_id": "0x8b8bbaf3ca9c5131",
        "trace_state": "{}"
    },
    "kind": "SpanKind.SERVER",
    "parent_id": null,
    "start_time": "2020-04-30T17:28:57.886397Z",
    "end_time": "2020-04-30T17:28:57.886490Z",
    "status": {
        "status_code": "OK"
    },
    "attributes": {
        "http.method": "GET",
        "http.server_name": "127.0.0.1",
        "http.scheme": "http",
        "host.port": 8082,
        "http.host": "localhost:8082",
        "http.target": "/server_request?param=testing",
        "net.peer.ip": "127.0.0.1",
        "net.peer.port": 52872,
        "http.flavor": "1.1"
    },
    "events": [],
    "links": [],
    "resource": {
        "telemetry.sdk.language": "python",
        "telemetry.sdk.name": "opentelemetry",
        "telemetry.sdk.version": "0.16b1"
    }
}
```

### Execute an automatically instrumented server

Stop the execution of `server_instrumented.py` by pressing <kbd>Control+C</kbd>
and run the following command instead:

```console
$ opentelemetry-instrument --traces_exporter console python server_uninstrumented.py
```

In the console where you previously executed `client.py`, run the following
command again:

```console
$ python client.py testing
```

The console running `server_uninstrumented.py` will display the spans generated
by instrumentation as JSON. The spans should appear similar to the following
example:

```json
{
    "name": "server_request",
    "context": {
        "trace_id": "0x9f528e0b76189f539d9c21b1a7a2fc24",
        "span_id": "0xd79760685cd4c269",
        "trace_state": "{}"
    },
    "kind": "SpanKind.SERVER",
    "parent_id": "0xb4fb7eee22ef78e4",
    "start_time": "2020-04-30T17:10:02.400604Z",
    "end_time": "2020-04-30T17:10:02.401858Z",
    "status": {
        "status_code": "OK"
    },
    "attributes": {
        "http.method": "GET",
        "http.server_name": "127.0.0.1",
        "http.scheme": "http",
        "host.port": 8082,
        "http.host": "localhost:8082",
        "http.target": "/server_request?param=testing",
        "net.peer.ip": "127.0.0.1",
        "net.peer.port": 48240,
        "http.flavor": "1.1",
        "http.route": "/server_request",
        "http.status_text": "OK",
        "http.status_code": 200
    },
    "events": [],
    "links": [],
    "resource": {
    "telemetry.sdk.language": "python",
    "telemetry.sdk.name": "opentelemetry",
    "telemetry.sdk.version": "0.16b1",
    "service.name": ""
    }
}
```

You can see that both outputs are the same because automatic instrumentation does
exactly what manual instrumentation does.

### Instrumentation while debugging

The debug mode can be enabled in the Flask app like this:

```python
if __name__ == "__main__":
    app.run(port=8082, debug=True)
```

The debug mode can break instrumentation from happening because it enables a
reloader. To run instrumentation while the debug mode is enabled, set the
`use_reloader` option to `False`:

```python
if __name__ == "__main__":
    app.run(port=8082, debug=True, use_reloader=False)
```

[distro]: https://github.com/open-telemetry/opentelemetry-python/tree/main/docs/examples/distro
[OpenTracing example]: https://github.com/yurishkuro/opentracing-tutorial/tree/master/python
[source files]: https://github.com/open-telemetry/opentelemetry-python/tree/main/docs/examples/auto-instrumentation