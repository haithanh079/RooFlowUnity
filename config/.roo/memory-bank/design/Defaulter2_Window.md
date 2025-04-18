# Defaulter2 Window for Asset Creation and Configuration Design Document

## 1. Feature Overview
Brief description of what this feature does and how it fits into the project
- [ ] Create a dedicated Unity Editor window for Defaulter2 configuration
- [ ] Provide a centralized interface for creating and managing Defaulter2 assets
- [ ] Improve user experience by offering a more intuitive UI compared to the Inspector view

## 2. Feature Context and Analysis

### 2.1 Current Implementation / Related Systems
- Currently, users interact with Defaulter2 through the Inspector of a Defaulter2 ScriptableObject asset
- `Defaulter2Editor.cs` provides custom Inspector UI with tabs for different asset types
- `DF2_EditorUI.cs` contains utility methods for drawing UI elements
- Users must know to create a Defaulter2 asset via the Assets > Create menu before they can use it

### 2.2 Feature Gap Analysis
| Feature Area | Desired State | Current State | Gap Summary |
|--------------|----------------|----------------|-------------|
| Asset Creation | Create Defaulter2 assets from dedicated window | Must use Assets > Create menu | Need "Create Asset" button in window |
| Window Access | Menu item to open window from Unity toolbar | No dedicated window exists | Need EditorWindow implementation |
| UI Organization | Well-organized UI with sections and improved layout | Inspector-based UI with tabs | Need to adapt current UI to window context |
| User Guidance | Improved help/tooltips for first-time users | Minimal guidance in Inspector | Need better documentation integration |

### 2.3 Dependencies & Integration Notes
- Interfaces with: `Defaulter2.cs`, `DF2_TabDefinition.cs`, `DF2_EditorUI.cs`
- Relies on: `AssetDatabase` for asset creation and management
- Affected by: Unity Editor UI constraints, serialization system, EditorWindow lifecycle

## 3. Key Components

### Milestone 1
Create the basic window structure and asset management functionality.

- [ ] Create `Defaulter2Window.cs` class extending EditorWindow
- [ ] Implement menu item to open window (Window > Defaulter2)
- [ ] Add functionality to detect existing Defaulter2 assets
- [ ] Implement "Create Asset" button for when no assets are found
- [ ] Persist window state using EditorPrefs

### Milestone 2
Migrate the existing UI from the Inspector to the window.

- [ ] Port tab system from Defaulter2Editor to Defaulter2Window (reuse `_tabs` list and tab drawing logic from `Defaulter2Editor.cs`)
- [ ] Adapt the current UI elements to work in window context (reuse `DrawPresetListUI`, `DrawPreset`, `DrawFilterUI` methods from `Defaulter2Editor.cs` and `DF2_EditorUI.cs`)
- [ ] Implement scrolling for better content management (reuse `scrollPosition` Vector2 from `Defaulter2Editor.cs`)
- [ ] Add window-specific UI enhancements (better spacing, organization)
- [ ] Ensure all existing functionality works in the window context (Validate/Enforce actions, reuse `ValidateCurrentTab` and `BulkProcess` methods from `Defaulter2Editor.cs`)

---

## 4. Detailed Implementation Plan
This implementation plan involves adding a new class (`Defaulter2Window.cs`), modifying an existing class (`Defaulter2Editor.cs`), and reusing code from `DF2_EditorUI.cs`. No enums are being added or removed, and no files are being removed.
Outline all the changes with detailed information & code snippet about what's change and for what

