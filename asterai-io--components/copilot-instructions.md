## components

> provides host interfaces like `host-ws` and `host-cron`, to make it easier

# Asterai Component Publishing Skill

This document describes how to develop, test, and publish Asterai Components.

## The Asterai Runtime

[Asterai][asterai] is a WASM component registry and open-source runtime (on top of wasmtime)
where each component implements the WebAssembly Component Model, which currently
is implemented by WASIp2.

Asterai also establishes the concept of an Environment. This is a set of
components and a set of environment variables. The asterai runtime, available
via the asterai CLI, can run environments. It is also possible to call
individual functions within components in an environment. When running an
environment, the runtime will start a web server that hosts the endpoints:

- HTTP POST /v1/environment/{env_ns}/{env_name}/call -- for calling functions within
  an environment. May require the Authorization header if specified.
- HTTP <any> /:env-namespace/:env-name/:comp-namespace/:comp-name/... -- this will
  forward the `...` request (any method) to the component's
  `wasi:http/incoming-handler` implementation, so this allows components
  that implement this interface to essentially host their own web servers.

The runtime also provides the [asterai:host-ws][host-ws] and
[asterai:host-cron][host-cron] interfaces which can be used if components
need to create client WS connections or set up cron jobs
(to call any function within the environment).
The host function [asterai:host][host] provides an interface for reflecting
on the environment components and dynamic component function calling.
Keep these in mind when building the component.

The main limitation of WASI at the moment is that languages do not 100% natively
support compiling against it. For example, in Rust, if you need to use an
HTTP client to make requests, you would need to use something like the [waki][waki]
crate, as `reqwest` does not work against WASI yet. This is also why asterai
provides host interfaces like `host-ws` and `host-cron`, to make it easier
for components to do these tasks within the WASI runtime.

## The Asterai Component Registry

The asterai registry can be thought of as NPM for WASM components. It allows
anyone to pull public components without any auth, while logged in users can
consume private components and also push environments (which are always private)
so that they can be pulled or run in the cloud.
Components can be easily composed together, as [WIT][wit] can be used to generate typed
references in whatever language the component is being written in.

## Building a component

While technically any WASM-compatible language can be used, the asterai CLI
tooling currently supports Rust and Typescript/Javascript.
Before you start building a new component, ensure the user has the CLI installed
and is authenticated:

`asterai auth status`

Download and install the CLI:
- via npm: `npm i -g @asterai/cli` (or pnpm, or yarn etc)
- Mac/Linux: `curl -fsSL https://asterai.io/install.sh | sh`
- Windows: `powershell -c "irm asterai.io/install.ps1 | iex"`

Ensure the user is authenticated, run `asterai auth login <api-key>`.
The user can get their API key from https://asterai.io/settings/api-keys.

Scaffold a new component project dir for Rust:

`asterai component init your-component-name -l rs`

For Typescript, change `-l rs` to `-l ts`.

Then, `cd` into the project dir and build with:

`asterai component build`

This will do two things: first it will generate type bindings from the
`component.wit`, including for all imports, and then it will compile the
component. It will autogenerate a `wit/` dir which resolves all imports.
It is important to NEVER touch the `wit/` dir manually, and it should be
.gitignored, as it can be generated at any time by running the build command.

To test a component, you can use the command:

`asterai component call . interface/fn-name "arg1" "arg2" ...`

This will call the provided function (eg interface/fn-name) on the current
component project, and is an easy way to test that the component works
as expected. This will load all system environment vars into the temporary
environment created for the function call, so make sure to set required env vars
(can be inline) before calling.

To push a component to the registry, run

`asterai component push`

Which will push a private component. Pass `-f` to override the current version,
though note this only works for private components. The version is defined in
the `component.wit` file.

To test a component within a proper environment, the component must be added
to that environment, and env vars need to be explicitly set as part of that
environment. For example, creating a new environment and defining vars:

`asterai env init my-environment`
`asterai env set-var my-environment --var MY_VAR=foo`

Adding a component to the environment:

`asterai env add-component my-environment username:comp-name`

Then you can call a function within it:

`asterai env call my-environment comp-ns:comp-name interface/fn-name "arg1" ...`

## Good Practices

1. Instead of exporting a function directly in the component world,
prefer to export an interface from the component world.
This is because interface is the unit of composability in WIT, and so
the component must export an interface for other components to be
able to call it.
2. Unless specified otherwise, prefer lower level languages, such as Rust and
Go, over Python and JS/TS, as those are more memory efficient.
3. Prefer typed [WIT][wit] functions. For example, prefer a function signature like
`a: string, b: s32` over something like  `args-json: string` which obfuscates
the real arg signature. Args can also contain WIT records if required.

## External References

- [asterai llms.txt][asterai]
- [host interface][host]
- [WS host interface][host-ws]
- [Cron host interface][host-cron]
- [waki library][waki]
- [WIT reference][wit]

[asterai]: http://asterai.io/llms.txt
[host]: https://github.com/asterai-io/asterai/blob/f15b6e0c5dd6cf4ae1fb7e77ad72dbfe4a2bdb63/runtime/wit/asterai_host.wit
[host-ws]: https://github.com/asterai-io/asterai/blob/f15b6e0c5dd6cf4ae1fb7e77ad72dbfe4a2bdb63/runtime/wit/asterai_host_ws.wit
[host-cron]: https://github.com/asterai-io/asterai/blob/f15b6e0c5dd6cf4ae1fb7e77ad72dbfe4a2bdb63/runtime/wit/asterai_host_cron.wit
[waki]: https://github.com/wacker-dev/waki
[wit]: https://component-model.bytecodealliance.org/design/wit.html

---
> Source: [asterai-io/components](https://github.com/asterai-io/components) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
