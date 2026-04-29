## million-merkle-tree

> You are working on a Hardhat-based Merkle Tree Gas Analysis project that focuses on realistic, production-scale gas measurement for Merkle tree operations on Ethereum. The project uses an external data generation approach where trees are created off-chain in TypeScript and only verification operations are tested on-chain.

# Merkle Tree Gas Analysis Project - Cursor AI Rules

## Project Overview

You are working on a Hardhat-based Merkle Tree Gas Analysis project that focuses on realistic, production-scale gas measurement for Merkle tree operations on Ethereum. The project uses an external data generation approach where trees are created off-chain in TypeScript and only verification operations are tested on-chain.

### Core Technologies
- **Framework**: Hardhat (NOT Foundry) with TypeScript
- **Language**: TypeScript for scripts/tests, Solidity 0.8.24 for contracts
- **Libraries**: 
  - OpenZeppelin Contracts v5.1+ (on-chain verification)
  - OpenZeppelin Merkle Tree JS (off-chain generation)
  - hardhat-gas-reporter (gas analysis)
- **Testing**: Mocha/Chai with Hardhat's testing framework
- **Integration**: Context7 MCP for AI-assisted development

### Project Goals
1. Analyze gas costs for Merkle trees up to depth 20 (1,048,576 leaves)
2. Provide statistical analysis of gas scaling patterns
3. Test real-world integration scenarios (airdrops, voting, whitelists)
4. Create reusable patterns and benchmarks

## Development Standards

### TypeScript Standards

#### File Structure
```typescript
// Always use explicit imports
import { ethers } from "hardhat";
import { StandardMerkleTree } from "@openzeppelin/merkle-tree";
import { BigNumber } from "ethers";

// Group imports: external, internal, types
import type { MerkleVerifier } from "../typechain-types";
```

#### Naming Conventions
- Files: kebab-case (e.g., `generate-trees.ts`, `gas-analysis.test.ts`)
- Classes: PascalCase (e.g., `TreeGenerator`, `ProofSampler`)
- Functions: camelCase (e.g., `generateTreesForDepth`, `analyzeGasScaling`)
- Constants: UPPER_SNAKE_CASE (e.g., `MAX_TREE_DEPTH`, `DEFAULT_SAMPLE_SIZE`)

#### Type Safety
```typescript
// Always define interfaces for data structures
interface TreeData {
  depth: number;
  root: string;
  leafCount: number;
  sampleProofs: ProofData[];
  metadata: TreeMetadata;
}

// Use discriminated unions for variants
type AnalysisResult = 
  | { type: "success"; data: GasReport }
  | { type: "error"; error: string };

// Prefer readonly arrays and objects
function processProofs(proofs: readonly Proof[]): void {
  // Implementation
}
```

### Solidity Standards

#### Contract Structure
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import {MerkleProof} from "@openzeppelin/contracts/utils/cryptography/MerkleProof.sol";

/**
 * @title MerkleVerifier
 * @notice Gas-optimized Merkle proof verification
 * @dev Focuses on minimal gas consumption for analysis
 */
contract MerkleVerifier {
    // State variables
    mapping(bytes32 => bool) public roots;
    
    // Events
    event RootUpdated(bytes32 indexed root);
    
    // Custom errors (gas-efficient)
    error InvalidProof();
    error RootNotSet();
    
    // External functions
    function setRoot(bytes32 _root) external {
        roots[_root] = true;
        emit RootUpdated(_root);
    }
    
    // View functions
    function verify(
        bytes32[] calldata proof,
        bytes32 leaf
    ) external view returns (bool) {
        // Implementation
    }
}
```

#### Gas Optimization Patterns
```solidity
// Use calldata for read-only arrays
function verifyBatch(
    bytes32[][] calldata proofs,
    bytes32[] calldata leaves
) external view returns (bool[] memory results) {
    // Avoid storage reads in loops
    uint256 length = proofs.length;
    results = new bool[](length);
    
    // Cache frequently accessed values
    for (uint256 i; i < length;) {
        results[i] = _verify(proofs[i], leaves[i]);
        unchecked { ++i; }
    }
}
```

## Hardhat Integration Requirements

### Configuration Pattern
```typescript
// hardhat.config.ts
import { HardhatUserConfig } from "hardhat/config";
import "@nomicfoundation/hardhat-toolbox";
import "hardhat-gas-reporter";

