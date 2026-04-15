## helpscout-to-csv

> List and filter conversations. Use the thread resource link inside the Conversation entity to load conversation threads.

# List Conversations

List and filter conversations. Use the thread resource link inside the Conversation entity to load conversation threads.

## Request

```
GET /v2/conversations HTTP/1.1
Authorization: Bearer oauth_token
```

Request parameters can be used to filter conversations. All parameters are joined by the AND operator.

If you want closed conversations in a specific inbox, call:

```bash
curl https://api.helpscout.net/v2/conversations?mailbox=123&status=closed
```

You can also exclude data from your search results using the NOT operator, calling:

```bash
curl https://api.helpscout.net/v2/conversations?query=(NOT email:"email@domain.com")
```

By default only active conversations are listed and they are sorted by `createdAt` (from newest to oldest). This is equivalent to `?status=active&sortField=createdAt&sortOrder=desc`.

If you want to list all conversations, use `?status=all`

**Note:** If using the `embed=threads` parameter with this call, you will see truncated chat threads. This is by design. To view Beacon chat threads in full, call the List Threads endpoint for that conversation instead.

## URL Parameters

| Parameter | Type | Examples | Description |
|---|---|---|---|
| `embed` | enumeration | `embed=threads` | Allows embedding/loading of sub-entities, allowed values are: `threads` |
| `mailbox` | number | `mailbox=123`<br>`mailbox=123,567` | Filters conversations from a specific inbox id. Use comma separated values for more inboxes |
| `folder` | number | `folder=993` | Filters conversations from a specific folder id |
| `status` | enumeration | `status=active` | Filter conversation by status (defaults to active):<br>- `active`<br>- `all`<br>- `closed`<br>- `open`<br>- `pending`<br>- `spam` |
| `tag` | string | `tag=red`<br>`tag=red,blue` | Filter conversation by tags. Use comma separated values for more tags |
| `assigned_to` | number | `assigned_to=1771` | Filters conversations by assignee id |
| `modifiedSince` | date | `modifiedSince=2018-05-04T12:00:03Z` | Returns only conversations that have been modified since this date |
| `number` | number | `number=123` | Look up a conversation by conversation number |
| `sortField` | enumeration | `sortField=createdAt` | Sorts the result by specified field. Allowed values:<br>- `createdAt`<br>- `customerEmail`<br>- `customerName`<br>- `mailboxid`<br>- `modifiedAt`<br>- `number`<br>- `score`<br>- `status`<br>- `subject`<br>- `waitingSince` |
| `sortOrder` | enumeration | `sortOrder=desc` | Sort in ascending (`asc`) or descending (`desc`) order, default is `desc` |
| `query` | special | `query=(number:123)` | Advanced query. If you use other filtering request parameters, they will be combined with `AND` operator. For example `?mailbox=567&query=tag:fire` effectively means `mailbox=567 AND tag=fire`. |
| `page` | number | `page=1` | The page number |
| `customFieldsByIds` | special | `customFieldsByIds=123:blue`<br>`customFieldsByIds=123:blue,234:99` | Filters by custom field values. Must be combined with `mailbox` parameter as well. Expected format is `id:value,id:value`. See Custom Fields Search for details |

## Custom Fields Search

### customFieldsByIds

This is a fairly low level search - the values are expected in a `fieldId:value` format and you can either:

- use multiple `customFieldsByIds` params like this: `?customFieldsByIds=123:green&customFieldsByIds=234:blue`
- or join the values yourself: `?customFieldsByIds=123:green,234:blue`

**Rules:**

- the filtering is done via an exact match for all field types with the exception of single line text field - for example a custom field with a number type and ID `123` with the following filter of `123:1000` will only match if the value is precisely `1000`
- single line text fields are searched in full text search manner
- multiple filters use `AND` operator - all custom fields filters must match
- for Dropdown custom field type the value is the dropdown option ID (you can use Inbox Fields endpoint to fetch all IDs)
- Multiline custom fields are not supported

