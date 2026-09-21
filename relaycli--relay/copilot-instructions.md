## 012-key-concepts

> We need to keep the number of user-facing concepts to a minimum. Those concepts need to be persistent and in service of the product. Their core functions are:


# Key concepts

We need to keep the number of user-facing concepts to a minimum. Those concepts need to be persistent and in service of the product. Their core functions are:
- authentication and account management
- core email interactions
- make it easy & appealing for developers to build apps

Here are our main concepts:
- email server: a VM that can run email interactions through a specific set of possible protocols
- email provider: organization hosting email servers
- email account: a domain-bound address to send and receive emails
- email protocols: convention for interactions with an email account
- email client: a GUI for interactions with email accounts
- credentials: user-defined secrets that enable a connection to an email account
- message: a single e-mail (body + headers), defined by its Message-ID or IMAP UID.
- thread: all messages sharing References/In-Reply-To headers

---
> Source: [relaycli/relay](https://github.com/relaycli/relay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
