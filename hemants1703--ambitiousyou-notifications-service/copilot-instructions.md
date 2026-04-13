## ambitiousyou-notifications-service

> This Node.js backend serves as the notifications service for AmbitiousYou. The main application is separate from this project but we just need to focus on the notifications service in this project.

# AmbitiousYou Notifications Service

This Node.js backend serves as the notifications service for AmbitiousYou. The main application is separate from this project but we just need to focus on the notifications service in this project.

## AmbitiousYou Overview
AmbitiousYou is a goal-tracking application. It's not another todo application rather it is a todo application on steroids! AmbitiousYou let's you become a superhuman by letting you focus on your life goals in a prioritised approach!

Ambitions are time-bound goals of users that they can accomplish by breaking it down into smaller more manageable forms such as Tasks or Milestones (only one form per ambition). Both of those forms are time-bound as well within the time frame of the Ambition they are in.

---
There are mainly Two Modes of Notifications
1. **Email**
2. **Push Notification for PWAs**
---

About Notifications
- Notifications are triggered based on _two situations_ - **User-triggered & Scheduled**
- **User-triggered**: explicitly sent when user makes some higher level changes to their account
- **Scheduled**: for any ambitions or tasks the user creates in the application, a notification is scheduled for them.

### User Triggered Notifications

When user makes any changes in the application that are serious changes in the applications such as reset or change password, they get these notifications through both the modes.

### Scheduled Notifications

1. **Dynamic** - When user makes any ambition and tasks/milestones in the application, we need to schedule notifications for the user so as to remind them about deadlines for each of their ambitions and tasks/milestones.

2. **Fixed** - Apart from reminder notifications we are also going to send them some daily scheduled notifications only in PWAs such as motivational quotes and questioning them about their progress towards their ambition!

### Tech Stack

0. pnpm - package manager
1. Node.js - Server Side Runtime
2. Express.js - Backend framework
3. Azure Communication Service for sending emails (prod), Using Gmail for dev purposes

We try to keep the tech stack as simple and lesser heavy as possible to keep the application super sleek, simple and performant

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hemants1703) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