const config: HardhatUserConfig = {
  solidity: {
    version: "0.8.24",
    settings: {
      optimizer: {
        enabled: true,
        runs: 200, // Optimize for deployment
      },
      viaIR: true, // Enable IR optimizer for complex contracts
    },
  },
  gasReporter: {
    enabled: process.env.REPORT_GAS === "true",
    currency: "USD",
    gasPrice: 30,
    outputFile: "data/gas-reports/latest.txt",
    noColors: true,
  },
  networks: {
    hardhat: {
      blockGasLimit: 30_000_000, // Support large operations
    },
  },
};

export default config;
```

### Test Patterns
```typescript
// test/gas-analysis/verification-scaling.test.ts
import { expect } from "chai";
import { ethers } from "hardhat";
import { loadFixture } from "@nomicfoundation/hardhat-network-helpers";
import { loadTreeData } from "../../scripts/utils/DataPersistence";

describe("Verification Gas Scaling", function () {
  // Fixture pattern for consistent test setup
  async function deployVerifierFixture() {
    const [deployer] = await ethers.getSigners();
    const Verifier = await ethers.getContractFactory("MerkleVerifier");
    const verifier = await Verifier.deploy();
    await verifier.deployed();
    
    return { verifier, deployer };
  }
  
  // Test multiple tree depths
  const depths = [5, 10, 15, 20];
  
  depths.forEach((depth) => {
    it(`should verify proof for depth ${depth} tree`, async function () {
      const { verifier } = await loadFixture(deployVerifierFixture);
      const treeData = await loadTreeData(depth);
      
      // Set root
      await verifier.setRoot(treeData.root);
      
      // Measure verification gas
      const tx = await verifier.verify(
        treeData.sampleProofs[0].proof,
        treeData.sampleProofs[0].leaf
      );
      
      const receipt = await tx.wait();
      console.log(`Depth ${depth} verification gas: ${receipt.gasUsed}`);
      
      // Verify logarithmic scaling
      const expectedGas = 700 * depth; // ~700 gas per proof element
      expect(receipt.gasUsed).to.be.closeTo(expectedGas, expectedGas * 0.1);
    });
  });
});
```

## External Data Management Best Practices

### Tree Generation Pattern
```typescript
// scripts/01-generate-trees.ts
import { StandardMerkleTree } from "@openzeppelin/merkle-tree";
import { ethers } from "ethers";
import fs from "fs/promises";
import path from "path";

export async function generateTreeForDepth(depth: number): Promise<void> {
  console.log(`Generating tree for depth ${depth}...`);
  
  const leafCount = Math.pow(2, depth);
  const leaves: [string, string][] = [];
  
  // Generate leaves in batches to manage memory
  const batchSize = Math.min(10000, leafCount);
  
  for (let i = 0; i < leafCount; i += batchSize) {
    const batch: [string, string][] = [];
    const end = Math.min(i + batchSize, leafCount);
    
    for (let j = i; j < end; j++) {
      const address = ethers.Wallet.createRandom().address;
      const value = ethers.utils.parseEther(String(j)).toString();
      batch.push([address, value]);
    }
    
    leaves.push(...batch);
    
    // Progress reporting
    if (i % 100000 === 0 && i > 0) {
      console.log(`  Generated ${i} / ${leafCount} leaves`);
    }
  }
  
  // Create tree
  const tree = StandardMerkleTree.of(leaves, ["address", "uint256"]);
  
  // Save tree data
  await saveTreeData(depth, tree, leaves);
}

async function saveTreeData(
  depth: number,
  tree: StandardMerkleTree<[string, string]>,
  leaves: [string, string][]
): Promise<void> {
  const dataDir = path.join("data", "trees", `depth-${depth}`);
  await fs.mkdir(dataDir, { recursive: true });
  
  // Save root
  const treeData = {
    depth,
    root: tree.root,
    leafCount: leaves.length,
    format: tree.leafEncoding,
    metadata: {
      generatedAt: Date.now(),
      version: "1.0.0",
    },
  };
  
  await fs.writeFile(
    path.join(dataDir, "tree.json"),
    JSON.stringify(treeData, null, 2)
  );
  
  // Sample and save proofs
  const proofs = await sampleProofs(tree, Math.min(100, leafCount));
  await fs.writeFile(
    path.join(dataDir, "proofs.json"),
    JSON.stringify(proofs, null, 2)
  );
}
```

### Data Loading Pattern
```typescript
// scripts/utils/DataPersistence.ts
export class DataPersistence {
  private static cache = new Map<string, TreeData>();
  
