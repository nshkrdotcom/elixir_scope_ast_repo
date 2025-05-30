# Design Document: ElixirScope.ASTRepository (elixir_scope_ast_repo)

## 1. Purpose & Vision

**Summary:** The central static analysis engine and repository for ElixirScope. Responsible for parsing Elixir code, building Abstract Syntax Trees (ASTs), Control Flow Graphs (CFGs), Data Flow Graphs (DFGs), and comprehensive Code Property Graphs (CPGs). It provides powerful querying, pattern matching, and algorithmic analysis over these static representations.

**(Greatly Expanded Purpose based on your existing knowledge of ElixirScope and CPG features):**

The `elixir_scope_ast_repo` library is the core intelligence hub for understanding Elixir code structure and semantics within the ElixirScope ecosystem. Its primary mission is to transform raw source code into rich, multi-faceted graph representations—most notably the Code Property Graph (CPG)—that enable deep, automated reasoning about code behavior, quality, security, and performance.

This library aims to:
*   **Provide Deep Code Understanding:** Go beyond simple ASTs by constructing CFGs (control flow), DFGs (data flow, potentially SSA-based), and unifying them into a CPG. This CPG will link syntax, control, and data flow information, including inter-procedural relationships.
*   **Enable Advanced Static Analysis:** Implement sophisticated graph algorithms (`CPGMath`) and code-aware semantic analysis (`CPGSemantics`) on the CPG to uncover complex patterns, dependencies, potential bugs, security vulnerabilities, and performance hotspots.
*   **Serve as a Centralized Static Knowledge Base:** Act as the single source of truth for the static structure of the analyzed project, accessible by other ElixirScope components.
*   **Support Powerful Querying:** Offer a flexible query language (`QueryBuilder`) to interrogate the ASTs, CFGs, DFGs, and CPGs, allowing for complex structural and semantic searches.
*   **Facilitate Pattern Matching:** Provide a `PatternMatcher` capable of identifying user-defined and predefined code patterns (including smells and best practices) within the CPG.
*   **Ensure AST Stability and Correlation:** Implement robust AST parsing and `NodeIdentifier` logic to assign stable, unique IDs to AST nodes, which are critical for linking runtime events back to specific code locations within the CPG.
*   **Manage Repository Lifecycle:** Handle project population (`ProjectPopulator`), file watching (`FileWatcher`), and synchronization (`Synchronizer`) to keep the static analysis data up-to-date with code changes.
*   **Optimize for Performance and Memory:** Employ efficient data structures (primarily ETS, as per `CPG_ETS_INTEGRATION.MD`), caching (`MemoryManager`), and graph processing algorithms (`CPG_OPTIMIZATION_STRATEGIES.MD`) to handle large codebases.

The CPG is the centerpiece of this library. It will integrate information as described in your CPG PRD, allowing queries that span syntax (e.g., "find all function calls to `File.write/2`"), control flow (e.g., "can this variable be null at this decision point?"), and data flow (e.g., "trace the origin of this data being written to a file"). The `CPGMath` and `CPGSemantics` APIs will provide the tools to unlock these insights.

This library will enable:
*   `elixir_scope_compiler` to obtain stable AST Node IDs for instrumentation.
*   `elixir_scope_correlator` to map runtime `ast_node_id`s to detailed static CPG nodes and their context.
*   `elixir_scope_ai` to leverage the rich CPG for code understanding, refactoring suggestions, and predictive analysis.
*   `elixir_scope_debugger_features` to implement CPG-aware breakpoints (e.g., "break if this data-flow path is taken").
*   `TidewaveScope` MCP tools to provide AI assistants with deep structural insights into the codebase, answering questions like "show me the control flow for this function" or "what parts of the code are most central/complex based on the CPG?".

## 2. Key Responsibilities

This library is responsible for:

*   **Parsing & AST Management:**
    *   Parsing Elixir source files into ASTs (`ElixirScope.ASTRepository.Parser`).
    *   Assigning unique, stable AST Node IDs (`ElixirScope.ASTRepository.NodeIdentifier`).
    *   Storing and managing ASTs for modules and functions.
