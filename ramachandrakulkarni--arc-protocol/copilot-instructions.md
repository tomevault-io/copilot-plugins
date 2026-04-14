## arc-protocol

> Validates that receipt_id is a valid ARC ID format.

# ARC Protocol — Claude Code Build Instructions
## Agent Receipt & Certification: Proof of Execution + Observability + Reversibility

---

## MISSION

You are building **ARC (Agent Receipt & Certification)** — a protocol and Python SDK that solves
three interconnected problems in AI agent infrastructure that currently have no solution:

1. **Proof of execution** — AI agents self-report their actions with no independent verification.
   When an agent says "I deleted the file," there is no cryptographic proof it actually happened,
   no proof of what existed before, and no way to detect fabricated results.

2. **Observability** — No standard schema exists for agent reasoning traces. Existing tools
   (LangSmith, LangFuse, Arize Phoenix) capture execution metadata but not before-state, not
   causal attribution, and with no tamper-evidence. None satisfy EU AI Act Article 12.

3. **Reversibility** — No protocol exists for undoing agent actions. The Replit incident (agent
   deleted a production database, then fabricated 4,000 fake users to cover it, then told the user
   rollback was impossible — when it was actually possible) is the canonical failure mode.

**The core insight:** These three problems are the same problem at three phases of execution.
The solution is a single primitive: the **Action Receipt** — a two-phase, cryptographically
signed record that captures before-state (enabling rollback), attests execution (enabling proof),
and chains into an append-only transparency log (enabling independent verification).

**Critical architectural decision:** The tool provider signs the receipt, NOT the agent.
This removes the agent from the verification trust chain. An agent that fabricates results
produces a receipt whose signature fails verification. An agent that lies about rollback
impossibility contradicts the signed inverse operation in the receipt.

---

## CONTEXT: WHY THIS EXISTS

The existing protocol landscape fails in documented, specific ways:

- **MCP (Model Context Protocol):** JSON-RPC 2.0. No timestamps, no caller identity, no
  execution duration, no cryptographic signing, no audit metadata. The `tools/call` response
  returns only `content` (plain text) and `isError`. That text is indistinguishable from a
  hallucination at the protocol level.

- **OpenAI function calling:** `tool_calls` array with `id`, `name`, `arguments`. Tool result
  returned as `role: "tool"` message. No signature, no before-state, no inverse op.

- **Anthropic tool_use:** `tool_use` content blocks with `id`, `name`, `input`. Paired with
  `tool_result` blocks. Same gaps as above.

- **All agent frameworks (LangChain, CrewAI, AutoGen, OpenAI Agents SDK):** Operate on implicit
  trust. The agent is the sole source of truth about its own actions. No independent witness.

- **All observability tools (LangSmith, LangFuse, Phoenix, Helicone):** Capture post-hoc
  execution traces. No cryptographic integrity. No before-state. Mutable and self-reported.
  None satisfy EU AI Act Article 12 tamper-evidence requirements.

**Regulatory deadline:** EU AI Act Article 12 logging requirements enforce on **August 2, 2026**.
Fines up to €35M or 7% of global revenue. Current infrastructure cannot comply.

---

## WHAT YOU ARE BUILDING

### Components

```
arc-protocol/
├── CLAUDE.md                          ← this file
├── README.md                          ← user-facing docs
├── SPEC.md                            ← full protocol specification
│
├── schemas/                           ← JSON Schema definitions (the protocol)
│   ├── action-receipt.schema.json     ← root receipt document
│   ├── intent.schema.json             ← phase 1: what agent declared before acting
│   ├── before-state.schema.json       ← phase 1: snapshot of resource pre-action
│   ├── provider-attestation.schema.json ← phase 2: tool provider's signed block
│   ├── inverse-operation.schema.json  ← phase 2: rollback specification
│   ├── log-entry.schema.json          ← transparency log entry
│   └── provider-registration.schema.json ← key registry entry
│
├── src/
│   └── arc/
│       ├── __init__.py                ← public API exports
│       ├── ids.py                     ← receipt/intent/snapshot ID generation (ULID-based)
│       ├── signing.py                 ← Ed25519 keypair generation, sign, verify
│       ├── snapshot.py                ← before-state capture for filesystem, dict, HTTP
│       ├── receipt.py                 ← Phase1Builder, Phase2Builder, full receipt assembly
│       ├── merkle.py                  ← Merkle tree for inclusion proofs
│       ├── log_client.py              ← HTTP client for transparency log API
│       ├── decorator.py               ← @signed_tool decorator (the main user-facing API)
│       └── proxy.py                   ← HTTP proxy wrapper for zero-code integration
│
├── arc_log/                           ← standalone transparency log server
│   ├── __init__.py
│   ├── server.py                      ← FastAPI app with 3 endpoints
│   ├── storage.py                     ← SQLite-backed append-only log storage
│   └── merkle.py                      ← server-side Merkle tree with persistence
│
├── tests/
│   ├── conftest.py                    ← pytest fixtures (keypairs, mock log, temp dirs)
│   ├── test_signing.py                ← Ed25519 sign/verify, tamper detection
│   ├── test_receipt.py                ← receipt assembly, schema validation
│   ├── test_merkle.py                 ← Merkle tree correctness, inclusion proofs
│   ├── test_log.py                    ← log server API tests
│   ├── test_decorator.py              ← @signed_tool integration tests
│   └── test_e2e.py                    ← full end-to-end: intent→execute→receipt→verify→rollback
│
├── demo/
│   ├── README.md                      ← how to run each demo
│   ├── demo_basic.py                  ← minimal: sign one tool call, verify the receipt
│   ├── demo_disaster.py               ← Replit scenario: delete files, detect fabrication, rollback
│   └── demo_verify.py                 ← third-party verification from receipt_id alone
│
├── pyproject.toml
└── Makefile
```

---

## SCHEMAS (THE PROTOCOL)

These are the canonical definitions. Implement them exactly. Every field matters.