  static async loadTreeData(depth: number): Promise<TreeData> {
    const cacheKey = `depth-${depth}`;
    
    // Check cache first
    if (this.cache.has(cacheKey)) {
      return this.cache.get(cacheKey)!;
    }
    
    // Load from disk
    const dataDir = path.join("data", "trees", `depth-${depth}`);
    const treeData = await fs.readFile(
      path.join(dataDir, "tree.json"),
      "utf-8"
    );
    const proofData = await fs.readFile(
      path.join(dataDir, "proofs.json"),
      "utf-8"
    );
    
    const result: TreeData = {
      ...JSON.parse(treeData),
      sampleProofs: JSON.parse(proofData),
    };
    
    // Cache for future use
    this.cache.set(cacheKey, result);
    
    return result;
  }
}
```

## Testing Patterns for Gas Analysis

### Gas Measurement Pattern
```typescript
// test/gas-analysis/base-gas-measurement.ts
export abstract class GasMeasurement {
  protected gasReports: GasReport[] = [];
  
  protected async measureGas(
    operation: string,
    fn: () => Promise<any>
  ): Promise<number> {
    const tx = await fn();
    const receipt = await tx.wait();
    
    const report: GasReport = {
      operation,
      gasUsed: receipt.gasUsed.toNumber(),
      timestamp: Date.now(),
    };
    
    this.gasReports.push(report);
    return report.gasUsed;
  }
  
  protected analyzeGasReports(): GasAnalysis {
    const gasCosts = this.gasReports.map(r => r.gasUsed);
    
    return {
      min: Math.min(...gasCosts),
      max: Math.max(...gasCosts),
      mean: gasCosts.reduce((a, b) => a + b, 0) / gasCosts.length,
      median: this.calculateMedian(gasCosts),
      p95: this.calculatePercentile(gasCosts, 95),
      p99: this.calculatePercentile(gasCosts, 99),
    };
  }
}
```

### Statistical Analysis Pattern
```typescript
// scripts/utils/StatisticalAnalysis.ts
export class StatisticalAnalysis {
  static analyzeScalingPattern(
    depths: number[],
    gasCosts: number[]
  ): ScalingAnalysis {
    // Linear regression for O(log n) validation
    const logDepths = depths.map(d => Math.log2(Math.pow(2, d)));
    const regression = this.linearRegression(logDepths, gasCosts);
    
    // Calculate R-squared
    const rSquared = this.calculateRSquared(
      gasCosts,
      logDepths.map(d => regression.slope * d + regression.intercept)
    );
    
    return {
      slope: regression.slope,
      intercept: regression.intercept,
      rSquared,
      predictedGas: (depth: number) => {
        return Math.round(regression.slope * depth + regression.intercept);
      },
    };
  }
}
```

## Code Quality Standards

### Error Handling
```typescript
// Always use custom errors with context
export class TreeGenerationError extends Error {
  constructor(
    public depth: number,
    public reason: string,
    public cause?: Error
  ) {
    super(`Failed to generate tree at depth ${depth}: ${reason}`);
    this.name = "TreeGenerationError";
  }
}

// Wrap operations with proper error handling
export async function safeTreeOperation<T>(
  operation: () => Promise<T>,
  context: string
): Promise<T> {
  try {
    return await operation();
  } catch (error) {
    console.error(`Error in ${context}:`, error);
    throw new TreeGenerationError(
      0,
      `Operation failed: ${context}`,
      error as Error
    );
  }
}
```

### Logging Standards
```typescript
// Use structured logging
import { createLogger } from "./utils/Logger";

const logger = createLogger("TreeGenerator");

logger.info("Starting tree generation", {
  depth,
  expectedLeaves: Math.pow(2, depth),
  timestamp: new Date().toISOString(),
});

logger.debug("Batch processed", {
  batchNumber: i / batchSize,
  processedLeaves: i,
  remainingLeaves: leafCount - i,
});

