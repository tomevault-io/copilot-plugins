## zombie-coder-local-ai-ollama

> এই document Editor/IDE extension (Cursor/VS Code) এর জন্য integration নির্দেশনা প্রদান করে। ZombieCoder Local AI Framework এখন সম্পূর্ণভাবে GGUF models support করে এবং HuggingFace Hub থেকে সরাসরি download ও run করতে পারে।

# ZombieCoder Local AI - GGUF Integration Guide for Editor Extension

## 📋 সারসংক্ষেপ (Summary)

এই document Editor/IDE extension (Cursor/VS Code) এর জন্য integration নির্দেশনা প্রদান করে। ZombieCoder Local AI Framework এখন সম্পূর্ণভাবে GGUF models support করে এবং HuggingFace Hub থেকে সরাসরি download ও run করতে পারে।

---

## 🗂️ নতুন Files যোগ করা হয়েছে

### 1. `/scripts/model_downloader.py`
**উদ্দেশ্য:** HuggingFace Hub থেকে GGUF models download করা

**মূল Functions:**
```python
download_model(repo_id, filename, dest_dir, token) -> Dict
get_gguf_files(repo_id, token) -> list
```

**ব্যবহার উদাহরণ:**
```python
from scripts.model_downloader import download_model

result = download_model(
    repo_id="TheBloke/TinyLlama-1.1B-Chat-v1.0-GGUF",
    filename="tinyllama-1.1b-chat-v1.0.Q2_K.gguf",
    dest_dir="./models"
)
```

---

### 2. `/scripts/gguf_loader.py`
**উদ্দেশ্য:** GGUF models load করা llama.cpp backend দিয়ে

**মূল Functions:**
```python
load_model(model_path, port, threads, gpu_layers, context_size, background) -> Dict
check_model_status(port) -> Dict
find_llama_cpp_server() -> Optional[Path]
```

**ব্যবহার উদাহরণ:**
```python
from scripts.gguf_loader import load_model

result = load_model(
    model_path="./models/tinyllama-gguf/tinyllama-1.1b-chat-v1.0.Q2_K.gguf",
    port=8080,
    threads=4,
    gpu_layers=0
)
```

---

### 3. `/scripts/registry_manager.py`
**উদ্দেশ্য:** Model registry management (installed এবং available models track করা)

**মূল Functions:**
```python
add_installed_model(model_name, repo_id, filename, location, size_mb, format)
remove_installed_model(model_name)
list_installed_models() -> Dict
validate_model_format(model_path) -> Dict
```

---

### 4. `/models/model_registry.json`
**উদ্দেশ্য:** সব models এর centralized registry

**Structure:**
```json
{
  "version": "1.0",
  "last_updated": "2025-10-18T03:00:00Z",
  "models": {
    "model-name": {
      "repo_id": "TheBloke/Model-GGUF",
      "filename": "model.Q4_K_M.gguf",
      "format": "gguf",
      "status": "installed",
      "location": "./models/model-name/model.Q4_K_M.gguf",
      "size_mb": 1500,
      "installed_at": "2025-10-18T03:00:00Z"
    }
  },
  "available": {
    "model-name": {
      "repo_id": "TheBloke/Model-GGUF",
      "filename": "model.Q4_K_M.gguf",
      "format": "gguf",
      "status": "available",
      "size_estimate_mb": 1500,
      "description": "Model description"
    }
  }
}
```

---

## 🔌 API Endpoints (Integration Points)

### **Download Endpoints**

#### 1. POST `/download/start`
Model download শুরু করে

**Request Body:**
```json
{
  "model_name": "phi-2-gguf",
  "repo_id": "TheBloke/phi-2-GGUF",
  "revision": null
}
```

**Response:**
```json
{
  "status": "started",
  "model_name": "phi-2-gguf",
  "repo_id": "TheBloke/phi-2-GGUF",
  "target_dir": "C:\\model\\models\\phi-2-gguf"
}
```

---

#### 2. GET `/download/status/{model}`
Download status check করে এবং complete হলে format validation করে

