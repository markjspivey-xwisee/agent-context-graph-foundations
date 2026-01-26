# Gas Town Inspirations: Principled Adoption

This document describes how Agent Context Graph can adopt operational patterns from Gas Town
while remaining true to our architectural pillars.

---

## 1. Execution Enclaves (inspired by Git Worktree Isolation)

### Gas Town Pattern
Each Polecat (worker agent) operates in its own git worktree, preventing file conflicts
and enabling parallel execution without coordination overhead.

### ACG Principled Adoption: "Execution Enclaves"

An **Execution Enclave** is a DID-bound, isolated execution environment with full PROV tracking.

#### Pillar Mapping
| Pillar | How It's Honored |
|--------|------------------|
| **B - Context Graphs** | Each enclave has its own Context Graph scope |
| **G - Provenance** | Enclave creation/destruction are PROV Activities |
| **H - DIDs** | Enclave is bound to agent DID; files are signed |
| **J - Threat Model** | Isolation prevents confused deputy attacks |

#### Design

```turtle
# New ontology class
acg:ExecutionEnclave a owl:Class ;
    rdfs:subClassOf prov:Entity ;
    rdfs:comment "Isolated execution environment bound to an agent DID" .

acg:hasEnclave a owl:ObjectProperty ;
    rdfs:domain acg:Agent ;
    rdfs:range acg:ExecutionEnclave .

acg:enclaveWorktree a owl:DatatypeProperty ;
    rdfs:domain acg:ExecutionEnclave ;
    rdfs:range xsd:string ;
    rdfs:comment "Git worktree path for this enclave" .

acg:enclaveCreatedBy a owl:ObjectProperty ;
    rdfs:subPropertyOf prov:wasGeneratedBy ;
    rdfs:domain acg:ExecutionEnclave ;
    rdfs:range prov:Activity .
```

#### New Affordance: `CreateEnclave`

```json
{
  "actionType": "CreateEnclave",
  "requiresCredential": [
    { "schema": "ExecutorCapability", "issuer": "did:web:authority.example.com" }
  ],
  "params": {
    "shaclRef": "https://agentcontextgraph.dev/shacl/params#CreateEnclaveParamsShape"
  },
  "effects": [
    { "type": "resource-create", "description": "Creates isolated git worktree" },
    { "type": "prov-emit", "description": "Emits EnclaveCreated trace" }
  ]
}
```

#### Key Difference from Gas Town
- Gas Town: Worktrees are infrastructure detail
- ACG: Enclaves are **first-class entities** with DIDs, traces, and credential requirements

#### Implementation Sketch

```typescript
interface ExecutionEnclave {
  id: string;                    // URN for this enclave
  boundAgentDID: string;         // Agent that owns this enclave
  worktreePath: string;          // Git worktree location
  createdAt: string;             // ISO timestamp
  createdByTrace: string;        // PROV trace ID that created it
  contextScope: string[];        // Resource URNs this enclave can access
  status: 'active' | 'sealed' | 'destroyed';
}

// Enclave creation is an affordance traversal
const enclaveAffordance = context.affordances.find(a => a.actionType === 'CreateEnclave');
const result = await broker.traverse({
  contextId: context.id,
  affordanceId: enclaveAffordance.id,
  parameters: {
    baseRepository: 'git@github.com:org/repo.git',
    branchPrefix: `agent/${agentDID.split(':').pop()}`
  }
});
// Result includes enclave ID and worktree path
// PROV trace automatically emitted
```

---

## 2. Resumable Context Graphs (inspired by Beads/Hooks)

### Gas Town Pattern
Beads persist work state in git-backed SQLite. Hooks are worktrees that survive crashes.
Agents can resume work without relying on memory.

### ACG Principled Adoption: "Context Checkpoints"

A **Context Checkpoint** is an immutable snapshot of a Context Graph plus agent state,
enabling crash recovery while maintaining full provenance.

#### Pillar Mapping
| Pillar | How It's Honored |
|--------|------------------|
| **B - Context Graphs** | Checkpoints are Context Graph snapshots |
| **C - HATEOAS** | Resume affordance appears dynamically when checkpoint exists |
| **G - Provenance** | Checkpoint/Resume are PROV Activities with derivation chains |
| **I - Semiotics** | Checkpoint labels are versioned conventions |

#### Design

```turtle
acg:ContextCheckpoint a owl:Class ;
    rdfs:subClassOf prov:Entity ;
    rdfs:comment "Immutable snapshot of a Context Graph for recovery" .

acg:checkpointOf a owl:ObjectProperty ;
    rdfs:subPropertyOf prov:wasDerivedFrom ;
    rdfs:domain acg:ContextCheckpoint ;
    rdfs:range acg:ContextGraph .

acg:checkpointState a owl:DatatypeProperty ;
    rdfs:domain acg:ContextCheckpoint ;
    rdfs:range xsd:string ;
    rdfs:comment "Serialized agent state at checkpoint time" .

acg:resumedFrom a owl:ObjectProperty ;
    rdfs:domain acg:ContextGraph ;
    rdfs:range acg:ContextCheckpoint ;
    rdfs:comment "Indicates this context was resumed from a checkpoint" .
```