logger.error("Generation failed", {
  depth,
  error: error.message,
  stack: error.stack,
});
```

## Context7 MCP Integration Guidelines

### Pattern Recognition
```typescript
// Mark patterns for Context7 analysis
/**
 * @context7-pattern gas-optimization
 * @description Batch verification reduces per-proof gas costs
 * @optimization-potential 30-40% gas reduction
 */
function batchVerify(proofs: Proof[]): boolean[] {
  // Implementation
}
```

### Workflow Automation
```typescript
// scripts/context7-workflow.ts
export const workflowSteps = {
  generateTrees: {
    description: "Generate Merkle trees for specified depths",
    inputs: ["depths[]"],
    outputs: ["treeData[]"],
  },
  runGasAnalysis: {
    description: "Execute gas measurements on generated trees",
    inputs: ["treeData[]"],
    outputs: ["gasReports[]"],
  },
  generateReport: {
    description: "Create comprehensive analysis report",
    inputs: ["gasReports[]"],
    outputs: ["analysisReport"],
  },
};
```

## File Naming Conventions

### Scripts
- Tree generation: `XX-generate-[specifics].ts` (e.g., `01-generate-trees.ts`)
- Deployment: `XX-deploy-[contract].ts` (e.g., `02-deploy-contracts.ts`)
- Analysis: `XX-run-[analysis-type].ts` (e.g., `03-run-gas-analysis.ts`)

### Tests
- Gas analysis: `[operation]-scaling.test.ts`
- Integration: `[scenario]-simulation.test.ts`
- Performance: `[scale]-performance.test.ts`

### Data Files
- Trees: `data/trees/depth-[N]/tree.json`
- Proofs: `data/trees/depth-[N]/proofs.json`
- Reports: `data/gas-reports/[YYYY-MM-DD]-[analysis-name].json`

## Development Workflow

### Daily Process
1. **Morning Setup**
   ```bash
   # Pull latest changes
   git pull origin main
   
   # Install any new dependencies
   npm install
   
   # Compile contracts
   npx hardhat compile
   
   # Run quick sanity tests
   npm run test:quick
   ```

2. **Development Cycle**
   ```bash
   # Generate new tree data if needed
   npm run generate:trees -- --depth 15
   
   # Run specific tests during development
   npx hardhat test test/gas-analysis/verification-scaling.test.ts
   
   # Check gas reports
   npm run analyze:gas
   ```

3. **Pre-Commit Checks**
   ```bash
   # Run linting
   npm run lint
   
   # Run all tests
   npm test
   
   # Generate gas report
   REPORT_GAS=true npm test
   
   # Update documentation
   npm run docs:generate
   ```

### Branch Strategy
- `main`: Production-ready code
- `develop`: Integration branch
- `feature/[name]`: New features
- `analysis/[test-name]`: Specific gas analysis experiments
- `optimize/[optimization]`: Gas optimization attempts

## Error Handling Patterns

### Contract Errors
```solidity
// Use custom errors for gas efficiency
error InvalidProof();
error RootNotSet();
error BatchSizeMismatch(uint256 proofsLength, uint256 leavesLength);