*   **Graph Generation:**
    *   Generating CFGs for functions (`ElixirScope.ASTRepository.Enhanced.CFGGenerator`).
    *   Generating DFGs for functions, potentially including SSA form (`ElixirScope.ASTRepository.Enhanced.DFGGenerator`).
    *   Building CPGs by unifying AST, CFG, DFG, and adding inter-procedural edges (`ElixirScope.ASTRepository.Enhanced.CPGBuilder`).
*   **CPG Algorithms & Semantics:**
    *   Implementing core graph algorithms (centrality, pathfinding, dominance, etc.) on CPGs (`ElixirScope.ASTRepository.Enhanced.CPGMath`).
    *   Implementing semantic analysis on CPGs (smell detection, impact analysis, taint tracking prototypes) (`ElixirScope.ASTRepository.Enhanced.CPGSemantics`).
*   **Repository Core (`EnhancedRepository` GenServer):**
    *   Providing a GenServer to manage all static analysis data (ASTs, CFGs, DFGs, CPGs).
    *   Handling CRUD operations for this data.
    *   Managing in-memory storage (ETS-based) and caching.
*   **Project Analysis Lifecycle:**
    *   Discovering project files and orchestrating initial analysis (`ElixirScope.ASTRepository.Enhanced.ProjectPopulator`).
    *   Monitoring file system changes and triggering re-analysis (`ElixirScope.ASTRepository.Enhanced.FileWatcher`).
    *   Synchronizing repository state with file system changes (`ElixirScope.ASTRepository.Enhanced.Synchronizer`).
*   **Querying & Pattern Matching:**
    *   Providing a query builder and execution engine for static analysis artifacts (`ElixirScope.ASTRepository.QueryBuilder`, `QueryExecutor`).
    *   Implementing a pattern matching engine for CPGs and ASTs (`ElixirScope.ASTRepository.PatternMatcher`).
*   **Resource Management:**
    *   Managing memory usage, including caching strategies and data eviction (`ElixirScope.ASTRepository.MemoryManager`).
    *   Optimizing performance of analysis and queries (`ElixirScope.ASTRepository.PerformanceOptimizer`).

## 3. Key Modules & Structure

The primary modules within this library will be:

*   `ElixirScope.ASTRepository.EnhancedRepository` (Main GenServer facade)
*   **Parsing & AST Foundation:**
    *   `ElixirScope.ASTRepository.Parser`
    *   `ElixirScope.ASTRepository.NodeIdentifier`
    *   `ElixirScope.ASTRepository.ASTAnalyzer` (if distinct from Parser)
*   **Graph Generators:**
    *   `ElixirScope.ASTRepository.Enhanced.CFGGenerator` (and its submodules: `ASTProcessor`, `ControlFlowProcessors`, `ExpressionProcessors`, `ASTUtilities`, `StateManager`, `ComplexityCalculator`, `PathAnalyzer`, `Validators`)
    *   `ElixirScope.ASTRepository.Enhanced.DFGGenerator` (and its submodules: `ASTAnalyzer` (DFG-specific), `DependencyAnalyzer`, `EdgeCreator`, `NodeCreator`, `OptimizationAnalyzer`, `PatternAnalyzer`, `StateManager`, `StructureBuilder`, `VariableTracker`)
    *   `ElixirScope.ASTRepository.Enhanced.CPGBuilder` (and its submodules: `Core`, `GraphMerger`, `ComplexityAnalyzer` (CPG-specific), `SecurityAnalyzer`, `PerformanceAnalyzer`, `QualityAnalyzer`, `PatternDetector`, `QueryProcessor`, `Validator`, `Helpers`)
*   **CPG Core Algorithms & Semantics:**
    *   `ElixirScope.ASTRepository.Enhanced.CPGMath`
    *   `ElixirScope.ASTRepository.Enhanced.CPGSemantics`