#### New Affordances: `Checkpoint` and `Resume`

```json
{
  "actionType": "Checkpoint",
  "effects": [
    { "type": "resource-create", "description": "Creates immutable checkpoint" },
    { "type": "prov-emit", "description": "Emits CheckpointCreated trace" }
  ]
}
```

```json
{
  "actionType": "Resume",
  "requiresCredential": [
    { "schema": "SameAgentAsCheckpoint" }
  ],
  "effects": [
    { "type": "state-restore", "description": "Restores context from checkpoint" },
    { "type": "prov-emit", "description": "Emits ContextResumed trace with derivation" }
  ]
}
```

#### Key Difference from Gas Town
- Gas Town: Beads are opaque blobs in SQLite
- ACG: Checkpoints are **PROV Entities** with derivation chains and semantic structure

#### Storage Design

```typescript
interface ContextCheckpoint {
  id: string;                    // urn:checkpoint:...
  contextGraphId: string;        // The context this checkpoints
  agentDID: string;              // Who created this checkpoint
  timestamp: string;             // When

  // Serialized state
  contextSnapshot: ContextGraph; // Full context at checkpoint time
  agentState: {                  // Agent-specific resumable state
    taskQueue: Task[];
    completedTasks: string[];
    workingMemory: Record<string, unknown>;
  };

  // PROV metadata
  createdByActivity: string;     // Trace ID of the checkpoint action
  supersedes?: string;           // Previous checkpoint (if any)

  // Integrity
  contentHash: string;           // SHA-256 of serialized state
  signature?: string;            // Agent's signature (optional)
}

// Resume creates a new context derived from checkpoint
interface ResumedContext extends ContextGraph {
  resumedFrom: string;           // Checkpoint ID
  resumeActivity: string;        // Trace ID of the resume action
}
```

#### PROV Trace for Resume

```json
{
  "@context": ["https://www.w3.org/ns/prov#"],
  "type": ["prov:Activity", "acg:ResumeActivity"],
  "used": {
    "type": "acg:ContextCheckpoint",
    "id": "urn:checkpoint:abc123"
  },
  "generated": {
    "type": "acg:ContextGraph",
    "id": "urn:context:xyz789",
    "prov:wasDerivedFrom": "urn:checkpoint:abc123"
  },
  "wasAssociatedWith": {
    "agentDID": "did:key:z6Mk..."
  }
}
```

---

## 3. Concurrent Context Graphs (inspired by Parallel Spawning)

### Gas Town Pattern
The Mayor spawns 20-30 Polecats in parallel via tmux. Work is distributed and
merged back through the Refinery agent.

### ACG Principled Adoption: "AAT-Aware Parallelism"

Parallel execution is permitted where **AAT behavioral contracts allow** and
**policy does not forbid**, with the broker managing concurrent context graphs.

#### Pillar Mapping
| Pillar | How It's Honored |
|--------|------------------|
| **A - AAT** | Parallelism rules derived from AAT composition constraints |
| **B - Context Graphs** | Each parallel agent has independent context |
| **F - Policy** | Concurrency limits are policy-controlled |
| **J - Threat Model** | Race conditions prevented by enclave isolation |

#### Design Principle: Parallelism as AAT Property

Not all agents can run in parallel. The AAT specification must declare:

```json
{
  "id": "aat:ExecutorAgentType",
  "compositionRules": {
    "parallelizable": true,
    "maxConcurrent": 10,
    "requiresIsolation": true,
    "conflictsWith": ["aat:ArbiterAgentType"]
  }
}
```

#### New Ontology

```turtle
aat:parallelizable a owl:DatatypeProperty ;
    rdfs:domain aat:AbstractAgentType ;
    rdfs:range xsd:boolean ;
    rdfs:comment "Whether multiple agents of this type can run concurrently" .

aat:maxConcurrent a owl:DatatypeProperty ;
    rdfs:domain aat:AbstractAgentType ;
    rdfs:range xsd:integer ;
    rdfs:comment "Maximum concurrent instances allowed" .

aat:conflictsWith a owl:ObjectProperty ;
    rdfs:domain aat:AbstractAgentType ;
    rdfs:range aat:AbstractAgentType ;
    rdfs:comment "Agent types that cannot run concurrently with this type" .

acg:ConcurrencyPolicy a owl:Class ;
    rdfs:subClassOf acg:Policy ;
    rdfs:comment "Policy governing parallel agent execution" .
```

#### Orchestrator Enhancement