// Revert with descriptive errors
function verify(bytes32[] calldata proof, bytes32 leaf) external view returns (bool) {
    if (!roots[root]) revert RootNotSet();
    if (!MerkleProof.verify(proof, root, leaf)) revert InvalidProof();
    return true;
}
```

### TypeScript Errors
```typescript
// Comprehensive error handling
export async function generateAndSaveTree(depth: number): Promise<TreeData> {
  // Validate input
  if (depth < 1 || depth > 22) {
    throw new RangeError(`Tree depth must be between 1 and 22, got ${depth}`);
  }
  
  try {
    // Check available memory for large trees
    if (depth > 18) {
      const requiredMemory = estimateMemoryUsage(depth);
      const availableMemory = process.memoryUsage().heapTotal;
      
      if (requiredMemory > availableMemory * 0.8) {
        throw new Error(
          `Insufficient memory for depth ${depth} tree. ` +
          `Required: ${formatBytes(requiredMemory)}, ` +
          `Available: ${formatBytes(availableMemory)}`
        );
      }
    }
    
    const tree = await generateTree(depth);
    const data = await saveTree(tree);
    
    return data;
  } catch (error) {
    // Log detailed error information
    logger.error("Tree generation failed", {
      depth,
      error: error instanceof Error ? error.message : String(error),
      stack: error instanceof Error ? error.stack : undefined,
      memoryUsage: process.memoryUsage(),
    });
    
    // Re-throw with context
    throw new TreeGenerationError(depth, "Generation failed", error as Error);
  }
}
```

### Test Error Handling
```typescript
// Proper test error handling
describe("Large Scale Tree Tests", function () {
  // Increase timeout for large operations
  this.timeout(300000); // 5 minutes
  
  beforeEach(async function () {
    // Check if tree data exists
    try {
      await loadTreeData(20);
    } catch (error) {
      // Skip test if data not available
      this.skip("Tree data for depth 20 not available. Run generate:trees first.");
    }
  });
  
  it("should handle memory constraints gracefully", async function () {
    try {
      const result = await testLargeTreeOperation();
      expect(result).to.have.property("gasUsed");
    } catch (error) {
      // Distinguish between expected and unexpected errors
      if (error instanceof OutOfMemoryError) {
        console.warn("Test skipped due to memory constraints");
        this.skip();
      } else {
        throw error; // Re-throw unexpected errors
      }
    }
  });
});
```

## Performance Optimization Priorities

### Memory Optimization
```typescript
// Stream processing for large datasets
export async function* generateLeavesStream(
  count: number,
  batchSize: number = 10000
): AsyncGenerator<[string, string][], void, unknown> {
  for (let i = 0; i < count; i += batchSize) {
    const batch: [string, string][] = [];
    const end = Math.min(i + batchSize, count);
    
    for (let j = i; j < end; j++) {
      const address = ethers.Wallet.createRandom().address;
      const value = ethers.utils.parseEther(String(j)).toString();
      batch.push([address, value]);
    }
    
    yield batch;
    
    // Allow garbage collection between batches
    if (global.gc) global.gc();
  }
}

// Use streaming for large tree generation
export async function generateLargeTree(depth: number): Promise<string> {
  const leafCount = Math.pow(2, depth);
  const leaves: [string, string][] = [];
  
  // Process in chunks to avoid memory overflow
  const stream = generateLeavesStream(leafCount);
  
  for await (const batch of stream) {
    leaves.push(...batch);
  }
  
  // Create tree with memory-efficient options
  const tree = StandardMerkleTree.of(leaves, ["address", "uint256"], {
    sortLeaves: false, // Avoid sorting for memory efficiency
  });
  
  return tree.root;
}
```

### Gas Optimization
```solidity
// Optimize storage access patterns
contract OptimizedVerifier {
    // Pack struct for efficient storage
    struct TreeConfig {
        bytes32 root;
        uint128 depth;
        uint128 timestamp;
    }
    
    // Use mappings for O(1) access
    mapping(uint256 => TreeConfig) public trees;
    
    // Batch operations to amortize base costs
    function batchVerify(
        uint256 treeId,
        bytes32[] calldata leaves,
        bytes32[][] calldata proofs
    ) external view returns (bool[] memory results) {
        TreeConfig memory config = trees[treeId]; // Single SLOAD
        uint256 length = leaves.length;
        results = new bool[](length);
        
        // Avoid repeated storage access
        bytes32 root = config.root;
        
        for (uint256 i; i < length;) {
            results[i] = MerkleProof.verify(proofs[i], root, leaves[i]);
            unchecked { ++i; }
        }
    }
}
```

### Analysis Performance
```typescript
// Parallel processing for analysis
import { Worker } from "worker_threads";
import os from "os";

export class ParallelAnalyzer {
  private workers: Worker[] = [];
  private readonly workerCount = os.cpus().length;
  
  async analyzeInParallel(depths: number[]): Promise<AnalysisResult[]> {
    // Split work among workers
    const chunksSize = Math.ceil(depths.length / this.workerCount);
    const chunks = [];
    
    for (let i = 0; i < depths.length; i += chunksSize) {
      chunks.push(depths.slice(i, i + chunksSize));
    }
    
    // Process chunks in parallel
    const promises = chunks.map((chunk, index) => 
      this.processChunk(chunk, index)
    );
    
    const results = await Promise.all(promises);
    return results.flat();
  }
  
