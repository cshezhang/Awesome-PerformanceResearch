# Awesome-PerformanceResearch

## 1 Introduction

This repo includes diverse works related to performance issues. We divide these works into two categories:

### (1) Empricial Study, especially in taxonomy
1. Semantics-based(ISSRE 2019)
    * Fast-path
    * Arguments
    * Cache memorization
2. Syntactic-based(PLDI 2012)
    * Uncoordinated functions
    * Skippable Function
    * Synchronization Issues
3. Configuration-related(CP-Detector, ASE 2020)
    * Optimization on-off
    * Non-functional tradeoff
    * Resource allocation
    * Functionality on-off
4. Specific performance issues
    * Inefficient Loop(L-Doctor, ICSE 2017)
    * Wasteful memory access
        1. Dead store
        2. Silent store
        3. Silent load
#### Other topics related to empircial study
* How performance bugs are exposed, introduced, fixed(patch)
* Bug fix time, Number of changed lines

### (2) Detection approach
1. Program analysis, including static analysis and dynamic analysis, e.g., profiler and instrumentation
    * L-Doctor ICSE 2017
        * Static-dynamic hybrid analysis to provide accruate performance diagnosis
    * JXPerf FSE 2019
        * Inefficiency Profiler
        * Use PMU and debug registers to recognize some wasteful memory accesses
        * Prior works mainly use bytecode instrumentation with high overhead
    * LearnConf EuroSys 2020
        * Leverage static analysis to infer performance properties of software configs
2. Statistical learning
    * OOPSLA 2014
3. Deep Learning
    * SPLASH Companion 2020
        * AST with data dependency edge + graph embedding(Deep Walk)

## 2 Detailed Paper Comments
### Pinpointing Performance Inefficiencies in Java
### FSE 2019
This paper leverages PMU and hardware debug register to detect wasteful memory access:
* Dead store<S1, S2>
    * S1 and S2 are two successive memory stores to location M (S1 occurs before S2)
    * S1 is a dead store iff there are no intervening memory loads from M between S1 and S2.
* Silent store<S1, S2>
    * A memory store S2, storing a valueV2 to locationM, is a silent store iff the previous memory store S1 performed on M stored a value V1, and V1 = V2.
* Silent load<L1, L2>
    * A memory load L2, loading a value V2 from location M is a silent load iff the previous memory load L1 performed on M loaded a value V1, and V1 = V2.

The methodology can be describled as the following figure(Silent Store):

1. PMU samples a memory store
2. Debug register monitor
3. Debug register trap
4. Check value

![SilentStore](https://raw.githubusercontent.com/HuaienZhang/Awesome-PerformanceResearch/main/img/SilentStore.png)

### Detecting Performance Patterns with Deep Learning
### SPLASH Companion 2020
This work may be the first one to leverage deep learning to recognize performance bugs. It uses ML to infer inefficient statements in code. The approach is easy:
1. Leverage AST to capture the control flow of corresponding program;
2. Add data dependency edges to capture data flow
    * Perform BFS search on the AST
    * Find Store or Param node
        * Create a stable entry in a dict with the node's value(define)
    * Find Load node
        * Create a data dependency edge from its parent store or param node to load node
3. Use DeepWalk to embedd the AST and it seems that efficient and inefficient ASTs are seperated:
![DeepWalk](https://raw.githubusercontent.com/HuaienZhang/Awesome-PerformanceResearch/main/img/DeepWalk.png)

So, the author argued that constructing every statements' AST and graph embedding can help us classify efficient or inefficient statements;

### Understanding and Detecting Real-World Performance Bugs
### PLDI 2012
This paper conducts a comprehensive study of 109 real-world performance bugs that are randomly sampled from five software suites. The important results are listed as two types below:
* Root cause(Taxonomy)
    1. Uncoordinated Functions: inefficient function-call combinations composed of efficient individual functions.
        * ![TransactionBug](https://raw.githubusercontent.com/HuaienZhang/Awesome-PerformanceResearch/main/img/TransactionBug.png)
        * These transactions are used to adding bookmarks, actually, they can be merged into one transaction.
    2. Skippable Function: call functions that conducts unnnecessary work given the calling context
        * ![SkippableFunction](https://raw.githubusercontent.com/HuaienZhang/Awesome-PerformanceResearch/main/img/SkippableFunction.png)
        * Calling nsImage::Draw for transparent figures is meaningless
        * ![IntensiveGC](https://raw.githubusercontent.com/HuaienZhang/Awesome-PerformanceResearch/main/img/IntensiveGC.png)
        * The GC call is useless, however, XMLHttpRequest is not popular at first, after Web 2.0, it becomes more and more popular. So, this problem lasted for five years before being fixed.
    3. Synchronization Issues: unnecessary synchronization that intensifies thread competition is also a common cause of performance
        * ![SyncBug](https://raw.githubusercontent.com/HuaienZhang/Awesome-PerformanceResearch/main/img/SyncBug.png)
loss
* Rule-based detection
    * Efficiency Rule contains two components: A transformation and a condition
        * Once code region satisfies the condition, the transformation can be applied to improve performance.
    * Concluded from patches(A total of 109)
        * 50 / 109 have efficiency rules, the other 59 do not contain rules(either target too specific program contexts or too general to be useful for rule-based bug detection)
        * ![Conditions](https://raw.githubusercontent.com/HuaienZhang/Awesome-PerformanceResearch/main/img/Conditions.png)
        * For example, doTransaction can be fixed by the No.3 rule. Obviously, this approach is naive and easily leads to many FPs.

### Inferring Performance Bug Patterns from Developer Commits
### ISSRE 2019
This paper leverages performance bug related commits to understand the ecosystem and provides a benchmark for the future work.

* Collect commits
    * Selection: Top popular projects from Debian repo
    * Identification: keywords-based, such as "performance" and "accerlate". All results will be manual checked
* Taxonomy: Semantics behind code changes, based on the observation proposed by authors: Perf bug fixing commits follow certain patterns
    1. Fast-path: avoid repeated or slow computation when possible, apply a different algorithm to achieve the same result as the slow-path
        ```C
        int foo(int bar) {
            if (some_cond(bar)) {
                return fast_path; // Fast-path
            }
            return very_heavy_computation(bar); //Slow-path
        }
        ```
    2. Arguments




