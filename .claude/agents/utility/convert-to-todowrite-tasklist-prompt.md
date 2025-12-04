---
name: convert-to-todowrite-tasklist-prompt
description: "Converts complex prompts to efficient TodoWrite tasklist methods for parallel execution"
---

# Convert Complex Prompts to TodoWrite Tasklist Method

**Purpose**: Transform verbose, context-heavy slash commands into efficient TodoWrite tasklist-based methods with parallel subagent execution for 60-70% speed improvements.

**Usage**: `/convert-to-todowrite-tasklist-prompt @/path/to/original-slash-command.md`

---

name: convert-to-todowrite-tasklist-prompt
description: "Converts complex prompts to efficient TodoWrite tasklist methods for parallel execution"

---

## Argument Variable Integration

When converting slash commands, ensure proper argument handling for dynamic inputs:

### Standard Argument Variables

    markdown

## ARGUMENT HANDLING

**File Input**: {file_path} or {code} - The primary file(s) or code to analyze
**Analysis Scope**: {scope} - Specific focus areas (security, performance, quality, architecture, all)
**Output Format**: {format} - Report format (detailed, summary, action_items)
**Target Audience**: {audience} - Intended audience (technical, executive, security_team)
**Priority Level**: {priority} - Analysis depth (quick, standard, comprehensive)
**Context**: {context} - Additional project context and constraints

### Usage Examples:

    bash

# Basic usage with file input

/comprehensive-review file_path="@src/main.py" scope="security,performance"

# Advanced usage with multiple parameters

/comprehensive-review file_path="@codebase/" scope="all" format="detailed" audience="technical" priority="comprehensive" context="Production deployment review"

# Quick analysis with minimal scope

/comprehensive-review file_path="@config.yaml" scope="security" format="summary" priority="quick"

### Argument Integration in TodoWrite Tasks

**Dynamic Task Content Based on Arguments:**

    json

[
{"id": "setup_analysis", "content": "Record start time and initialize analysis for {file_path}", "status": "pending", "priority": "high"},
{"id": "security_analysis", "content": "Security Analysis of {file_path} - Focus: {scope}", "status": "pending", "priority": "high"},
{"id": "report_generation", "content": "Generate {format} report for {audience}", "status": "pending", "priority": "high"}
]

---

name: convert-to-todowrite-tasklist-prompt
description: "Converts complex prompts to efficient TodoWrite tasklist methods for parallel execution"

---

## TodoWrite Structure for Parallel Execution

### Enhanced Task JSON Template with Argument Integration

    json

[
{"id": "setup_analysis", "content": "Record start time and initialize analysis for {file_path}", "status": "pending", "priority": "high"},

// Conditional Parallel Groups Based on {scope} Parameter
// If scope includes "security" or "all":
{"id": "security_auth", "content": "Security Analysis of {file_path} - Authentication & Validation (Subagent A)", "status": "pending", "priority": "high", "parallel_group": "security", "condition": "security in {scope}"},
{"id": "security_tools", "content": "Security Analysis of {file_path} - Tool Isolation & Parameters (Subagent B)", "status": "pending", "priority": "high", "parallel_group": "security", "condition": "security in {scope}"},
{"id": "security_protocols", "content": "Security Analysis of {file_path} - Protocols & Transport (Subagent C)", "status": "pending", "priority": "high", "parallel_group": "security", "condition": "security in {scope}"},

// If scope includes "performance" or "all":
{"id": "performance_complexity", "content": "Performance Analysis of {file_path} - Algorithmic Complexity (Subagent A)", "status": "pending", "priority": "high", "parallel_group": "performance", "condition": "performance in {scope}"},
{"id": "performance_io", "content": "Performance Analysis of {file_path} - I/O Patterns & Async (Subagent B)", "status": "pending", "priority": "high", "parallel_group": "performance", "condition": "performance in {scope}"},
{"id": "performance_memory", "content": "Performance Analysis of {file_path} - Memory & Concurrency (Subagent C)", "status": "pending", "priority": "high", "parallel_group": "performance", "condition": "performance in {scope}"},

// If scope includes "quality" or "architecture" or "all":
{"id": "quality_patterns", "content": "Quality Analysis of {file_path} - Code Patterns & SOLID (Subagent A)", "status": "pending", "priority": "high", "parallel_group": "quality", "condition": "quality in {scope}"},
{"id": "architecture_design", "content": "Architecture Analysis of {file_path} - Modularity & Interfaces (Subagent B)", "status": "pending", "priority": "high", "parallel_group": "quality", "condition": "architecture in {scope}"},

// Sequential Dependencies
{"id": "synthesis_integration", "content": "Synthesis & Integration - Consolidate findings for {file_path}", "status": "pending", "priority": "high", "depends_on": ["security", "performance", "quality"]},
{"id": "report_generation", "content": "Generate {format} report for {audience} - Analysis of {file_path}", "status": "pending", "priority": "high"},
{"id": "verification_parallel", "content": "Parallel verification of {file_path} analysis with multiple validation streams", "status": "pending", "priority": "high"},
{"id": "final_integration", "content": "Final integration and completion for {file_path}", "status": "pending", "priority": "high"}
]

