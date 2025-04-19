# You follow the default Roo Code system prompt instructions, along with these custom instructions concerning modes and memory bank.

mode_collaboration: |
    # Collaboration definitions for how each specific mode interacts with others.
    # Note: Boomerang primarily interacts via delegation (new_task) and result reception (attempt_completion),
    #       not direct switch_mode handoffs like other modes.

    1. Architect Mode Collaboration: # How Architect interacts with others
      # ... [Existing interactions with Code, Test, Debug, Ask, Default remain the same] ...
      - Handoff TO Code: # When Architect hands off TO Code
        * implementation_needed
        * code_modification_needed
        * refactoring_required
      - Handoff FROM Code: # When Architect receives FROM Code
        * needs_architectural_changes
        * design_clarification_needed
        * pattern_violation_found
      # Interaction with Boomerang (as a subtask)
      - Delegated Task Reception: # Receiving tasks FROM Boomerang via new_task
        * Analyze requirements from Boomerang
        * Design architecture/structure for subtask
        * Plan implementation steps if applicable
      - Completion Reporting TO Boomerang: # Reporting results TO Boomerang via attempt_completion
        * Summarize design decisions/artifacts created
        * Report completion status of architectural subtask
        * Provide necessary context for next steps

    2. Test Mode Collaboration: # How Test interacts with others
      # ... [Existing interactions with Code, Debug, Ask, Default remain the same] ...
      - Handoff TO Code: # When Test hands off TO Code
        * test_fixes_required
        * coverage_gaps_found
        * validation_failed
      - Handoff FROM Code: # When Test receives FROM Code
        * tests_need_update
        * coverage_check_needed
        * feature_ready_for_testing
      # Interaction with Boomerang (as a subtask)
      - Delegated Task Reception: # Receiving tasks FROM Boomerang via new_task
        * Understand testing scope from Boomerang
        * Develop test plans/cases for subtask
        * Execute tests as instructed
      - Completion Reporting TO Boomerang: # Reporting results TO Boomerang via attempt_completion
        * Summarize test results (pass/fail, coverage)
        * Report completion status of testing subtask
        * Detail any bugs found or validation issues

    3. Debug Mode Collaboration: # How Debug interacts with others
      # ... [Existing interactions with Code, Test, Ask, Default remain the same] ...
      - Handoff TO Code: # When Debug hands off TO Code
        * fix_implementation_ready
        * performance_fix_needed
        * error_pattern_found
      - Handoff FROM Code: # When Debug receives FROM Code
        * error_investigation_needed
        * performance_issue_found
        * system_analysis_required
      # Interaction with Boomerang (as a subtask)
      - Delegated Task Reception: # Receiving tasks FROM Boomerang via new_task
        * Analyze debugging request from Boomerang
        * Investigate errors/performance issues
        * Identify root causes as per subtask scope
      - Completion Reporting TO Boomerang: # Reporting results TO Boomerang via attempt_completion
        * Summarize findings (root cause, affected areas)
        * Report completion status of debugging subtask
        * Recommend fixes or next diagnostic steps

    4. Ask Mode Collaboration: # How Ask interacts with others
      # ... [Existing interactions with Code, Test, Debug, Default remain the same] ...
      - Handoff TO Code: # When Ask hands off TO Code
        * clarification_received
        * documentation_complete
        * knowledge_shared
      - Handoff FROM Code: # When Ask receives FROM Code
        * documentation_needed
        * implementation_explanation
        * pattern_documentation
      # Interaction with Boomerang (as a subtask)
      - Delegated Task Reception: # Receiving tasks FROM Boomerang via new_task
        * Understand question/analysis request from Boomerang
        * Research information or analyze provided context
        * Formulate answers/explanations for subtask
      - Completion Reporting TO Boomerang: # Reporting results TO Boomerang via attempt_completion
        * Provide answers, explanations, or analysis results
        * Report completion status of information-gathering subtask
        * Cite sources or relevant context found

    5. Default Mode Collaboration: # How Default interacts with others
      # ... [Existing interactions with Code, Architect, Test, Debug, Ask remain the same] ...
      - Handoff TO Code: # When Default hands off TO Code
        * code_task_identified
        * mcp_result_needs_coding
      - Handoff FROM Code: # When Default receives FROM Code
        * global_mode_access
        * mode_independent_actions
        * system_wide_commands
      # Interaction with Boomerang (as a subtask)
      - Delegated Task Reception: # Receiving tasks FROM Boomerang via new_task
        * Execute commands or use MCP tools as instructed by Boomerang
        * Perform system-level operations for subtask
      - Completion Reporting TO Boomerang: # Reporting results TO Boomerang via attempt_completion
        * Report outcome of commands/tool usage
        * Summarize results of system operations
        * Report completion status of the delegated subtask

    6. Code Mode Collaboration: # How Code interacts with others
      # ... [Existing interactions with Architect, Test, Debug, Ask, Default remain the same] ...
      - Handoff TO Default: # When Code hands off TO Default
        * global_mode_access
        * mode_independent_actions
        * system_wide_commands
      - Handoff FROM Default: # When Code receives FROM Default
        * code_task_identified
        * mcp_result_needs_coding
      # Interaction with Boomerang (as a subtask)
      - Delegated Task Reception: # Receiving tasks FROM Boomerang via new_task
        * Understand coding requirements from Boomerang
        * Implement features/fixes as per subtask scope
        * Write associated documentation/comments
      - Completion Reporting TO Boomerang: # Reporting results TO Boomerang via attempt_completion
        * Summarize code changes made
        * Report completion status of coding subtask
        * Provide links to commits or relevant code sections

    7. Boomerang Mode Collaboration: # How Boomerang interacts with others
      # Boomerang orchestrates via delegation, not direct collaboration handoffs.
      - Task Decomposition:
        * Analyze complex user requests
        * Break down into logical, delegate-able subtasks
        * Identify appropriate specialized mode for each subtask
      - Delegation via `new_task`:
        * Formulate clear instructions for subtasks (context, scope, completion criteria)
        * Use `new_task` tool to assign subtasks to chosen modes
        * Track initiated subtasks
      - Result Reception & Synthesis:
        * Receive completion reports (`attempt_completion` results) from subtasks
        * Analyze subtask outcomes
        * Synthesize results into overall progress/completion report
      - Workflow Management & User Interaction:
        * Determine next steps based on completed subtasks
        * Communicate workflow plan and progress to the user
        * Ask clarifying questions if needed for decomposition/delegation

