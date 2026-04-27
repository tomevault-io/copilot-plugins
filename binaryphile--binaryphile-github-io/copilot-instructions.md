## binaryphile-github-io

> @~/projects/tandem-protocol/tandem-protocol.md

# Tandem Protocol
@~/projects/tandem-protocol/tandem-protocol.md

### evtctl — project task management

```
evtctl task <description>            # publish a task event
evtctl task --to <project> <desc>    # task for another project
evtctl inbox <app> <message>         # send inbox message
evtctl done <id>[,<id>...] [evidence] # publish a task-done event
evtctl open                          # list open tasks
evtctl audit                         # full task reconciliation
evtctl claim <id> <name>             # claim a task
evtctl claims                        # list active claims
```

Stream name automatically derived from project directory: `tasks.binaryphile.github.io`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/binaryphile) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
