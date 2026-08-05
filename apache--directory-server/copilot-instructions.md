## directory-server

> SPDX-License-Identifier: Apache-2.0

<!--
SPDX-License-Identifier: Apache-2.0

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

# Agent Guide for Apache Directory Server (ApacheDS)

This file is read by automated agents (security scanners, code analyzers,
AI assistants) operating on this repository.

## Security

Security model: [SECURITY.md](./SECURITY.md) -> [THREAT_MODEL.md](./THREAT_MODEL.md)

This repository hosts the **Apache Directory umbrella threat model**, which
covers the shared identity-stack trust model across the project's components —
LDAP bind/ACI (ApacheDS), the LDAP API codec, Kerberos KDC (Kerby), Fortress
RBAC, SCIM (SCIMple), and MVCC storage (Mavibot) — with per-domain addenda.
Agents should consult `SECURITY.md` and the linked `THREAT_MODEL.md` before
reporting issues.

---
> Source: [apache/directory-server](https://github.com/apache/directory-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