**Response (Completed):**
```json
{
  "status": "completed",
  "model": "phi-2-gguf",
  "format_validation": {
    "format": "gguf",
    "valid": true,
    "files": ["phi-2.Q4_K_M.gguf"],
    "message": "GGUF format detected - compatible with llama.cpp"
  },
  "registry_updated": true
}
```

**Response (Invalid Format):**
```json
{
  "status": "completed",
  "model": "phi-2",
  "format_validation": {
    "format": "safetensors",
    "valid": false,
    "files": ["model.safetensors"],
    "message": "SafeTensors format detected - NOT compatible with current runtime. Please download GGUF version."
  }
}
```

---

### **Registry Endpoints**

#### 3. GET `/registry/models`
সব installed এবং available models list করে

**Response:**
```json
{
  "installed": {
    "tinyllama-gguf": {
      "repo_id": "TheBloke/TinyLlama-1.1B-Chat-v1.0-GGUF",
      "filename": "tinyllama-1.1b-chat-v1.0.Q2_K.gguf",
      "format": "gguf",
      "status": "installed",
      "location": "./models/tinyllama-gguf/tinyllama-1.1b-chat-v1.0.Q2_K.gguf",
      "size_mb": 641.5,
      "installed_at": "2025-10-18T03:00:00Z"
    }
  },
  "available": {
    "phi-2-gguf": {
      "repo_id": "TheBloke/phi-2-GGUF",
      "filename": "phi-2.Q4_K_M.gguf",
      "format": "gguf",
      "status": "available",
      "size_estimate_mb": 1500,
      "description": "Phi-2 2.7B GGUF Q4_K_M quantization"
    }
  },
  "count_installed": 1,
  "count_available": 3
}
```

---

#### 4. GET `/registry/validate/{model}`
Model format validate করে

**Response (GGUF - Valid):**
```json
{
  "format": "gguf",
  "valid": true,
  "files": ["model.Q4_K_M.gguf"],
  "message": "GGUF format detected - compatible with llama.cpp"
}
```

**Response (SafeTensors - Invalid):**
```json
{
  "format": "safetensors",
  "valid": false,
  "files": ["model.safetensors"],
  "message": "SafeTensors format detected - NOT compatible with current runtime. Please download GGUF version."
}
```

---

### **Runtime Endpoints**

#### 5. POST `/runtime/load/{model}`
Model load করে (শুধু GGUF support করে)

**Query Parameters:**
- `threads` (optional, default: 4)

**Response:**
```json
{
  "status": "ready",
  "model": "tinyllama-gguf",
  "port": 8080,
  "pid": 12345,
  "command": "C:\\model\\config\\llama.cpp\\server.exe --model ... --port 8080 --threads 4"
}
```

**Error Response (Non-GGUF):**
```json
{
  "detail": "GGUF weights not found under C:\\model\\models\\phi-2"
}
```

---

## ⚙️ Integration Workflow

### **Complete Download → Validate → Load Flow**

```typescript
// 1. Start Download
const downloadResp = await fetch('http://localhost:8155/download/start', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    model_name: 'phi-2-gguf',
    repo_id: 'TheBloke/phi-2-GGUF'
  })
});

// 2. Poll Download Status
const checkStatus = async (modelName: string) => {
  const resp = await fetch(`http://localhost:8155/download/status/${modelName}`);
  const status = await resp.json();
  
  if (status.status === 'completed') {
    // 3. Check Format Validation
    if (status.format_validation?.valid) {
      console.log('✅ GGUF format - Ready to load');
      return true;
    } else {
      console.warn('❌ Invalid format:', status.format_validation?.message);
      return false;
    }
  }
  
  // Continue polling
  return null;
};

// 4. Load Model (if GGUF)
const loadResp = await fetch('http://localhost:8155/runtime/load/phi-2-gguf?threads=4', {
  method: 'POST'
});
const loadResult = await loadResp.json();

