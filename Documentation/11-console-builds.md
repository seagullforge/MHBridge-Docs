# Build for consoles

[Guide contents](../README.md#start-here) · [Troubleshooting](08-troubleshooting.md)

MHBridge evaluates face and body rigs through the native `MHBridgeRig` adapter. To run
that adapter on Xbox or PlayStation, you need to build it with your console SDK and
integrate the result with Unity's platform-specific native plugin system.

**We have not built or tested the adapter with Xbox or PlayStation SDKs because we do
not have access to them.** The steps on this page describe the expected integration
process, not a verified console build recipe. The adapter or OpenRigLogic may need
changes for your platform. Desktop test results do not establish console compatibility.

If you run into a build or integration problem, contact us. We are always happy to help
investigate and work through the changes with you. See [Get help](#get-help) for what
to include in a report.

## On this page

- [Before you start](#before-you-start)
- [Step 1 — Get the adapter source kit](#step-1--get-the-adapter-source-kit)
- [Step 2 — Get the matching OpenRigLogic source](#step-2--get-the-matching-openriglogic-source)
- [Step 3 — Configure and build for your SDK](#step-3--configure-and-build-for-your-sdk)
- [Step 4 — Integrate the native library with Unity](#step-4--integrate-the-native-library-with-unity)
- [Step 5 — Test on the target device](#step-5--test-on-the-target-device)
- [Get help](#get-help)

## Before you start

You need your console SDK, a compatible Unity console module, CMake 3.15 or newer,
Git, and a C++17 compiler. Use the MHBridge package and source kit from the same release.
The included CMake presets target desktop platforms; supply your SDK's toolchain for
console builds.

## Step 1 — Get the adapter source kit

Open `Assets/MHBridge/Plugins/MHBridgeRig-console-source.zip` and extract it outside
`Assets`. The package version and kit
revision are recorded inside the archive in `OpenRigLogic.lock.json`, together with
the required OpenRigLogic commit.

The kit contains the following files:

| File or folder | Purpose |
|---|---|
| `OpenRigAdapter/` | Adapter C++ source, public C header, and CMake build files. |
| `OpenRigLogic.lock.json` | Exact OpenRigLogic commit and hashes of the matching managed bindings. |
| `cmake/ConsoleDefaults.cmake` | Initial build settings with CPU-specific optimizations disabled. This is not a console toolchain. |
| `LICENSE.md` and `Third Party Notices.md` | Adapter license and dependency notices. |

The ZIP does not contain OpenRigLogic, console SDKs, compiled libraries, C# bindings,
or DNA. Use the managed bindings from the matching Unity package.

## Step 2 — Get the matching OpenRigLogic source

Read the repository URL and commit in `OpenRigLogic.lock.json`. Clone that repository,
then check out the exact commit. Do not substitute the latest branch: an upstream change
can alter the adapter's build or runtime behavior.

From the extracted kit root, run this template after replacing `YOUR_LOCKED_COMMIT`
with the commit recorded in your lock file:

```sh
git clone --no-checkout https://github.com/EpicGames/OpenRigLogic.git OpenRigLogic
git -C OpenRigLogic checkout --detach YOUR_LOCKED_COMMIT
git -C OpenRigLogic rev-parse HEAD
git -C OpenRigLogic status --porcelain
```

Confirm that the reported commit matches the lock file and that the status output is
empty. Keep the entire checkout, including its build files and headers.

## Step 3 — Configure and build for your SDK

Use the toolchain and generator required by your licensed SDK and Unity platform
documentation. The kit supplies common adapter settings; you supply the actual console
compiler configuration. Keep a separate build directory for each target and SDK version.

From the kit root, configure and build with your SDK settings substituted for `YOUR_*`:

```sh
cmake -S OpenRigAdapter -B build/YOUR_TARGET -C cmake/ConsoleDefaults.cmake -G "YOUR_SDK_GENERATOR" -DCMAKE_TOOLCHAIN_FILE="YOUR_ABSOLUTE_TOOLCHAIN_PATH" -DOPENRIGLOGIC_DIR="YOUR_ABSOLUTE_OPENRIGLOGIC_PATH" -DMHR_LIBRARY_TYPE=STATIC -DCMAKE_BUILD_TYPE=Release
cmake --build build/YOUR_TARGET --config Release --target MHBridgeRig --parallel
```

If your SDK uses generator-specific arguments or a native project format, adapt the
configuration to that supported workflow.

Start with the scalar settings in `ConsoleDefaults.cmake`. Keep host CPU autodetection,
upstream tests, examples, benchmarks, and Python wrappers disabled. Enable CPU-specific
optimizations only after the target build passes numerical tests.

Use the same SDK runtime settings for the adapter and OpenRigLogic. The adapter uses
C++ exceptions; a toolchain that disables them requires source changes.

## Step 4 — Integrate the native library with Unity

Choose static or shared linking according to the target Unity module's documentation.
The static setting in the template is a starting point, not a claim that every console
uses the same plugin format.

### Link the native dependencies

With `MHR_LIBRARY_TYPE=STATIC`, the build produces an adapter archive. **That archive
does not contain OpenRigLogic.** Copying only `MHBridgeRig.lib` or `libMHBridgeRig.a`
into Unity is insufficient.

If a parent CMake project performs the final native link, link its target against the
`MHBridgeRig` CMake target so that CMake carries the dependency requirements. If Unity
performs the final link separately, supply the adapter, required OpenRigLogic dependency
archives, and SDK system dependencies through the platform's supported build integration.
Use the generated CMake link requirements to identify those dependencies.

Native consumers of the static archive must define `MHR_STATIC` when including
`MHBridgeRig.h`. Preserve the required `MHR_*` entry points through release linking
and symbol stripping.

Use `MHR_LIBRARY_TYPE=SHARED` only if the target Unity module supports that module type.
In this mode, the adapter links OpenRigLogic into the module. Verify exported functions,
runtime dependencies, and any SDK-specific module requirements. The header's desktop
export macros may need platform-specific changes.

### Configure plugins and managed bindings

In Unity, enable each console artifact only for its intended target. Keep the desktop
plugin enabled for the Editor on its host platform, and exclude console artifacts from
Editor loading.

All native imports use the `DLL` constant in
`Assets/MHBridge/Runtime/Native/MHBridgeNative.cs`. It currently equals `"MHBridgeRig"`
for every platform; there is no console-specific branch.

- **Shared plugin loaded as `MHBridgeRig`:** leave the bindings unchanged. If the
  platform requires a different import name, change `DLL` for that Player target.
- **Static library linked into the Player:** use the import convention required by
  the Unity console module. If it specifies `DllImport("__Internal")`, set `DLL` to
  `"__Internal"` in a conditional branch for that console Player (`!UNITY_EDITOR`
  and its platform define). Keep `"MHBridgeRig"` in the other branch. This changes
  every import at once; individual declarations and `MHRig.cs` need no edits.

`__Internal` selects functions linked into the Player; it does not add the native
archives to the link. Those still need the integration described in
[Link the native dependencies](#link-the-native-dependencies). See Unity's
[DllImport reference](https://docs.unity.com/en-us/engine/6000.0/manual/scripting/compilation-and-code-reload/plug-ins/native/dllimport)
for the general distinction. The console module determines which convention applies.

## Step 5 — Test on the target device

Run a Player on the device with your final scripting backend, including IL2CPP where applicable, and release linking and stripping
settings. Start with a rig test that does not depend on character rendering:

1. Load the native library and check its reported version against the kit's lock file.
2. Load both head and body DNA from memory, then create the rig evaluator and instance.
3. Check metadata and output counts at every level of detail (LOD).
4. Evaluate neutral and driven poses. Check every output array for NaN or infinity.
5. Repeat evaluation, reset controls, and repeat the load and disposal cycle.
6. Compare results with a desktop baseline using an explicit floating-point tolerance.

When generating body test inputs, controls ending in `.qx`, `.qy`, `.qz`, and `.qw`
form quaternion groups. Their neutral value is `(0, 0, 0, 1)`, and driven values must
form unit quaternions. Setting every control to zero can produce invalid output even
when the library works correctly.

## Get help

If a step fails, open an [issue in the MHBridge documentation repository](https://github.com/seagullforge/MHBridge-Docs/issues).
We are always happy to help investigate build errors, linking problems, and runtime
failures. We cannot reproduce SDK-specific problems locally without platform access,
so your diagnostics and target-device checks help us work through them with you.

Include the information that you can share publicly:

- MHBridge version, source kit revision, and OpenRigLogic commit.
- Target console family and the stage that fails: configuration, compilation, linking,
  library loading, or rig evaluation.
- Unity version and scripting backend.
- The first relevant error and a minimal reproduction, with confidential details removed.
- Any changes you made to the adapter, dependency, or build settings.

Do not attach restricted SDK files, confidential platform documentation, credentials,
or NDA-covered logs to a public issue. If diagnosis needs confidential details, ask us
to arrange an appropriate support channel before sharing them.