  private async processChunk(
    depths: number[],
    workerId: number
  ): Promise<AnalysisResult[]> {
    return new Promise((resolve, reject) => {
      const worker = new Worker("./workers/gas-analyzer.js", {
        workerData: { depths, workerId }
      });
      
      worker.on("message", resolve);
      worker.on("error", reject);
      worker.on("exit", (code) => {
        if (code !== 0) {
          reject(new Error(`Worker stopped with exit code ${code}`));
        }
      });
    });
  }
}
```

## Common Patterns and Examples

### Tree Data Interface
```typescript
// Consistent data structure across the project
export interface TreeData {
  depth: number;
  root: string;
  leafCount: number;
  format: string[];
  sampleProofs: Array<{
    index: number;
    leaf: [string, string];
    proof: string[];
  }>;
  metadata: {
    generatedAt: number;
    version: string;
    memorySizeBytes?: number;
    generationTimeMs?: number;
  };
}
```

### Gas Report Structure
```typescript
export interface GasReport {
  operation: string;
  treeDepth: number;
  gasUsed: number;
  breakdown?: {
    baseGas: number;
    perProofElement: number;
    memoryGas: number;
  };
  context: {
    blockNumber: number;
    timestamp: number;
    gasPrice?: bigint;
  };
}

export interface AnalysisSummary {
  depths: number[];
  operations: {
    setRoot: GasStatistics;
    verify: GasStatistics;
    batchVerify: GasStatistics;
  };
  scaling: {
    slope: number;
    intercept: number;
    rSquared: number;
    formula: string; // e.g., "gas = 700 * depth + 21000"
  };
  recommendations: string[];
}
```

### Testing Utilities
```typescript
// Reusable test helpers
export class TestHelpers {
  static async deployContracts() {
    const [deployer] = await ethers.getSigners();
    
    const Verifier = await ethers.getContractFactory("MerkleVerifier");
    const verifier = await Verifier.deploy();
    
    const BatchVerifier = await ethers.getContractFactory("BatchMerkleVerifier");
    const batchVerifier = await BatchVerifier.deploy();
    
    const MultiTree = await ethers.getContractFactory("MultiTreeManager");
    const multiTree = await MultiTree.deploy();
    
    await Promise.all([
      verifier.deployed(),
      batchVerifier.deployed(),
      multiTree.deployed(),
    ]);
    
    return { verifier, batchVerifier, multiTree, deployer };
  }
  
  static generateMockProof(depth: number): string[] {
    const proof: string[] = [];
    for (let i = 0; i < depth; i++) {
      proof.push(ethers.utils.keccak256(ethers.utils.toUtf8Bytes(`level-${i}`)));
    }
    return proof;
  }
  
  static async measureGasForDepths(
    depths: number[],
    operation: (depth: number) => Promise<any>
  ): Promise<Map<number, number>> {
    const results = new Map<number, number>();
    
    for (const depth of depths) {
      const tx = await operation(depth);
      const receipt = await tx.wait();
      results.set(depth, receipt.gasUsed.toNumber());
    }
    
    return results;
  }
}
```

## Important Notes

1. **Never mention Foundry** - This project uses Hardhat exclusively
2. **External Data First** - Always generate trees off-chain, never on-chain
3. **Memory Awareness** - Be conscious of memory limits when handling large trees
4. **Gas Focus** - Every design decision should consider gas implications
5. **Statistical Rigor** - Ensure proper statistical analysis with sufficient sample sizes
6. **Error Context** - Always provide detailed error context for debugging
7. **Progress Reporting** - Long operations should report progress
8. **Caching Strategy** - Cache frequently accessed data to improve performance

## Quick Reference Commands

```bash
# Generate all standard test trees
npm run generate:all

# Run gas analysis for specific depth
npm run analyze:depth -- --depth 20

# Generate comprehensive report
npm run report:full

# Run memory-intensive tests
node --max-old-space-size=8192 ./node_modules/.bin/hardhat test test/performance/

# Profile gas usage
REPORT_GAS=true npx hardhat test --grep "gas"

# Generate documentation
npm run docs:generate

# Run Context7 workflow
npm run context7:analyze
```

Remember: The goal is production-ready gas analysis that provides actionable insights for developers implementing Merkle tree solutions at scale. Always prioritize accuracy, efficiency, and real-world applicability in all implementations.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Na3aga) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