### Class: Defaulter2Window.cs
```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using UnityEditor;
using UnityEngine;
using vietlabs.df2;

namespace vietlabs.df2
{
    public class Defaulter2Window : EditorWindow
    {
        // Window state
        private Defaulter2 _settings;
        private int _selectedTab;
        private Vector2 _scrollPosition;
        private bool _isInitialized;
        private List<DF2_TabDefinition> _tabs;
        
        // Window constants
        private const string WindowTitle = "Defaulter2";
        private const string PreferencePrefix = "Defaulter2Window_";
        
        // Window creation
        [MenuItem("Window/Defaulter2/Settings")]
        public static void ShowWindow()
        {
            // Get existing open window or create a new one
            var window = GetWindow<Defaulter2Window>(WindowTitle);
            window.minSize = new Vector2(450, 600);
            window.Show();
        }
        
        // Initialization
        private void OnEnable()
        {
            // Load settings
            _settings = FindDefaulterAsset();
            
            // Initialize UI
            InitializeTabs();
            
            // Restore window state
            _selectedTab = EditorPrefs.GetInt($"{PreferencePrefix}SelectedTab", 0);
            _isInitialized = true;
        }
        
        private void OnDisable()
        {
            // Save window state
            if (_isInitialized)
            {
                EditorPrefs.SetInt($"{PreferencePrefix}SelectedTab", _selectedTab);
            }
        }
        
        // Core UI method
        private void OnGUI()
        {
            if (_settings == null)
            {
                DrawNoSettingsUI();
                return;
            }
            
            // Main layout
            EditorGUILayout.BeginVertical();
            
            // Top toolbar and mode toggle
            DrawToolbar();
            DrawModeToggle();
            
            // Main content area
            _scrollPosition = EditorGUILayout.BeginScrollView(_scrollPosition);
            if ((_selectedTab >= 0) && (_selectedTab < _tabs.Count))
            {
                _tabs[_selectedTab].DrawContent();
            }
            EditorGUILayout.EndScrollView();
            
            // Action buttons
            DF2_EditorUI.DrawSeparator();
            DrawActionButtons();
            
            EditorGUILayout.EndVertical();
        }
        
        // Asset management
        private Defaulter2 FindDefaulterAsset()
        {
            string[] guids = AssetDatabase.FindAssets("t:Defaulter2");
            if (guids.Length > 0)
            {
                string path = AssetDatabase.GUIDToAssetPath(guids[0]);
                return AssetDatabase.LoadAssetAtPath<Defaulter2>(path);
            }
            
            return null;
        }
        
        private void CreateDefaulterAsset()
        {
            // TODO: Copy/move asset creation logic from Defaulter2Editor.cs
        }
        
        // UI Sections
        private void DrawNoSettingsUI()
        {
            // TODO: Implement UI for creating new asset
        }
        
        private void InitializeTabs()
        {
            // TODO: Initialize tab definitions
        }
    }
}
```

### Updates to Defaulter2Editor.cs
- Extract common UI drawing code into shared methods that can be used by both the Editor and Window
- Add static helper methods for asset management

## 5. Testing Strategy
Define how user can help to test this features
- Test asset creation when no Defaulter2 asset exists in the project
- Verify that all tabs function correctly in the window context
- Test window persistence across Unity Editor restarts
- Check that all UI elements appear correctly and are properly sized
- Verify that Validate/Enforce actions work from the window
- Test with both light and dark editor themes
- Verify that the UI elements and functionalities reused from `Defaulter2Editor.cs` and `DF2_EditorUI.cs` work as expected in the new window context.
- Ensure window works correctly on different screen resolutions

## 6. Acceptance Criteria
List all the criteria to make this feature useful and bug free
- Window is accessible via Unity menu (Window > Defaulter2)
- Users can create a new Defaulter2 asset if none exists (verify that a new Defaulter2 asset is created in the project when the "Create Asset" button is clicked in the window)
- All existing functionality from Inspector view works in window (verify that all tabs and their respective UI elements are displayed correctly and function as expected in the window)
- Window state (selected tab, scroll position) persists between sessions (verify that the selected tab and scroll position are restored when the window is reopened after a Unity Editor restart)
- UI is properly sized and laid out in the window context (verify that all UI elements are properly sized and laid out in the window, and that there are no overlapping or misaligned elements)
- All actions (validate, enforce, etc.) function correctly (verify that the "Validate" and "Enforce" buttons function correctly and apply the selected presets to the assets)
- Window provides clear guidance for new users (verify that tooltips and help messages are displayed for all UI elements)
- Context-sensitive help is available where needed (verify that context-sensitive help is available for all UI elements)

## 7. Timeline Estimate
- Phase 1 (Window Creation & Asset Management): 3 days
- Phase 2 (UI Migration & Enhancement): 3 days
- Final Testing and Polish: 2 days

## 8. ConclusionPlanning Notes
- The window implementation should prioritize reusing code from the existing Editor
- Consider using a more modular approach for UI components to facilitate future extensions
- Potential follow-up features could include preset templates, batch operations, and improved analytics