*   **Project Management:**
    *   `ElixirScope.ASTRepository.Enhanced.ProjectPopulator` (and submodules)
    *   `ElixirScope.ASTRepository.Enhanced.FileWatcher`
    *   `ElixirScope.ASTRepository.Enhanced.Synchronizer`
*   **Querying & Matching:**
    *   `ElixirScope.ASTRepository.QueryBuilder` (and submodules: `Executor`, `Normalizer`, `Optimizer`, `Validator`, `Cache`, `Types`)
    *   `ElixirScope.ASTRepository.PatternMatcher` (and submodules: `Core`, `Analyzers`, `PatternLibrary`, `PatternRules`, `Cache`, `Stats`, `Types`, `Validators`)
*   **Resource Management:**
    *   `ElixirScope.ASTRepository.MemoryManager` (and submodules: `CacheManager`, `Cleaner`, `Compressor`, `Config`, `Monitor`, `PressureHandler`, `Supervisor`)
    *   `ElixirScope.ASTRepository.PerformanceOptimizer` (and submodules)
*   `ElixirScope.ASTRepository.Config` (If AST Repo specific configurations are extensive and not in `elixir_scope_config`)

### Proposed File Tree (Simplified Top Level):

```
elixir_scope_ast_repo/
├── lib/
│   └── elixir_scope/
│       └── ast_repository/
│           ├── enhanced_repository.ex # Main GenServer
│           ├── parser.ex
│           ├── node_identifier.ex
│           ├── enhanced/               # Subdirectory for CFG, DFG, CPG, ProjectPopulator, etc.
│           │   ├── cfg_generator.ex
│           │   ├── cfg_generator/      # CFG submodules
│           │   ├── dfg_generator.ex
│           │   ├── dfg_generator/      # DFG submodules
│           │   ├── cpg_builder.ex
│           │   ├── cpg_builder/        # CPG submodules
│           │   ├── cpg_math.ex
│           │   ├── cpg_semantics.ex
│           │   ├── project_populator.ex
│           │   ├── project_populator/  # ProjectPopulator submodules
│           │   ├── file_watcher.ex
│           │   └── synchronizer.ex
│           ├── query_builder.ex
│           ├── query_builder/          # QueryBuilder submodules
│           ├── pattern_matcher.ex
│           ├── pattern_matcher/        # PatternMatcher submodules
│           ├── memory_manager.ex
│           ├── memory_manager/         # MemoryManager submodules
│           └── performance_optimizer.ex
│               └── performance_optimizer/ # PerformanceOptimizer submodules
├── mix.exs
├── README.md
├── DESIGN.MD
└── test/
    ├── test_helper.exs
    └── elixir_scope/
        └── ast_repository/ # Tests for each major component
```

**(Greatly Expanded - Module Description - Selected Examples):**
*   **`ElixirScope.ASTRepository.EnhancedRepository` (GenServer):** The central nervous system of static analysis. Manages the lifecycle of, and access to, all static analysis artifacts (ASTs, CFGs, DFGs, CPGs). It coordinates with `ProjectPopulator` for initial analysis and `Synchronizer` for updates. It uses ETS tables (as per `CPG_ETS_INTEGRATION.MD`) for storing the graphs and their metadata. It exposes the primary API for querying these graphs.
*   **`ElixirScope.ASTRepository.Enhanced.CPGBuilder.Core`**: Orchestrates the CPG construction process. It takes AST, CFG, and DFG data as input, uses `GraphMerger` to combine them, and then invokes various analyzers (`ComplexityAnalyzer`, `SecurityAnalyzer`, `PerformanceAnalyzer`, `QualityAnalyzer`) to enrich the CPG nodes and edges with properties.
*   **`ElixirScope.ASTRepository.Enhanced.CPGMath`**: Implements the mathematical graph algorithms defined in `CPG_MATH_API.MD`. Functions like `calculate_node_centrality/3`, `find_shortest_path/4`, `get_dominators/2` operate on `CPGData.t()` structures. Results might be cached or stored as properties on CPG nodes.
*   **`ElixirScope.ASTRepository.Enhanced.CPGSemantics`**: Implements the code-aware semantic analysis functions defined in `CPG_SEMANTICS_API.MD`. Functions like `identify_data_source/2`, `trace_taint_flow/3`, `get_impact_of_change/3` use both CPG structure and results from `CPGMath` to provide higher-level insights.
*   **`ElixirScope.ASTRepository.QueryBuilder` & `Executor`**: Provides a domain-specific language (or structured API) for querying CPGs (and other graphs). The `Executor` translates these queries into efficient ETS lookups and graph traversals, leveraging indexes and optimizations detailed in `CPG_QUERY_ENHANCEMENTS.MD`.
*   **`ElixirScope.ASTRepository.PatternMatcher`**: Allows defining and matching structural and semantic patterns on the CPG, as per `CPG_PATTERN_DETECTION_ADVANCED.MD`. Patterns could be defined via Elixir code or a specialized pattern language.