if (loadResult.status === 'ready') {
  console.log('✅ Model loaded on port', loadResult.port);
}
```

---

## 🚨 Important Rules for Editor Extension

### ✅ করণীয় (DO's)

1. **GGUF-Only Policy:**
   - শুধুমাত্র GGUF models download এবং load করতে হবে
   - `/registry/validate/{model}` দিয়ে format check করতে হবে
   - Invalid format হলে user কে warning দেখাতে হবে

2. **Download Completion:**
   - Download complete হলে automatically registry update হবে
   - `/download/status/{model}` থেকে `format_validation` check করতে হবে

3. **Registry Sync:**
   - Model install/uninstall হলে registry automatically update হবে
   - `/registry/models` endpoint use করে latest state পাওয়া যাবে

4. **Error Handling:**
   - SafeTensors models load করার চেষ্টা করলে clear error message দিতে হবে
   - User কে GGUF version download করতে suggest করতে হবে

### ❌ করণীয় নয় (DON'Ts)

1. **SafeTensors/PyTorch Models:**
   - SafeTensors বা PyTorch models load করার চেষ্টা করবেন না
   - এগুলো current runtime (llama.cpp) support করে না

2. **Manual Registry Edit:**
   - `model_registry.json` manually edit করবেন না
   - API endpoints ব্যবহার করুন

3. **Skip Format Validation:**
   - Download complete হওয়ার পরে format validation skip করবেন না
   - এটা guarantee করে যে model compatible

---

## 📦 Recommended GGUF Models

### Entry Level (2-3 GB RAM)
```json
{
  "tinyllama-1.1b-gguf": {
    "repo_id": "TheBloke/TinyLlama-1.1B-Chat-v1.0-GGUF",
    "filename": "tinyllama-1.1b-chat-v1.0.Q2_K.gguf",
    "size_mb": 641
  }
}
```

### Mid Range (4-6 GB RAM)
```json
{
  "phi-2-gguf": {
    "repo_id": "TheBloke/phi-2-GGUF",
    "filename": "phi-2.Q4_K_M.gguf",
    "size_mb": 1500
  },
  "llama-3.2-3b-gguf": {
    "repo_id": "bartowski/Llama-3.2-3B-Instruct-GGUF",
    "filename": "Llama-3.2-3B-Instruct-Q4_K_M.gguf",
    "size_mb": 2000
  }
}
```

---

## 🧪 Testing Commands

### Test Model Downloader
```bash
python scripts/model_downloader.py
```

### Test GGUF Loader
```bash
python scripts/gguf_loader.py "./models/tinyllama-gguf/tinyllama-1.1b-chat-v1.0.Q2_K.gguf" 8080 4
```

### Test Registry Manager
```python
from scripts.registry_manager import ModelRegistry

registry = ModelRegistry("models/model_registry.json")
print(registry.list_installed_models())
```

### Test API Endpoints
```bash
# Validate format
curl http://localhost:8155/registry/validate/tinyllama-gguf

# Get all models
curl http://localhost:8155/registry/models
```

---

## 📝 Notes for Extension Developer

1. **Server Base URL:** `http://localhost:8155`
2. **Default Model Port:** `8080` (auto-incremented for multiple models)
3. **llama.cpp Location:** `C:\model\config\llama.cpp\server.exe`
4. **Models Directory:** `C:\model\models\`
5. **Registry File:** `C:\model\models\model_registry.json`

---

## 🎯 Summary Checklist

- ✅ GGUF models download করা যায় HuggingFace থেকে
- ✅ Format validation automatic হয় download complete এর পরে
- ✅ Registry automatic update হয় install/uninstall এ
- ✅ SafeTensors models detect করে warning দেয়
- ✅ শুধুমাত্র GGUF models load হতে পারে llama.cpp দিয়ে
- ✅ সব API endpoints documented এবং tested

---

**Integration Complete! 🚀**

Editor ভাই এখন এই endpoints ব্যবহার করে seamless GGUF model management implement করতে পারবেন।

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zombiecoder1)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/zombiecoder1)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
