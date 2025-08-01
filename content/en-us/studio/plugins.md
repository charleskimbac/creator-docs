---
title: Plugins
description: Explains how to create, publish, and monetize extensions to Studio that add custom functionality.
---

A **plugin** is an extension that adds additional features or functionality to Studio. You can [install](../production/creator-store.md#find-assets) community-made plugins from the Creator Store, or you can [create](#create-new-plugins) and [publish](#upload-distribute-and-monetize-plugins) your own to the [Toolbox](../projects/assets/toolbox.md) to use across your experiences.

If you choose to also distribute your plugins to the Creator Store, you can either offer them for free or sell them for **United States Dollars** (the minimum price is $4.99). Roblox offers a market-leading revenue share for these sales, as only taxes and payment processing fees are deducted. For more information on selling plugins, see [Selling on the Creator Store](../production/sell-on-creator-store.md).

## Create new plugins

You can create your own plugins to improve your workflow in Studio. The following code sample is a plugin called **EmptyScriptAdder** that inserts an empty script as the child of an object or in `Class.ServerScriptService`. The following sections explain the major parts to creating this plugin.

To begin, you should enable **Plugin Debugging Enabled** in the **Studio** section of Studio's settings. This will expose the `Class.PluginDebugService` in Studio which provides real-time debugging for your plugin's code and makes it easier to reload and save your plugin.

```lua title="EmptyScriptAdder Plugin"
local ChangeHistoryService = game:GetService("ChangeHistoryService")
local Selection = game:GetService("Selection")

-- Create a new toolbar section titled "Custom Script Tools"
local toolbar = plugin:CreateToolbar("Custom Script Tools")

-- Add a toolbar button named "Create Empty Script"
local newScriptButton = toolbar:CreateButton("Create Empty Script", "Create an empty script", "rbxassetid://14978048121")

-- Make button clickable even if 3D viewport is hidden
newScriptButton.ClickableWhenViewportHidden = true

local function onNewScriptButtonClicked()
	local selectedObjects = Selection:Get()
	local parent = game:GetService("ServerScriptService")
	if #selectedObjects > 0 then
		parent = selectedObjects[1]
	end

	local newScript = Instance.new("Script")
	newScript.Source = ""
	newScript.Parent = parent
	ChangeHistoryService:SetWaypoint("Added new empty script")
end

newScriptButton.Click:Connect(onNewScriptButtonClicked)
```

### Save plugin scripts

Plugins start from scripts. To create a plugin, create a `Class.Script` and save it as a plugin using the [Explorer](../studio/explorer.md). For example, to create the **EmptyScriptAdder Plugin**:

1. Insert a new `Class.Script` inside `Class.ServerStorage` and rename it to **EmptyScriptAdder**.

   <img src="../assets/studio/plugins/Plugin-Empty-Script-Adder.png" width="320" />

2. Copy and paste the **EmptyScriptAdder Plugin** code into the new script.
3. In the **Explorer** window, right-click the script and select **Save as Local Plugin**.
4. In the popup window, click **Save** to insert the plugin script into your local **Plugins** folder of the Studio installation.
5. The plugin should appear in `Class.PluginDebugService` and start running.

<Alert severity="warning">
Make sure to delete the original script in `Class.ServerStorage` and work from the plugin inside `Class.PluginDebugService`, otherwise you may end up applying changes to the wrong script.
</Alert>

### Reload and save changes

With your `Class.Plugin` inside `Class.PluginDebugService`, you can easily update the plugin by right-clicking it and then selecting **Save and Reload Plugin** from the context menu. If you simply want to reload the plugin, for example to step through a section of code using a breakpoint without saving the plugin, you can
alternatively select **Reload Plugin**.

### Add toolbar buttons

To add a button for your plugin to the **Plugins** tab of the Studio toolbar, use the `Class.Plugin:CreateToolbar()` and `Class.PluginToolbar:CreateButton()` methods. In the code for **EmptyScriptAdder**, line 5 creates a new section in the toolbar named **Custom Script Tools** and line 8 creates a button named **Create Empty Script**.

<img src="../assets/studio/plugins/Plugin-Toolbar-Button.png" width="800" alt="New plugin button added to toolbar in Studio" />

### Execute code on click

To make the plugin execute code when a user clicks the toolbar button, connect a function to the button's `Class.PluginToolbarButton.Click` event. In the code for **EmptyScriptAdder**, the connecting function is `onNewScriptButtonClicked()`.

### Check user selection

To modify a plugin's behavior based on what the user has selected, use the `Class.Selection` service. The `onNewScriptButtonClicked()` function checks if the user has anything selected and creates the new script as its child instead of inside `Class.ServerScriptService`. If the user doesn't have anything selected, it creates the new script in `Class.ServerScriptService`.

<img src="../assets/studio/plugins/Plugin-Inserted-Script.png" width="320" />

### Support undo and redo

Use `Class.ChangeHistoryService` to allow users to undo and redo changes made by a plugin within an experience. In your script, set the plugin to call `Class.ChangeHistoryService:TryBeginRecording()` and store the identifier assigned to the API call before making changes. Then set the plugin to call `Class.ChangeHistoryService:FinishRecording()` after making changes, so it captures any changes made during the recording session for undo and redo.

The following code sample creates an example plugin that can apply the neon material to selected parts. It uses `Class.ChangeHistoryService` to record and manage the changes made by the plugin:

```lua title="Example Material Plugin with Recordings for Undo and Redo"
local ChangeHistoryService = game:GetService("ChangeHistoryService")
local Selection = game:GetService("Selection")

-- Create an example plugin
local toolbar = plugin:CreateToolbar("Example Plugin")
local button = toolbar:CreateButton("Neon it up", "", "")

-- Connect a function to the click event
button.Click:Connect(function()
    local parts = {}
    for _, part in Selection:Get() do
        if part:IsA("BasePart") then
            parts[#parts + 1] = part
        end
    end
    if #parts < 1 then
        -- Nothing to do.
        return
    end

    -- Try to begin a recording with a specific identifier
    local recording = ChangeHistoryService:TryBeginRecording("Set selection to neon")

    -- Check if recording was successfully initiated
    if not recording then
        -- Handle error here. This indicates that your plugin began a previous
        -- recording and never completed it. You may only have one recording
        -- per plugin active at a time.
        return
    end

    -- Iterate through the selected parts
    for _, part in parts do
        part.Material = Enum.Material.Neon -- Set the material of the part to Neon
    end

    -- Finish the recording, committing the changes to the history
    ChangeHistoryService:FinishRecording(recording, Enum.FinishRecordingOperation.Commit)
end)
```

## Upload, distribute and monetize plugins

As with [models](../parts/models.md), [meshes](../parts/meshes.md), [images](../parts/textures-decals.md), and [animations](../animation/editor.md#creating-an-animation), you can distribute plugins to Roblox to make them easy to reuse from the [Toolbox](../projects/assets/toolbox.md). You can choose to make them publicly available to all other creators on the [Creator Store](../production/creator-store.md), or to distribute them privately for your own use. If you choose to distribute your plugin publicly, you can set a price at which to sell it to other creators.

<Alert severity="info">
The only way to distribute and set a price for a plugin is through the Creator Dashboard. You can always upload a plugin using this process and monetize it later from the dashboard. For instructions on this process, see [Distributing Assets](../production/creator-store.md#through-creator-dashboard).
</Alert>

To distribute a plugin:

1. In the [Explorer](../studio/explorer.md) window, right-click a plugin script, then select **Publish as Plugin** from the contextual menu.
1. **(Optional)** In the upper-left corner of the asset configuration window, click the image to upload a 512&times;512 image.
1. Fill in the following fields:

   - **Name**: A title for your plugin.
   - **Description**: A description that describes what a potential user should expect the plugin to do.
   - **Creator**: The creator you'd like to attribute as the creator of the plugin.

1. Click the **Submit** button. Your plugin is now available to you in the **Inventory** and **Creations** tabs of the [Toolbox](../projects/assets/toolbox.md).
1. Click the link to Creator Dashboard to [configure distribution](../production/creator-store.md#through-creator-dashboard).

<Alert severity="info">
To publicly distribute or monetize a plugin you have previously uploaded, use the [Creations section of the Creator Dashboard](https://create.roblox.com/dashboard/creations?activeTab=Model). For more information, see [Distributing Assets](../production/creator-store.md).
</Alert>