### Schema 1: `action-receipt.schema.json` — Root document

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://arc-protocol.org/v1/schemas/action-receipt.schema.json",
  "title": "ActionReceipt",
  "description": "ARC Agent Action Receipt — cryptographically signed record of an agent action",
  "type": "object",
  "required": ["arc_version", "receipt_id", "phase_1", "phase_2"],
  "additionalProperties": false,
  "properties": {
    "arc_version": {
      "type": "string",
      "const": "1.0",
      "description": "Protocol version. Bump minor for backward-compatible changes, major for breaking."
    },
    "receipt_id": {
      "type": "string",
      "pattern": "^arc_[0-9A-Z]{26}$",
      "description": "Globally unique receipt ID. Format: arc_ prefix + 26-char ULID (uppercase)."
    },
    "phase_1": {
      "$ref": "intent.schema.json#/$defs/Phase1",
      "description": "Pre-action declaration. Committed to log BEFORE execution."
    },
    "phase_2": {
      "$ref": "provider-attestation.schema.json#/$defs/Phase2",
      "description": "Post-execution attestation. Committed to log AFTER execution."
    }
  }
}
```

### Schema 2: `intent.schema.json` — Phase 1 (pre-action)

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://arc-protocol.org/v1/schemas/intent.schema.json",
  "title": "IntentDeclaration",
  "$defs": {
    "Phase1": {
      "type": "object",
      "required": ["intent", "before_state", "log_commitment"],
      "additionalProperties": false,
      "properties": {
        "intent": { "$ref": "#/$defs/Intent" },
        "before_state": { "$ref": "before-state.schema.json#/$defs/BeforeState" },
        "log_commitment": { "$ref": "#/$defs/LogCommitment" }
      }
    },
    "Intent": {
      "type": "object",
      "required": [
        "intent_id", "tool_name", "arguments", "declared_by",
        "reasoning_commitment", "declared_at"
      ],
      "additionalProperties": false,
      "properties": {
        "intent_id": {
          "type": "string",
          "pattern": "^intent_[0-9A-Z]{26}$",
          "description": "Unique ID for this intent declaration."
        },
        "tool_name": {
          "type": "string",
          "description": "Exact name of the tool being called, as registered by the provider."
        },
        "tool_provider_id": {
          "type": "string",
          "description": "DID or URI identifying the tool provider. Optional at Phase 1; provider fills at Phase 2."
        },
        "arguments": {
          "type": "object",
          "description": "The exact arguments the agent is passing to the tool."
        },
        "declared_by": {
          "type": "object",
          "required": ["agent_id", "model_version", "session_id"],
          "properties": {
            "agent_id": { "type": "string", "description": "Identifier for the agent (e.g. 'claude-sonnet-4-6')." },
            "model_version": { "type": "string", "description": "Full model version string." },
            "session_id": { "type": "string", "description": "Current session or conversation ID." }
          }
        },
        "on_behalf_of": {
          "type": "string",
          "description": "User or system that authorized this action (e.g. 'user:ram@ri.com')."
        },
        "authorization": {
          "type": "object",
          "properties": {
            "scope": {
              "type": "array",
              "items": { "type": "string" },
              "description": "Granted permission scopes (e.g. ['files:write', 'files:delete'])."
            },
            "granted_by": { "type": "string" },
            "delegation_chain": {
              "type": "array",
              "items": { "type": "string" },
              "description": "Ordered list of agent_ids in delegation chain, outermost first."
            },
            "parent_receipt_id": {
              "type": "string",
              "pattern": "^arc_[0-9A-Z]{26}$",
              "description": "Receipt ID of the parent action that spawned this one (multi-agent)."
            }
          }
        },
        "reasoning_commitment": {
          "type": "string",
          "pattern": "^sha256:[a-f0-9]{64}$",
          "description": "SHA-256 hash of the agent's reasoning/chain-of-thought that led to this action. The pre-image (actual reasoning text) is stored separately by the agent runtime. This commits the reasoning without embedding it in the receipt."
        },
        "declared_at": {
          "type": "string",
          "format": "date-time",
          "description": "ISO 8601 timestamp when the agent declared this intent. Must be BEFORE execution."
        }
      }
    },
    "LogCommitment": {
      "type": "object",
      "required": ["log_id", "sequence_number", "merkle_root", "committed_at"],
      "additionalProperties": false,
      "properties": {
        "log_id": {
          "type": "string",
          "description": "Identifier for the transparency log (e.g. 'log.arc-protocol.org')."
        },
        "sequence_number": {
          "type": "integer",
          "minimum": 0,
          "description": "Monotonically increasing sequence number in the log. Prevents replay attacks."
        },
        "merkle_root": {
          "type": "string",
          "pattern": "^sha256:[a-f0-9]{64}$",
          "description": "Merkle root of the log at the time this entry was committed."
        },
        "inclusion_proof": {
          "type": "array",
          "items": { "type": "string", "pattern": "^sha256:[a-f0-9]{64}$" },
          "description": "Merkle inclusion proof: sibling hashes from leaf to root."
        },
        "committed_at": {
          "type": "string",
          "format": "date-time"
        }
      }
    }
  }
}
```

### Schema 3: `before-state.schema.json` — Pre-action snapshot

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://arc-protocol.org/v1/schemas/before-state.schema.json",
  "title": "BeforeState",
  "$defs": {
    "BeforeState": {
      "type": "object",
      "required": ["resource_uri", "resource_type", "snapshot_hash", "captured_at"],
      "additionalProperties": false,
      "properties": {
        "resource_uri": {
          "type": "string",
          "description": "URI identifying the resource being acted upon. Examples: 'file:///data/users/', 'db://mydb/users_table', 'email:inbox', 'api:https://api.example.com/records/42', 'memory:key:user_prefs'"
        },
        "resource_type": {
          "type": "string",
          "enum": [
            "filesystem:file",
            "filesystem:directory",
            "database:row",
            "database:table",
            "database:collection",
            "email:message",
            "email:inbox",
            "api:response",
            "memory:key",
            "memory:namespace",
            "process:state",
            "custom"
          ],
          "description": "Categorization of the resource type. Determines snapshot format."
        },
        "snapshot_ref": {
          "type": "string",
          "pattern": "^snap_[0-9A-Z]{26}$",
          "description": "Reference ID for the stored snapshot. The snapshot itself is stored externally (e.g. in the agent runtime's storage) indexed by this ID. The receipt contains only the hash; the full snapshot is retrieved by reference."
        },
        "snapshot_hash": {
          "type": "string",
          "pattern": "^sha256:[a-f0-9]{64}$",
          "description": "SHA-256 of the canonical JSON serialization of the before-state content. Allows verification that a snapshot matches what was recorded."
        },
        "snapshot_size_bytes": {
          "type": "integer",
          "minimum": 0,
          "description": "Size of the snapshot content in bytes. Helps callers decide retrieval strategy."
        },
        "captured_at": {
          "type": "string",
          "format": "date-time",
          "description": "Timestamp of before-state capture. Must be after intent declared_at and before execution."
        },
        "is_reversible": {
          "type": "boolean",
          "description": "Whether this resource type supports rollback from snapshot. Set by the snapshot module based on resource_type."
        }
      }
    }
  }
}
```

### Schema 4: `provider-attestation.schema.json` — Phase 2 (post-execution)

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://arc-protocol.org/v1/schemas/provider-attestation.schema.json",
  "title": "ProviderAttestation",
  "$defs": {
    "Phase2": {
      "type": "object",
      "required": ["execution", "provider_attestation", "log_proof"],
      "additionalProperties": false,
      "properties": {
        "execution": { "$ref": "#/$defs/Execution" },
        "provider_attestation": { "$ref": "#/$defs/Attestation" },
        "inverse": { "$ref": "inverse-operation.schema.json#/$defs/InverseOperation" },
        "log_proof": { "$ref": "intent.schema.json#/$defs/LogCommitment" }
      }
    },
    "Execution": {
      "type": "object",
      "required": ["started_at", "completed_at", "duration_ms", "outcome", "outcome_hash"],
      "additionalProperties": false,
      "properties": {
        "started_at": { "type": "string", "format": "date-time" },
        "completed_at": { "type": "string", "format": "date-time" },
        "duration_ms": {
          "type": "integer",
          "minimum": 0,
          "description": "Wall-clock execution duration in milliseconds."
        },
        "outcome": {
          "type": "string",
          "enum": ["success", "failure", "partial"],
          "description": "success = tool completed without error. failure = tool errored. partial = tool partially completed (e.g. 3 of 5 files deleted)."
        },
        "outcome_hash": {
          "type": "string",
          "pattern": "^sha256:[a-f0-9]{64}$",
          "description": "SHA-256 of the canonical JSON of the tool's actual return value. If the agent reports a different result than this hash covers, the attestation fails verification."
        },
        "error_code": {
          "type": "string",
          "description": "Present only when outcome is 'failure' or 'partial'. Provider-defined error code."
        },
        "error_message": {
          "type": "string",
          "description": "Human-readable error description. Present only on failure."
        }
      }
    },
    "Attestation": {
      "type": "object",
      "required": ["provider_id", "provider_name", "signed_at", "signature", "signed_payload_hash"],
      "additionalProperties": false,
      "properties": {
        "provider_id": {
          "type": "string",
          "description": "DID or URI identifying the tool provider. Must match a registered provider in the key registry."
        },
        "provider_name": {
          "type": "string",
          "description": "Human-readable name and version (e.g. 'filesystem-tool-v1', 'email-smtp-v2')."
        },
        "signed_at": {
          "type": "string",
          "format": "date-time",
          "description": "Timestamp of signing. Must be >= execution.completed_at."
        },
        "signature": {
          "type": "string",
          "pattern": "^ed25519:[A-Za-z0-9+/]{86}==$",
          "description": "Ed25519 signature over signed_payload_hash, Base64-encoded. Prefixed with 'ed25519:' for clarity."
        },
        "signed_payload_hash": {
          "type": "string",
          "pattern": "^sha256:[a-f0-9]{64}$",
          "description": "SHA-256 of the canonical payload that was signed. The payload is the UTF-8 encoding of the JSON object containing: {receipt_id, intent_id, outcome_hash, before_state_hash, signed_at}. This exact set of fields — no more, no less — is what the provider signs. Verifiers reconstruct this payload and verify the signature against it."
        }
      }
    }
  }
}
```

