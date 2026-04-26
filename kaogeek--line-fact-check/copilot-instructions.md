## line-fact-check

> Rules and context for this project


# Iron rules
- Always use Context7
- Always run tests after edits
- Never remove tests
- Always explain you thoughts, step-by-step, before suggesting edits
- Ask us first when you need to make edits in unopened files
- Ask us very explicitly that you are installing new tools
- Keep changes localized and minimum
- Keep variable names short, DO NOT name something like fooDataUpdaterAdapter
- If your decisions used reference to links from forums or documentations, give us the URLs (without previewing it)

# About this project
line-fact-check is an free and open source software app for submitting and flagging fake news, especially Thai news.
Our developers are volunteers, and come from across Thailand.

It is a monorepo, and each directory at root can be treated as separate projects.
For example, [factcheck](/factcheck/) is Go monolith HTTP API server, while [backoffice-webapp](/backoffice-webapp/) is the web frontend.

# Nix use
The monorepo is also a Nix flake. This is done to ensure stuff builds at all times.

While Nix flake is fully embraced on CICD pipelines, although some of our devs might not want to install Nix. This is why we need to provide a way for our devs to interact with our Nix flake unless we want to maintain multiple version of Dockerfiles for the same program (one for Nix dev, another for non-Nix), which we will not do.

Because we can assume our devs do not have Nix installed, you cannot by default suggest that they run Nix commands. 

If we really need Nix to debug anything in chat session, use `docker run -it` instead. Say you need:

```sh
nix build .#foo
```

You can wrap the Nix command inside `docker run -it nixos/nix`:

```sh
docker run -it nixos/nix nix build .#foo
```

# CICD
The Git repository is hosted on GitHub and we use GitHub Actions for CICD.
We try to use Nix everywhere when we could, for example, in [.github/workflows/release.yaml](/.github/workflows/release.yaml)

# Jargons (in code and chat)
- A Go integration test is called "it-test", and Go integration tests end with `_it_test.go` and tag `integration_test`.

# Business logic
## Business users
As a fact-checker platform, we have 2 types of "users"

- Users

    Users can access our app via chat apps like LINE or other entrypoints, and the web portal.
    They can submit their messages any time on any entrypoints, and this will be reflected in
    the user_message metadata that records the whole submission.

- Admins

    Admins access our app via web portal and are the "human verifiers" of our system.
    Admin creates "topics", reviews and approves "topic-message assignment", as well as resolve topics with result.

## Business entities

These are our main "data types"

- Messages (currently table `messages_v2`, previously `messages`)

    Messages are posted by users, and represent a "statement" to be fact-checked.
    Messages fall under topics, which group multiple messages together.

    Messages could enter our system via a user client (chat),
    or via the admin web UI.

- Message groups (currently table `message_groups`)

    Message groups group messages together based on "similarity".
    Currently, the similarity engine is just a SHA-1 hash match,
    i.e. messages with identical texts belong to the same group.

    On message submission, the system checks if we already have
    a group with "similar" statement. If there's one, we assign
    the new message to the existing group. If there's none, the
    system will create a new message group for the message automatically.

    Assignment of message to a message group could be automatic
    (i.e. when the system does this), or manually by admins

- Topics

    Topics are high-level groupings of messages/message groups.
    Currently, all assignment of a message to a topic is manual,
    and must be done by human admins.

    A topic could be "answered" by admins. Once answered, the topic
    answer is assumed to have resolved all messages of the topic.

- Answers

    Answer is as direct as it is - it's an answer. It resolves a topic
    with answer text. A topic could be answered many times, and we intend
    to model answers as append-only logs.

## Business flow

As a fact-checker platform, our business logic can be classified into 3 steps:

1. "Users" post/submit some news or statement into our system

    When this happens, 2 things kinda happen:

    - `messages` created

    - `messages` is assigned its `message_groups`

        - If we already have a group with "similar" text

            We assign the new message to this group.
            If the group was already assigned topic,
            this new message will also automatically
            inherit its group's topic.

        - If we have no similar groups

            We create a new group with our message as base,
            and assign the new message to this new group.

            When this happens, the message group's topic
            stays unassigned

2. "Admins" assign topic to new message submissions

    Human admins will come in and review new submissions
    from step 1.

    Should the admins decide to verify new submissions,
    they will create a new topic for the group, or assigned
    it to an existing topic.

3. "Admins" add answers to topic

    Once human admins got their answer for a particular topic,
    they add the answers to the topic, toggling the topic status
    to resolved.

    Another flow is "update answer" flow, where admins want to
    update the topic's answer.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kaogeek) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
