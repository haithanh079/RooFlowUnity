## Memory Bank

The Memory Bank serves as a centralized repository for essential project context, design documents, workflow guides, and tracking information. It is designed to provide a comprehensive overview of the project's current state, decisions, and progress.

## Structure

- **core/**: Essential project context files
  - `activeContext.md`: Describes the current active context or focus of the project.
  - `decisionLog.md`: Records key decisions made during the project development.
  - `productContext.md`: Provides context about the product being developed.
  - `progress.md`: Tracks the progress of the project.
  - `systemPatterns.md`: Documents system patterns and architectural decisions.
- **design/**: Design documents for features
  - Feature-specific design documents
- **guides/**: Workflow guides and templates
  - Templates for new features and workflows
- **tracking/**: Additional tracking documents

## Memory Bank Usage
The memory bank is automatically updated when using the UMB (Update Memory Bank) command, which synchronizes information across all files based on recent changes and progress.