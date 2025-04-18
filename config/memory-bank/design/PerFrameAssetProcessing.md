# Per-Frame Asset Processing with Progress Bar Design Document

## 1. Feature Overview
Brief description of what this feature does and how it fits into the project
- [ ] Implement per-frame asset processing to prevent editor freezing during bulk operations
- [ ] Display a progress bar to provide visual feedback during processing
- [ ] Allow processing to be cancelled by the user if needed

## 2. Feature Context and Analysis

### 2.1 Current Implementation / Related Systems
- The current `ProcessTrackedAssets()` method in `Defaulter2.Processor.cs` processes all assets in a single batch
- `Defaulter2AssetPostProcessor` uses `EditorApplication.delayCall` to schedule processing
- `DF2_EditorUI.DrawProgressBar()` exists but is used only for validation, not incremental processing
- Asset processing can block the main thread, causing the editor to become unresponsive

### 2.2 Feature Gap Analysis
| Feature Area | Desired State | Current State | Gap Summary |
|--------------|----------------|----------------|-------------|
| Processing Model | Non-blocking, distributed across frames | Blocking, single-batch | Need to implement per-frame distribution |
| Visual Feedback | Progress bar showing current status | No visual feedback | Need progress bar integration |
| Cancellation | Allow user to cancel long-running operations | No cancellation support | Need cancellation mechanism |
| Responsiveness | Editor remains responsive during processing | Editor may freeze during large operations | Need to limit per-frame workload |

### 2.3 Dependencies & Integration Notes
- Interfaces with: `Defaulter2.Processor.cs`, `Defaulter2AssetPostProcessor`
- Relies on: `EditorApplication.update` for per-frame execution, `EditorUtility.DisplayProgressBar` for UI
- Affected by: Unity Editor update cycle, asset database operations, UI thread constraints

## 3. Key Components

### Milestone 1
Implement core per-frame processing logic.

- [ ] Modify `ProcessTrackedAssets()` to support incremental processing
- [ ] Create data structures to track processing state across frames
- [ ] Use `EditorApplication.update` to distribute work instead of immediate execution
- [ ] Implement throttling to limit assets processed per frame

### Milestone 2
Add visual feedback and user control.

- [ ] Integrate progress bar to display current processing status
- [ ] Add cancellation mechanism
- [ ] Ensure proper cleanup if processing is interrupted
- [ ] Add UI elements in Defaulter2 window to show processing status

---

## 4. Detailed Implementation Plan
Outline all the changes with detailed information & code snippet about what's change and for what

### Changes to Defaulter2.Processor.cs

```csharp
public partial class Defaulter2
{
    // Processing state moved to DF2_AssetProcessor
    
    // Configuration moved to DF2_AssetProcessor
    
    public DF2_AssetProcessor assetProcessor = new DF2_AssetProcessor(this);

    // Current ProcessTrackedAssets will be renamed to ProcessTrackedAssetsBlocking
    // and kept for backward compatibility
    public void ProcessTrackedAssetsBlocking()
    {
        // Existing implementation
    }
    
    // New method for distributed processing
    public void ProcessTrackedAssetsPerFrame()
    {
        assetProcessor.ProcessAssetsPerFrame();
    }
}
```

### New Class DF2_AssetProcessor.cs

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using UnityEditor;
using UnityEngine;
using vietlabs.df2; // Assuming extension methods are in this namespace

public class DF2_AssetProcessor
{
    [NonSerialized] private bool _isProcessing;
    [NonSerialized] private int _processedCount;
    [NonSerialized] private List<string> _guidsToProcess;
    [NonSerialized] private bool _cancelRequested;
    [SerializeField] private int assetsPerFrame = 10;
    private Dictionary<string, ImportPreset> _presetCache = new Dictionary<string, ImportPreset>();
    [NonSerialized] internal readonly HashSet<string> _processedGuids = new HashSet<string>();
    private readonly Defaulter2 _settings;

    public DF2_AssetProcessor(Defaulter2 settings)
    {
        _settings = settings;
    }

    public void ProcessAssetsPerFrame()
    {
        if (_isProcessing) return;

        _isProcessing = true;
        _processedCount = 0;
        _cancelRequested = false;
        _guidsToProcess = new List<string>(_settings.processQueue.assetGuids);
        _settings.processQueue.Clear();
        _presetCache.Clear();
        _processedGuids.Clear();

        // Register update callback
        EditorApplication.update += ProcessAssetsInUpdate;
    }