## Query

You can create complex search queries by combining operators:

```
?query=(assigned:"spock" AND (customerIds: 123 OR customerIds: 567))
```

Some characters (particularly `+` signs in email addresses) are reserved and will need to be encoded. For more info, see [this article](https://www.w3schools.com/tags/ref_urlencode.asp).

### Subject

| Example | What it does |
|---|---|
| `query=(subject:"rainbow")` | Searches for conversation subjects that contain the word "rainbow" |
| `query=(subject:"have a question")` | Searches for conversation subjects that contain the phrase "have a question" (not case sensitive) |
| `query=(subject:"coffee" OR subject:"tea")` | Searches for conversations with a subject that contain the word "coffee" or the word "tea" |
| `query=(subject:"coffee tea")` | Searches for conversation subjects that contain the word "coffee" and the word "tea" in no particular order |

### Tags

| Example | What it does |
|---|---|
| `query=(tag:"carrots")` | Searches for conversations that have the tag "carrots" |
| `query=(tag:"dogs love carrots")` | Searches for conversations that have the tag "dogs love carrots" |
| `query=(tag:"dogs" OR tag:"carrots")` | Searches for conversations that are tagged with either "dogs" or "carrots" |

### Inbox

| Example | What it does |
|---|---|
| `query=(mailbox:"Orders")` | Searches the "Orders" inbox by exact match |
| `query=(mailboxid:100)` | Searches the inbox with an id of 100 |

### Modified At

| Example | What it does |
|---|---|
| `query=(modifiedAt:[2014-02-27T00:00:00Z TO *])` | Searches conversations that have been modified since 2014-02-27 |
| `query=(modifiedAt:[2014-02-27T00:00:00Z TO 2014-02-27T23:59:59Z])` | Searches conversations that were modified on 2014-02-27 |
| `query=(modifiedAt:[NOW-1HOUR TO *])` | Searches conversations that were modified in the last hour |
| `query=(modifiedAt:[2013-01-01T00:00:00Z TO 2013-12-31T23:59:59Z])` | Searches conversations that were modified in 2013 |
| `query=(modifiedAt:[2014 TO 2014-12-01])` | Searches conversations that were modified from the start of 2014 to the beginning of December, 2014 |
| `query=(modifiedAt:2000-11)` | Searches conversations that were modified in November, 2000 |

### Created At

| Example | What it does |
|---|---|
| `query=(createdAt:[2014-02-27T00:00:00Z TO *])` | Searches conversations that have been created since 2014-02-27 |
| `query=(createdAt:[2014-02-27T00:00:00Z TO 2014-02-27T23:59:59Z])` | Searches conversations that were created on 2014-02-27 |
| `query=(createdAt:[NOW-1HOUR TO *])` | Searches conversations that were created in the last hour |
| `query=(createdAt:[2013-01-01T00:00:00Z/YEAR TO 2014-01-01T00:00:00Z/YEAR])` | Searches conversations that were created in 2013 |

### Email

| Example | What it does |
|---|---|
| `query=(email:"john@appleseed.com")` | Searches conversations where the mail `john@appleseed.com` is either in to, cc, bcc fields or it's one of the emails in a customer profile. |
| `query=(email:(john@appleseed.com OR vbear@mywork.com))` | Searches conversations using the fields mentioned above, but returns results for multiple emails or customers |

### Body

| Example | What it does |
|---|---|
| `query=(body:"what a beautiful day")` | Searches conversation threads for the phrase, "what a beautiful day" |
| `query=(body:"what a day" OR body:"how about tomorrow")` | Searches conversation threads for the phrases "what a day" or "how about tomorrow" |

### Other

| Example | What it does |
|---|---|
| `query=(customerIds:100)` | Returns conversations (across all inboxes) belonging to the given customer |
| `query=(number:1234)` | Searches for conversation #1234 |
| `query=(id:123456)` | Searches for the unique conversation ID |
| `query=(assigned:"spock")` | Searches for conversations assigned to Spock |
| `query=(attachments:true)` | Searches for conversations with attachments |
| `query=(assigned:"Unassigned")` | Searches for unassigned conversations |
| `query=(replyUserIds:(23 OR 45)` | Searches for conversations with replies from specific Help Scout users (by ID) |

## Response

```http
HTTP/1.1 200 OK
Content-Type: application/hal+json
```

```json
{
  "_embedded" : {
    "conversations" : [ {
      "id" : 10,
      "number" : 12,
      "threads" : 2,
      "type" : "email",
      "folderId" : 11,
      "status" : "closed",
      "state" : "published",
      "subject" : "Help",
      "preview" : "Preview",
      "mailboxId" : 13,
      "assignee" : {
        "id" : 99,
        "type" : "user",
        "first" : "Mr",
        "last" : "Robot",
        "email" : "none@nowhere.com"
      },
      "createdBy" : {
        "id" : 12,
        "type" : "customer",
        "email" : "bear@acme.com"
      },
      "createdAt" : "2012-03-15T22:46:22Z",
      "closedBy" : 14,
      "closedByUser" : {
        "id" : 14,
        "type" : "user",
        "first" : "Clo",
        "last" : "Ser",
        "photoUrl" : "pic.jpg",
        "email" : "closer@closers.com"
      },
      "closedAt" : "2012-03-16T14:07:23Z",
      "userUpdatedAt" : "2012-03-16T14:07:23Z",
      "customerWaitingSince" : {
        "time" : "2012-07-24T20:18:33Z",
        "friendly" : "20 hours ago"
      },
      "source" : {
        "type" : "email",
        "via" : "customer"
      },
      "tags" : [ {
        "id" : 9150,
        "color" : "#929499",
        "tag" : "vip"
      } ],
      "cc" : [ "bear@normal.com" ],
      "bcc" : [ "bear@secret.com" ],
      "primaryCustomer" : {
        "id" : 238604,
        "type" : "customer",
        "first" : "Rob",
        "last" : "Robertovic",
        "email" : "rob@acme.com"
      },
      "snooze" : {
        "snoozedBy" : 4,
        "snoozedUntil" : "2024-06-03T12:00:00Z",
        "unsnoozeOnCustomerReply" : true
      },
      "nextEvent" : {
        "time" : "2024-06-03T12:00:00Z",
        "eventType" : "snooze",
        "userId" : 4,
        "cancelOnCustomerReply" : true
      },
      "customFields" : [ {
        "id" : 8,
        "name" : "Account Type",
        "value" : "8518",
        "text" : "Free"
      }, {
        "id" : 6688,
        "name" : "Account Status",
        "value" : "33077",
        "text" : "Trial"
      } ],
      "_embedded" : {
        "threads" : [ ]
      },
      "_links" : {
        "self" : {
          "href" : "..."
        },
        "mailbox" : {
          "href" : "..."
        },
        "primaryCustomer" : {
          "href" : "..."
        },
        "createdByCustomer" : {
          "href" : "..."
        },
        "closedBy" : {
          "href" : "..."
        },
        "threads" : {
          "href" : "..."
        },
        "assignee" : {
          "href" : "..."
        },
        "web" : {
          "href" : "..."
        }
      }
    } ]
  },
  "_links" : {
    "self" : {
      "href" : "..."
    },
    "first" : {
      "href" : "..."
    },
    "last" : {
      "href" : "..."
    },
    "page" : {
      "href" : "...",
      "templated" : true
    }
  },
  "page" : {
    "size" : 25,
    "totalElements" : 0,
    "totalPages" : 0,
    "number" : 0
  }
}
```

## Response Fields

| Path | Type | Description |
|---|---|---|
| `_embedded.conversations` | Array | Array of conversation objects |

# List Threads

Threads are by default sorted by `createdAt` (from newest to oldest).

## Request

```
GET /v2/conversations/123/threads HTTP/1.1
Authorization: Bearer oauth_token
```

## Response

```http
HTTP/1.1 200 OK
Content-Type: application/hal+json
```

```json
{
  "_embedded" : {
    "threads" : [ {
      "id" : 1234,
      "type" : "customer",
      "status" : "active",
      "state" : "published",
      "action" : {
        "type" : "manual-workflow",
        "text" : "You ran the Assign to Spam workflow",
        "associatedEntities" : { }
      },
      "body" : "this is what i have to say: I love your service.",
      "source" : {
        "type" : "email",
        "via" : "customer"
      },
      "customer" : {
        "id" : 29418,
        "first" : "Vernon",
        "last" : "Bear",
        "photoUrl" : "http://whatever-path-to-url/image.jpg",
        "email" : "vbear@mywork.com"
      },
      "createdBy" : {
        "id" : 29418,
        "type" : "customer",
        "first" : "Vernon",
        "last" : "Bear",
        "photoUrl" : "http://whatever-path-to-url/image.jpg",
        "email" : "vbear@mywork.com"
      },
      "assignedTo" : {
        "id" : 1234,
        "type" : "team",
        "first" : "Jack",
        "last" : "Sprout",
        "email" : "jack.sprout@gmail.com"
      },
      "savedReplyId" : 10,
      "to" : [ "to1@somewhere.com", "to2@somewhere.com" ],
      "cc" : [ "cc1@somewhere.com", "cc2@somewhere.com" ],
      "bcc" : [ "bcc1@somewhere.com", "bcc2@somewhere.com" ],
      "createdAt" : "2015-06-05T20:18:33Z",
      "openedAt" : "2015-06-07T10:01:25Z",
      "rating" : {
        "customerId" : 1234,
        "rating" : "great",
        "comments" : "Great job!"
      },
      "scheduled" : {
        "scheduledBy" : 4,
        "createdAt" : "2024-06-10T10:20:30Z",
        "scheduledFor" : "2034-06-10T10:20:30Z",
        "unscheduleOnCustomerReply" : false
      },
      "_embedded" : {
        "attachments" : [ {
          "id" : 1234,
          "filename" : "photo1.jpg",
          "mimeType" : "image/jpeg",
          "width" : 600,
          "height" : 800,
          "size" : 20191,
          "state" : "valid",
          "_links" : {
            "self" : {
              "href" : "..."
            },
            "data" : {
              "href" : "..."
            },
            "web" : {
              "href" : "..."
            }
          }
        } ]
      },
      "_links" : {
        "assignedTo" : {
          "href" : "..."
        },
        "createdByCustomer" : {
          "href" : "..."
        },
        "customer" : {
          "href" : "..."
        }
      }
    } ]
  },
  "_links" : {
    "self" : {
      "href" : "..."
    },
    "first" : {
      "href" : "..."
    },
    "last" : {
      "href" : "..."
    },
    "page" : {
      "href" : "...",
      "templated" : true
    }
  },
  "page" : {
    "size" : 25,
    "totalElements" : 0,
    "totalPages" : 0,
    "number" : 0
  }
}
```

## Moved or Merged Conversations

When a conversation is merged with another conversation, it is no longer accessible using the old ID. Get Conversation endpoint will return a `HTTP 301 Moved Permanently` status code and the response will contain a `Location` header with the URI of the new conversation.

This request will return a `HTTP 404 Not Found` in such case. If you suspect the conversation you are trying to change was merged, call the Get Conversation endpoint to get a new conversation location.

## Response Fields

| Path | Type | Description |
|---|---|---|
| `.id` | Number | Unique identifier |
| `assignedTo` | Object | The user or team assigned to the conversation |
| `status` | String | Thread status, accepted values:<br>- `active`<br>- `closed`<br>- `nochange`<br>- `pending`<br>- `spam` |
| `state` | String | Thread state, accepted values:<br>- `draft`<br>- `hidden`<br>- `published`<br>- `review` |
| `.action.type` | String | Type of action performed |
| `.action.text` | String | Description of the action (for `lineitem` only) |
| `.action.associatedEntities` | Object | Associated entities with the action |
| `body` | String | The body or text of the thread |
| `source.type` | String | Source type, accepted values:<br>`api`, `beacon`, `channel`, `chat`, `consumer`, `coreapi`, `csv`, `cvs`, `desk`, `docs`, `email`, `emailfwd`, `heymarket`, `internal`, `jira`, `manual`, `mobile`, `notification`, `orchestration`, `support`, `unknown`, `uservoice`, `web`, `workflows`, `zendesk` |
| `source.via` | String | Accepted values:<br>- `user`<br>- `customer` |
| `customer` | Object | Customer information |
| `createdBy` | Object | Creator information. The `type` property will specify whether it was created by a user or a customer |
| `savedReplyId` | Number | ID of the saved reply used |
| `type` | String | Thread type, accepted values:<br>- `beaconchat`<br>- `chat`<br>- `customer`<br>- `forwardchild`<br>- `forwardparent`<br>- `lineitem`<br>- `message`<br>- `note`<br>- `phone` |
| `to` | Array | Email addresses in the `to:` field |
| `cc` | Array | Email addresses in the `cc:` field |
| `bcc` | Array | Email addresses in the `bcc:` field |
| `createdAt` | String | When the thread was created (ISO 8601 date string) |
| `openedAt` | String | When the thread was opened (only present for `message` type) |
| `linkedConversationId` | String | Linked conversation ID |
| `rating` | Object | Rating information (only present for `message` type) |
| `scheduled` | Object | Scheduled send information |
| `_embedded.attachments` | Array | Array of attachments |

## Rating Response Fields

| Path | Type | Description |
|---|---|---|
| `customerId` | Number | The ID of the customer who provided the rating |
| `rating` | String | Rating enumeration, one of: `great`, `not_good`, `okay` |
| `comments` | String | Optional comment left by the customer |

## Schedule Response Fields

| Path | Type | Description |
|---|---|---|
| `scheduledBy` | Number | User ID that scheduled the thread |
| `sendAsId` | Number | Original draft author that will be used for sending the scheduled thread |
| `createdAt` | String | When the thread was scheduled (ISO 8601 date string) |
| `scheduledFor` | String | When the thread will be sent (ISO 8601 date string) |
| `unscheduleOnCustomerReply` | Boolean | Whether a new customer reply should automatically unschedule the thread |

## Attachment Response Fields

| Path | Type | Description |
|---|---|---|
| `[].id` | Number | Unique identifier |
| `[].filename` | String | File name |
| `[].mimeType` | String | Mime type |
| `[].width` | Number | Width in pixels - not guaranteed to be present |
| `[].height` | Number | Height in pixels - not guaranteed to be present |
| `[].size` | Number | Size in bytes - not guaranteed to be present |
| `[].state` | String | Accepted values:<br>- `valid`<br>- `virus` |

## Resource Links

| Relation | Description |
|---|---|
| `web` | Link to the attachment in the web application for easier download |
| `data` | Link to Get Data endpoint |

## Thread Type

`lineitem` represents a change of state on the conversation. This could include, but not limited to, the conversation was assigned, the status changed, the conversation was moved from one inbox to another, etc. A line item won't have a body, to/cc/bcc lists, or attachments.

When a conversation is forwarded, a new conversation is created to represent the forwarded conversation. `forwardparent` is the type set on the thread of the original conversation that initiated the forward event. `forwardchild` is the type set on the first thread of the new forwarded conversation.

## Thread State

A state of `underreview` means the thread has been stopped by Collision Detection and is waiting to be confirmed (or discarded) by the person that created the thread.

A state of `hidden` means the thread was hidden (or removed) from customer-facing emails.

## Thread Status

Thread status is only updated when there is a status change. Otherwise, the status will be set to `nochange`.

## Attachment State

A state of `valid` means the attachment is free from malicious content.

A state of `virus` means malicious content has been detected for the attachment.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DIY-Recording-Equipment)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DIY-Recording-Equipment)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