## 4. Public API (Conceptual)

Via `ElixirScope.ASTRepository.EnhancedRepository` (GenServer facade):

*   `start_link(opts :: keyword()) :: GenServer.on_start()`
*   `get_ast(repo_ref, module, function, arity) :: {:ok, ast_term} | :not_found`
*   `get_cfg(repo_ref, module, function, arity) :: {:ok, ElixirScope.AST.CFGData.t()} | :not_found`
*   `get_dfg(repo_ref, module, function, arity) :: {:ok, ElixirScope.AST.DFGData.t()} | :not_found`
*   `get_cpg(repo_ref, module, function, arity) :: {:ok, ElixirScope.AST.CPGData.t()} | :not_found`
    *   (Or a single `get_function_analysis(repo_ref, m, f, a)` returning `EnhancedFunctionData.t()`)
*   `get_module_analysis(repo_ref, module) :: {:ok, ElixirScope.AST.EnhancedModuleData.t()} | :not_found`
*   `query_cpg(repo_ref, query_spec :: map()) :: {:ok, list_of_results} | {:error, term()}` (delegates to `QueryBuilder.execute_query`)
*   `match_cpg_pattern(repo_ref, pattern_spec :: map()) :: {:ok, list_of_matches} | {:error, term()}` (delegates to `PatternMatcher`)
*   `get_ast_node_id_details(repo_ref, ast_node_id :: String.t()) :: {:ok, map_with_cpg_context} | :not_found`
*   `get_repository_stats(repo_ref) :: {:ok, map()}`

Via `ElixirScope.ASTRepository.Enhanced.ProjectPopulator`:

*   `populate_project(repo_ref, project_path :: String.t(), opts :: keyword()) :: {:ok, summary_map} | {:error, term()}`

Via `ElixirScope.ASTRepository.Enhanced.CPGMath` / `CPGSemantics` (direct module calls, stateless):

*   Functions as defined in `CPG_MATH_API.MD` and `CPG_SEMANTICS_API.MD`.

## 5. Core Data Structures

This library consumes and produces the structures defined in `elixir_scope_ast_structures`. Key among them:

*   `ElixirScope.AST.EnhancedModuleData.t()`
*   `ElixirScope.AST.EnhancedFunctionData.t()`
*   `ElixirScope.AST.CFGData.t()`, `DFGData.t()`, `CPGData.t()` (and their node/edge components)
*   `ElixirScope.AST.ComplexityMetrics.t()`
*   Internal structures for query plans, pattern states, cache entries.

## 6. Dependencies

This library will depend on the following ElixirScope libraries:

*   `elixir_scope_utils` (for utilities)
*   `elixir_scope_config` (for operational parameters, thresholds, feature flags for CPG features)
*   `elixir_scope_events` (if it emits events about its analysis progress or findings)
*   `elixir_scope_ast_structures` (crucially, for all its input and output data types)

It will depend on Elixir core libraries (`GenServer`, `:ets`, `File`, `Path`, `Code`, `Module`, `Macro`).

