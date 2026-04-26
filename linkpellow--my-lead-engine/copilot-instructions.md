## my-lead-engine

> gRPC Contract Protocol - Unified Service Communication


# gRPC CONTRACT PROTOCOL

## 🎯 PURPOSE

All cross-service communication in the Triple-Vessel system must use gRPC with a shared protocol buffer contract. This ensures type safety, performance, and clear service boundaries.

---

## 📍 CONTRACT LOCATION

**Shared Proto File:** `@proto/chimera.proto`  
**Used By:** The General (Node.js), The Brain (Python), The Body (Rust)

---

## 🔧 PROTO DEFINITION STRUCTURE

### Service Definitions
```protobuf
syntax = "proto3";

package chimera;

// The Brain Service (Python)
service Brain {
  // Process screenshot with VLM
  rpc ProcessVision(ProcessVisionRequest) returns (VisionResponse);
  
  // Query Hive Mind memory
  rpc QueryMemory(QueryMemoryRequest) returns (MemoryResponse);
  
  // Update world model
  rpc UpdateWorldModel(WorldModelUpdate) returns (WorldModelResponse);
}

// The Body Service (Rust)
service Body {
  // Execute browser mission
  rpc ExecuteMission(MissionRequest) returns (MissionResponse);
  
  // Validate stealth (CreepJS)
  rpc ValidateStealth(StealthValidationRequest) returns (StealthValidationResponse);
  
  // Get worker status
  rpc GetWorkerStatus(WorkerStatusRequest) returns (WorkerStatusResponse);
}

// The General Service (Node.js)
service General {
  // Request lead processing
  rpc ProcessLead(LeadRequest) returns (LeadResponse);
  
  // Get enrichment status
  rpc GetEnrichmentStatus(StatusRequest) returns (StatusResponse);
}
```

### Message Types
```protobuf
// Vision Processing
message ProcessVisionRequest {
  bytes screenshot = 1;
  string context = 2;  // Optional context
}

message VisionResponse {
  string description = 1;
  double confidence = 2;
  repeated UIElement elements = 3;
}

message UIElement {
  string type = 1;
  string selector = 2;
  map<string, string> attributes = 3;
}

// Memory Query
message QueryMemoryRequest {
  string query = 1;
  int32 top_k = 2;
}

message MemoryResponse {
  repeated MemoryResult results = 1;
}

message MemoryResult {
  string text = 1;
  double similarity = 2;
  map<string, string> metadata = 3;
}

// Mission Execution
message MissionRequest {
  string mission_id = 1;
  string target_url = 2;
  map<string, string> parameters = 3;
}

message MissionResponse {
  bool success = 1;
  string result_data = 2;  // JSON string
  repeated string errors = 3;
}

// Stealth Validation
message StealthValidationRequest {
  string worker_id = 1;
}

message StealthValidationResponse {
  double trust_score = 1;  // 0.0 to 100.0
  bool is_human = 2;  // true if score >= 100.0
  map<string, string> fingerprint_details = 3;
}
```

---

## 🔄 COMMUNICATION FLOWS

### Flow 1: The General → The Brain (VLM Processing)
```
The General (Node.js)
  ↓ gRPC: ProcessVision
The Brain (Python)
  ↓ Returns: VisionResponse
The General
```

**Use Case:** Process screenshot to understand page structure

### Flow 2: The General → The Body (Mission Execution)
```
The General (Node.js)
  ↓ gRPC: ExecuteMission
The Body (Rust)
  ↓ Returns: MissionResponse
The General
```

**Use Case:** Execute stealth browser mission

### Flow 3: The Brain → The Body (Stealth Validation)
```
The Brain (Python)
  ↓ gRPC: ValidateStealth
The Body (Rust)
  ↓ Returns: StealthValidationResponse
The Brain
```

**Use Case:** Validate worker stealth capabilities

---

## 🛠️ IMPLEMENTATION PATTERNS

### Node.js (The General)
```typescript
// lib/grpc/brain-client.ts
import * as grpc from '@grpc/grpc-js';
import * as protoLoader from '@grpc/proto-loader';

const PROTO_PATH = '../../@proto/chimera.proto';

const packageDefinition = protoLoader.loadSync(PROTO_PATH, {
  keepCase: true,
  longs: String,
  enums: String,
  defaults: true,
  oneofs: true
});

const proto = grpc.loadPackageDefinition(packageDefinition) as any;

export class BrainClient {
  private client: any;
  
  constructor(address: string = 'chimera-brain:50051') {
    this.client = new proto.chimera.Brain(
      address,
      grpc.credentials.createInsecure()  // Use TLS in production
    );
  }
  
  async processVision(screenshot: Buffer): Promise<any> {
    return new Promise((resolve, reject) => {
      this.client.ProcessVision(
        { screenshot, context: '' },
        (error: any, response: any) => {
          if (error) reject(error);
          else resolve(response);
        }
      );
    });
  }
}
```

