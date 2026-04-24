## cursor-batch-framework

> Best practices for creating Cursor Batch Framework jobs - metadata-driven coordinators, thin workers, and chaining patterns


# Cursor Batch Framework Best Practices

## Prefer Metadata-Driven Jobs (CursorJob)

Always prefer `CursorJob` over custom coordinators. Zero boilerplate — just configure metadata.

```apex
// ✅ GOOD: Metadata-driven - no coordinator class needed
// Job name is the MasterLabel (not DeveloperName)
CursorJob.run('Process Pending Accounts');

// ❌ AVOID: Custom coordinator when not needed
new MyCustomCoordinator().submit();
```

**Required metadata configuration** (`CursorBatch_Config__mdt`):
- `Query_Builder_Class__c` — Selector class implementing `ICursorBatchQueryBuilder` (or `ICursorBatchMetadataQueryBuilder` if it needs runtime parameters)
- `Query_Builder_Method__c` — Method name returning the query string
- `Worker_Class__c` — Worker class extending `CursorBatchWorker`

---

## Thin Worker Pattern

Workers should be thin — focus ONLY on processing logic, not infrastructure.

```apex
// ✅ GOOD: Thin worker - single responsibility
public class AccountStatusWorker extends CursorBatchWorker {
    public override void process(List<SObject> records) {
        List<Account> accounts = (List<Account>) records;
        for (Account acc : accounts) {
            acc.Status__c = 'Processed';
        }
        update accounts;
    }
}

// ❌ BAD: Fat worker with infrastructure concerns
public class AccountStatusWorker extends CursorBatchWorker {
    private String customQuery;
    private Integer customPageSize;
    
    public AccountStatusWorker(String query, Integer pageSize) {
        this.customQuery = query;  // Don't manage config in worker
        this.customPageSize = pageSize;
    }
    // ...
}
```

**Thin worker rules**:
1. No constructor needed — the inherited default constructor handles everything. Only add one if you need to call `setLogger()`.
2. `process()` does one thing: process records
3. No state management — use `CursorBatchContext` for execution parameters
4. Don't catch exceptions unless you want to suppress retry (see below)

---

## Automatic Exception Handling & Retry

The framework automatically catches exceptions in `process()` and retries with exponential backoff. Logging and retry are **free** — don't add infrastructure code.

```apex
// ✅ GOOD: Let exceptions bubble up - framework handles retry & logging
public override void process(List<SObject> records) {
    update records;  // If this fails, framework retries automatically
}

// ❌ BAD: Catching exceptions prevents automatic retry
public override void process(List<SObject> records) {
    try {
        update records;
    } catch (Exception e) {
        System.debug(e);  // Now framework can't retry!
    }
}

// ❌ BAD: Catch and re-throw is pointless - framework already does this
public override void process(List<SObject> records) {
    try {
        update records;
    } catch (Exception e) {
        Logger.error('Update failed', e);  // Framework logs automatically
        throw e;  // Unnecessary - just let it bubble up
    }
}

// ✅ OK: Catch only if you explicitly DON'T want retry
public override void process(List<SObject> records) {
    try {
        callExternalApi(records);
    } catch (CalloutException e) {
        // Intentionally swallow - don't retry this specific error
        markRecordsAsFailed(records, e.getMessage());
    }
}
```

**Built-in retry delays** (exponential backoff, capped at 10 min):
- Retry 1: `base` minutes (default base: 1 min)
- Retry 2: `base × 2` minutes
- Retry 3: `base × 4` minutes (capped at 10 min max)
- Formula: `min(base × 2^retryCount, 10)`
- Max retries controlled by `Worker_Max_Retries__c` (default: 3)

Configure `Worker_Retry_Delay__c` in `CursorBatch_Config__mdt` — no code changes needed.

**For caller-controlled retry** (e.g., rate limiting), throw `CursorBatchRetryException`:

```apex
if (rateLimited) {
    throw CursorBatchRetryException.create('Rate limited', 5); // Retry in 5 mins
}
```

---

## Chaining Options (Cleanest to Most Flexible)

### Option 1: Chain_To_Job__c (Cleanest — Metadata Only)

Configure in `CursorBatch_Config__mdt`:
- `Chain_To_Job__c` = `'Next Job Name'` (uses **MasterLabel**, not DeveloperName)

No code required. When job completes → automatically runs next job.

### Option 2: Worker finish() Method (When Logic Needed)

Override `finish()` in worker for conditional chaining:

```apex
public override void finish(CursorBatch_Job__c jobRecord) {
    if (jobRecord.Status__c == 'Completed') {
        if (shouldRunFollowUp(jobRecord)) {
            CursorJob.run('Follow Up Job');  // MasterLabel, not DeveloperName
        }
    }
}
```

### Option 3: Callable Class (Complex Chaining Rules)

Create dedicated class for reusable chaining logic:

```apex
public class BatchChainRouter implements Callable {
    public Object call(String action, Map<String, Object> args) {
        CursorBatch_Job__c job = (CursorBatch_Job__c) args.get('jobRecord');
        
        switch on action {
            when 'routeAccountJobs' {
                return routeAccountJobs(job);
            }
            when 'routeBillingJobs' {
                return routeBillingJobs(job);
            }
        }
        return null;
    }
    
    private Object routeAccountJobs(CursorBatch_Job__c job) {
        if (job.Records_Processed__c > 0) {
            CursorJob.run('Account Notification Job');  // MasterLabel
        }
        return null;
    }
}
```

Configure metadata:
- `Chain_To_Class__c` = `'BatchChainRouter'`
- `Chain_To_Method__c` = `'routeAccountJobs'`

---

## Query Builder Pattern

Implement `ICursorBatchQueryBuilder` in selector classes:

```apex
public class AccountSelector implements ICursorBatchQueryBuilder {
    public String buildQuery(String methodName) {
        switch on methodName {
            when 'pendingAccounts' { return buildPendingAccountsQuery(); }
            when 'staleAccounts' { return buildStaleAccountsQuery(); }
            when else { return null; }
        }
    }
    
    private String buildPendingAccountsQuery() {
        // IMPORTANT: Use inline values, NOT bind variables
        return 'SELECT Id, Name FROM Account WHERE Status__c = \'Pending\'';
    }
}
```

---

## Job Invocation Metadata

Pass runtime parameters (record IDs, flags, context) to jobs via metadata JSON. The metadata is persisted on the `CursorBatch_Job__c` record and carried to every worker.

```apex
// ✅ Invoke with metadata — serialized to JSON internally
CursorJob.run('Process Account Children', new Map<String, Object>{
    'accountId' => parentAccountId
});

// With delay
CursorJob.runWithDelay('Sync Records', 5, new Map<String, Object>{
    'batchId' => myBatchId
});

// Custom coordinator with metadata (setMetadata accepts a JSON string)
MyCoordinator coord = new MyCoordinator('Job Name');
coord.setMetadata('{"recordId": "001xxx"}');
coord.submit();
```

### Metadata-Aware Query Builder

When the query needs runtime parameters, implement `ICursorBatchMetadataQueryBuilder` instead of `ICursorBatchQueryBuilder`. The framework deserializes the metadata JSON once and passes a ready-to-use `Map<String, Object>` (empty map when no metadata, never null):

```apex
public class ContactsByAccountSelector implements ICursorBatchMetadataQueryBuilder {
    public String buildQuery(String methodName, Map<String, Object> metadata) {
        String accountId = (String) metadata.get('accountId');

        switch on methodName {
            when 'getContactsByAccount' {
                if (String.isBlank(accountId)) {
                    return null;
                }
                return 'SELECT Id, Name FROM Contact WHERE AccountId = \'' +
                       String.escapeSingleQuotes(accountId) + '\'';
            }
            when else { return null; }
        }
    }
}
```

Existing query builders implementing `ICursorBatchQueryBuilder` continue to work unchanged — the framework checks for the metadata interface first and falls back automatically.

### Accessing Metadata in Workers

Metadata scopes the query (query builder's job). Workers typically don't need it during `process()` — they just process whatever records the cursor returns. Use `getMetadata()` in `finish()` to post results back to the originating record:

```apex
public class MyWorker extends CursorBatchWorker {
    public override void process(List<SObject> records) {
        update records;
    }

    public override void finish(CursorBatch_Job__c jobRecord) {
        Map<String, Object> state = getCurrentState();
        Map<String, Object> meta  = getMetadata();
        if (meta != null) {
            Id targetId = (Id) meta.get('recordId');
        }
    }
}
```

`getMetadata()` and `getCurrentState()` work in both `process()` and `finish()`. Metadata is automatically preserved across pages and retries.

---

## Parent/Child Pattern (Avoid Record Locks)

When processing child records, query **parents** to avoid `UNABLE_TO_LOCK_ROW`:

```apex
// Coordinator queries parents
public String buildQuery() {
    return 'SELECT Id FROM Account WHERE Id IN ' +
           '(SELECT AccountId FROM Opportunity WHERE StageName = \'Prospecting\')';
}

// Worker queries children of received parents
public override void process(List<SObject> records) {
    Set<Id> accountIds = new Map<Id, SObject>(records).keySet();
    List<Opportunity> opps = [
        SELECT Id, StageName FROM Opportunity 
        WHERE AccountId IN :accountIds AND StageName = 'Prospecting'
    ];
    // Process opportunities - all children of same parent in same worker
}
```

---

## New Job Checklist

1. [ ] Create `CursorBatch_Config__mdt` record with `Active__c = true` (MasterLabel is the job name used in code)
2. [ ] Implement `ICursorBatchQueryBuilder` in selector (or extend `CursorBatchCoordinator`)
3. [ ] Create thin worker extending `CursorBatchWorker`
4. [ ] Configure chaining via metadata (`Chain_To_Job__c`) when possible
5. [ ] Use parent/child pattern for child record processing
6. [ ] Set appropriate `Parallel_Count__c` and `Page_Size__c`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ndrg2010) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
