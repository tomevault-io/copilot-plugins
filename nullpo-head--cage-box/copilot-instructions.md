## cage-box

> Cage Box is a sandbox tool to enable scripts and binaries to declare their allowed permissions,

# AGENTS.md for Cage Box project

## What is Cage box?

Cage Box is a sandbox tool to enable scripts and binaries to declare their allowed permissions,
ensuring they can only interact with allowed file paths. Even when running with
`sudo`, these programs are prevented from accessing any unauthorized files or
directories. Cage Box is a rootless tool; it allows you to restrict target
processes without root privilege.

Cage Box is like an AppArmor, but a standalone tool that can be used with any script or binary, not just those that are designed to work with it. It uses Linux's Landlock to restrict file access, and seccomp_unotify together to dynamically allow or deny file path access in an interactive mode.

With Cage Box,
- a developer can declare file paths their script or binary is allowed to access
- an end user can check what files are accessed by looking at the beginning of the file or `$ Cage Box -l app`, without inspecting the whole contents
- you can also run unverified applications that don't natively support Cage Box in an interactive sandbox that asks you whether each path accessed should be allowed at runtime
- you can also make a quick and secure sandbox for your AI agent to run commands, without worrying about it accessing unauthorized files or directories.

## Sandbox mechanism overview

Cage Box restricts file access by Landlock rules, which is converted from a Cage Box rule file. Since Landlock does not restrict some critical system calls such as truncate or setxattr, Cage Box combines Landlock with seccomp-bpf and seccomp notify; Cage Box spawns a monitor process to restrict such system calls by the target process, and emulates those system calls on behalf of the target process.

These sandbox are implemented mainly by src/monitor.rs

Cage Box also supports an interactive monitoring mode. Cage Box asks users to accept or deny file paths when it detects file accesses which are not allowed by the rules. This is achieved by a novel mechanism; Cage Box denies all file access by an empty Landlock rule first, then emulates file path system calls such as `openat` or `mkdir`. The recent seccomp notify mechanism enables this method, since it enables Cage Box to inject result file descriptors to the target process.

See src/interactive.rs

The novel points of Cage Box are
- Handy Cage Box script UX, which is convenient to distribute binaries and sandbox rules together
- Cage Box allows you to interactively allow or deny file access when you run it in the interactive sandbox mode, which is powerful and handy, but not a common feature among existing sandbox tools.
- It is a fully rootless sandbox solution. Vulnerability of Cage Box does not give more privileges than you originally have.
- It does not use user namespaces either; it runs on Debian and Ubuntu without additional configuration, where user namespaces are restricted

## Implementation design


Cage Box restricts the file access of scripts and binaries with Landlock LSM (Linux Security Module).

The restriction implementation is based on the following three components:

### 1. Landlock LSM:

Cage Box runs the binary to restrict the file access, applying the Landlock LSM rules defined in the Cage Box script.

This is the core mechanism of Cage Box. Cage Box converts the sandbox rules into Landlock rulesets.
Landlock does not require root privilege to apply restriction, which allows Cage Box to be a rootless sandbox solution.

### 2. seccomp_unotify (seccomp-bpf + unotify):

While Landlock is the core mechanism of Cage Box, it has two problems as-is;

* Some of critical syscalls are not restricted by Landlock, including `chmod`, `chown`, `truncate`, `ioctl`, and so on.
   You can find an exhaustive list in `man landlock`. This is a known limitation of Landlock.
   So, we need an additional mechanism to restrict these syscalls.

* Landlock does not allow us to hook file access. Thus, we cannot achieve interactive file path restriction by Landlock.

To solve these two problems, Cage Box utilizes seccomp_unotify.
seccomp_unotify is relatively a new feature of Linux. It is similar to regular seccomp-BPF + ptrace,
but it allows a monitor process to inject a FD to the target process,
by which the monitor process can emulate file related syscalls.

