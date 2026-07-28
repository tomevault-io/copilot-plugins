## firecracker-containerd

> The agent runs inside the Firecracker microVM and is responsible for interfacing

# agent

The agent runs inside the Firecracker microVM and is responsible for interfacing
with runc to create the containers.  The agent communicates with the runtime
outside the VM over a vsock.

The following things still need to be done with the agent:

* making it available inside the VM image (either bundling or injecting)
* starting the agent on boot
* handle stdin, stdout, and stderr of the container

---
> Source: [firecracker-microvm/firecracker-containerd](https://github.com/firecracker-microvm/firecracker-containerd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
