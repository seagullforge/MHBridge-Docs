# Materials

[Guide contents](../README.md#start-here) · [Troubleshooting](08-troubleshooting.md)

The DCC/DNA importer copies the material templates assigned in **Tools > MHBridge > Import Settings**.
The shipped settings use the following materials for each part and LOD:

| Role | Material template | Shader graph | Visible effect |
|---|---|---|---|
| Body | `MH_Body.mat` | `MHBody_Graph` | Body skin color, surface detail, and surface shading. |
| Face skin, LOD0 | `Head/MHSkin_Animated_StuckLit.mat` | `MHSkin_Animated_StuckLit_Graph` | Skin shading with animated wrinkle maps. |
| Face skin, LOD1 | `Head/MHSkin_AnimatedSS.mat` | `MHSkin_AnimatedSS_Graph` | Skin scattering with animated wrinkle maps. |
| Face skin, LOD2–3 | `Head/MHSkin_SS.mat` | `MHSkin_SS_Graph` | Skin scattering without animated wrinkle maps. |
| Face skin, LOD4 and deeper | `Head/MHSkin_NoSS.mat` | `MHSkin_NoSS_Graph` | Skin shading without subsurface scattering or animated wrinkle maps. |
| Teeth | `MH_Teeth.mat` | HDRP/Lit | Tooth color, surface detail, and scattering. |
| Eye | `MH_Eye.mat` | `MHEye_Graph` | Sclera, iris, and pupil. |
| EyeShell | `MH_EyeShell.mat` | `MHEyeShell_Graph` | Darkening under the eyelids. |
| EyeEdge | `MH_EyeEdge.mat` | `MHEyeEdge_Graph` | Wet edge where the eye meets the lid. |
| Eyelashes | `MH_Eyelashes.mat` | HDRP/Lit | Eyelash color and alpha-clipped shape. |
| Saliva | `MH_Saliva.mat` | HDRP/Lit | Transparent saliva surface. |
| Cartilage | `MH_Cartilage.mat` | HDRP/Lit | Inner-eye cartilage, using the head color and normal maps. |

Templates are in `Assets/MHBridge/Materials`; MHBridge graphs are in `Assets/MHBridge/Shaders`.
HDRP/Lit is an HDRP shader rather than an MHBridge graph.
Each material array uses its index as the LOD and repeats its last entry for deeper LODs.
Parts other than face skin use one template across their LODs in the shipped settings.
The DCC importer assigns available character textures to the copied materials.
A bare DNA file carries no textures, so its materials retain the templates' textures.

## On this page

- [Tune an extracted material](#tune-an-extracted-material)
- [Body](#body)
- [Face skin](#face-skin)
  - [Wrinkle maps](#wrinkle-maps)
- [Eyes](#eyes)
  - [Sclera](#sclera)
  - [Iris](#iris)
  - [Pupil](#pupil)
  - [Limbal ring](#limbal-ring)
  - [Caustics](#caustics)
  - [EyeShell and EyeEdge](#eyeshell-and-eyeedge)
- [Hair materials](#hair-materials)
  - [Eyelashes](#eyelashes)
  - [Cards and grooms](#cards-and-grooms)
- [Diffusion profiles](#diffusion-profiles)
- [Visual problems](#visual-problems)

## Tune an extracted material

To tune a DCC/DNA material, first click **Extract Textures & Materials** on the character's **Material** tab.
For hair or cards, tune the materials created by the corresponding import tool.

Tune materials under consistent lighting, then check animation and LOD changes.
For texture extraction and iris-boundary rebuilding, use the [import reference](04-import-reference.md#extract-preview-versus-production).

## Body

`MH_Body.mat` uses the character's body base color, normal, and mask maps.
Start with these maps and the material's surface settings, then check the body beside the face
under the same lighting.

## Face skin

The head materials share the face's base color, normal, and mask maps.
The shipped LOD0 and LOD1 materials also receive animated wrinkle maps; LOD2–3 keep
subsurface scattering, and LOD4 and deeper use the material without scattering.
Check your changes at each LOD because tuning one extracted material does not tune every variant.

### Wrinkle maps

OpenRig calculates the animated wrinkle weights.
With **Apply Wrinkle Maps** enabled on `MHFace`, MHBridge passes these weights to the skin
shader, which blends the wrinkle maps.
If you create a custom material or shader, provide compatible wrinkle inputs and blending,
using the shipped `MHSkin_AnimatedSS_Graph` shader as a reference.

## Eyes

Start from the shipped material, then tune for your character and lighting. The values listed
here are the shipped `MH_Eye.mat` defaults.

![Eye material groups and controlled-light comparison](images/05-materials/01-eye-materials.png)

The importer assigns these four eye texture properties:

| Inspector slot | Shader property | Extracted map |
|---|---|---|
| **Iris Texture** | `_IrisTexture` | `Eyes_Iris_Color` |
| **Iris Normal** | `_IrisNormal` | `Eyes_Iris_Normal` |
| **Sclera Texture** | `_Sclera_Texture` | `Eyes_Sclera_Color` |
| **Sclera Normal** | `_ScleraNormal` | `Eyes_Sclera_Normal` |

### Sclera

| Parameter | Shipped value | Effect and use |
|---|---|---|
| **Sclera Texture** | Assigned at import | Color of the white of the eye. Edit the extracted map to change its coloration. |
| **Sclera Smoothness** | `0.81` | Controls the surface highlight. Increase for a smoother, wetter appearance. |
| **Sclera Normal** | Assigned at import | Surface detail, including blood vessels. Keep its texture type on **Normal map**. |
| **Sclera Normal Strength** | `1` | Strength of surface detail. Reduce if vessels look too raised. |
| **Sclera Diffusion Profile** | `MH_ScleraDiffusionProfile` | Subsurface scattering of the sclera. Assign and register this profile before tuning lighting. |

### Iris

| Parameter | Shipped value | Effect and use |
|---|---|---|
| **Cornea Smoothness** | `1` | Sharpness of the corneal highlight. Lower to broaden it. |
| **Iris Texture** | Assigned at import | Iris color and pattern. Edit this texture to change eye color. |
| **Iris Clamp Color** | Gray | Keep the shipped value while tuning iris color with **Iris Texture**. |
| **Iris Normal** | Assigned at import | Iris surface structure used in diffuse shading. Keep its texture type on **Normal map**. |
| **Iris Normal Strength** | `7.46` | Strength of the iris pattern's shading. Reduce if the relief looks excessive. |
| **Iris Diffusion Profile** | `MH_IrisDiffusionProfile` | Iris scattering and caustic softness. Assign and register the iris profile. |
| **Inner Radius** | `0.195` | Radius used to lay out the iris texture. Adjust texture coverage separately from the split boundary. |
| **Iris Radius** | `0.1385` | Visible iris aperture. Adjust the iris size on the eyeball. |
| **Iris Depth** | `0.85` | Iris depth used by refraction, clamped by the shader. Adjust while checking oblique views. |

### Pupil

| Parameter | Shipped value | Effect and use |
|---|---|---|
| **Pupil Radius** | `0.121` | Controls the radial compression of the iris texture around the pupil. Check it together with the aperture limits. |
| **Pupil Aperture** | `0.313` | Authored neutral pupil size. Animation drives aperture relative to this value. |
| **Minimal Pupil Aperture** | `0.045` | Narrow endpoint. Preview with **Pupil Aperture** at `0`, then restore the intended neutral. |
| **Maximal Pupil Aperture** | `0.693` | Wide endpoint. Preview with **Pupil Aperture** at `1`, then restore the intended neutral. |

With `MHFace.applyPupilDilation` enabled, RigLogic's animation outputs drive the pupil.
`MHFace` reads the authored **Pupil Aperture** independently for each eye; it uses `0.5`
only if no eye material provides the value. It applies the animated aperture to every LOD of that eye. Full wide and narrow outputs
reach aperture `1` and `0`. At LODs where those rig channels return zero, the authored neutral
is used. The material's minimum and maximum still define the visible endpoint sizes.


### Limbal ring

| Parameter | Shipped value | Effect and use |
|---|---|---|
| **Limbal Ring Size Iris** | `0.0775` | Width of the ring inside the iris boundary. Adjust the inner rim. |
| **Limbal Ring Size Sclera** | `0.01715` | Width outside the boundary. Adjust the outer rim. |
| **Limbal Ring Fade** | `1` | Falloff into the sclera. Adjust to soften the transition. |
| **Limbal Ring Intensity** | `5` | Ring darkness. Reduce if the iris has a heavy outline. |

### Caustics

| Parameter | Shipped value | Effect and use |
|---|---|---|
| **Caustics Offset** | `0` | Depth placement of the caustic ring. Check while changing the light direction. |
| **Caustics Intensity** | `1` | Caustic lighting multiplier. Reduce an overly bright ring. |
| **Caustics Blend** | `1` | Blend of the iris caustic effect. Adjust its contribution under fixed lighting. |
| **Caustics Radius** | `0.0066` | Radius of the caustic ring. Adjust its spread across the iris. |

### EyeShell and EyeEdge

| Material and parameter | Effect and use |
|---|---|
| EyeShell **Top Occlusion Power** | Shapes upper-lid occlusion. Increasing the power reduces the affected area. |
| EyeShell **Bottom Occlusion Power** | Shapes lower-lid occlusion. Tune separately from the upper lid. |
| EyeShell **Occlusion Intensity** | Overall strength of the occlusion layer. Reduce excessive darkening. |
| EyeShell **Depth Occlusion Distance** | Controls how depth differences affect opacity. Adjust the transition near the lids. |
| EyeShell **Tint** | Color of the occlusion layer. Check it in both bright and dark scenes. |
| EyeEdge **Depth Distance** | Controls how depth differences affect the wet edge's transparency. Adjust its visible extent. |
| EyeEdge **Depth Offset** | Corrects the depth comparison. Tune when the wet edge sits incorrectly against the lid. |

EyeShell uses alpha blending, so its tint is not a physically exact multiply of the surface
beneath it. A colored tint can appear too bright in a dark scene; use black for neutral
occlusion and inspect your result under low light.

![EyeShell and EyeEdge controls](images/05-materials/02-eye-shell-edge.png)


## Hair materials

### Eyelashes

`MH_Eyelashes.mat` uses HDRP/Lit with alpha clipping for the imported eyelash mesh.
The importer assigns `Eyelashes_Color` to its base color map; the texture's alpha supplies
the clip mask. Tune this extracted material separately from cards and strand grooms.

### Cards and grooms

A strand groom is not an ordinary mesh — its vertices are produced by the hair package's GPU solver
— so an HDRP/Lit material renders nothing useful on it. Any groom material has to carry the
package's `HairVertex` subgraph node. MHBridge ships profile-specific materials in
`Assets/MHBridge/Hair/Materials`:

| Material | Shader | Used by |
|---|---|---|
| `Hair_Groom.mat`, `Eyebrows_Groom.mat`, `Beard_Groom.mat`, `Mustache_Groom.mat` | `Shaders/MHGroom.shadergraph` | The strand grooms, through the profile's **Groom Material** field. |
| `Hair_Cards.mat`, `Eyebrows_Cards.mat` | `Shaders/MHCards.shadergraph` | The hair cards, through the profile's **Card Material** field. |

Hair and Eyebrows have dedicated cards materials. Beard and Mustache use `Hair_Cards.mat`
as their fallback. Check the profile's **Card Material** field for the assignment.
The cards materials are there from the first import. The groom material and its shader arrive with
the sub-package, for the reason in [Unpack the groom assets](06-hair.md#unpack-the-groom-assets).

They expose the same parameters on purpose, so a groom and its cards can be given one look and the
swap between them at distance does not show. The cards shader adds one option of its own,
**Recalculate Card Normals**, which replaces the FBX's own normals with normals derived from the
card's shape — useful when the baked mesh lights flat.

![Hair_Groom material controls](images/06-hair/23-groom-material-renderer.png)


The ones worth reaching for first:

| Parameter | What it does |
|---|---|
| **Root Color**, **Length Color**, **Tip Color** | The three stops of the gradient along a strand. These are the three the Hair Tool and the Cards Tool write from an entry's colors, so setting them here is the same edit you can make before the import. |
| **Root Distance**, **Root Fade**, **Tip Distance**, **Tip Fade** | Where each stop sits along the strand and how far it bleeds into the next. |
| **Opacity Threshold** | The alpha cutoff, mostly applied for Cards. Lower it and more of the strand's soft edge survives, which reads as thicker, denser hair; raise it and the groom thins out. The single most useful knob for density, and the first one to try when a groom looks sparse or when it looks like a solid helmet. |
| **Opacity Multiplier**, **Opacity Fading** | Global opacity, and how much a strand fades towards its tip. |
| **Going Grey** | The share of strands that turn grey. `0` on the shipped materials. |
| **Per Strand Hue Variation**, **Per Strand Smoothness Variation** | Small per-strand differences, seeded from the strand index. They are what keep a groom from reading as one solid mass. |
| **Specular**, **Specular Shift**, **Min Smoothness**, **Max Smoothness** | The primary highlight: how strong it is, where along the strand it sits, and how tight it is. |
| **Secondary Specular Multiplier**, **Secondary Specular Shift**, **Secondary Smoothness Offset** | The second, offset highlight that hair has and other surfaces do not. |
| **Rim Transmittance Color**, **Rim Transmittance Reach** | The light that passes through the hair from behind. This is where a groom stops looking like plastic. |
| **Masks**, **Alpha Depth**, **Tiling** | The two tiling textures the shipped materials use for per-strand breakup and soft alpha, and the tiling applied to them. |
| **Ambient Occlusion**, **Baked Ambient Occlusion**, **Depth Offset** | Contact shading and the depth bias that keeps strands from z-fighting the scalp. |
| **HighQualityLineMultiplier** | Scales the strand width used by [high quality lines](06-hair.md#high-quality-lines-unity-6). It does nothing in the other renderers, and nothing below Unity 6. |

![The same groom before and after tuning: flat default color, then a root-to-tip gradient with grey variation](images/05-materials/03-material-compare.png)

## Diffusion profiles

A missing profile asset or an empty material profile field needs a manual assignment.
Read the material and field named by the issue, then assign the matching profile:

| Material field | Profile asset in `Assets/MHBridge/Shaders` |
|---|---|
| Skin **Diffusion Profile** | `MH_Skin.asset` |
| Teeth **Diffusion Profile** | `MH_Teeth.asset` |
| Eye **Iris Diffusion Profile** | `MH_IrisDiffusionProfile.asset` |
| Eye **Sclera Diffusion Profile** | `MH_ScleraDiffusionProfile.asset` |

Assigning a profile to a material and registering it with HDRP are separate steps.
Open **Tools > MHBridge > Validate Setup**. After assignment, click **Re-check**, then **Register** for any unregistered profile.

HDRP applies only the diffusion profiles listed in the default volume profile. An
unregistered profile is silently replaced by the neutral one, so subsurface scattering stops
working with no error anywhere. The **Register** button is the shortest way out. To do it by
hand, open **Project Settings > Graphics > HDRP > Volume > Default Profile** and add the
profile to the **Diffusion Profile List** override. On Unity 2022.3 the same list lives in
**Project Settings > Graphics > HDRP Global Settings > Default Volume**.

![Material-specific issues and registered diffusion profiles](images/05-materials/04-diffusion-profiles.png)


Registering the profile on a Volume in a scene works too. The check cannot see that, which is
why it is a warning you can dismiss rather than an error.

## Visual problems

| Symptom | Cause | Fix |
|---|---|---|
| The iris boundary is wrong | The automatic split needs adjustment | Use **Material > Override Iris To Sclera Point** before stripping source textures. **Apply** and **Use Automatic** rebuild the four maps and update extracted eye materials. |
| The iris boundary override is disabled | One or both original combined eye maps are absent from the `.dcc` | Restore the original export with both `Eyes_Color` and `Eyes_Normal`. Existing split PNGs alone cannot supply this preview. |
| Extracted eye normals look inverted | The green channel was flipped again after extraction | MHBridge already converts the maps to OpenGL convention. Undo the extra flip and keep **Texture Type** on **Normal map** and wrapping on **Clamp**. |
| The pupil does not change size | The eye material has no `_PupilAperture` property, so the write is a silent no-op | Use `MH_Eye.mat` with `MHEye_Graph`, enable `applyPupilDilation`, and supply animation with pupil rig outputs. The shader controls pupil size relative to the material's authored neutral. |

For skin scattering and missing profiles, use [Diffusion profiles](#diffusion-profiles).