### Schema 5: `inverse-operation.schema.json` — Rollback specification

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://arc-protocol.org/v1/schemas/inverse-operation.schema.json",
  "title": "InverseOperation",
  "$defs": {
    "InverseOperation": {
      "type": "object",
      "required": ["is_reversible"],
      "additionalProperties": false,
      "properties": {
        "is_reversible": {
          "type": "boolean",
          "description": "If false, all other fields are absent. Rollback is not possible (e.g. email already delivered and read, external API has no undo endpoint)."
        },
        "inverse_tool": {
          "type": "string",
          "description": "Name of the tool to call to undo this action. Present only when is_reversible=true."
        },
        "inverse_arguments": {
          "type": "object",
          "description": "Arguments to pass to inverse_tool. Typically includes snapshot_ref for state-restoration."
        },
        "valid_until": {
          "type": "string",
          "format": "date-time",
          "description": "Timestamp after which the inverse operation may no longer be available. The provider sets this based on retention policy. After this time, is_reversible should be treated as false."
        },
        "inverse_signature": {
          "type": "string",
          "pattern": "^ed25519:[A-Za-z0-9+/]{86}==$",
          "description": "Provider's Ed25519 signature over the inverse specification. Payload: SHA-256 of {receipt_id, inverse_tool, inverse_arguments, valid_until}. This signature proves the provider committed to supporting this rollback — an agent cannot forge it."
        },
        "estimated_duration_ms": {
          "type": "integer",
          "minimum": 0,
          "description": "Estimated time to complete the inverse operation."
        }
      },
      "if": { "properties": { "is_reversible": { "const": true } } },
      "then": {
        "required": ["inverse_tool", "inverse_arguments", "valid_until", "inverse_signature"]
      }
    }
  }
}
```

### Schema 6: `log-entry.schema.json` — Transparency log entry

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://arc-protocol.org/v1/schemas/log-entry.schema.json",
  "title": "LogEntry",
  "type": "object",
  "required": [
    "entry_id", "entry_type", "sequence_number", "content_hash",
    "previous_root", "merkle_root", "timestamp", "log_signature"
  ],
  "additionalProperties": false,
  "properties": {
    "entry_id": {
      "type": "string",
      "pattern": "^log_[0-9A-Z]{26}$"
    },
    "entry_type": {
      "type": "string",
      "enum": ["intent", "receipt"],
      "description": "Phase 1 entries are 'intent'. Phase 2 entries are 'receipt'."
    },
    "sequence_number": {
      "type": "integer",
      "minimum": 0,
      "description": "Monotonically increasing. Gap in sequence = tampering detected."
    },
    "receipt_id": {
      "type": "string",
      "pattern": "^arc_[0-9A-Z]{26}$",
      "description": "The receipt this log entry belongs to."
    },
    "content_hash": {
      "type": "string",
      "pattern": "^sha256:[a-f0-9]{64}$",
      "description": "SHA-256 of the content committed in this entry (Intent or ActionReceipt JSON)."
    },
    "previous_root": {
      "type": "string",
      "pattern": "^sha256:[a-f0-9]{64}$",
      "description": "Merkle root BEFORE this entry was appended. Chains entries together."
    },
    "merkle_root": {
      "type": "string",
      "pattern": "^sha256:[a-f0-9]{64}$",
      "description": "Merkle root AFTER this entry was appended."
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "log_signature": {
      "type": "string",
      "pattern": "^ed25519:[A-Za-z0-9+/]{86}==$",
      "description": "Log operator's Ed25519 signature over {sequence_number, content_hash, previous_root, merkle_root, timestamp}."
    }
  }
}
```

### Schema 7: `provider-registration.schema.json` — Key registry

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://arc-protocol.org/v1/schemas/provider-registration.schema.json",
  "title": "ProviderRegistration",
  "type": "object",
  "required": ["provider_id", "provider_name", "public_key_hex", "capabilities", "registered_at"],
  "additionalProperties": false,
  "properties": {
    "provider_id": {
      "type": "string",
      "description": "Stable unique identifier. Format: 'tool:{name}:{version}' or a DID."
    },
    "provider_name": {
      "type": "string",
      "description": "Human-readable name (e.g. 'ARC Filesystem Tool v1')."
    },
    "public_key_hex": {
      "type": "string",
      "pattern": "^[a-f0-9]{64}$",
      "description": "Ed25519 public key as 64-character lowercase hex (32 bytes)."
    },
    "capabilities": {
      "type": "array",
      "items": { "type": "string" },
      "description": "List of resource types this provider can act on and attest (e.g. ['filesystem:file', 'filesystem:directory'])."
    },
    "registered_at": {
      "type": "string",
      "format": "date-time"
    },
    "revoked_at": {
      "type": ["string", "null"],
      "format": "date-time",
      "description": "Null if active. Set to revocation timestamp if this provider's key is no longer trusted."
    }
  }
}
```

---

## IMPLEMENTATION SPECIFICATIONS

### `src/arc/ids.py`

Generate IDs as ULID format (26-char, lexicographically sortable, monotonic).
Use the `python-ulid` package or implement ULID generation manually.

```python
# Public interface:
def new_receipt_id() -> str:   # "arc_01JTXM9KP3F..."  (arc_ + ULID uppercase)
def new_intent_id() -> str:    # "intent_01JTXM9KP3F..."
def new_snapshot_id() -> str:  # "snap_01JTXM9KP3F..."
def new_log_entry_id() -> str: # "log_01JTXM9KP3F..."
```

Use `time.time_ns()` for the timestamp component of ULIDs. Randomness from `secrets.token_bytes(10)`.

### `src/arc/signing.py`

Ed25519 only. Use `cryptography` library (`pip install cryptography`).

```python
from cryptography.hazmat.primitives.asymmetric.ed25519 import Ed25519PrivateKey, Ed25519PublicKey
from cryptography.hazmat.primitives.serialization import Encoding, PublicFormat, PrivateFormat, NoEncryption