## 7. Role in TidewaveScope & Interactions

Within the `TidewaveScope` ecosystem, the `elixir_scope_ast_repo` library will:

*   Be a central, long-running service, likely started by `TidewaveScope`'s main application supervisor.
*   Its `ProjectPopulator` will be invoked by `TidewaveScope` (on startup in dev, or via an MCP tool) to analyze the target application.
*   `FileWatcher` and `Synchronizer` will keep its data current during development.
*   Provide the static analysis backend for numerous `TidewaveScope` MCP tools:
    *   "Explain this code" -> query CPG for node properties, semantic links.
    *   "Find similar functions" -> use CPG structure/semantics, embedding queries.
    *   "What's the complexity of X?" -> query complexity metrics.
    *   "Show data flow for Y" -> query DFG/CPG for data flow paths.
    *   "Suggest refactorings for Z" -> use pattern matching and semantic analysis results.
*   Supply stable `ast_node_id`s (via its Parser/NodeIdentifier) to the `elixir_scope_compiler` for injection.
*   Serve `elixir_scope_correlator` with detailed CPG node information when runtime events are mapped back using `ast_node_id`.
*   Provide CPGs and analysis results as input to `elixir_scope_ai`.

## 8. Future Considerations & CPG Enhancements

*   **Persistence:** Beyond ETS, for very large projects or for sharing analysis results, explore persisting CPGs/analysis to disk (e.g., RocksDB, specialized graph DB, or a binary format).
*   **Distributed CPG:** Strategies for building/querying CPGs in a distributed Elixir environment (connecting CPGs from different nodes).
*   **Language Server Protocol (LSP) Integration:** Expose CPG query results and analysis through an LSP server for direct IDE integration.
*   **Full Implementation of all CPG_*.md documents:** This is the primary path for future enhancements, including all advanced algorithms, optimizations, and query capabilities.
*   **Schema Evolution:** Managing changes to CPG node/edge properties as new analyses are added.

## 9. Testing Strategy

This is a complex library requiring extensive testing:

*   **Unit Tests for each Generator (`CFGGenerator`, `DFGGenerator`, `CPGBuilder` and their submodules):**
    *   Test with various simple and complex Elixir code snippets.
    *   Verify correct graph structure (nodes, edges, properties) for known inputs.
    *   Test edge cases, error handling, and specific language constructs.
*   **Unit Tests for `Parser`, `NodeIdentifier`:**
    *   Verify correct AST parsing and stable, unique Node ID assignment.
*   **Unit Tests for `CPGMath` and `CPGSemantics`:**
    *   Test each algorithm with sample CPG data.
    *   Verify correctness of computed metrics and semantic interpretations.
    *   Use test cases derived from `CPG-MATH_TEST_LIST.MD` and `CPG-SEMANTICS_TEST_LIST.MD`.
*   **Unit Tests for `QueryBuilder` and `PatternMatcher`:**
    *   Test query construction and execution for various query types.
    *   Test pattern definition and matching against sample CPGs.
*   **Unit Tests for `ProjectPopulator`, `FileWatcher`, `Synchronizer`:**
    *   Mock file system interactions to test discovery, change detection, and sync logic.
*   **`EnhancedRepository` GenServer Tests:**
    *   Test all API calls for storing and retrieving data.
    *   Test concurrent access.
    *   Test memory management and caching effectiveness (using `MemoryManager` mocks or by observing ETS stats).
*   **Integration Tests (within the library):**
    *   Test the end-to-end flow: `ProjectPopulator` -> `Parser` -> Graph Generators -> `EnhancedRepository` storage -> `QueryBuilder`.
    *   Test CPG-based queries retrieving expected results after populating with sample code.
*   **Property-Based Tests:** For graph generation and query logic to ensure robustness against a wide variety of code structures.
*   **CPG Validation Tests:** After CPG generation, run validation checks based on `CPG_VALIDATION_CHECKLIST.MD`.