### Python (The Brain)
```python
# src/grpc/server.py
import grpc
from concurrent import futures
from proto import chimera_pb2, chimera_pb2_grpc

class BrainService(chimera_pb2_grpc.BrainServicer):
    def __init__(self, vlm_processor, hive_mind):
        self.vlm_processor = vlm_processor
        self.hive_mind = hive_mind
    
    async def ProcessVision(self, request, context):
        result = await self.vlm_processor.process(request.screenshot)
        return chimera_pb2.VisionResponse(
            description=result['description'],
            confidence=result['confidence'],
            elements=[chimera_pb2.UIElement(**elem) for elem in result['elements']]
        )
    
    async def QueryMemory(self, request, context):
        results = await self.hive_mind.search(request.query, top_k=request.top_k)
        return chimera_pb2.MemoryResponse(
            results=[chimera_pb2.MemoryResult(**r) for r in results]
        )

async def serve():
    server = grpc.aio.server(futures.ThreadPoolExecutor(max_workers=10))
    chimera_pb2_grpc.add_BrainServicer_to_server(
        BrainService(vlm_processor, hive_mind),
        server
    )
    server.add_insecure_port('[::]:50051')
    await server.start()
    await server.wait_for_termination()
```

### Rust (The Body)
```rust
// src/grpc/client.rs
use tonic::Request;
use proto::chimera::{brain_client::BrainClient, ProcessVisionRequest};

pub async fn send_vision_to_brain(
    screenshot: Vec<u8>
) -> Result<proto::chimera::VisionResponse, tonic::Status> {
    let mut client = BrainClient::connect("http://chimera-brain:50051").await?;
    
    let request = Request::new(ProcessVisionRequest {
        screenshot,
        context: String::new(),
    });
    
    let response = client.process_vision(request).await?;
    Ok(response.into_inner())
}
```

---

## 🚨 CRITICAL RULES

### 1. Contract-First Development
- ✅ **Define proto contract first** before implementing services
- ✅ **Generate code from proto** (don't write manually)
- ✅ **Version proto contracts** if breaking changes needed
- ❌ **Never bypass gRPC** for direct HTTP calls

### 2. Type Safety
- ✅ **Use generated types** from proto files
- ✅ **Validate message types** at service boundaries
- ✅ **Handle errors gracefully** (gRPC status codes)

### 3. Network Security
- ✅ **Use Railway's private network** for service-to-service calls
- ✅ **Use TLS in production** (insecure only for local dev)
- ✅ **Authenticate requests** if needed (add auth to proto)

### 4. Error Handling
- ✅ **Use gRPC status codes** (OK, INVALID_ARGUMENT, NOT_FOUND, etc.)
- ✅ **Return structured errors** in response messages
- ✅ **Implement retry logic** with exponential backoff
- ✅ **Log all gRPC errors** for debugging

---

## 📊 PROTO GENERATION COMMANDS

### Node.js
```bash
# Install tools
npm install -g grpc-tools

# Generate TypeScript types
grpc_tools_node_protoc \
  --plugin=protoc-gen-ts=./node_modules/.bin/protoc-gen-ts \
  --ts_out=./src/grpc/generated \
  --js_out=import_style=commonjs,binary:./src/grpc/generated \
  --grpc_out=./src/grpc/generated \
  -I ../../@proto \
  ../../@proto/chimera.proto
```

### Python
```bash
# Generate Python code
python -m grpc_tools.protoc \
  --proto_path=../../@proto \
  --python_out=./proto \
  --grpc_python_out=./proto \
  ../../@proto/chimera.proto
```

### Rust
```bash
# In Cargo.toml
[build-dependencies]
prost-build = "0.12"

# In build.rs
prost_build::compile_protos(
    &["../../@proto/chimera.proto"],
    &["../../@proto"]
)?;
```

---

## ✅ SUCCESS CRITERIA

A properly implemented gRPC contract:

- ✅ Proto file defined in `@proto/chimera.proto`
- ✅ All services generate code from proto
- ✅ The General can call The Brain via gRPC
- ✅ The General can call The Body via gRPC
- ✅ The Brain can call The Body via gRPC
- ✅ All communication uses Railway's private network
- ✅ Type safety enforced by generated code
- ✅ Error handling uses gRPC status codes

---

## 🎯 REMEMBER

**Contract-first:** Define proto before implementing.  
**Type safety:** Use generated types, don't write manually.  
**Private network:** Use Railway's private network for security.  
**No HTTP bypass:** All cross-service communication via gRPC.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/linkpellow) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
