# Toggle Flag to Show/Hide Preset Names in the Project Panel Design Document

## 1. Feature Overview
Brief description of what this feature does and how it fits into the project
- [ ] Add a toggle in the Defaulter2 window to show/hide preset names in Unity's Project panel
- [ ] Display the name of applied presets under assets in the Project panel when enabled
- [ ] Provide visual indication of which assets are managed by Defaulter2

## 2. Feature Context and Analysis

### 2.1 Current Implementation / Related Systems
- Currently, there's no visual indication in the Project panel of which preset is applied to an asset
- Users must inspect individual assets or use the Defaulter2 interface to determine preset assignments
- Unity provides `EditorApplication.projectWindowItemOnGUI` callback for drawing custom UI in the Project panel
- `Defaulter2` already has methods to determine which preset is applied to an asset

### 2.2 Feature Gap Analysis
| Feature Area | Desired State | Current State | Gap Summary |
|--------------|----------------|----------------|-------------|
| Visual Feedback | Show preset names under assets in Project panel | No visual indication of preset assignment | Need to implement Project panel UI integration |
| Toggle Control | User-configurable visibility via toggle | No toggle exists | Need to add toggle in Defaulter2 window |
| Persistence | Toggle state persists between editor sessions | No persistent setting | Need to implement preference storage |

### 2.3 Dependencies & Integration Notes
- Interfaces with: Unity's `ProjectWindowItemOnGUI` callback system
- Relies on: `Defaulter2` for preset assignment lookup
- Affected by: Project window view modes, asset label display constraints, Unity UI scaling

## 3. Key Components

### Milestone 1
Implement the toggle control and persistence.

- [ ] Add toggle control to Defaulter2 window
- [ ] Store toggle state as serialized field in Defaulter2
- [ ] Create logic to initialize and handle toggle state
- [ ] Update toggle state when changed

### Milestone 2
Implement the Project panel integration.

- [ ] Subscribe to `EditorApplication.projectWindowItemOnGUI`
- [ ] Identify assets with applied presets
- [ ] Draw preset names under matching assets
- [ ] Handle different Project panel view modes
- [ ] Ensure proper cleanup when disabled

---

## 4. Detailed Implementation Plan
Outline all the changes with detailed information & code snippet about what's change and for what

### 4.1 Caching Mechanism
To improve performance, a static dictionary in the `Defaulter2` class will be used to cache the preset for each asset. This is particularly important for the Project panel integration, where the preset lookup would be performed for every visible asset.

- Cache will be implemented as a static field in `Defaulter2` to ensure it persists regardless of window state
- Cache will be initialized when the preset names feature is first enabled
- Cache will be cleared when the feature is disabled
- Cache will store key-value pairs where:
  - Key: Asset path (string)
  - Value: The ImportPreset for that asset

### 4.2 Add Toggle and Cache to Defaulter2.cs

```csharp
public partial class Defaulter2 : ScriptableObject
{
    // Existing code...
    
    // Toggle state - serialized to persist with the asset
    [SerializeField] public bool showPresetNamesInProjectPanel = false;
    
    // Static cache for preset lookups to improve performance
    // Static ensures it persists even without an open window
    [NonSerialized] private static Dictionary<string, ImportPreset> _presetCache = new Dictionary<string, ImportPreset>();
    
    // Add method to clear cache (called when disabling the feature)
    public static void ClearPresetCache()
    {
        _presetCache.Clear();
    }
    
    // Method to toggle preset names visibility
    public void TogglePresetNamesVisibility(bool isVisible)
    {
        if (showPresetNamesInProjectPanel != isVisible)
        {
            showPresetNamesInProjectPanel = isVisible;
            
            if (showPresetNamesInProjectPanel)
            {
                // Subscribe to callback when enabled
                EditorApplication.projectWindowItemOnGUI += OnProjectWindowItemGUI;
            }
            else
            {
                // Unsubscribe from callback and clear cache when disabled
                EditorApplication.projectWindowItemOnGUI -= OnProjectWindowItemGUI;
                ClearPresetCache();
            }
            
            // Mark asset as dirty to save the serialized value
            EditorUtility.SetDirty(this);
            
            // Force project window repaint
            EditorApplication.RepaintProjectWindow();
        }
    }
    
    // Initialize the callbacks during OnEnable or after domain reload
    [InitializeOnLoadMethod]
    private static void InitializeCallbacks()
    {
        if (Api != null && Api.showPresetNamesInProjectPanel)
        {
            EditorApplication.projectWindowItemOnGUI += Api.OnProjectWindowItemGUI;
        }
    }
    
    // Handle project window GUI drawing
    private void OnProjectWindowItemGUI(string guid, Rect selectionRect)
    {
        // Skip if disabled
        if (!showPresetNamesInProjectPanel) return;
        
        // Get asset path from GUID
        string assetPath = AssetDatabase.GUIDToAssetPath(guid);
        
        // Skip if not a valid asset
        if (string.IsNullOrEmpty(assetPath) || !assetPath.StartsWith("Assets")) return;
        
        // Get preset from cache or calculate and cache if not found
        ImportPreset preset;
        if (!_presetCache.TryGetValue(assetPath, out preset))
        {
            // Use the existing GetBestMatchedPreset method directly
            preset = GetBestMatchedPreset(assetPath);
            _presetCache[assetPath] = preset; // Cache even null results to avoid repeated lookups
        }
        
        // Skip if no preset found
        if (preset == null) return;
        
        // Determine label position based on view mode
        Rect labelRect = selectionRect;
        
        // If icon size is small (list view), position to the right
        // If icon size is large (grid view), position below the name
        bool isListView = selectionRect.height <= 20;
        
        if (isListView)
        {
            // Position to the right of the asset name
            float originalWidth = labelRect.width;
            labelRect.x += originalWidth;
            labelRect.width = 100;
        }
        else
        {
            // Position below the asset name
            labelRect.y += 32; // Adjust as needed
            labelRect.height = 16;
        }
        
        // Draw the preset name
        GUIStyle style = new GUIStyle(EditorStyles.miniLabel);
        style.normal.textColor = new Color(0.6f, 0.6f, 1.0f);
        
        GUI.Label(labelRect, preset.name, style);
    }
}
```