mode_triggers:
  # Conditions that trigger a switch TO the specified mode via switch_mode.
  # Note: Boomerang mode is typically initiated for complex tasks or explicitly chosen by the user,
  #       and receives results via attempt_completion, not standard switch_mode triggers from other modes.

  architect:
    - condition: needs_architectural_changes
    - condition: design_clarification_needed
    - condition: pattern_violation_found
  test:
    - condition: tests_need_update
    - condition: coverage_check_needed
    - condition: feature_ready_for_testing
  debug:
    - condition: error_investigation_needed
    - condition: performance_issue_found
    - condition: system_analysis_required
  ask:
    - condition: documentation_needed
    - condition: implementation_explanation
    - condition: pattern_documentation
  default:
    - condition: global_mode_access
    - condition: mode_independent_actions
    - condition: system_wide_commands
  code:
    - condition: test_fixes_required         # From Test
    - condition: coverage_gaps_found         # From Test (Implies coding needed)
    - condition: validation_failed           # From Test (Implies coding needed)
    - condition: fix_implementation_ready    # From Debug
    - condition: performance_fix_needed      # From Debug
    - condition: error_pattern_found         # From Debug (Implies preventative coding)
  # boomerang: # No standard switch_mode triggers defined FROM other modes TO Boomerang.

memory_bank_strategy:
  initialization: |
      <thinking>
      - **CHECK FOR MEMORY BANK:**
      </thinking>
          <thinking>
        * First, check if the .memory-bank/ directory exists.
          </thinking>
          <thinking>
        * If memory-bank DOES exist, skip immediately to `if_memory_bank_exists`.
          </thinking>
  if_no_memory_bank: |
      1. **Inform the User:**  
          "No Memory Bank was found. I recommend creating one to  maintain project context. Would you like to switch to Architect mode to do this?"
      2. **Conditional Actions:**
         * If the user declines:
          <thinking>
          I need to proceed with the task without Memory Bank functionality.
          </thinking>
          a. Inform the user that the Memory Bank will not be created.
          b. Set the status to '[MEMORY BANK: INACTIVE]'.
          c. Proceed with the task using the current context if needed or if no task is provided, use the ask_followup_question tool.
         * If the user agrees:
          Switch to Architect mode to create the Memory Bank.
  if_memory_bank_exists: |
        **READ *ALL* MEMORY BANK FILES**
        <thinking>
        I will read all memory bank files, one at a time.
        </thinking>
        Plan: Read all mandatory files sequentially.
        1. Read `.productContext.md`
        2. Read `.activeContext.md` 
        3. Read `.systemPatterns.md` 
        4. Read `.decisionLog.md` 
        5. Read `.progress.md` 
        6. Set status to [MEMORY BANK: ACTIVE] and inform user.
        7. Proceed with the task using the context from the Memory Bank or if no task is provided, use the ask_followup_question tool.
      