For the first issue, Cage Box uses seccomp-bpf + unotify to hook those syscalls,
check if the requested paths are allowed by applied Cage Box ruleset,
and call syscalls against the given paths on behalf of the target process and injects the result to the target process.

For the second issue, Cage Box first restricts all file access by Landlock.
Then, Cage Box hooks `open` syscall family by seccomp_unotify.
When file accesses happen, Cage Box asks the user interactively whether they want to allow that access or not.
If users allow them, Cage Box emulates `open` syscalls and injects result FDs to the target process.

### 3. regular seccomp-BPF + ptrace (to replace execve with execveat so that we can hook it via notify):

A remaining issue of the approach of Landlock + seccomp_unotify for the interactive mode is a fact that a monitor process cannot emulate execve for the target process.
However, we also want to interactively allow or deny `execve` as a type of file accesses as well.

For this purpose, Cage Box also uses seccomp-BPF + ptrace to hook `execve` syscall by ptrace. When starting the target process at start up, the forked child stops itself with SIGSTOP, and the parent attaches it by PTRACE_SEIZE and restarts the child.

#### 3.1 Basic: ELF apps

After trapping an `execve` call, Cage Box replaces the syscall number to call with `execveat`, and modify args so that the target process does `execve` against a FD instead of the original path.

1. The target process calls `execve`.
2. The ptrace monitor of Cage Box traps it by ptrace (The seccomp bpf filter returns the ptrace value). Replace the system call number with that of `execveat`. Replace the fd with an unused FD. Remember the path and args in a shared map structure between the ptrace monitor and the notification monitor. Replace `pathname` with the pointer to the null character at the end.
   - List the current sibling threads and send `PTRACE_INTERRUPT` to each one,
     but do not wait for the resulting stops. Keep the known sibling TIDs for
     failure cleanup.
   - Reserve a high unused FD, bounded by the target's `RLIMIT_NOFILE`, to reduce
     the chance that a still-running sibling allocates the same descriptor.
     This is a best-effort collision mitigation rather than a race-free FD
     reservation.
   - Allow only one rewritten exec per thread group. Reject another `execve`
     from the same TGID with `EBUSY` while the first is active.
   - Cage Box does not support FD tables shared across different thread groups,
     such as `clone(CLONE_FILES)` without `CLONE_THREAD`. FD reservation only
     coordinates tasks in the exec caller's thread group.
3. The ptrace monitor continues the target process by `PTRACE_SYSCALL`.
4. The notification monitor of Cage Box is notified for `execveat`. Get the args and the FD number to inject from the shared data structure. Do the permission check against the requested path. If the access is allowed, create or reuse a memfd copy of the executable and inject that FD, not the original executable FD. Executing a memfd works inside the restricted Landlock domain and avoids requiring read permission in Cage Box rules for executable paths.
   - Before injecting a memfd, verify that the original object is a regular file
     with at least one executable mode bit and that it passes an `X_OK` check
     under the target's effective credentials. This prevents a failed exec from
     exposing the contents of an arbitrary readable, non-executable file through
     the intentionally retained memfd.
   - Cache recently copied executable memfds up to 300 MiB in total.
   - Before reusing a cached memfd, compare the source executable's stat metadata. If it changed, refresh the memfd contents so Cage Box never executes stale cached bytes.
   - Bound each copy to the source's pre-copy size and compare stat metadata
     again afterward. Reject the exec if the source changed while being copied.
5. If `execveat` succeeds, the ptrace monitor traps `PTRACE_EVENT_EXEC`. If it fails, the ptrace monitor should trap the syscall exit event, or a process exit event. In either case, the ptrace monitor cleans up the exec args data from the shared map structure. On failure, send `PTRACE_SYSCALL` to every known sibling without tracking whether each one had actually stopped. A delayed interrupt stop that arrives afterward is resumed by the normal ptrace event loop.
   - For implementation simplicity, a memfd injected before a failed `execveat`
     remains open in the target process. The memfd is sealed against mutation,
     but the old image can read its contents even when Cage Box granted only
     execute permission. It is closed when the target closes it, a later
     successful exec applies `O_CLOEXEC`, or the process exits. Repeated failed
     execs can consume the target's FD slots.