### 4.3 Update Defaulter2Window.cs to use the Serialized Toggle

```csharp
public class Defaulter2Window : EditorWindow
{
    // Existing code...
    
    private void DrawToggleControls()
    {
        EditorGUILayout.Space(10);
        EditorGUILayout.BeginVertical(EditorStyles.helpBox);
        
        EditorGUILayout.LabelField("Project Panel Integration", EditorStyles.boldLabel);
        
        // Show preset names toggle
        EditorGUI.BeginChangeCheck();
        bool showPresetNames = EditorGUILayout.Toggle(
            new GUIContent("Show Preset Names",
            "Display preset names under assets in the Project panel"),
            _settings.showPresetNamesInProjectPanel);
            
        if (EditorGUI.EndChangeCheck())
        {
            // Use the method on Defaulter2 to toggle visibility
            _settings.TogglePresetNamesVisibility(showPresetNames);
        }
        
        EditorGUILayout.EndVertical();
    }
}
```

### 4.4 Using Existing GetBestMatchedPreset Method

We'll directly use the existing `GetBestMatchedPreset` method already available in the Defaulter2 class. This method already handles all the asset type determination and preset matching logic we need, so there's no need to create a separate `GetPresetForAsset` method:

```csharp
// Existing method in Defaulter2.cs (already available, no changes needed)
public ImportPreset GetBestMatchedPreset(string assetPath)
{
    if (assetPath.IsAudioPath())
    {
        float duration = AudioUtils.GetAudioDuration(assetPath);
        return GetAudioPresetForDuration(duration);
    }
    
    // Handle texture only
    if (!assetPath.IsTexturePath()) return null;
    
    string assetGuid = AssetDatabase.AssetPathToGUID(assetPath);
    bool isSpineAtlas = IsSpineAtlasTexture(assetGuid);
    if (isSpineAtlas) return spineAtlasPreset.WillProcessAsset(assetPath) ? spineAtlasPreset : null;
    
    var allMatches = PresetMatchingUtility.FindMatchingPresets(assetPath, texturePresets, "Texture")
        .Concat(PresetMatchingUtility.FindMatchingPresets(assetPath, spritePresets, "Sprite"))
        .ToList();
    return allMatches.Count == 0 ? null : PresetMatchingUtility.GetBestMatch(allMatches)?.Preset;
}
```

## 5. Testing Strategy
Define how user can help to test this features
- Test with various Project panel view modes (list view, icon view, different sizes)
- Verify that toggle properly shows/hides preset names
- Test with different asset types (textures, sprites, audio, etc.)
- Confirm that toggle state persists when Unity Editor is restarted
- Test with both light and dark editor themes
- Verify that performance remains acceptable with large project folders
- Check visual clarity of preset names with different background colors/assets
- Test performance with and without caching to verify improvement
- Confirm cache works without Defaulter2Window being open

## 6. Acceptance Criteria
List all the criteria to make this feature useful and bug free
- Toggle in Defaulter2 window correctly shows/hides preset names
- Preset names are visible under assets in the Project panel when enabled
- Toggle state persists between editor sessions (using serialized field instead of EditorPrefs)
- Preset names are legible in all Project panel view modes
- Implementation doesn't significantly impact Project panel performance
- Preset names automatically update when asset assignments change
- Visual style is consistent with Unity's UI guidelines
- Feature works correctly with all supported asset types
- Caching significantly improves performance for large projects
- Feature works even when Defaulter2Window is not open

## 7. Timeline Estimate
- Phase 1 (Toggle Control): 1 day
- Phase 2 (Project Panel Integration): 3 days
- Final Testing and Polish: 2 days

## 8. Conclusion/Planning Notes
- Consider adding color coding for different preset types
- Future enhancement could include filtering the Project panel by preset type
- Caching mechanism implemented to improve performance
- Consider making text style/color configurable via preferences