### Conditional Task Execution Based on Arguments

**Scope-Based Task Filtering:**

    markdown

## CONDITIONAL EXECUTION LOGIC

**Full Analysis (scope="all")**:

- Execute all security, performance, quality, and architecture tasks
- Use comprehensive parallel subagent deployment

**Security-Focused (scope="security")**:

- Execute only security_auth, security_tools, security_protocols tasks
- Skip performance, quality, architecture parallel groups
- Faster execution with security specialization

**Performance-Focused (scope="performance")**:

- Execute only performance_complexity, performance_io, performance_memory tasks
- Include synthesis and reporting phases
- Targeted performance optimization focus

**Custom Scope (scope="security,quality")**:

- Execute selected parallel groups based on comma-separated values
- Flexible analysis depth based on specific needs

**Priority-Based Execution:**

- priority="quick": Use single subagent per domain, reduced file scope
- priority="standard": Use 2-3 subagents per domain (default)
- priority="comprehensive": Use 3-4 subagents per domain, expanded file scope

### Task Delegation Execution Framework

**CRITICAL: Use Task Tool Delegation Pattern (Prevents Context Overflow)**

    markdown

## TASK DELEGATION FRAMEWORK

### Phase 1: Security Analysis (Task-Based)

**TodoWrite**: Mark "security_analysis" as in_progress
**Task Delegation**: Use Task tool with focused analysis:

Task Description: "Security Analysis of Target Codebase"
Task Prompt: "Analyze security vulnerabilities focusing on:

- STRIDE threat modeling for architecture
- OWASP Top 10 assessment (adapted for context)
- Authentication and credential management
- Input validation and injection prevention
- Protocol-specific security patterns

**CONTEXT MANAGEMENT**: Analyze only 3-5 key security files:

- Main coordinator file (entry point security)
- Security/validation modules (2-3 files max)
- Key protocol handlers (1-2 files max)

Provide specific findings with file:line references and actionable recommendations."

### Phase 2: Performance Analysis (Task-Based)

**TodoWrite**: Mark "security_analysis" completed, "performance_analysis" as in_progress
**Task Delegation**: Use Task tool with performance focus:

Task Description: "Performance Analysis of Target Codebase"
Task Prompt: "Analyze performance characteristics focusing on:

- Algorithmic complexity (Big O analysis)
- I/O efficiency patterns (async/await, file operations)
- Memory management (caching, object lifecycle)
- Concurrency bottlenecks and optimization opportunities

**CONTEXT MANAGEMENT**: Analyze only 3-5 key performance files:

- Core algorithm modules (complexity focus)
- I/O intensive modules (async/caching focus)
- Memory management modules (lifecycle focus)

Identify specific bottlenecks with measured impact and optimization opportunities."

### Phase 3: Quality & Architecture Analysis (Task-Based)

**TodoWrite**: Mark "performance_analysis" completed, "quality_analysis" as in_progress
**Task Delegation**: Use Task tool with quality focus:

Task Description: "Quality & Architecture Analysis of Target Codebase"
Task Prompt: "Evaluate code quality and architectural design focusing on:

- Clean code principles (function length, naming, responsibility)
- SOLID principles compliance and modular design
- Architecture patterns and dependency management
- Interface design and extensibility considerations

**CONTEXT MANAGEMENT**: Analyze only 3-5 representative files:

- Core implementation patterns (2-3 files)
- Module interfaces and boundaries (1-2 files)
- Configuration and coordination modules (1 file)

Provide complexity metrics and specific refactoring recommendations with examples."

**CRITICAL SUCCESS PATTERN**: Each Task operation stays within context limits by analyzing only 3-5 files maximum, using fresh context for each analysis phase.

---

name: convert-to-todowrite-tasklist-prompt
description: "Converts complex prompts to efficient TodoWrite tasklist methods for parallel execution"

---

## Context Management for Task Delegation

### CRITICAL: Context Overflow Prevention Rules

**NEVER Generate These Patterns:**
❌ `@file1 @file2 @file3 @file4 @file5...` (bulk file loading)
❌ `Analyze all files simultaneously`
❌ `Load entire codebase for analysis`

**ALWAYS Use These Patterns:**
✅ `Task tool to analyze: [3-5 specific files max]`
✅ `Progressive analysis through Task boundaries`
✅ `Fresh context for each analysis phase`

### File Selection Strategy (Maximum 5 Files Per Task)

**Security Analysis Priority Files (3-5 max):**

Task tool to analyze:

- Main coordinator file (entry point security)
- Primary validation/security modules (2-3 files)
- Key protocol handlers (1-2 files)

**Performance Analysis Priority Files (3-5 max):**

Task tool to analyze:

- Core algorithm modules (complexity focus)
- I/O intensive modules (async/caching focus)
- Memory management modules (lifecycle focus)

**Quality Analysis Priority Files (3-5 max):**

Task tool to analyze:

- Representative implementation patterns (2-3 files)
- Module interfaces and boundaries (1-2 files)

### Context Budget Allocation for Task Delegation

Total Context Limit per Task: ~200k tokens

- Task Instructions: ~10k tokens (focused, domain-specific)
- File Analysis: ~40k tokens (3-5 files maximum)
- Analysis Output: ~20k tokens (specialized findings)
- Buffer/Overhead: ~10k tokens
  Total per Task: ~80k tokens (safe task execution)

Context Efficiency:

- 3 Task operations: 3 × 80k = 240k total analysis capacity
- Fresh context per Task prevents overflow accumulation
- Progressive analysis maintains depth while respecting limits

CRITICAL: Never exceed 5 files per Task operation

---

name: convert-to-todowrite-tasklist-prompt
description: "Converts complex prompts to efficient TodoWrite tasklist methods for parallel execution"

---

## Quality Gates for Parallel Execution

### Completion Verification Checklist

**Before Synthesis Phase:**

- [ ] All security subagents completed with specific file:line references
- [ ] All performance subagents completed with measurable impact assessments
- [ ] All quality subagents completed with concrete refactoring examples
- [ ] No parallel streams terminated due to context overflow
- [ ] All findings include actionable recommendations

**Synthesis Quality Gates:**

- [ ] Cross-domain conflicts identified and resolved
- [ ] Priority matrix spans all parallel finding categories
- [ ] Implementation roadmap coordinates across all domains
- [ ] No critical findings lost during consolidation
- [ ] Final recommendations maintain parallel analysis depth

### Success Metrics

**Parallel Execution Effectiveness:**

- **Speed Improvement**: Target 50-70% reduction in total analysis time
- **Coverage Enhancement**: More detailed analysis per domain through specialization
- **Context Efficiency**: No subagent context overflow, optimal token utilization
- **Quality Maintenance**: Same or higher finding accuracy vs sequential analysis
- **Actionability**: All recommendations include specific file:line references and metrics

---

name: convert-to-todowrite-tasklist-prompt
description: "Converts complex prompts to efficient TodoWrite tasklist methods for parallel execution"

---

## CONVERSION WORKFLOW EXECUTION

Now, apply this framework to convert the original slash command file provided in $ARGUMENT:

### TodoWrite Task: Conversion Process

    json

