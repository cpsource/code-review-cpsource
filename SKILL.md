---
name: code-review-cpsource
description: |
  Provides comprehensive code review guidance for React 19, Vue 3, Rust, TypeScript, Java, Python, and C/C++.
  Helps catch bugs, improve code quality, and give constructive feedback.
  Includes a mandatory verification phase: every suspected bug must be cross-referenced against
  the test suite, callers, and concrete execution traces before being reported.
  Use when: reviewing pull requests, conducting PR reviews, code review, reviewing code changes,
  establishing review standards, mentoring developers, architecture reviews, security audits,
  checking code quality, finding bugs, giving feedback on code.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash      # 运行 lint/test/build 命令验证代码质量
  - WebFetch  # 查阅最新文档和最佳实践
---

# Code Review Excellence

Transform code reviews from gatekeeping to knowledge sharing through constructive feedback, systematic analysis, and collaborative improvement.

## When to Use This Skill

- Reviewing pull requests and code changes
- Establishing code review standards for teams
- Mentoring junior developers through reviews
- Conducting architecture reviews
- Creating review checklists and guidelines
- Improving team collaboration
- Reducing code review cycle time
- Maintaining code quality standards

## Core Principles

### 1. The Review Mindset

**Goals of Code Review:**
- Catch bugs and edge cases
- Ensure code maintainability
- Share knowledge across team
- Enforce coding standards
- Improve design and architecture
- Build team culture

**Not the Goals:**
- Show off knowledge
- Nitpick formatting (use linters)
- Block progress unnecessarily
- Rewrite to your preference

### 2. Effective Feedback

**Good Feedback is:**
- Specific and actionable
- Educational, not judgmental
- Focused on the code, not the person
- Balanced (praise good work too)
- Prioritized (critical vs nice-to-have)

```markdown
❌ Bad: "This is wrong."
✅ Good: "This could cause a race condition when multiple users
         access simultaneously. Consider using a mutex here."

❌ Bad: "Why didn't you use X pattern?"
✅ Good: "Have you considered the Repository pattern? It would
         make this easier to test. Here's an example: [link]"

❌ Bad: "Rename this variable."
✅ Good: "[nit] Consider `userCount` instead of `uc` for
         clarity. Not blocking if you prefer to keep it."
```

### 3. Review Scope

**What to Review:**
- Logic correctness and edge cases
- Security vulnerabilities
- Performance implications
- Test coverage and quality
- Error handling
- Documentation and comments
- API design and naming
- Architectural fit

**What Not to Review Manually:**
- Code formatting (use Prettier, Black, etc.)
- Import organization
- Linting violations
- Simple typos

## Review Process

### Phase 1: Context Gathering (2-3 minutes)

Before diving into code, understand:
1. Read PR description and linked issue
2. Check PR size (>400 lines? Ask to split)
3. Review CI/CD status (tests passing?)
4. Understand the business requirement
5. Note any relevant architectural decisions

### Phase 2: High-Level Review (5-10 minutes)

1. **Architecture & Design** - Does the solution fit the problem?
   - For significant changes, consult [Architecture Review Guide](reference/architecture-review-guide.md)
   - Check: SOLID principles, coupling/cohesion, anti-patterns
2. **Performance Assessment** - Are there performance concerns?
   - For performance-critical code, consult [Performance Review Guide](reference/performance-review-guide.md)
   - Check: Algorithm complexity, N+1 queries, memory usage
3. **File Organization** - Are new files in the right places?
4. **Testing Strategy** - Are there tests covering edge cases?

### Phase 3: Line-by-Line Review (10-20 minutes)

For each file, check:
- **Logic & Correctness** - Edge cases, off-by-one, null checks, race conditions
- **Security** - Input validation, injection risks, XSS, sensitive data
- **Performance** - N+1 queries, unnecessary loops, memory leaks
- **Maintainability** - Clear names, single responsibility, comments

### Phase 3.5: Verify Before Reporting (CRITICAL)

