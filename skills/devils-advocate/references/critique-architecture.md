# Critique Module: Architecture Stage

## Pre-Mortem Prompt

"The system was built to this architecture and failed catastrophically in production
at 10x expected load. The team discovered the architecture couldn't be fixed without
a complete rewrite. What was the root cause?"

## Dimension Checklist

### 1. Coupling and Cohesion
- For each module, count incoming dependencies (afferent, Ca) and outgoing (efferent, Ce)
- Calculate instability: Ce / (Ca + Ce) -- modules with high instability AND high
  importance are architectural risks
- Check dependency direction: do dependencies flow from unstable toward stable?
  (Stable Dependencies Principle)
- Flag any circular dependencies as S1
- For each module, estimate change impact -- if >3 other modules must change when
  this module's interface changes, flag as coupling concern

### 2. Single Points of Failure (SPOF)
- For each component: "What happens if this is completely unavailable for 5 minutes?"
  If answer is "entire system down," it is a SPOF
- Audit redundancy on critical path: database replica, load balancer, circuit breaker, queue
- Trace the longest chain of synchronous dependencies -- each link is a potential SPOF
- Check state storage: where is critical state stored? Is it replicated? Can system
  recover from state loss?

### 3. Scalability Bottlenecks
- Trace the request path for the highest-volume operation -- identify which component
  saturates first (database, network, CPU, memory, disk I/O)
- Check Amdahl's Law: what percentage of workload is inherently sequential?
- Evaluate state scaling: vertical-only scaling (bigger machine) is a bottleneck;
  horizontal requires state partitioning
- Project data growth at 10x and 100x -- flag any O(n^2) or O(n!) operations

### 4. Module Boundary Violations
- Build a file ownership matrix -- any file with >1 module owner is S1
- Check for exposed implementation details that other modules depend on
- Audit shared mutable state: files, database tables, or config that multiple
  modules read AND write -- each instance is a conflict vector
- Verify Conway's Law alignment: does module structure map to the agent/team structure?

### 5. DAG Optimization
- Identify the critical path (longest sequential chain) -- this is minimum execution time
- Count parallelization width at each tier -- narrow widths (1-2) indicate serialization
- For each dependency edge: "Does Task B truly require Task A's output?" Remove false dependencies
- Check DAG depth vs. width ratio -- deep narrow DAGs suggest over-serialization
- Assess critical path sensitivity: what happens if one critical-path task takes 2x longer?

### 6. Interface Contract Completeness
- Verify every module-to-module interaction has a contract: input types, output types,
  error types, preconditions, postconditions, invariants
- Check error contract completeness -- contracts must specify all possible errors, not just happy path
- Verify idempotency specification for any operation that might be retried
- Check timeout and cancellation behavior in contracts

### 7. ATAM-Style Tradeoff Analysis
- Identify sensitivity points: properties critical for achieving a quality attribute
- Identify tradeoff points: decisions affecting multiple quality attributes in
  opposing directions
- For each tradeoff, verify the architecture plan explicitly acknowledges and
  justifies the tradeoff

## Deterministic Pre-Checks

- Verify the architecture plan contains all Output Contract items: Module Boundary Map,
  Dependency DAG, Interface Contracts, Integration Checkpoints, CLAUDE.md Recommendations
- Check that no file appears in more than one module's file manifest
- Trace the DAG for cycles (follow dependency chains -- if any chain returns to its start, flag S1)
- Verify all file paths in the module boundary map use absolute paths
