## Initial Context Building

1. **Read Architecture Documentation First**
   - ALWAYS begin by reading `.memory-bank/design/project_architecture.md` to understand the overall project structure, patterns, and design principles.
   - This step is mandatory before proceeding with any task analysis.

## Workflow

1. Scout First  
   Use `read_file`, `search_files`, or other tools to gather context. Reuse existing utilities/components. Avoid introducing new types unless absolutely necessary.

2. Ask Clarifying Questions  
   Focus only on the happy path. Avoid future-proofing, edge cases, testing, or documentation. No implementation code is allowed at this stage.

3. Create Implementation Plan (only after fully understanding the task)  
   Focus on:
   - Best effort-to-impact return
   - Bare minimum needed
   - Reusable or adaptable components
   - Tasks that can be postponed

4. Design Feedback Loop  
   Reflect on purpose, effort vs benefit:
   - What to exclude, simplify, or delay
   - Note risks, flaws, or missing logic (not necessarily solve)
   Then:
   - Ask user for review
   - Redesign and update existing design file only
   - Iterate until user confirms

5. Use `switch_mode` to hand off implementation once design is confirmed.

---

## Design Output Expectations

1. Plan Overview  
   - High-level system description  
   - Major module/system interactions  
   - Avoid low-level detail unless essential

2. Change List  
   List all new/removed/modified items, each with purpose:
   - Classes / Structs / Enums  
   - Fields / Properties  
   - Method signatures with `// TODO: ...` in body explaining implementation flow or dependencies (no actual code)

---

## Guidelines

- Ensure logical correctness (refine iteratively)
- Prefer reuse over new implementation
- Keep it simple, focused, and clear
- Do exactly what's needed — no more
- Follow Unity standards (e.g. serializable types, supported syntax)
- Prefer:
  - Early return
  - Flat control flow
  - Fields over properties
  - `Array.Empty<T>()` over `new T[0]`
  - Maintainable, readable structure
- Update imports after changes
- Respect scopes (`public`, `private`, `internal`)
- Must support Unity 2020.3+

---

## Restrictions

- No `/// <summary>` or doc tags  
- No `try/catch`, `#region`, `#pragma`, or PowerShell  
- No over-engineering or abstracting prematurely  
- No constructors (use `Init()` instead)  
- No tiny (<5 line) utility methods  
- No constructor for serializable classes (use `public T Init()` if needed)  
- No new files, types, or members unless absolutely required  
- Code must compile — no broken imports, brackets, or scopes  
- No duplicated logic — extract and reuse existing code