**Every suspected bug MUST be verified before it is reported.** Static reading
of code is not sufficient — you must cross-reference callers, tests, and runtime
behavior to confirm a finding is real. Unverified findings damage review
credibility and waste the author's time.

**Verification steps for each finding:**

1. **Check the test suite.** Search for tests that exercise the suspected buggy
   code path. If tests exist and assert specific behavior that contradicts your
   assumption, the code may be correct and your mental model wrong.

   ```
   # Find tests for the function you think is buggy
   grep -rn "functionName" tests/
   ```

2. **Check callers.** Search for all call sites of the function. How do real
   callers use the API? Do they pass the arguments you think are problematic?
   A function that looks wrong in isolation may be correct given its actual
   usage contract.

   ```
   grep -rn "functionName(" src/ --include="*.c"
   ```

3. **Trace through edge cases manually.** For tricky logic (pointer arithmetic,
   loop bounds, state machines), pick concrete input values and trace execution
   step by step. Write down the variable states. Do not rely on intuition for
   off-by-one or signed/unsigned boundary conditions.

4. **Distinguish "looks wrong" from "is wrong."** Common false positives:
   - Intentional sentinel values (e.g., `-1` meaning "end of list")
   - Deliberately asymmetric push/pop semantics
   - Legacy API contracts that differ from what the name suggests
   - Code that is correct but poorly documented

5. **If you cannot verify, say so.** Label unverified findings explicitly:
   ```
   [unverified] This appears to allow negative indices, but I could not
   find tests or callers that exercise this path. Needs manual confirmation.
   ```

**What NOT to do:**
- Do not report a function as buggy based solely on reading its implementation
  without checking how it is actually called
- Do not assume a doc comment is the specification — the test suite is the
  ground truth for intended behavior
- Do not flag intentional design patterns (sentinel values, walk-to-end loops)
  as bugs without first checking whether tests confirm the behavior

### Phase 4: Summary & Decision (2-3 minutes)

1. Summarize key concerns
2. Highlight what you liked
3. Make clear decision:
   - ✅ Approve
   - 💬 Comment (minor suggestions)
   - 🔄 Request Changes (must address)
4. Offer to pair if complex

## Review Techniques

### Technique 1: The Checklist Method

Use checklists for consistent reviews. See [Security Review Guide](reference/security-review-guide.md) for comprehensive security checklist.

### Technique 2: The Question Approach

Instead of stating problems, ask questions:

```markdown
❌ "This will fail if the list is empty."
✅ "What happens if `items` is an empty array?"

❌ "You need error handling here."
✅ "How should this behave if the API call fails?"
```

### Technique 3: Suggest, Don't Command

Use collaborative language:

```markdown
❌ "You must change this to use async/await"
✅ "Suggestion: async/await might make this more readable. What do you think?"

❌ "Extract this into a function"
✅ "This logic appears in 3 places. Would it make sense to extract it?"
```

### Technique 4: Differentiate Severity

Report findings in four buckets:

1. **Critical** — Memory corruption, auth bypass, data race with real impact,
   undefined behavior reachable in production, data loss. Must fix before merge.
2. **Major** — Wrong behavior, resource leak, missing validation, bad API
   design, strong maintainability issue. Should fix; discuss if disagree.
3. **Minor** — Style, readability, small inefficiency, naming, comment cleanup.
   Non-blocking.
4. **Questions / Assumptions** — Things that may be okay, but need confirmation
   from the author. Not asserting a bug — asking for clarification.

### Technique 5: The Compact C/C++ Review Prompt

For every C or C++ review, answer these ten questions explicitly. If a question
has no findings, say so — do not skip it silently:

1. **What can crash?** (null deref, assert, abort, unhandled error)
2. **What can corrupt memory?** (buffer overflow, use-after-free, double free,
   uninitialized read, wild pointer)
3. **What can leak resources?** (memory, files, sockets, locks, handles)
4. **What can be exploited by hostile input?** (injection, format string, path
   traversal, overflow to code execution)