```typescript
interface ConcurrencyPolicy {
  maxTotalAgents: number;        // e.g., 30
  maxPerType: Record<string, number>;  // e.g., { executor: 10, planner: 2 }
  conflictMatrix: Record<string, string[]>;  // Which types can't coexist
  resourceLimits: {
    maxTokensPerMinute: number;
    maxCostPerHour: number;       // Gas Town burns $100/hr - we can cap this
  };
}

class ConcurrentOrchestrator {
  private activeContexts: Map<string, ContextGraph> = new Map();
  private enclaves: Map<string, ExecutionEnclave> = new Map();

  async spawnParallelAgents(
    tasks: Task[],
    policy: ConcurrencyPolicy
  ): Promise<void> {
    // Group tasks by AAT type
    const tasksByType = this.groupByAgentType(tasks);

    // Check AAT parallelization rules
    for (const [type, typeTasks] of tasksByType) {
      const aat = await this.aatRegistry.getAAT(type);
      if (!aat.compositionRules?.parallelizable) {
        // Must run sequentially
        await this.runSequential(typeTasks);
        continue;
      }

      // Respect maxConcurrent from AAT and policy
      const maxConcurrent = Math.min(
        aat.compositionRules.maxConcurrent ?? Infinity,
        policy.maxPerType[type] ?? policy.maxTotalAgents
      );

      // Check for conflicts with currently running agents
      const conflicts = aat.compositionRules.conflictsWith ?? [];
      const runningConflicts = conflicts.filter(c =>
        this.hasRunningAgentOfType(c)
      );

      if (runningConflicts.length > 0) {
        // Queue for later
        await this.queueBehindConflicts(typeTasks, runningConflicts);
        continue;
      }

      // Spawn parallel agents with isolated enclaves
      await this.spawnBatch(typeTasks, maxConcurrent);
    }
  }

  private async spawnBatch(tasks: Task[], maxConcurrent: number): Promise<void> {
    const batches = this.chunk(tasks, maxConcurrent);

    for (const batch of batches) {
      // Create enclave for each agent
      const agents = await Promise.all(batch.map(async task => {
        const enclave = await this.createEnclave(task.agentDID);
        const context = await this.broker.getContext({
          agentDID: task.agentDID,
          agentType: task.agentType,
          scope: { enclave: enclave.id }
        });

        return this.runAgentInEnclave(task, context, enclave);
      }));

      // Wait for batch to complete
      await Promise.all(agents);
    }
  }
}
```

#### Key Difference from Gas Town
- Gas Town: Spawn as many Polecats as tmux can handle
- ACG: Parallelism is **AAT-constrained** and **policy-governed** with formal conflict detection

---

## Summary: Principled Adoption Matrix

| Gas Town Feature | ACG Adoption | Key Principle Preserved |
|------------------|--------------|------------------------|
| Git worktrees | Execution Enclaves | DIDs bind enclaves; PROV traces creation |
| Beads/Hooks | Context Checkpoints | Checkpoints are PROV Entities with derivation |
| Polecat spawning | AAT-Aware Parallelism | Parallelism rules in AAT spec, not hardcoded |
| Mayor coordination | Orchestrator + Broker | HATEOAS discovery, not command dispatch |
| Refinery merging | Merge affordance | Merge is credential-gated, traced |

---

## Implementation Priority

### Phase 1: Execution Enclaves
- Add `acg:ExecutionEnclave` to ontology
- Add `CreateEnclave` / `DestroyEnclave` affordances
- Implement git worktree creation in ActionExecutor
- Update PROV traces to include enclave context

### Phase 2: Context Checkpoints
- Add checkpoint storage (extend TraceStore or new CheckpointStore)
- Add `Checkpoint` / `Resume` affordances
- Implement context serialization/deserialization
- Add derivation chains to PROV traces

### Phase 3: Concurrent Context Graphs
- Add `parallelizable`, `maxConcurrent`, `conflictsWith` to AAT spec
- Add `ConcurrencyPolicy` to policy engine
- Implement concurrent orchestrator
- Add cost/token limiting (learn from Gas Town's $100/hr burn rate)

---

## What We Explicitly Do NOT Adopt

1. **Opaque SQLite state**: Our checkpoints are semantic PROV entities
2. **Tmux as infrastructure**: Our parallelism is AAT-declared, not tmux-limited
3. **Implicit agent identity**: Our enclaves are DID-bound
4. **Watchers-on-watchers**: Our monitoring is Observer AAT, not meta-agents
5. **Untraced merges**: Our merges are affordance traversals with full PROV

---

## References

- [Gas Town GitHub](https://github.com/steveyegge/gastown)
- [Two Kinds of Multi-Agent](https://paddo.dev/blog/gastown-two-kinds-of-multi-agent/)
- [ARCHITECTURE_INDEX.md](./ARCHITECTURE_INDEX.md) - Our architectural pillars
