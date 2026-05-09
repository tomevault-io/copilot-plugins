## better-auth-orgs

> Full docs: https://www.better-auth.com/docs/plugins/organization

# Better Auth — Organization Plugin

Full docs: https://www.better-auth.com/docs/plugins/organization

## Setup

```ts
// server
import { organization } from "better-auth/plugins"
export const auth = betterAuth({ plugins: [organization()] })

// client
import { organizationClient } from "better-auth/client/plugins"
export const authClient = createAuthClient({ plugins: [organizationClient()] })
```

## Key Client Methods

```ts
// Org management
authClient.organization.create({ name, slug })
authClient.organization.update({ data: { name, slug, logo } })
authClient.organization.delete({ organizationId })
authClient.organization.setActive({ organizationId })
authClient.useActiveOrganization()
authClient.useListOrganizations()

// Members
authClient.organization.inviteMember({ email, role, organizationId })
authClient.organization.acceptInvitation({ invitationId })
authClient.organization.removeMember({ memberIdOrEmail, organizationId })
authClient.organization.updateMemberRole({ role, memberId, organizationId })
authClient.organization.leave({ organizationId })

// Permissions
authClient.organization.hasPermission({ permissions: { resource: ["action"] } })
```

## Key Server Methods

```ts
auth.api.createOrganization({ body: { name, slug }, headers })
auth.api.getFullOrganization({ query: { organizationId }, headers })
auth.api.createInvitation({ body: { email, role, organizationId } })
auth.api.addMember({ body: { userId, role, organizationId } })
auth.api.hasPermission({ headers, body: { permissions: { resource: ["action"] } } })
```

## Default Roles

| Role | Permissions |
|------|-------------|
| `owner` | Full control |
| `admin` | Full control except delete org / change owner |
| `member` | Read only on org/member/invitation resources |

## Custom Permissions

```ts
import { createAccessControl } from "better-auth/plugins/access"

const statement = { project: ["create", "update", "delete"] } as const
const ac = createAccessControl(statement)
const member = ac.newRole({ project: ["create"] })
const admin = ac.newRole({ project: ["create", "update"] })

// Pass to both server and client plugins: { ac, roles: { member, admin, owner } }
```

## Lifecycle Hooks

```ts
organization({
  organizationHooks: {
    beforeCreateOrganization: async ({ organization, user }) => ({ data: { ...organization } }),
    afterCreateOrganization: async ({ organization, member, user }) => { /* ... */ },
    beforeAddMember: async ({ member, user, organization }) => { /* throw to block */ },
    afterAddMember: async ({ member, user, organization }) => { /* ... */ },
    beforeCreateInvitation: async ({ invitation, inviter, organization }) => ({ data: { ...invitation } }),
    afterAcceptInvitation: async ({ invitation, member, user, organization }) => { /* ... */ },
  },
  sendInvitationEmail: async (data) => {
    const link = `https://yourapp.com/accept-invitation/${data.id}`
    // send email with link
  },
})
```

## Teams (optional)

```ts
organization({ teams: { enabled: true, maximumTeams: 10 } })
organizationClient({ teams: { enabled: true } })

// Methods: createTeam, listTeams, updateTeam, removeTeam, addTeamMember, removeTeamMember
```

---
> Source: [inboundemail/inbound](https://github.com/inboundemail/inbound) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