    private void ProcessAssetsInUpdate()
    {
        // Check for cancellation
        if (_cancelRequested)
        {
            CleanupProcessing();
            return;
        }

        // Process a limited number of assets per frame
        int assetsProcessedThisFrame = 0;

        while (_processedCount < _guidsToProcess.Count && assetsProcessedThisFrame < assetsPerFrame)
        {
            string guid = _guidsToProcess[_processedCount];
            string assetPath = AssetDatabase.GUIDToAssetPath(guid);

            if (string.IsNullOrEmpty(assetPath))
            {
                Debug.LogWarning($"Defaulter2: Asset with GUID {guid} path not found, skipping.");
                _processedCount++;
                continue;
            }

            _processedGuids.Add(guid);

            ImportPreset preset;
            if (!_presetCache.TryGetValue(guid, out preset))
            {
                preset = _settings.Api.GetBestMatchedPreset(assetPath);
                _presetCache[guid] = preset;
            }

            if (preset != null)
            {
                try
                {
                    preset.Apply(assetPath);
                }
                catch (Exception e)
                {
                    Debug.LogError($"Defaulter2: Error processing asset {assetPath}: {e.Message}");
                }
            }

            _processedCount++;
            assetsProcessedThisFrame++;
        }

        // Update progress bar
        float progress = (float)_processedCount / _guidsToProcess.Count;
        string title = "Defaulter2";
        string message = $"Processing assets ({_processedCount}/{_guidsToProcess.Count})...";
        EditorUtility.DisplayProgressBar(title, message, progress);

        // Check if processing is complete
        if (_processedCount >= _guidsToProcess.Count)
        {
            CleanupProcessing();
            Debug.Log($"Defaulter2: Successfully processed {_processedCount} assets.");
        }
    }

    private void CleanupProcessing()
    {
        EditorApplication.update -= ProcessAssetsInUpdate;
        EditorUtility.ClearProgressBar();
        _isProcessing = false;
        _cancelRequested = false;
    }

    public void CancelProcessing()
    {
        if (_isProcessing)
        {
            _cancelRequested = true;
            EditorApplication.delayCall += () => { _isProcessing = false; EditorUtility.ClearProgressBar(); };
        }
    }

    public bool IsProcessing()
    {
        return _isProcessing;
    }

    // Moved from Defaulter2AssetPostProcessor
    public static void OnPostprocessAllAssets(string[] importedAssets, string[] deletedAssets, string[] movedAssets, string[] movedFromAssetPaths, Defaulter2 settings)
    {
        if (settings == null) return;
        var requireProcess = false;
        var iter = importedAssets
            .Concat(movedAssets)
            .Where(path => path.StartsWith("Assets"))
            .Where(path => path.IsTexturePath() || path.IsAudioPath() || path.IsModelPath())
            .Distinct();

        foreach (string assetPath in iter)
        {
            if (!settings.TrackAssetChange(assetPath)) continue;
            requireProcess = true;
            // Debug.Log($"Defaulter2 <OnPostprocessAllAssets>: Detected change in {assetPath}");
        }

        if (!requireProcess) return;
        EditorApplication.delayCall -= DelayedProcessTrackedAssets;
        EditorApplication.delayCall += DelayedProcessTrackedAssets;

        void DelayedProcessTrackedAssets()
        {
            EditorApplication.delayCall -= DelayedProcessTrackedAssets;
            if (settings == null) return;
            if (!settings.isAutoMode) return;
            settings.ProcessTrackedAssets();
        }
    }

