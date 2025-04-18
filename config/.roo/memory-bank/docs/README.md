# Defaulter2

A professional Unity editor extension that automatically configures import settings for assets according to predefined presets and a powerful rule-based filtering system.

## Overview

Defaulter2 streamlines the Unity asset pipeline by automatically applying consistent import settings based on configurable rules and presets. It saves development time by eliminating manual configuration for textures, sprites, audio, models, and more. The core configuration is managed through a central `Defaulter2` ScriptableObject.

## Key Features

- **Rule-Based Filtering**: Apply presets using a flexible system based on folder paths, name patterns (prefix, suffix, wildcard, regex), and asset dimensions. Supports include/exclude logic.
- **Custom Presets**: Define and manage import settings for various asset types (Texture, Sprite, Audio, Model, SpineAtlas).
- **Automatic Processing**: Applies settings automatically when assets are imported or modified (non-blocking).
- **Manual Control**: Option to manually validate settings and enforce presets on selected assets or folders.
- **Editor Integration**: Seamlessly integrates into the Unity Editor via a dedicated Defaulter2 window.
- **Project Panel Integration**: Shows which preset is affecting a specific asset directly in the Project panel.

## Installation

1.  Import the Defaulter2 package into your Unity project.
2.  Access Defaulter2 through the Unity menu: `Tools > Defaulter2`.
3.  Create a Defaulter2 configuration asset through the Defaulter2 window.
4.  Configure your presets and rules via the editor window.

## Usage

1.  Define import presets in the Defaulter2 window.
2.  Configure the `Inclusive Scope` and `Exclusive Scope` for each preset using rules (folder, name pattern, dimensions).
3.  Assets imported or modified will automatically have matching presets applied (non-blocking).
4.  Use the "Validate" and "Enforce" buttons for manual checks and application.
5.  Enable the "Show Preset Names" toggle in Defaulter2 to see which preset is affecting a specific asset in the Project panel.

## Support

For technical support or feature requests, please contact:
- Email: [support@yourcompany.com] (Update if necessary)

## License

This is a commercial product. Please refer to the license agreement included with the package.