Note that the ptrace is configured to trace all descendants.

Regarding `execveat`, we don't support this relatively rare system call for now.
1. The notification monitor traps `execveat`. It can tell that this `execveat` is an organic one by checking if the shared map structure has information of this system call. If it finds the system call is organic, it denies the system call showing an error message to users that says `execveat` is not supported currently and encourage users to report their use cases.

By implementing this basic flow, `execve` for regular ELF apps.

#### 3.2 Make scripts work

Some script interpreters such as Node.js check the real path of the given script path, since the parent directory of the executed script path is crucial for their semantics.

To cover these interpreters, we need one more trick; we implement a undocumented helper subcommand, `shebang`, to interpret shebangs in the user space. We will use this `shebang` subcommand as a trampoline shebang interpreter to bypass Landlock restriction discussed later.

0. In modes that trace exec (`--interactive` and `--seccomp-all-syscalls`),
   Cage Box adds its current executable with read and execute permissions and
   `%system_libs` with read permission to the effective rules. Although
   `%system_libs` might not contain every library needed by Cage Box, this is
   an accepted compromise.

1. After the normal execute permission checks, Cage Box reads the first
   `BINPRM_BUF_SIZE` bytes (defined as 256 bytes) and checks whether the first
   two bytes are `#!`. Non-scripts continue through the ELF flow in 3.1.

2. Before proceeding, the notification monitor follows the interpreter chain
   up to `BINPRM_MAX_RECURSION` (defined as 4) script expansions. Intermediate
   interpreters require read and execute permission. If the chain does not
   reach a non-script executable, `execve` fails with `ELOOP`.

3. Next, Cage Box does `execve` following the steps of 3.1, but it executes the following unnamed script instead of the real target binary:

   ```
   #!/path/to/cage-box shebang
   /path/to/the/original/shebang original_shebang_parameters
   <private length-prefixed original-path metadata>
   ```

   The first line is a shebang to the Cage Box binary with the `shebang`
   argument. The second line is a normalized copy of the original shebang with
   the `#!` prefix stripped. The private trailer preserves the exact original
   pathname bytes because Linux replaces the caller's `argv[0]` with
   `/dev/fd/N` when executing the unnamed script.

   The trampoline FD is injected without `O_CLOEXEC`; Linux otherwise rejects
   an `execveat(AT_EMPTY_PATH)` script with `ENOENT`. The hidden helper closes
   it after reading the payload. If exec fails before the helper starts, the FD
   intentionally remains open in the old target image.

3. This setup makes Cage Box executed as a trampoline binary to execute scripts under the Landlock restriction with no file access.
   Executing shebang scripts will fail unless the Landlock allows read to interpreters.
   `shebang` subcommand of Cage Box is a hidden subcommand to read interpreters and triggers seccomp notify of the parent Cage Box command.

   `shebang` parses the shebang by the same logic as the Linux kernel has:
      - Skip any whitespaces (spaces or tabs) until we find the path to the interpreter. If we reach a newline or the `BINPERM_BUF_SIZE` limit, it should fail with ENOEXEC
      - If the shebang has non-whitespace characters following whitespaces after the path to the interpreter, the whole non-whitespace characters will be parsed as a single argument to the interpreter

4. We repeat this shebang expansion logic recursively up to `BINPRM_MAX_RECURSION` (defined as 4) times.
   If we cannot reach a non-shebang path, the `shebang` subcommand exits with status 126.
   This should be a rare case since we already checked the depth of the shebang loop in step 2.
   Strictly speaking, this behavior doesn't perfectly simulate the semantics of execve, but since it's an edge case, we compromise here.

5. After constructing the final arguments and the interpreter path, `shebang` does `execve` to the final interpreter binary with correctly constructed program arguments by the steps described in 3.1.

---
> Source: [nullpo-head/cage-box](https://github.com/nullpo-head/cage-box) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