5. **What can deadlock or race?** (lock ordering, check-then-act, unprotected
   shared state)
6. **What is undefined or non-portable?** (signed overflow, strict aliasing,
   misaligned access, platform assumptions)
7. **What is hard to maintain?** (coupling, unclear ownership, magic numbers,
   duplicated logic, missing abstraction)
8. **What tests are missing?** (edge cases, error paths, concurrency, security
   boundaries)
9. **What is the highest-risk issue?** (single most important finding)
10. **What is the simplest fix?** (for the highest-risk issue)

### Always-On C/C++ Checks

These checks are **mandatory even in a quick review** of C or C++ code. Never
skip them regardless of time pressure:

1. Bounds and length handling
2. Ownership and lifetime
3. Integer conversion and overflow
4. Error-path cleanup
5. Thread-safety assumptions
6. UB triggers
7. Input trust boundaries
8. Raw pointer use
9. Copy/move/destructor correctness (C++)
10. Test coverage for edge and failure cases

### C/C++ Section Framework

For C and C++ reviews, the review MUST walk through all 20 sections of the
[C/C++ Master Checklist](reference/c-cpp-master-checklist.md). Report findings
under the section headings so nothing is missed. Sections with no findings
should be noted as clean.

See the language-specific guides for detailed patterns and examples:
- [C Guide](reference/c.md) — memory safety, UB, conditional compilation, format strings
- [C++ Guide](reference/cpp.md) — RAII, lifetime, move semantics, templates

## Language-Specific Guides

根据审查的代码语言，查阅对应的详细指南：

| Language/Framework | Reference File | Key Topics |
|-------------------|----------------|------------|
| **React** | [React Guide](reference/react.md) | Hooks, useEffect, React 19 Actions, RSC, Suspense, TanStack Query v5 |
| **Vue 3** | [Vue Guide](reference/vue.md) | Composition API, 响应性系统, Props/Emits, Watchers, Composables |
| **Rust** | [Rust Guide](reference/rust.md) | 所有权/借用, Unsafe 审查, 异步代码, 错误处理 |
| **TypeScript** | [TypeScript Guide](reference/typescript.md) | 类型安全, async/await, 不可变性 |
| **Python** | [Python Guide](reference/python.md) | 可变默认参数, 异常处理, 类属性 |
| **Java** | [Java Guide](reference/java.md) | Java 17/21 新特性, Spring Boot 3, 虚拟线程, Stream/Optional |
| **Go** | [Go Guide](reference/go.md) | 错误处理, goroutine/channel, context, 接口设计 |
| **C** | [C Guide](reference/c.md) | 指针/缓冲区, 内存安全, UB, 错误处理 |
| **C++** | [C++ Guide](reference/cpp.md) | RAII, 生命周期, Rule of 0/3/5, 异常安全 |
| **CSS/Less/Sass** | [CSS Guide](reference/css-less-sass.md) | 变量规范, !important, 性能优化, 响应式, 兼容性 |
| **Qt** | [Qt Guide](reference/qt.md) | 对象模型, 信号/槽, 内存管理, 线程安全, 性能 |

## Additional Resources

- [C/C++ Master Checklist](reference/c-cpp-master-checklist.md) - 20-section mandatory checklist for C/C++ reviews
- [Architecture Review Guide](reference/architecture-review-guide.md) - 架构设计审查指南（SOLID、反模式、耦合度）
- [Performance Review Guide](reference/performance-review-guide.md) - 性能审查指南（Web Vitals、N+1、复杂度）
- [Common Bugs Checklist](reference/common-bugs-checklist.md) - 按语言分类的常见错误清单
- [Security Review Guide](reference/security-review-guide.md) - 安全审查指南
- [Code Review Best Practices](reference/code-review-best-practices.md) - 代码审查最佳实践
- [PR Review Template](assets/pr-review-template.md) - PR 审查评论模板
- [Review Checklist](assets/review-checklist.md) - 快速参考清单