    [InitializeOnLoadMethod]
    private static void OnAfterAssemblyReload()
    {
        //Defaulter2._processedGuids.Clear(); //This is no longer needed
        //if (Defaulter2.Api == null) return;
        //EditorApplication.delayCall -= DelayedProcessTrackedAssets;
        //EditorApplication.delayCall += DelayedProcessTrackedAssets;
    }
}
```

### Changes to Defaulter2AssetPostProcessor.cs

```csharp
// This file is now empty and will be deleted
```

### Updates to Defaulter2Window.cs

```csharp
private void OnGUI()
{
    // Existing code...
    
    // Add processing status and cancel button
    if (_settings.assetProcessor.IsProcessing())
    {
        EditorGUILayout.Space(10);
        EditorGUILayout.BeginHorizontal();
        string currentAsset = (_settings.assetProcessor._processedCount < _settings.assetProcessor._guidsToProcess.Count) ? AssetDatabase.GUIDToAssetPath(_settings.assetProcessor._guidsToProcess[_settings.assetProcessor._processedCount]) : "Finishing...";
        EditorGUILayout.LabelField($"Processing: {currentAsset} ({_settings.assetProcessor._processedCount}/{_settings.assetProcessor._guidsToProcess.Count})", EditorStyles.boldLabel);
        
        if (GUILayout.Button("Cancel", GUILayout.Width(80)))
        {
            _settings.assetProcessor.CancelProcessing();
        }
        
        EditorGUILayout.EndHorizontal();
    }
    
    // Existing code...
}
```

### Updates to Editor/Core/Defaulter2.cs

```csharp
// Add this method to Defaulter2.cs
    private static void OnPostprocessAllAssets(string[] importedAssets, string[] deletedAssets, string[] movedAssets, string[] movedFromAssetPaths)
    {
        Defaulter2.Api.assetProcessor.OnPostprocessAllAssets(importedAssets, deletedAssets, movedAssets, movedFromAssetPaths, Defaulter2.Api);
    }
```

### Changes to Editor/Core/Defaulter2.Processor.cs

```csharp
// Remove Defaulter2AssetPostProcessor class completely
```

### 5. Testing Strategy
Define how user can help to test this features
- Test with different batch sizes (few assets vs. many assets)
- Verify that the editor remains responsive during processing
- Check that progress bar correctly reflects processing status
- Test cancellation at different points during processing
- Verify that large asset imports don't freeze the editor
- Confirm that all assets are processed correctly when distributed across frames
- Test with different asset types (textures, audio, etc.)

### 6. Acceptance Criteria
List all the criteria to make this feature useful and bug free
- Unity Editor remains responsive during asset processing
- Progress bar accurately displays current processing state
- Processing can be cancelled by the user at any time
- All assets are correctly processed when using per-frame distribution
- Proper cleanup occurs if processing is interrupted
- System works with both automatic and manual processing modes
- Performance impact on editor responsiveness is minimal
- Asset processing completes in a reasonable timeframe

### 7. Timeline Estimate
- Phase 1 (Per-Frame Processing Logic): 3 days
- Phase 2 (Progress Bar and Cancellation): 2 days
- Final Testing and Polish: 2 days

### 8. ConclusionPlanning Notes
- Balance between processing speed and editor responsiveness is crucial
- Consider making assets-per-frame configurable via preferences
- Future enhancements could include background thread processing for even better performance
- Monitor memory usage during distributed processing to ensure it remains stable

### 9. Affected Classes, Structs, Enums, and Methods

#### New
- `DF2_AssetProcessor` (top-level class)
- `DF2_AssetProcessor.ProcessAssetsPerFrame()`
- `DF2_AssetProcessor.ProcessAssetsInUpdate()`
- `DF2_AssetProcessor.CleanupProcessing()`
- `DF2_AssetProcessor.CancelProcessing()`
- `DF2_AssetProcessor.IsProcessing()`
- `DF2_AssetProcessor._processedGuids`
- `DF2_AssetProcessor.OnPostprocessAllAssets`

#### Moved
- `_isProcessing` (from `Defaulter2` to `DF2_AssetProcessor`)
- `_processedCount` (from `Defaulter2` to `DF2_AssetProcessor`)
- `_guidsToProcess` (from `Defaulter2` to `DF2_AssetProcessor`)
- `_cancelRequested` (from `Defaulter2` to `DF2_AssetProcessor`)
- `assetsPerFrame` (from `Defaulter2` to `DF2_AssetProcessor`)
- Logic from `Defaulter2AssetPostProcessor` to `DF2_AssetProcessor`

#### Modified
- `Defaulter2.ProcessTrackedAssetsPerFrame()`
- `Defaulter2Window.OnGUI()`
- `Defaulter2` to instantiate `DF2_AssetProcessor`

#### Deleted
- `Defaulter2AssetPostProcessor`

### 10. TODO
- Verify if logic exists elsewhere to copy or move into the new `DF2_AssetProcessor` class.
- Ensure extension methods are accessible to `DF2_AssetProcessor`.