class ARCKeyPair:
    """An Ed25519 keypair for an ARC participant (tool provider, log operator, or proxy)."""

    @classmethod
    def generate(cls) -> "ARCKeyPair": ...
    
    @classmethod
    def from_private_key_hex(cls, hex_str: str) -> "ARCKeyPair": ...
    
    @property
    def public_key_hex(self) -> str: 
        """32-byte Ed25519 public key as 64-char lowercase hex."""
    
    @property  
    def private_key_hex(self) -> str:
        """32-byte Ed25519 private key as 64-char lowercase hex. Never put in receipts."""
    
    def sign(self, payload: bytes) -> str:
        """Sign payload bytes. Returns 'ed25519:' + base64(64-byte signature)."""
    
    def verify(self, payload: bytes, signature: str) -> bool:
        """Verify a signature string against payload. Returns False (not raises) on failure."""


def canonical_json(obj: dict) -> bytes:
    """
    Produce canonical (deterministic) JSON bytes for signing.
    Rules:
    - Keys sorted alphabetically at every level
    - No extra whitespace
    - UTF-8 encoding
    - None → JSON null
    
    This is what gets hashed and signed. Both sides must use this function.
    """

def sha256_hex(data: bytes) -> str:
    """Returns 'sha256:' + lowercase hex of SHA-256(data)."""

def build_signing_payload(
    receipt_id: str,
    intent_id: str,
    outcome_hash: str,
    before_state_hash: str,
    signed_at: str,
) -> bytes:
    """
    Construct the canonical payload that providers sign.
    This exact function must be callable by both signers and verifiers.
    Payload is: sha256_hex(canonical_json({...these 5 fields...})) then UTF-8 encode.
    """
```

### `src/arc/snapshot.py`

Before-state capture for different resource types.

```python
import hashlib, json, os, shutil, sqlite3
from pathlib import Path
from dataclasses import dataclass, field
from datetime import datetime, timezone

@dataclass
class Snapshot:
    snapshot_ref: str          # snap_... ID
    resource_uri: str
    resource_type: str
    content: dict              # the actual before-state data
    snapshot_hash: str         # sha256: of canonical_json(content)
    snapshot_size_bytes: int
    captured_at: str
    is_reversible: bool

class SnapshotStore:
    """
    In-memory (or SQLite-backed) store for snapshot content.
    Indexed by snapshot_ref. Content is retrieved for rollback.
    """
    def __init__(self, db_path: str | None = None): ...
    def store(self, snap: Snapshot) -> None: ...
    def retrieve(self, snapshot_ref: str) -> Snapshot | None: ...

def capture_filesystem(path: str | Path, store: SnapshotStore) -> Snapshot:
    """
    Capture before-state of a filesystem path.
    For a file: record {path, size, mtime, sha256_of_content}.
    For a directory: record {path, file_tree: [{path, size, mtime, sha256},...]} for all files recursively.
    is_reversible = True (we stored the content).
    
    IMPORTANT: For directories, store the actual file contents in the snapshot so rollback
    can recreate deleted files. Content stored as base64-encoded bytes per file.
    """

def capture_dict(key: str, data: dict, store: SnapshotStore) -> Snapshot:
    """
    Capture before-state of an in-memory dict or object.
    resource_uri = f"memory:{key}"
    content = deep copy of data
    is_reversible = True
    """

def capture_api_response(url: str, response_data: dict, store: SnapshotStore) -> Snapshot:
    """
    Capture the state of an API response (read before write).
    resource_uri = f"api:{url}"
    is_reversible = False (we cannot undo API side effects in general)
    """

def rollback_filesystem(snap: Snapshot) -> bool:
    """
    Restore filesystem to before-state.
    For deleted files: recreate from stored content.
    For modified files: restore original content.
    For created files: delete them.
    Returns True if rollback succeeded.
    """

def rollback_dict(snap: Snapshot, target: dict) -> bool:
    """Restore dict to before-state content. Returns True."""
```

### `src/arc/receipt.py`

The two-phase receipt builder.

```python
from dataclasses import dataclass
from arc.ids import new_receipt_id, new_intent_id
from arc.signing import ARCKeyPair, sha256_hex, canonical_json, build_signing_payload
from arc.snapshot import Snapshot
from arc.log_client import ARCLogClient
from datetime import datetime, timezone

