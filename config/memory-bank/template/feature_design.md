# [Feature Name] Design Document

## 1. Overview
- What this feature does
- Why it’s needed (problem → solution)
- Expected outcome

## 2. System Design
- Key system/module interactions
- Dependencies or impacted components
- Optional: Mermaid diagram

## 3. Change Outline
List all new/removed/modified items in logical order — **new additions first**, followed by changes and removals.  
Group fields and methods under their respective classes. Use `// TODO` in method bodies to describe intent. No implementation code.

1. Class: `MyFeatureManager`
   - Field: `bool isFeatureEnabled`  
     Used to gate logic at runtime.
   - Method: `Init()`  
     `// TODO: Initialize config and internal state`
   - Method: `Start()`  
     `// TODO: Entry point when feature is activated`
   - Method: `HandleEvent(Event e)`  
     `// TODO: Validate input and trigger logic`

2. Enum: `FeatureState`
   - Values: `Idle`, `Active`, `Complete`  
     Controls lifecycle state of the feature.

...

## 4. Notes
- Risks, edge cases, or simplifications
- What’s excluded or postponed
- Any remaining questions or assumptions