[
{"id": "read_original", "content": "Read and analyze original slash command from $ARGUMENT", "status": "pending", "priority": "high"},
{"id": "identify_patterns", "content": "Identify context overload patterns and conversion opportunities", "status": "pending", "priority": "high"},
{"id": "decompose_tasks", "content": "Decompose workflow into parallel TodoWrite tasks", "status": "pending", "priority": "high"},
{"id": "design_subagents", "content": "Design specialized subagent prompts for parallel execution", "status": "pending", "priority": "high"},
{"id": "generate_conversion", "content": "Generate optimized slash command with TodoWrite framework", "status": "pending", "priority": "high"},
{"id": "validate_output", "content": "Validate converted prompt for context efficiency and completeness", "status": "pending", "priority": "high"},
{"id": "overwrite_original", "content": "Overwrite original file with converted optimized version", "status": "pending", "priority": "high"}
]

### Execution Instructions

**Mark "read_original" as in_progress and begin analysis of $ARGUMENT**

1. **Read the original file** and identify:
   - Total line count and instruction complexity
   - File loading patterns (@filename usage)
   - Sequential vs parallel execution opportunities
   - Context overflow risk factors

2. **Apply the conversion framework** systematically:
   - Break complex workflows into discrete tasks
   - Design parallel subagent execution strategies
   - Implement context management techniques
   - Create TodoWrite task structure

3. **Generate the optimized version** with:
   - Efficient TodoWrite task JSON
   - Parallel subagent delegation instructions
   - Context-aware file selection strategies
   - Quality gates and verification procedures

4. **Overwrite the original file** (mark "validate_output" completed, "overwrite_original" as in_progress):
   - Use Write tool to overwrite $ARGUMENT with the converted slash command
   - Ensure the optimized version maintains the same analytical depth while avoiding context limits
   - Include proper error handling and validation before overwriting

5. **Confirm completion** (mark "overwrite_original" completed):
   - Display confirmation message: "✅ Original file updated with optimized TodoWrite version"
   - Verify all 7 conversion tasks completed successfully

---

## CRITICAL SUCCESS PATTERNS FOR CONVERTED PROMPTS

### Context Overflow Prevention Framework

**The conversion tool MUST generate these patterns to prevent context overflow:**

1. **Task Delegation Instructions**:

   ```markdown
   ### Phase 1: Security Analysis

   **TodoWrite**: Mark "security_analysis" as in_progress
   **Task Delegation**: Use Task tool with focused analysis:

   Task Description: "Security Analysis of Target Codebase"
   Task Prompt: "Analyze security focusing on [specific areas]

   **CONTEXT MANAGEMENT**: Analyze only 3-5 key files:

   - [File 1] (specific purpose)
   - [File 2-3] (specific modules)
   - [File 4-5] (specific handlers)

   Provide findings with file:line references."
   ```

2. **Verification Using Task Tool**:

   ```markdown
   ### GEMINI VERIFICATION (Task-Based)

   **Task Delegation**: Use Task tool for verification:

   Task Description: "Gemini Verification of Analysis Report"
   Task Prompt: "Verify analysis accuracy using progressive examination

   **PROGRESSIVE VERIFICATION**:

   - Verify findings through targeted code review
   - Cross-reference specific sections progressively

   Report file: {report_file_path}"
   ```

3. **Explicit Context Rules**:

   ```markdown
   **CONTEXT MANAGEMENT RULES**:

   - Maximum 5 files per Task operation
   - Use Task tool for all analysis phases
   - Progressive analysis through Task boundaries
   - Fresh context for each Task operation

   **AVOID**: @file1 @file2 @file3... bulk loading patterns
   **USE**: Task delegation with strategic file selection
   ```

### Success Validation Checklist

**Converted prompts MUST include:**

- [ ] Task delegation instructions for each analysis phase
- [ ] Maximum 5 files per Task operation
- [ ] Progressive verification using Task tool
- [ ] Explicit context management warnings
- [ ] No bulk @filename loading patterns
- [ ] Fresh context strategy through Task boundaries

This framework transforms any complex, context-heavy prompt into an efficient TaskWrite tasklist method that avoids context overflow while maintaining analytical depth and coverage, automatically updating the original file with the optimized version.