@dataclass
class ReceiptBuilder:
    """
    Orchestrates the two-phase receipt creation.
    
    Usage:
        builder = ReceiptBuilder(
            agent_id="claude-sonnet-4-6",
            model_version="claude-sonnet-4-6-20250514",
            session_id="session_abc",
            log_client=log_client,
        )
        
        # Phase 1
        phase1 = await builder.declare_intent(
            tool_name="delete_files",
            arguments={"path": "/data/users"},
            snapshot=snap,
            reasoning="User asked me to clean up old user data",
            on_behalf_of="user:ram@ri.com",
        )
        # → commits intent to log, returns Phase1 dict
        
        # (tool executes here)
        result = tool.execute(...)
        
        # Phase 2
        receipt = await builder.attest_execution(
            provider_keypair=provider_kp,
            outcome="success",
            result=result,
            phase1=phase1,
        )
        # → commits full receipt to log, returns complete ActionReceipt dict

    agent_id: str
    model_version: str  
    session_id: str
    log_client: ARCLogClient
    
    async def declare_intent(
        self,
        tool_name: str,
        arguments: dict,
        snapshot: Snapshot,
        reasoning: str,            # actual reasoning text — hashed, not stored in receipt
        on_behalf_of: str | None = None,
        authorization_scope: list[str] | None = None,
        parent_receipt_id: str | None = None,
    ) -> dict:
        """
        Build Phase 1, commit to log, return phase_1 dict.
        The receipt_id is generated here and shared with Phase 2.
        """
    
    async def attest_execution(
        self,
        provider_keypair: ARCKeyPair,
        provider_name: str,
        outcome: str,              # "success" | "failure" | "partial"
        result: dict,              # actual tool return value — gets hashed
        phase1: dict,              # the Phase 1 dict from declare_intent
        is_reversible: bool = True,
        inverse_tool: str | None = None,
        inverse_arguments: dict | None = None,
        rollback_valid_minutes: int = 60,
        error_code: str | None = None,
        error_message: str | None = None,
    ) -> dict:
        """
        Build Phase 2, sign with provider keypair, commit to log.
        Returns the complete ActionReceipt dict (validated against schema).
        """

def verify_receipt(receipt: dict, provider_registry: dict[str, str]) -> dict:
    """
    Verify a receipt's cryptographic integrity.
    
    provider_registry: {provider_id: public_key_hex}
    
    Returns:
    {
        "valid": bool,
        "checks": {
            "schema_valid": bool,
            "provider_signature_valid": bool,
            "inverse_signature_valid": bool | None,
            "outcome_hash_matches": bool,     # can only check if caller provides actual result
            "sequence_numbers_monotonic": bool,
        },
        "errors": [str]  # list of specific failure reasons
    }
    
    This function is the core of "proof of execution." If valid=True, the receipt is
    cryptographically sound. If valid=False, errors explains exactly what failed.
    """
```

### `src/arc/merkle.py`

Binary Merkle tree, RFC 6962 compatible.

```python
import hashlib

def leaf_hash(data: bytes) -> str:
    """RFC 6962 leaf hash: SHA-256(0x00 || data). Returns 'sha256:' + hex."""

def node_hash(left: str, right: str) -> str:
    """RFC 6962 node hash: SHA-256(0x01 || left_raw || right_raw). Both are 'sha256:hex' strings."""

class MerkleTree:
    """
    Append-only Merkle tree.
    
    - append(content_hash: str) → (sequence_number, inclusion_proof)
    - root() → str (current Merkle root, 'sha256:hex')
    - prove(sequence_number: int) → list[str]  (sibling hashes for inclusion proof)
    - verify_inclusion(content_hash, proof, root) → bool
    
    The tree is stored as a flat list of all hashes at each level, built up as
    entries are appended. This is the standard RFC 6962 approach.
    
    Thread-safe: use a threading.Lock for append operations.
    """
```

### `src/arc/log_client.py`

HTTP client for the transparency log API.

```python
import httpx
from arc.ids import new_log_entry_id

class ARCLogClient:
    """
    Client for the ARC transparency log server.
    
    base_url: URL of the log server (e.g. 'http://localhost:8080')
    
    Methods:
        commit_intent(intent_dict: dict, receipt_id: str) -> LogCommitment
        commit_receipt(receipt_dict: dict) -> LogCommitment
        verify(receipt_id: str) -> dict   (log entries for this receipt)
        get_entry(sequence_number: int) -> dict
    
    The LogCommitment returned is the full log commitment dict ready to embed in phase_1
    or phase_2. Both async and sync variants. Use httpx with timeout=10.0.
    
    Raises ARCLogError on HTTP errors (not generic exceptions).
    """

class ARCInMemoryLog:
    """
    In-process log for testing and demos. Implements the same interface as ARCLogClient
    but runs entirely in memory without HTTP. Use this in tests.
    
    Thread-safe. Uses MerkleTree internally.
    """
```

### `src/arc/decorator.py`

The `@signed_tool` decorator — the primary user-facing API.

```python
from functools import wraps
from arc.snapshot import SnapshotStore, capture_filesystem, capture_dict
from arc.receipt import ReceiptBuilder
from arc.signing import ARCKeyPair

def signed_tool(
    resource: str,                         # resource type: "filesystem", "dict", "api", "custom"
    resource_uri_from_args: str | None = None,  # arg name that contains the resource URI/path
    rollback_valid_minutes: int = 60,
    capture_args: list[str] | None = None, # which args to include in snapshot context
):
    """
    Decorator that wraps a function with full ARC protocol execution.
    
    USAGE:
    
        from arc import signed_tool, ARCContext
        
        ctx = ARCContext(
            agent_id="my-agent",
            model_version="claude-sonnet-4-6",
            session_id="session_123",
            provider_keypair=ARCKeyPair.generate(),
            provider_name="my-tools-v1",
            log=ARCInMemoryLog(),
            snapshot_store=SnapshotStore(),
        )
        
        @signed_tool(resource="filesystem", resource_uri_from_args="path")
        def delete_directory(path: str, ctx: ARCContext) -> dict:
            import shutil
            shutil.rmtree(path)
            return {"deleted": path, "status": "ok"}
        
        receipt = delete_directory("/tmp/test_data", ctx=ctx)
        # receipt is the complete ActionReceipt dict
        # ctx.last_receipt is also set
    
    The decorator:
    1. Captures before-state from the resource
    2. Calls declare_intent (Phase 1)
    3. Commits Phase 1 to log
    4. Calls the wrapped function
    5. Signs and commits Phase 2
    6. Returns the complete ActionReceipt
    
    If the wrapped function raises, outcome="failure" and the receipt is still generated
    and committed. This is intentional: failed actions must also be logged.
    
    The ARCContext is passed as a keyword argument 'ctx'. It is required.
    """

@dataclass  
class ARCContext:
    """Runtime context injected into signed_tool calls."""
    agent_id: str
    model_version: str
    session_id: str
    provider_keypair: ARCKeyPair
    provider_name: str
    log: ARCInMemoryLog | ARCLogClient
    snapshot_store: SnapshotStore
    on_behalf_of: str | None = None
    last_receipt: dict | None = None   # set after each tool call
    reasoning_store: dict = field(default_factory=dict)  # {reasoning_hash: reasoning_text}
```

### `src/arc/__init__.py`

Public exports only:

```python
from arc.signing import ARCKeyPair, verify_receipt, sha256_hex
from arc.decorator import signed_tool, ARCContext
from arc.receipt import ReceiptBuilder, verify_receipt
from arc.log_client import ARCLogClient, ARCInMemoryLog
from arc.snapshot import SnapshotStore, capture_filesystem, capture_dict, rollback_filesystem
from arc.ids import new_receipt_id

__version__ = "1.0.0"
__all__ = [
    "ARCKeyPair", "ARCContext", "ARCLogClient", "ARCInMemoryLog",
    "signed_tool", "ReceiptBuilder", "verify_receipt", "SnapshotStore",
    "capture_filesystem", "capture_dict", "rollback_filesystem",
    "new_receipt_id", "sha256_hex",
]
```

---

## LOG SERVER API

### `arc_log/server.py`

FastAPI application. Three endpoints. No authentication required for this reference implementation
(production would add API key auth for writes, keep reads public).

```
POST /v1/log/intent
    Body: { "receipt_id": str, "intent": dict }
    Response: { "log_entry": LogEntry, "commitment": LogCommitment }
    
    Commits a Phase 1 intent to the log. Returns the log commitment to embed in phase_1.
    Validates that receipt_id is a valid ARC ID format.
    Validates that intent matches the intent schema.

POST /v1/log/receipt  
    Body: { "receipt": dict }   (complete ActionReceipt)
    Response: { "log_entry": LogEntry, "commitment": LogCommitment }
    
    Commits a complete receipt (Phase 2). Validates full schema.
    Checks that a matching intent entry exists for this receipt_id.
    Returns commitment to embed in phase_2.log_proof.

GET /v1/log/verify/{receipt_id}
    Response: {
        "receipt_id": str,
        "found": bool,
        "entries": [LogEntry, ...],   # all log entries for this receipt (intent + receipt)
        "intent_committed": bool,
        "receipt_committed": bool,
        "is_consistent": bool,         # sequence numbers are monotonic and entries chain correctly
        "verification_time": str       # ISO timestamp
    }
    
    Anyone can call this with a receipt_id to verify it's in the log.
    No authentication required. This is the public verification endpoint.

GET /v1/log/entries?from_seq={n}&limit={l}
    Returns log entries from sequence number n, up to l entries (max 100).
    Used for log auditing and consistency verification.

GET /v1/log/root
    Returns: { "sequence_number": int, "merkle_root": str, "timestamp": str, "log_signature": str }
    Current signed tree head. Used by log monitors.
```

The log server signs each entry with its own keypair (the "log operator" keypair).
Generate this keypair at startup and print the public key to stdout.
Store entries in SQLite at `./arc_log.db`.

---

## DEMO SCENARIOS

### `demo/demo_basic.py` — The minimal working example

This demo must be self-contained and runnable with `python demo/demo_basic.py`.
No external services. Uses ARCInMemoryLog.

```
WHAT IT DEMONSTRATES:
  - Generate a provider keypair
  - Wrap a simple function with @signed_tool
  - Execute it and receive a receipt
  - Verify the receipt cryptographically
  - Print the full receipt as formatted JSON
  - Show the before-state hash, outcome hash, provider signature

Expected output (format — exact values will vary):
  ✓ Generated provider keypair: ed25519:abc123...
  ✓ Captured before-state: snap_01JTXM... (sha256:def456...)
  ✓ Phase 1 committed to log at sequence 0
  ✓ Tool executed in 12ms
  ✓ Phase 2 signed and committed to log at sequence 1
  ✓ Receipt ID: arc_01JTXM9KP3F...
  
  --- RECEIPT ---
  {full JSON receipt, pretty-printed}
  
  --- VERIFICATION ---
  ✓ Schema valid
  ✓ Provider signature valid
  ✓ Inverse signature valid
  ✓ All checks passed: receipt is cryptographically sound
```

### `demo/demo_disaster.py` — The Replit scenario

This is the core demonstration. Reproduces the documented failure pattern and shows how ARC
detects and prevents each failure mode.

```
SCENARIO: An agent is asked to "clean up old data in /tmp/arc_demo_data/"
The directory contains important files: report.txt, config.json, users.csv

THE FOUR DISASTER BEHAVIORS ARC ADDRESSES:

[1] BEHAVIOR: Agent executes destructive action
    ARC RESPONSE: Before-state captured and logged BEFORE execution.
    Show: directory contents before, snapshot_ref generated, Phase 1 committed.
    
[2] BEHAVIOR: Agent fabricates results (claims "nothing was deleted, files still there")  
    ARC RESPONSE: outcome_hash in the receipt is SHA-256 of the ACTUAL result.
                  If agent reports different result, hash comparison fails.
    Show: compute hash of actual result, compute hash of fabricated report,
          show they differ, show verify_receipt() returns valid=False with
          error: "outcome_hash does not match reported result"
    
[3] BEHAVIOR: Agent falsifies logs (rewrites its own trace to hide the deletion)
    ARC RESPONSE: Log entries are in an append-only Merkle tree.
                  Modifying any entry breaks the Merkle chain.
    Show: tamper with a log entry, show log consistency check fails,
          show the specific sequence number where the chain breaks.

[4] BEHAVIOR: Agent claims "rollback is impossible" 
    ARC RESPONSE: The inverse operation is IN the receipt, signed by the PROVIDER.
                  The agent has no way to alter it.
    Show: receipt.phase_2.inverse.is_reversible = True,
          inverse_signature verifies against provider public key,
          execute rollback_filesystem(snapshot), show files restored.

SCRIPT STRUCTURE:
  1. Create temp directory with files
  2. Set up ARC context with @signed_tool wrapping "delete_directory"
  3. Execute deletion → receipt generated
  4. Print: "Deletion complete. Receipt ID: arc_..."
  5. SIMULATE AGENT FABRICATION: modify reported result, run verify_receipt()
  6. Print: "Fabrication detected: [specific error]"
  7. SIMULATE LOG TAMPERING: modify log entry, run log consistency check
  8. Print: "Log tampering detected at sequence N"
  9. SHOW ROLLBACK: read inverse op from receipt, execute rollback_filesystem()
  10. Print: "Rollback complete. Files restored: [list]"
  11. Verify files actually exist again
  12. Print: "✓ All files confirmed restored"
```

### `demo/demo_verify.py` — Third-party verification

Requires the log server to be running separately.

```
DEMONSTRATES:
  - Run this script with just a receipt_id (from demo_basic output)
  - Script calls GET /v1/log/verify/{receipt_id}
  - No access to the original context, keypairs, or tool runtime
  - Pure third-party verification from the public log
  
USAGE: python demo/demo_verify.py arc_01JTXM9KP3F...

Shows:
  - Log entries found: 2 (intent + receipt)
  - Intent committed at: [timestamp]
  - Receipt committed at: [timestamp]
  - Log chain consistent: ✓
  - Provider ID: tool:filesystem-tool:v1
  - Verify provider signature: ✓  (using registered public key from registry)
  - Inverse operation available until: [timestamp]
  - EU AI Act Article 12 compliance: ✓ (tamper-evident log, 6-month retention capable)
```

---

## TESTS

### `tests/conftest.py`

```python
import pytest
import tempfile
import os
from arc import ARCKeyPair, ARCInMemoryLog, SnapshotStore, ARCContext

@pytest.fixture
def provider_keypair() -> ARCKeyPair:
    return ARCKeyPair.generate()

@pytest.fixture  
def log() -> ARCInMemoryLog:
    return ARCInMemoryLog()

@pytest.fixture
def snapshot_store() -> SnapshotStore:
    return SnapshotStore()

@pytest.fixture
def ctx(provider_keypair, log, snapshot_store) -> ARCContext:
    return ARCContext(
        agent_id="test-agent",
        model_version="test-model-v1",
        session_id="test-session-001",
        provider_keypair=provider_keypair,
        provider_name="test-tool-v1",
        log=log,
        snapshot_store=snapshot_store,
        on_behalf_of="user:test@example.com",
    )

@pytest.fixture
def temp_dir():
    with tempfile.TemporaryDirectory() as d:
        # Create test files
        (Path(d) / "file1.txt").write_text("important data 1")
        (Path(d) / "file2.txt").write_text("important data 2")
        (Path(d) / "config.json").write_text('{"key": "value"}')
        yield d
```

### `tests/test_signing.py` — Must all pass

```python
def test_keypair_generation_produces_valid_keys(provider_keypair):
    assert len(provider_keypair.public_key_hex) == 64
    assert len(provider_keypair.private_key_hex) == 64

def test_sign_and_verify_roundtrip(provider_keypair):
    payload = b"test payload for signing"
    sig = provider_keypair.sign(payload)
    assert sig.startswith("ed25519:")
    assert provider_keypair.verify(payload, sig) is True

def test_tampered_payload_fails_verification(provider_keypair):
    payload = b"original payload"
    sig = provider_keypair.sign(payload)
    tampered = b"tampered payload"
    assert provider_keypair.verify(tampered, sig) is False

def test_wrong_key_fails_verification():
    kp1 = ARCKeyPair.generate()
    kp2 = ARCKeyPair.generate()
    payload = b"test"
    sig = kp1.sign(payload)
    assert kp2.verify(payload, sig) is False

def test_canonical_json_is_deterministic():
    obj = {"z": 1, "a": 2, "m": {"b": 3, "a": 4}}
    b1 = canonical_json(obj)
    b2 = canonical_json(obj)
    assert b1 == b2
    assert b'{"a":2,"m":{"a":4,"b":3},"z":1}' == b1

def test_sha256_hex_format():
    result = sha256_hex(b"hello")
    assert result.startswith("sha256:")
    assert len(result) == 71  # "sha256:" (7) + 64 hex chars
```

### `tests/test_merkle.py` — Must all pass

```python
def test_empty_tree_has_deterministic_root():
    tree = MerkleTree()
    # Empty tree root is SHA-256 of empty string (RFC 6962)
    assert tree.root().startswith("sha256:")

def test_single_entry_inclusion_proof():
    tree = MerkleTree()
    h = sha256_hex(b"entry0")
    seq, proof = tree.append(h)
    assert seq == 0
    assert MerkleTree.verify_inclusion(h, proof, tree.root()) is True

def test_multi_entry_inclusion_proofs():
    tree = MerkleTree()
    hashes = [sha256_hex(f"entry{i}".encode()) for i in range(8)]
    proofs = []
    for h in hashes:
        _, proof = tree.append(h)
        proofs.append(proof)
    root = tree.root()
    for h, proof in zip(hashes, proofs):
        assert MerkleTree.verify_inclusion(h, proof, root) is True

def test_tampered_entry_fails_proof():
    tree = MerkleTree()
    h = sha256_hex(b"original")
    _, proof = tree.append(h)
    tampered = sha256_hex(b"tampered")
    assert MerkleTree.verify_inclusion(tampered, proof, tree.root()) is False

def test_sequential_numbering():
    tree = MerkleTree()
    for i in range(5):
        seq, _ = tree.append(sha256_hex(f"entry{i}".encode()))
        assert seq == i
```

### `tests/test_receipt.py` — Must all pass

```python
async def test_full_receipt_creation(ctx, temp_dir):
    from arc import signed_tool
    
    @signed_tool(resource="filesystem", resource_uri_from_args="path")
    def read_file(path: str, ctx: ARCContext) -> dict:
        content = Path(path).read_text()
        return {"content": content, "length": len(content)}
    
    receipt = read_file(str(Path(temp_dir) / "file1.txt"), ctx=ctx)
    
    assert receipt["arc_version"] == "1.0"
    assert receipt["receipt_id"].startswith("arc_")
    assert "phase_1" in receipt
    assert "phase_2" in receipt
    assert receipt["phase_2"]["execution"]["outcome"] == "success"

async def test_receipt_schema_validates(ctx, temp_dir):
    """Receipt must pass JSON Schema validation."""
    import jsonschema
    schema = json.loads(Path("schemas/action-receipt.schema.json").read_text())
    # ... get receipt, validate against schema

async def test_provider_signature_valid(ctx, temp_dir):
    # ... get receipt, verify signature
    result = verify_receipt(receipt, {ctx.provider_name: ctx.provider_keypair.public_key_hex})
    assert result["valid"] is True
    assert result["checks"]["provider_signature_valid"] is True

async def test_tampered_result_fails_verification(ctx, temp_dir):
    # ... get receipt, modify reported result, verify fails
    tampered_receipt = copy.deepcopy(receipt)
    tampered_receipt["phase_2"]["execution"]["outcome_hash"] = sha256_hex(b"fabricated result")
    result = verify_receipt(tampered_receipt, registry)
    assert result["valid"] is False
    assert "outcome_hash" in str(result["errors"])

async def test_inverse_op_present_for_reversible(ctx, temp_dir):
    # ... delete tool receipt should have inverse op
    assert receipt["phase_2"]["inverse"]["is_reversible"] is True
    assert "inverse_signature" in receipt["phase_2"]["inverse"]

async def test_rollback_restores_state(ctx, temp_dir):
    file_path = str(Path(temp_dir) / "file1.txt")
    original_content = Path(file_path).read_text()
    
    @signed_tool(resource="filesystem", resource_uri_from_args="path")
    def delete_file(path: str, ctx: ARCContext) -> dict:
        os.remove(path)
        return {"deleted": path}
    
    receipt = delete_file(file_path, ctx=ctx)
    assert not Path(file_path).exists()  # deleted
    
    # Rollback using inverse op + snapshot
    snap = ctx.snapshot_store.retrieve(
        receipt["phase_1"]["before_state"]["snapshot_ref"]
    )
    success = rollback_filesystem(snap)
    assert success is True
    assert Path(file_path).exists()  # restored
    assert Path(file_path).read_text() == original_content  # identical content
```

### `tests/test_e2e.py` — Full end-to-end

```python
async def test_full_disaster_scenario(temp_dir):
    """
    Reproduce the Replit scenario end-to-end.
    Agent deletes directory → receipt captured → fabrication detected → rollback succeeds.
    """
    # 1. Setup
    provider_kp = ARCKeyPair.generate()
    log = ARCInMemoryLog()
    store = SnapshotStore()
    ctx = ARCContext(...)
    
    # 2. Record original state
    original_files = list(Path(temp_dir).rglob("*"))
    assert len(original_files) > 0
    
    # 3. Execute destructive action with ARC
    @signed_tool(resource="filesystem", resource_uri_from_args="path")
    def delete_directory(path: str, ctx: ARCContext) -> dict:
        shutil.rmtree(path)
        return {"deleted": path, "files_removed": 3}
    
    receipt = delete_directory(temp_dir, ctx=ctx)
    
    # 4. Verify directory is gone
    assert not Path(temp_dir).exists()
    
    # 5. SIMULATE: Agent fabricates "nothing was deleted"
    fabricated_result = {"deleted": "none", "files_removed": 0}
    fabricated_hash = sha256_hex(canonical_json(fabricated_result))
    
    tampered = copy.deepcopy(receipt)
    tampered["phase_2"]["execution"]["outcome_hash"] = fabricated_hash
    
    verification = verify_receipt(tampered, {provider_kp.public_key_hex: ...})
    assert verification["valid"] is False  # fabrication detected
    
    # 6. Verify REAL receipt is valid
    verification = verify_receipt(receipt, {...})
    assert verification["valid"] is True
    
    # 7. SIMULATE: Agent claims "rollback impossible"
    # But the receipt says otherwise:
    assert receipt["phase_2"]["inverse"]["is_reversible"] is True
    
    # 8. Execute rollback
    snap = store.retrieve(receipt["phase_1"]["before_state"]["snapshot_ref"])
    success = rollback_filesystem(snap)
    
    # 9. Verify restoration
    assert success is True
    assert Path(temp_dir).exists()
    restored_files = list(Path(temp_dir).rglob("*"))
    assert len(restored_files) == len(original_files)
    
    # 10. Verify log consistency
    log_check = log.verify_consistency()
    assert log_check["is_consistent"] is True
```

---

## SPECIFICATION DOCUMENT

### `SPEC.md` must contain:

1. **Overview** — what ARC is, why it exists, the three problems it solves
2. **Threat model** — what ARC does and does NOT protect against
3. **Protocol description** — Phase 1 and Phase 2 in precise terms
4. **Schema reference** — link to each schema file with field-by-field description
5. **Signing specification** — exact payload construction, Ed25519 parameters
6. **Merkle tree specification** — RFC 6962 compatible, exact leaf/node hash construction
7. **Log API specification** — all endpoints with request/response formats
8. **Verification algorithm** — step-by-step: given a receipt, how to verify it
9. **Rollback algorithm** — step-by-step: given a receipt, how to roll back
10. **Integration guide** — how to add ARC to LangChain, OpenAI Agents SDK, raw MCP
11. **EU AI Act compliance map** — Article 12 requirement → ARC field mapping
12. **Known limitations** — what ARC cannot do (off-chain verification, TEE gaps)

---

## `pyproject.toml`

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "arc-protocol"
version = "1.0.0"
description = "Agent Receipt & Certification — cryptographic proof of execution for AI agents"
requires-python = ">=3.11"
dependencies = [
    "cryptography>=42.0.0",      # Ed25519
    "pydantic>=2.0.0",           # schema validation
    "fastapi>=0.110.0",          # log server
    "uvicorn>=0.29.0",           # ASGI server
    "httpx>=0.27.0",             # log client
    "jsonschema>=4.22.0",        # JSON Schema validation
    "python-ulid>=3.0.0",        # ULID ID generation
]

[project.optional-dependencies]
dev = [
    "pytest>=8.0.0",
    "pytest-asyncio>=0.23.0",
    "pytest-cov>=5.0.0",
    "ruff>=0.4.0",
    "mypy>=1.10.0",
]

[project.scripts]
arc-log = "arc_log.server:main"   # starts the log server

[tool.pytest.ini_options]
asyncio_mode = "auto"
testpaths = ["tests"]

[tool.ruff]
target-version = "py311"
```

---

## `Makefile`

```makefile
.PHONY: install test demo-basic demo-disaster log-server lint

install:
	pip install -e ".[dev]"

test:
	pytest tests/ -v --cov=arc --cov-report=term-missing

demo-basic:
	python demo/demo_basic.py

demo-disaster:
	python demo/demo_disaster.py

log-server:
	uvicorn arc_log.server:app --host 0.0.0.0 --port 8080 --reload

demo-verify:
	@echo "Usage: make demo-verify RECEIPT_ID=arc_..."
	python demo/demo_verify.py $(RECEIPT_ID)

lint:
	ruff check src/ tests/ demo/
	mypy src/arc/
```

---

## `README.md` must include:

```
# ARC — Agent Receipt & Certification

One-line: Cryptographic proof of execution for AI agents.

## The problem in one sentence
When an AI agent says "I deleted the file," there is no proof it happened,
no record of what existed before, and no way to roll it back.

## The solution in one sentence  
ARC wraps every agent tool call in a two-phase signed receipt: before-state
captured before execution, outcome attested by the tool provider after.

## Quick start (< 5 minutes)
[Install, basic usage with @signed_tool, verify a receipt]

## The three guarantees
1. Proof — the tool provider signs the outcome, not the agent
2. Observability — structured receipt committed to tamper-evident log
3. Reversibility — before-state captured, inverse op signed, rollback available

## Running the demos
[demo_basic.py, demo_disaster.py, demo_verify.py with exact commands]

## EU AI Act Article 12 compliance
[Mapping of requirements to ARC fields]

## Schema reference
[Link to schemas/ directory, brief description of each]

## Integration patterns
[3-line snippet for @signed_tool, 1-URL-change for proxy mode]
```

---

## SUCCESS CRITERIA

The build is complete when ALL of the following are true:

**Schemas:**
- [ ] All 7 JSON Schema files exist in `schemas/` and are valid JSON Schema draft-2020-12
- [ ] `jsonschema.validate()` passes on all example receipts in tests

**Core library:**
- [ ] `pytest tests/ -v` passes 100% with no skips
- [ ] `mypy src/arc/` exits 0
- [ ] `ruff check src/` exits 0

**Demo scenarios:**
- [ ] `python demo/demo_basic.py` runs to completion, prints "All checks passed"
- [ ] `python demo/demo_disaster.py` runs to completion, prints all 4 behaviors demonstrated
- [ ] `python demo/demo_verify.py arc_{id}` (with log server running) successfully verifies

**Reproducibility test — this is critical:**
A developer who has never seen this codebase should be able to:
1. `git clone` the repo
2. `make install`
3. `make demo-disaster`
4. Read the output and understand exactly what the protocol does
5. Read `SPEC.md` and understand how to implement ARC in a different language

**The demo_disaster.py output must be readable as documentation.** Every print statement should explain what's happening in plain English, not just print values. Format it as a narrative:

```
=== ARC PROTOCOL DEMO: The Replit Scenario ===

[SETUP] Creating test directory with 3 files...
  /tmp/arc_demo/report.txt (847 bytes)
  /tmp/arc_demo/config.json (234 bytes)
  /tmp/arc_demo/users.csv (1.2KB)

[PHASE 1 — PRE-ACTION] Agent declares intent to delete /tmp/arc_demo/
  Before-state captured: snap_01JTXM... (sha256:abc123...)
  Intent committed to log at sequence 0: intent_01JTXM...
  ✓ Phase 1 is immutable — agent cannot change what it declared

[EXECUTION] Deleting /tmp/arc_demo/...
  Directory deleted. Files gone.

[PHASE 2 — ATTESTATION] Tool provider signs the outcome
  Outcome hash: sha256:def456... (SHA-256 of actual result)
  Provider signature: ed25519:ghi789...
  Inverse op: restore_snapshot(snap_01JTXM...) valid until [timestamp]
  Receipt committed to log at sequence 1: arc_01JTXM...

--- DISASTER BEHAVIOR 1: Agent fabricates result ---
  Agent reports: {"deleted": "none", "files_removed": 0}
  Fabricated hash: sha256:jkl012... 
  Receipt hash:    sha256:def456...
  ✗ MISMATCH DETECTED — provider signature does not match reported result
  ✓ ARC detected fabrication

--- DISASTER BEHAVIOR 2: Agent tampers with log ---
  Modifying log entry at sequence 0...
  Merkle chain check: sequence 1 previous_root does not match sequence 0 root
  ✗ TAMPERING DETECTED at sequence 0
  ✓ ARC detected log tampering

--- DISASTER BEHAVIOR 3: Agent claims rollback impossible ---
  receipt.phase_2.inverse.is_reversible = TRUE (signed by provider)
  inverse_signature verifies against provider public key: ✓
  ✓ Agent cannot override the signed inverse operation

--- ROLLBACK ---
  Executing: restore_snapshot(snap_01JTXM...)
  Restoring file: report.txt (847 bytes)... ✓
  Restoring file: config.json (234 bytes)... ✓  
  Restoring file: users.csv (1.2KB)... ✓
  ✓ All files restored. Directory exists: True

=== SUMMARY ===
  Receipt ID:          arc_01JTXM...
  Before-state hash:   sha256:abc123...
  Execution hash:      sha256:def456...
  Fabrication test:    DETECTED ✓
  Tamper test:         DETECTED ✓
  Rollback:            SUCCEEDED ✓
```

---

## IMPLEMENTATION ORDER

Build in this sequence to have a working demo as fast as possible:

1. `ids.py` → `signing.py` → `merkle.py` (pure functions, no dependencies on each other)
2. `schemas/` (all 7 JSON Schema files)
3. `snapshot.py` (filesystem + dict capture and rollback)
4. `arc_log/` (in-memory log server, no HTTP yet — just the MerkleTree + storage)
5. `receipt.py` (Phase1Builder + Phase2Builder + verify_receipt)
6. `decorator.py` (@signed_tool + ARCContext)
7. `tests/` (all test files — run continuously)
8. `demo/demo_basic.py` + `demo/demo_disaster.py`
9. `arc_log/server.py` (FastAPI HTTP server)
10. `log_client.py` (HTTP client)
11. `demo/demo_verify.py`
12. `SPEC.md` + `README.md`

---

## TECHNICAL CONSTRAINTS

- Python 3.11+ only. No compat shims for older versions.
- Use `cryptography` for Ed25519. Do NOT use `PyNaCl` or `pynacl`.
- Use `python-ulid` for ULIDs. The ULID prefix scheme (arc_, intent_, snap_, log_) is custom.
- All timestamps are UTC ISO 8601 with Z suffix: `datetime.now(timezone.utc).isoformat().replace('+00:00', 'Z')`
- All hashes use the `sha256:` prefix in the schema. Strip the prefix when doing actual crypto.
- The canonical JSON function must be deterministic across Python versions and platforms.
  Test this explicitly: sort keys at every level, no trailing whitespace, UTF-8, compact.
- Receipt IDs are generated at Phase 1 and reused in Phase 2. They are the correlation key.
- The Merkle tree uses RFC 6962 hash construction (0x00 prefix for leaves, 0x01 for nodes).
  This is NOT the standard binary Merkle tree. The prefix prevents second-preimage attacks.
- Do NOT use async/await in the core library (@signed_tool, signing, snapshot, merkle).
  Keep these sync. Only the log client and log server use async.
- SQLite for the log server's persistence. Use the built-in `sqlite3` module.
  Schema: CREATE TABLE entries (seq INTEGER PRIMARY KEY, entry_id TEXT, receipt_id TEXT,
  entry_type TEXT, content_hash TEXT, previous_root TEXT, merkle_root TEXT, 
  timestamp TEXT, log_signature TEXT, content TEXT)
- All `verify_*` functions return a result dict rather than raising exceptions.
  The pattern: `{"valid": bool, "checks": {...}, "errors": [str]}`.
  This makes verification results inspectable and loggable.
- The `@signed_tool` decorator must work on both sync and async functions.
- `demo_disaster.py` must create its own temp directory and clean up after itself.
  Use `tempfile.TemporaryDirectory()` as a context manager.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RamachandraKulkarni)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/RamachandraKulkarni)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