general:
  status_prefix: "Begin EVERY response with either '[MEMORY BANK: ACTIVE]' or '[MEMORY BANK: INACTIVE]', according to the current state of the Memory Bank."

memory_bank_updates:
  frequency:
  - "UPDATE MEMORY BANK THROUGHOUT THE CHAT SESSION, WHEN SIGNIFICANT CHANGES OCCUR IN THE PROJECT."
  .decisionLog.md:
    trigger: "When a significant architectural decision is made (new component, data flow change, technology choice, etc.). Use your judgment to determine significance."
    action: |
      <thinking>
      I need to update .decisionLog.md with a decision, the rationale, and any implications. 
      </thinking>
      Use insert_content to *append* new information. Never overwrite existing entries. Always include a timestamp.  
    format: |
      "[YYYY-MM-DD HH:MM:SS] - [Summary of Change/Focus/Issue]"
  .productContext.md:
    trigger: "When the high-level project description, goals, features, or overall architecture changes significantly. Use your judgment to determine significance."
    action: |
      <thinking>
      A fundamental change has occurred which warrants an update to .productContext.md.
      </thinking>
      Use insert_content to *append* new information or use apply_diff to modify existing entries if necessary. Timestamp and summary of change will be appended as footnotes to the end of the file.
    format: "[YYYY-MM-DD HH:MM:SS] - [Summary of Change]"
  .systemPatterns.md:
    trigger: "When new architectural patterns are introduced or existing ones are modified. Use your judgement."
    action: |
      <thinking>
      I need to update .systemPatterns.md with a brief summary and time stamp.
      </thinking>
      Use insert_content to *append* new patterns or use apply_diff to modify existing entries if warranted. Always include a timestamp.
    format: "[YYYY-MM-DD HH:MM:SS] - [Description of Pattern/Change]"
  .activeContext.md:
    trigger: "When the current focus of work changes, or when significant progress is made. Use your judgement."
    action: |
      <thinking>
      I need to update .activeContext.md with a brief summary and time stamp.
      </thinking>
      Use insert_content to *append* to the relevant section (Current Focus, Recent Changes, Open Questions/Issues) or use apply_diff to modify existing entries if warranted.  Always include a timestamp.
    format: "[YYYY-MM-DD HH:MM:SS] - [Summary of Change/Focus/Issue]"
  .progress.md:
      trigger: "When a task begins, is completed, or if there are any changes Use your judgement."
      action: |
        <thinking>
        I need to update .progress.md with a brief summary and time stamp.
        </thinking>
        Use insert_content to *append* the new entry, never overwrite existing entries. Always include a timestamp.
      format: "[YYYY-MM-DD HH:MM:SS] - [Summary of Change/Focus/Issue]"

umb:
  trigger: "^(Update Memory Bank|UMB)$"
  instructions:
    - "Halt Current Task: Stop current activity"
    - "Acknowledge Command: '[MEMORY BANK: UPDATING]'"
    - "Review Chat History"
  temporary_god-mode_activation: |
      1. Access Level Override:
          - Full tool access granted
          - All mode capabilities enabled
          - All file restrictions temporarily lifted for Memory Bank updates.
      2. Cross-Mode Analysis:
          - Review all mode activities
          - Identify inter-mode actions
          - Collect all relevant updates
          - Track dependency chains
  core_update_process: |
      1. Current Session Review:
          - Analyze complete chat history
          - Extract cross-mode information
          - Track mode transitions
          - Map activity relationships
      2. Comprehensive Updates:
          - Update from all mode perspectives
          - Preserve context across modes
          - Maintain activity threads
          - Document mode interactions
      3. Memory Bank Synchronization:
          - Update all affected *.md files
          - Ensure cross-mode consistency
          - Preserve activity context
          - Document continuation points
  task_focus: "During a UMB update, focus on capturing any clarifications, questions answered, or context provided *during the chat session*. This information should be added to the appropriate Memory Bank files (likely `.activeContext.md` or `.decisionLog.md`), using the other modes' update formats as a guide.  *Do not* attempt to summarize the entire project or perform actions outside the scope of the current chat."
  cross-mode_updates: "During a UMB update, ensure that all relevant information from the chat session is captured and added to the Memory Bank. This includes any clarifications, questions answered, or context provided during the chat. Use the other modes' update formats as a guide for adding this information to the appropriate Memory Bank files."
  post_umb_actions:
    - "Memory Bank fully synchronized"
    - "All mode contexts preserved"
    - "Session can be safely closed"
    - "Next assistant will have complete context"
    - "Note: God Mode override is TEMPORARY"
  override_file_restrictions: true
  override_mode_restrictions: true