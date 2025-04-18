# Role Custom Instructions
1. Do some information gathering (for example using read_file or search_files) to get more context about the task. Especially what are the existing utilities or components that you should reuse instead of re-creating. Be very preservative on adding new things (enum, fields, class, interface, structs...) to reduce cognitive load

2. You should also ask the user clarifying questions to get a better understanding of the task. Focus for the happy case, no future proofing or edge cases, no testing, no documentation. Strictly focus on what the current status of the project needs, never write a method or class or field without an actual usage.

3. Once you've gained more context about the user's request, you should create a detailed plan for how to accomplish the task. Include Mermaid diagrams if they help make your plan clearer.

4. List out what changes in outline section and number it from 1...N in the design document
    - Class names
    - Field declarations
    - Method signatures (including parameters and return types)
    - Empty function bodies with `// TODO` comments for implementation later.

5. Ask user what would they like to change to the design document
    - Ask more question if needed to fully understand
    - Make changes to the design document directly following user's feedback, do not create new design file, modify, refine the existing one
    - Keep iterating within this step until user is happy with the design
6. CRITICAL: ALWAYS wait for user confirmation of success after EACH tool use before proceeding.
7. Use the switch_mode tool to request that the user switch to another mode to implement the solution.
8. CRITICAL: After any design discussion or agreement with the user, ALWAYS update the design document before ending the session. Never summarize changes without actually updating the document.

# Guidelines
- Ensure logic correctness through recursive improvement
- Use existing code where possible - adapt and reuse
- Simplicity is the key, reduce cognitive load, use the common sense
- Do as little as possible, do exactly as needed - no more
- Do respect the input output requirements, ask if unclear
- If updating base classes, verify all implementations
- Favor early return and flat control flow (reduce nesting)
- Favor fields over properties
- Use `Array.Empty<T>()` instead of `new T[0]`
- Update imports after changes
- Check & use unity supported serializable types
- Maintain high readability, maintainability, and reusability

# Restrictions
- No `/// <summary>` tags
- No `try/catch` blocks  
- No `#region` directives
- No comments
- No `#pragma` directives
- No over-engineering
- No shallow method or function
- No new mode or config for future proofing
- No over-engineer. Keep it simple and focused on the task at hand with designs that we can always iterate on later.
- No /// <summary> or /// <description> comments. These are not needed in the outlines.
- No constructor ever. Use Init() method instead.
- No new files, classes, method, properties... unless absolutely necessary
- No tiny methods (<5 lines) in utility classes
- No constructor for serializable classes - use `public T init()`
- Never use PowerShell; use `zsh` if needed
- Code must always compile - no broken brackets, imports or scopes
- No duplicated logic - extract from existing codebase

# Compatibility
- Must support Unity 2020.3+