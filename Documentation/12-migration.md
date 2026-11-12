# Migrate from an earlier version

[Guide contents](../README.md#start-here) · [Troubleshooting](08-troubleshooting.md)

Each section lists the steps for one version. If you skip versions, follow every section between
your version and the new one, oldest first.

## On this page

- [Migrate to 1.1.0](#migrate-to-110)

## Migrate to 1.1.0

Version 1.1.0 changes the following:

- **Pipeline suffixes.** The shipped HDRP assets carry an `_HDRP` suffix, for example
  `MH_Eye.mat` is now `MH_Eye_HDRP.mat`, and `MHEye_Graph.shadergraph` is now
  `MHEye_HDRP.shadergraph`. MHBridge renames them during the update and keeps their GUIDs.
  Characters, scenes, and materials that reference them keep working.
- **Skin attachment.** Skin attachment is part of the hair package. MHBridge no longer uses
  `com.unity.demoteam.digital-human`, and skin attachment works under both HDRP and URP.

If none of your grooms uses skin attachment, complete
[Step 1](#step-1--update-the-mhbridge-package),
[Step 2](#step-2--update-the-hair-package), and optionally
[Step 5](#step-5--remove-the-digital-human-package).

Before you start, commit your project to version control or back it up.

> [!WARNING]
> The update overwrites the shipped templates and shaders under `Assets/MHBridge`. Changes that
> you made in them directly are lost. Before you update, move your changes to copies outside
> `Assets/MHBridge`, and assign the copies in **Import Settings**.

### Step 1 — Update the MHBridge package

1. Import the new MHBridge package, and wait for Unity to finish importing and compiling.
2. Wait for the second, automatic import. MHBridge renames the earlier assets and unpacks the
   material set of the active render pipeline. The Console line names how many assets moved.

Do not open the sub-packages in `Assets/MHBridge/Setup` yourself. Unity then adds second copies
instead of updating the assets. If that happened, delete the copies that the update added, then
open **Tools > MHBridge > Validate Setup**, and click **Unpack**.

### Step 2 — Update the hair package

If `com.unity.demoteam.hair` is not in your project and you don't need hair, the migration is
complete.

If the package is in your project, update it:

1. Select **Window > Package Manager**, and then select **In Project**.
2. Select **Demo Team Hair System**, and click **Update**.
3. Wait for the recompile to finish.

### Step 3 — Import the grooms again

Import the skin-attached grooms of each character again, with the same `.abc` files and
profiles. See [Hair](06-hair.md).

### Step 4 — Verify the result

1. Enter Play mode, and play a facial animation on the character.
2. Check that the eyebrows, beard, and mustache move with the expression.

### Step 5 — Remove the digital-human package

Optional: MHBridge no longer needs `com.unity.demoteam.digital-human`. If no other part of your
project uses it, remove it:

1. In Package Manager, select **In Project**.
2. Select **Demo Team Digital Human**, and click **Remove**.
