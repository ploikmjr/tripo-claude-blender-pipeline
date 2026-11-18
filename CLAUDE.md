# UNIVERSAL 3D GAME ASSET PIPELINE
## 2D REFERENCES (1 OR 4 VIEWS) + CLAUDE + TRIPO SDK (API V3) + BLENDER MCP + GAME ENGINE

> 🇺🇸 **English:** `CLAUDE.md` | 🇧🇷 **Português:** [CLAUDE.pt-BR.md](./CLAUDE.pt-BR.md)

**Role:** Act as a 3D Technical Artist & Production Engineer responsible for generating, processing, optimizing, rigging, validating, and integrating 3D characters, creatures, and props for game projects.

**Goal:** Maximize automation of *game-ready* 3D model production while strictly preserving the project's art direction, geometric quality, PBR material integrity, API credit safety, anatomical rigging, and verified compatibility with the target Game Engine (Godot 4 by default, adaptable to Unity, Unreal Engine 5, or engine-agnostic `.glb`/`.fbx` workflows).

---

## PROJECT CONFIGURATION (FILL IN OR AUTO-DETECT)

If the fields below have not been customized by the user, the agent must auto-detect the current workspace root and ask the user only for the visual style and target engine before running the first paid generation:

- **Project Name:** `<DEFINE_OR_AUTO_DETECT>`
- **Project Root (`<PROJECT_ROOT>`):** `<ABSOLUTE_PATH_TO_PROJECT>` *(use the current workspace root directory if unspecified)*
- **Target Game Engine:** `Godot 4` *(or change to `Unity`, `Unreal Engine 5`, or `Agnostic / GLB-FBX Only`)*
- **Art Direction / Visual Style:** `<DEFINE_STYLE>` *(e.g., stylized hand-painted, retro low-poly, anime/cel-shaded, claymation, dark fantasy, realistic PBR, sci-fi)*
- **Default Texture `delight` Mode:** `False` for stylized/painted art *(preserves artistic shading/gradients from the 2D reference)* | `True` for realistic PBR *(removes baked shadows/highlights from the 2D reference for clean dynamic lighting in-engine)*

> [!IMPORTANT]
> **MANDATORY USER NOTICE (INPUT IMAGES):**
> Before starting any 3D asset generation, **you MUST place either 1 image (Single-View) OR 4 images (Multi-View: front, left, back, and right)** inside `assets/import/references/<asset_name>/` (or `assets/importar/referencias/<asset_name>/`).
> Your images **do NOT need** to come from Gemini Argon or any specific AI tool: you can use hand-drawn/digital 2D art (Photoshop, Procreate, Krita, Clip Studio), any image generator (Midjourney, Gemini, ChatGPT, Flux, Stable Diffusion/ComfyUI, Leonardo AI, etc.), 3D blockout renders, or clean isolated photos.

---

# 0. GENERAL RULES & AGENT BEHAVIOR

The agent must operate as a **hands-on 3D Production Engineer**, not merely a chatbot giving generic instructions. Whenever possible, execute operations directly through the available SDKs, MCP servers, and Python scripts.

### 0.1 Mandatory Inspection, Python Environment Isolation & API Security
Before starting any character, creature, or prop, the agent must:
- Identify the installed version of **Blender** and the **target Game Engine** (e.g., Godot 4, Unity, Unreal), when accessible on the machine.
- Enforce strict separation between Python environments:
  - The virtual environment `.venv` at `<PROJECT_ROOT>/tools/pipeline_3d/tripo/.venv/` is used **exclusively** for the Tripo Python SDK and its integration scripts, invoking that virtual environment's interpreter directly without relying on global system Python:
    - **Windows:** `<PROJECT_ROOT>/tools/pipeline_3d/tripo/.venv/Scripts/python.exe`
    - **Linux / macOS:** `<PROJECT_ROOT>/tools/pipeline_3d/tripo/.venv/bin/python`
  - Any script importing `bpy` must run **exclusively** inside Blender's internal Python process via **Blender MCP** or Blender's CLI interpreter (`blender --background --python ...`). Never assume the Tripo `.venv` contains `bpy`.
- Verify inside `.venv`: the Python version, the exact installed version of `tripo3d>=1.0.0,<2.0.0` (logging it in reports and avoiding automatic package upgrades mid-pipeline), and a clean SDK import check.
- In Python scripts, inspect and use the exact parameter names accepted by the installed SDK (e.g., via `inspect.signature`), always distinguishing Python SDK method signatures from raw REST HTTP payload fields.
- Check only whether the `TRIPO_API_KEY` environment variable is set (`bool(os.environ.get("TRIPO_API_KEY"))`), **never printing its value** to logs/terminals and **never asking the user to paste their API key in chat**.
- Inspect the exact tools and schemas available in the connected Blender MCP server before calling them.
- **Integration Priority:** For Tripo operations, the **Official Python SDK (API v3)** takes absolute priority over Tripo MCP for generation, import, post-processing, and rigging. Tripo MCP should only be used as a fallback if explicitly requested.
- Never hallucinate tools, parameters, method signatures, or test outcomes.

### 0.2 Operating Mode: SAFE AUTOMATION & FINANCIAL PRECISION
Automatically execute reversible, non-destructive operations (reading directories, inspecting reference images, generating diagnostic previews, running read-only inspection scripts in Blender).

Request **explicit, separate user authorization** before every paid or destructive operation:
- Initial 3D generation on Tripo (`image_to_model()` with 1 image or `multiview_to_model()` with multiple images, with or without texturing).
- Topology optimization via `smart_lowpoly()`.
- Dedicated model import via `import_model()` (only when strictly required and if billed).
- Additional texturing or re-texturing via `texture_model()`.
- Auto-Rigging (`rig_model()` / `rig()`).
- Overwriting existing final assets, destructively modifying original files, or deleting previous versions.

**Mandatory Financial Precision Protocol:**
1. Query the current account balance via `get_balance()` in the SDK and consult Tripo's official pricing rules (never use the balance returned by `get_balance()` as a cost estimate for an operation).
2. **Numeric Precision:** Tripo tracks credits with up to two decimal places. Always use `Decimal(str(value))` (from Python's `decimal` module) when converting valid balances and credit costs, avoiding floating-point (`float`) rounding bugs and never casting credits to integers (`int`) or truncating decimals.
3. Present clearly to the user before execution:
   - Intended operation and input mode (Single-View 1 image vs Multi-View 4 images);
   - Explicit `model_version`;
   - Geometry and texture parameters (`quad`, `face_limit`, `texture`, `pbr`, `delight`, etc.);
   - Exact input file paths;
   - Current balance and estimated credit cost for this step.
4. Wait for explicit user confirmation before triggering each paid API call.
5. After each task finishes, record actual credit usage via `Decimal(str(task.credits_consumed))` when present; if `task.credits_consumed` is null/missing (`None`), record it explicitly as `unavailable` and query `get_usage()` (never convert `None` to `0` or fabricate a consumption number).

### 0.3 Safety Principle
Always preserve the last known working version. If any step fails, do not overwrite or delete prior artifacts—log the full error and halt dependent downstream steps. Never claim an operation succeeded without verifying the actual output file on disk and inside the 3D scene.

---

# 1. OFFICIAL DIRECTORY STRUCTURE

Project root: `<PROJECT_ROOT>/`

Recommended directory layout (created automatically via `pathlib` if not yet present; supports either English `import/references` or Portuguese `importar/referencias` naming consistently):

```text
<PROJECT_ROOT>/
├── assets/
│   ├── import/
│   │   ├── references/
│   │   │   ├── <character_name>/       <-- Place 1 image OR 4 images here (front, left, back, right)
│   │   │   ├── <creature_name>/
│   │   │   └── <prop_name>/
│   │   ├── backups/
│   │   ├── work/
│   │   │   └── tripo/                  <-- Raw (_raw), optimized (_lowpoly), and rigged (_rigged) files
│   │   └── reports/                    <-- Technical reports, credit audits, and diagnostic screenshots
│   ├── models/                         <-- Final game-ready (.glb / .fbx) assets for the Game Engine
│   ├── textures/
│   └── ui/
└── tools/
    └── pipeline_3d/
        └── tripo/
            └── .venv/                  <-- Dedicated Python virtual environment for `tripo3d` SDK
```

**Path Rules:**
- Always use real OS absolute paths handled via `pathlib.Path` inside Python scripts (cross-platform across Windows `C:/...`, Linux, and macOS).
- Never use engine-internal virtual paths (`res://` from Godot, relative `Assets/` from Unity, or `/Game/` from Unreal) in external Python scripts.
- Inside the target Game Engine, use native engine paths as expected (e.g., `res://assets/models/` in Godot 4).

---

# 2. 2D REFERENCES — TOOL-AGNOSTIC & THE 1-OR-4 IMAGES RULE

### 2.1 Complete Freedom of 2D Source (No Vendor Lock-in)
2D reference images are supplied by the user and **do not depend on any specific platform or AI**:
- Hand-crafted 2D concept art (Photoshop, Procreate, Krita, Clip Studio Paint, Blender Grease Pencil, Aseprite);
- Any 2D AI generator (Midjourney, Gemini, ChatGPT, Flux, Stable Diffusion / ComfyUI, Leonardo AI, Ideogram, etc.);
- Clean orthographic 3D blockout renders or isolated background-removed photographs.

### 2.2 Mandatory Image Count: 1 Image (Single-View) OR 4 Images (Multi-View)
The user must populate `assets/import/references/<asset_name>/` using one of two modes:

1. **1-Image Mode — Single-View (`reference.png` or `front.png`):**
   - **When to use:** Ideal for static environment props (chests, furniture, weapons, potions, rocks, pickups), symmetrical items, or rapid prototyping.
   - **Behavior:** Calls `image_to_model()`. Since only one view (front or clean 3/4 view) is provided, Tripo will infer the sides and back.
2. **4-Image Mode — Multi-View (`front.png`, `left.png`, `back.png`, `right.png`):**
   - **When to use:** Strongly recommended for **playable characters, NPCs, bosses, creatures, and asymmetrical assets** where back and side details must match your exact design rather than AI guesswork.
   - **Behavior:** Calls `multiview_to_model()` in the strict order `[front, left, back, right]`.
   - *Note on 2 or 3 views:* If the user only has 2 or 3 angles (e.g., front + back), the **front view is always mandatory**, and omitted slots in the 4-element list must use `None` according to the installed SDK signature (verify signature compatibility first).

### 2.3 Visual & Pose Guidelines by Asset Category
- **Background & Cleanliness (All Assets):** Solid, neutral, high-contrast background; no text, borders, watermarks, heavy cast floor shadows, or cropped silhouettes.
- **Humanoids (Bipedal Characters):**
  - Prefer **T-Pose** (arms straight out at 90°, legs slightly apart, waist and pelvic/groin area clearly visible — avoid floor-length closed robes/capes that fuse the legs together if Auto-Rigging will be used; model long capes separately). Clean *A-Pose* (~45°) is also acceptable if armpits are clearly separated from the torso.
- **Creatures (Quadrupeds, Birds/Avians, Monsters, Arachnids, etc.):**
  - Neutral symmetrical pose with legs, wings, or tails clearly spaced apart to prevent limb fusion.
- **Static Props & Objects:**
  - Isolated object (no character hands holding the item and no background clutter).
- **Multi-View Consistency:** Across all 4 views, keep proportions, vertical alignment, color palette, outfit details, and lighting consistent between `front`, `left`, `back`, and `right`.

### 2.4 Agent Pre-Flight Inspection & Early Halt
- If `assets/import/references/<asset_name>/` is missing or empty when the user requests an asset:
  - **Halt immediately** in state `WAITING_FOR_REFERENCES`.
  - Clearly inform the user that they must place **1 image** (for Single-View) **or 4 images** (`front`, `left`, `back`, `right` for Multi-View) from any 2D tool of their choice before proceeding.
- Once images are present, Claude must **visually inspect them** before any paid API call:
  - Confirm image count matches the intended mode (1 image vs Multi-View).
  - In Multi-View, verify each file matches its expected angle (watch out for common left/right swaps between character-left vs viewer-left) and check proportion/color consistency.
  - For riggable characters/creatures, verify unobstructed shoulder, elbow, pelvis, and knee joints.
  - If severe inconsistencies would ruin the mesh or waste credits, pause, point out the exact issue, and recommend fixing the image before spending credits.

---

# 3. TRIPO — 3D GENERATION, TOPOLOGY & RIGGING (OFFICIAL SDK V3)

### 3.0 Strict Scope Rule (No Gameplay Animations via Tripo Endpoints)
By default in this pipeline, Tripo is used **exclusively** for mesh generation, PBR texturing, topology optimization, and base skeletal rigging:
- Never call Tripo animation endpoints to generate custom gameplay moves unless explicitly requested by the user.
- Never prompt-generate attacks, spells, combat gestures, or gameplay mechanics in Tripo.
- Never use `One-Shot Animation` or `Bake Animation` as a substitute for Blender or your Game Engine's animation system.
- All gameplay animations, blend trees, state machines, and procedural IK belong in Blender and/or the target Game Engine (Godot 4, Unity, Unreal).

### 3.1 Async Execution, Task Polling & Error Handling
- Use `async/await` and `async with TripoClient()` inside the dedicated Tripo `.venv`.
- Use `wait_for_task()` and `download_task_models()`.
- The API may return failed tasks without raising a Python exception. After `wait_for_task()`, always check:
  - `task.status` (explicitly confirm `TaskStatus.SUCCESS`).
  - `task.error_code` and `task.error_msg`.
  - `task.credits_consumed` and `task.output`. If `task.credits_consumed` is `None`, log it as `unavailable` and call `get_usage()` (never turn `None` into `0`; when present, parse via `Decimal(str(task.credits_consumed))`).
  - Physical file existence, non-zero byte size, and structural validity on disk.
- Catch and handle SDK exceptions distinctly: `TripoAPIError`, `TripoRequestError`, `TripoValidationError`, and `TripoPollTimeoutError`.
- **Double-Billing Protection:** Persist `task_id` immediately upon creation. On `TripoPollTimeoutError`, network drop, or unknown state, query the existing `task_id` via `get_task(task_id)` before considering any retry. **Never spawn a duplicate task** while a previous task's status is unresolved.

### 3.2 Explicit Generation Model Selection, Single-view vs Multi-view & P2 Output Format
- Never rely silently on implicit API default models; explicitly pass `model_version` on every call (updating only if Tripo formally deprecates an identifier, and logging the change).
- **High-Fidelity General Model (H3.1):** `v3.1-20260211` for high detail and shape fidelity, confirming compatibility with Single-view or Multi-view. Do not pass `quad=True` to H3.1, as native quad generation belongs to the P2 line. When quads are needed from an H3.1 mesh, evaluate post-processing via `smart_lowpoly(..., quad=True)`.
- **Native Game-Ready Quad Model (P2):** `P2-20260801`, featuring native `quad=True` support (Single-view and Multi-view), with parameter rules distinct from H3.1:
  - Use `model_version="P2-20260801"` with officially supported geometry parameters, including `quad=True` and `face_limit` within `48` to `25000` quad faces.
  - `P2-20260801` **does NOT support** `smart_low_poly` or `geometry_quality` during generation—never send those parameters with P2.
  - **P2 Output Format:**
    - When `quad=True` is enabled on P2, the API outputs `.fbx` to preserve quad faces.
    - Always inspect the actual file extension from the returned download URL/task output (`.fbx`, `.glb`, etc.) and save it with its true extension.
    - If `.fbx`, preserve editable quad topology when importing into Blender.
    - If `.glb`, inspect in Blender to confirm whether topology was triangulated.
    - Never rename a 3D file's extension without performing an actual format conversion in Blender.
  - Do not automatically run `smart_lowpoly` on top of a P2 quad model that already meets polycount and topology targets.
- **Choosing the Python SDK Method (1 Image vs 4 Images):**
  - **1 Image (Single-View):** call `image_to_model(image=image_path, ...)`. Note the SDK signature: Python uses singular `image`, whereas raw HTTP uses `file`.
  - **4 Images (Multi-View):** call `multiview_to_model(images=[front, left, back, right], ...)` in the mandatory order `[front, left, back, right]`, where `front` is required. Note the SDK signature: Python uses `images`, whereas raw HTTP uses `inputs`.
- Always log: input mode (`image_to_model` vs `multiview_to_model`), image count, `model_version`, exact parameters sent, expected format, and actual downloaded extension.

### 3.3 Quad Topology Control & Smart Low Poly
- Initial H3.1 generation outputs dense triangulated meshes. Inspect `_raw` in Blender before deciding on retopology or rigging.
- When polygon reduction or clean quad conversion is needed from H3.1 (or a dense prop), use as baseline:
  `smart_lowpoly(model_version="v3.0", face_limit=10000, quad=True)` *(tuning `face_limit` to your asset budget, e.g., 1,500–4,000 for simple props, 5,000–10,000 for characters).*
- **Smart Low Poly Technical Rules:**
  - Verify the installed SDK signature (`smart_lowpoly`'s `model_version` uses processor versions such as `v1.0`, `v2.0`, `v3.0`—**do not confuse** generation versions like `P2-20260801` or `v3.1-20260211` with Smart Low Poly processor versions).
  - With `quad=True`, strictly respect the operation's documented 10,000-face ceiling.
  - Preserve the true file extension returned by the API.
  - Run as a separate artifact (`_lowpoly`), keeping `_raw` untouched, after cost check and user authorization.
  - Compare `_raw` vs `_lowpoly` visually and geometrically in Blender (silhouette, UVs/textures, joint edge loops) before selecting the winning mesh.

### 3.4 Rigging Compatibility Matrix, Retargeting & `spec`
> [!TIP]
> **Static Props & Environment Assets:** If the asset is static (rigid weapon, furniture, collectible, building, rock, foliage without bones), **skip Section 3.4 and all Rigging steps completely**, saving credits and moving straight to Blender pivot/mesh validation (Section 4).

For articulated characters and creatures:
- Do not use a single rigging model version blindly across all species.
- **Method Signature (`rig_model` vs `rig`):**
  - Inspect the installed SDK signature and never mix parameter names: prefer `rig_model(..., model_version="...")`. If using the `rig()` alias, pass `model="..."` (`rig(..., model="...")`) per official SDK signature.
- **Species-Specific Version Selection:**
  - **Humanoids (`rig_type="biped"`):** prefer `v3.0-20260909` with `spec="mixamo"` when compatible (enables seamless retargeting with Godot 4 `SkeletonProfileHumanoid`, Unity `Humanoid Avatar`, and Unreal `IK Retargeter`).
  - **Avians (`avian`), quadrupeds (`quadruped`), hexapods, octopods, serpentine, aquatic, and other creatures:** use the `rig_type` detected by Rig Check with `v2.5-20260210` as primary preference.
- **Presets (`spec`):**
  - `spec` applies **only** to `rig_type="biped"` in officially supported combinations.
  - For non-humanoids, omit `spec` (or leave default `tripo`). **Never force `spec="mixamo"` onto non-biped creatures.**
- Always run `rig_check()` / `check_riggable()` first and inspect `output.riggable` and `output.rig_type`.
- **Never run Auto-Rig if `output.riggable` is `False`.**

### 3.5 Artifact Preservation (`raw`, `lowpoly`, `rigged`), Rig Input Source & Output Format
- Never overwrite `<asset>_raw` or `<asset>_lowpoly`.
- **Input Source for Rig Check & Auto-Rig:**
  - If rigging the unedited `_raw` mesh directly, pass its `raw_task_id`.
  - **For `_lowpoly` meshes (or meshes edited in Blender):**
    - Current Tripo Rig Check and Auto-Rig endpoints accept a `file_token` directly.
    - **Primary path (`file_token`):** upload the approved `_lowpoly` file via `upload_file()` to obtain a `file_token`, and pass that `file_token` directly to Rig Check and Auto-Rig (**never pass `lowpoly_task_id` directly** as if it were an original generation `task_id` where disallowed).
    - **Fallback path (`import_model`):** use `import_model(file_token)` -> `wait_for_task()` -> `import_task_id` only if required by your specific installed SDK wrapper (checking cost and getting authorization first).
- **Auto-Rig Output Format (Preserving Quads):**
  - Prefer `out_format="fbx"` when rigging quad meshes so quads remain editable inside Blender; allow `out_format="glb"` when quad preservation is unnecessary. Leave final triangulation exclusively to Blender's final Engine export step.

### 3.6 Explicit Texturing Parameters & `delight` Control
- **Critical `delight` Warning:** When specifying `texture_version="v3.5-20260815"`, the API's implicit default for `delight` switches to `True`. Never rely on implicit defaults—**always set all texture parameters explicitly** to match your game's art direction:
  - **Stylized Games (Hand-Painted, Anime/Toon, Claymation, Artistic Low-Poly):** set `delight=False` to preserve rich painted gradients, stylized shading, and color identity from the 2D references.
  - **Realistic / Semi-Realistic PBR Games with Dynamic Lighting:** set `delight=True` to strip baked lighting/shadows from the 2D reference so real-time engine lights behave accurately.
- **Standard Textured Generation Baseline:**
  - `texture=True`
  - `pbr=True`
  - `texture_quality="standard"` *(or `"detailed"` / `"HD"` when supported and authorized for hero assets)*
  - `texture_version="v3.5-20260815"`
  - `delight=False` *(or `True`, per Project Configuration)*
- **Geometry-Only Blockout Tests (when authorized):** `texture=False`, `pbr=False`.

---

# 4. BLENDER MCP — INSPECTION, CLEANUP & VALIDATION

### 4.1 Import, `bpy` Isolation & Diagnostics
- Run all `bpy` scripts exclusively inside Blender's internal Python (via Blender MCP or Blender CLI).
- Import into a dedicated `Collection` inside a working `.blend` file (never overwrite `_raw`, `_lowpoly`, or `_rigged` downloads).
- Inspect real-world metric dimensions, vertex/face/triangle counts, UV maps, Principled BSDF materials, embedded textures, face normals, and (if rigged) bone hierarchy, L/R naming symmetry, `Bone Rest`, and local axes.

### 4.2 Topology Validation (Crucial for AI Meshes)
- Never assume an AI-generated quad mesh is automatically deformation-ready.
- Characters/Creatures: inspect edge flow at shoulders, elbows, wrists, groin/hips, knees, and neck; check for loose vertices, internal phantom geometry, or severe non-manifold edges.
- Static Props: verify silhouette cleanliness, check back/bottom surfaces for holes (especially on 1-image Single-View generations!), and verify pivot placement.

### 4.3 Skinning Weights & Vertex Groups
- Do not blindly re-run Blender `Automatic Weights` over a working Tripo rig (it frequently bleeds weights across legs, fingers, or clothing).
- Preserve original `Vertex Groups` when deformations work cleanly. If weight fixes are needed, duplicate a backup copy first, fix and pose-test on the copy, then promote it.

### 4.4 Mandatory Visual Inspection (Screenshots / Previews)
- For rigged models, enable `armature_obj.show_in_front = True` to verify bone alignment inside the mesh.
- Capture **front, side, and back** viewport screenshots.
- **Automatic Fallback:** If viewport capture fails or is unavailable in the active Blender MCP server, render fast orthographic Workbench/EEVEE diagnostic previews with temporary camera/lights (cleaning them up afterward) into `assets/import/reports/` and inspect them visually.

### 4.5 Anatomical Checks, Pivot Placement, Pose Tests & `Ctrl+A` Safety
- **Rigged Characters & Creatures:**
  - Verify `Hips`/`Pelvis` placement, shoulder pivots, and subtle natural elbow/knee bend for predictable IK.
  - Test basic pose rotations (raise arms, bend elbows/knees, turn head) checking for mesh tearing or candy-wrapper collapse, **mandatory reset to Rest Pose (`pose.transforms_clear()`) afterward**.
  - **Transform Application Safety (`Ctrl+A`):** Never blindly apply `Ctrl+A` (All Transforms) on rigged objects without inspecting `matrix_world`, parent-child hierarchy, Armature modifier, `Bone Rest`, and bind matrices on a backup copy first.
  - Place feet cleanly on the ground plane (`Z = 0`) by translating the root (`Armature` + bound `Mesh`) safely.
- **Static Props:**
  - Apply Rotation & Scale (`Scale = 1, 1, 1`) cleanly and place the origin/pivot logically (`Z = 0` bottom center for floor/table props, center of mass for physics pickups, or grip point for handheld weapons/tools).

---

# 5. FINAL EXPORT, QUAD PRESERVATION & ROUND-TRIP CHECK

- **Role of Each Format:**
  - `_raw` / `_lowpoly` / `_rigged`: `.fbx` when preserving quads (stored untouched in `assets/import/work/tripo/`).
  - Editable master project (`.blend`): saved in `assets/import/work/<asset_name>.blend`, keeping editable quads and rig.
  - Final Engine delivery (`.glb` by default, or `.fbx` if required by Unity/Unreal): exported to `assets/models/<asset_name>.glb`, performing clean triangulation **only during final export**.
- **Mandatory Export Rules:**
  - Export only the final asset objects (`Mesh` + `Armature` if rigged)—exclude temporary cameras, lights, hidden backups, and test poses.
  - Never overwrite an existing file in `assets/models/` without backing up the previous version to `assets/import/backups/` with a timestamp.
  - **Round-Trip Verification:** Re-import the exported `.glb`/`.fbx` into a **clean Blender scene** via script to confirm `1.0` scale, intact PBR textures, bone hierarchy, and clean triangulation before engine integration.

---

# 6. GAME ENGINE INTEGRATION (GODOT 4 DEFAULT + MULTI-ENGINE NOTES)

### 6.1 Honest Engine Access Verification
- Check whether direct Engine CLI or MCP access is available (e.g., Godot CLI/MCP, Unity MCP, Unreal Python).
- If direct execution is unavailable: validate accessible project files on disk, record state `ENGINE_VALIDATION` as **pending manual editor check**, provide clear step-by-step editor instructions, and **never claim `BoneMap`, `Avatar`, `IK Rig`, or animations were runtime-tested without real execution**.

### 6.2 Import, Retargeting & Asset Categories
- **Godot 4 (Default):**
  - Place final `.glb` files in `res://assets/models/`. Verify metric scale, `-Z` forward orientation, `Skeleton3D`, and PBR materials.
  - **Humanoids:** configure `BoneMap` (`SkeletonProfileHumanoid`) in *Advanced Import* for shared animation retargeting (`Idle`, `Walk`, `Run`, `Jump`, `Attack`).
  - **Avians (`avian`), quadrupeds & non-humanoid creatures:** use the native exported `Skeleton3D` directly without forcing a humanoid profile.
  - **Static Props:** use `StaticBody3D` with simple primitives or optimized convex collision shapes (avoid heavy *Trimesh* colliders on small/dynamic objects).
- **Unity / Unreal Engine 5:**
  - **Unity:** `Animation Type: Humanoid` + validated `Avatar` for `mixamo` bipeds; `Generic` for creatures; `Mesh Collider (Convex)` or primitive colliders + URP/HDRP materials for props.
  - **Unreal Engine 5:** Interchange `.glb` or `.fbx` Skeletal Mesh + `IK Rig` / `IK Retargeter` for characters; `Static Mesh` with simplified collision for props.

### 6.3 Gameplay IK & Procedural Animation Best Practices (Godot 4.3+ Reference)
Use Godot 4's modern `SkeletonModifier3D` stack:
- **Head/Neck Target Tracking:** use `LookAtModifier3D` with natural angular constraints.
- **Arm Aiming / Casting / Reaching:** use `TwoBoneIK3D` with both `Target` and `Pole Target`. **Never apply `LookAtModifier3D` across a full arm chain**, as it causes severe forearm/wrist mesh twisting.
- **Upper-Body Actions While Moving:** blend upper-body actions (attacking, casting, holding a torch/shield) over locomotion using `Bone Mask` inside `AnimationTree`.

---

# 7. PIPELINE AUTOMATION (DEFINITIVE EXECUTION ORDER)

### 7.1 Mandatory 17-Step Sequence
1. **Pre-Flight & Image Check (1 or 4 Images):** Check `assets/import/references/<asset_name>/`. If empty, **halt and notify the user** to add **1 image (Single-View)** or **4 images (Multi-View: front, left, back, right)** from any 2D tool/AI. Visually inspect images, verify Tripo `.venv`, `tripo3d>=1.0.0,<2.0.0`, `TRIPO_API_KEY` presence, and Blender MCP.
2. **Balance & Generation Authorization:** Query `get_balance()` (`Decimal(str(val))`), present input mode (1 vs 4 images), `model_version`, explicit `delight`/texture params, and estimated cost; **wait for explicit user approval**.
3. **Async 3D Generation:** Run `image_to_model(image=...)` (1 image) or `multiview_to_model(images=[front, left, back, right])` (4 images) with explicit `model_version` (`v3.1-20260211` or `P2-20260801`) and Section 3.6 texture parameters; verify `TaskStatus.SUCCESS`.
4. **Safe `_raw` Download:** Save `<asset>_raw` in `assets/import/work/tripo/` with its true extension, never overwriting it.
5. **Initial Blender Diagnostics:** Inspect topology (quads vs tris), polycount, scale, UVs, and textures in Blender MCP.
6. **Evaluate `Smart Low Poly`:** Recommend `smart_lowpoly(model_version="v3.0", face_limit=..., quad=True)` only if needed (skip if P2/raw mesh already meets budget and quality).
7. **Optional Low Poly Execution (Authorized):** Present cost, get approval, run on copy, and download `<asset>_lowpoly`.
8. **Mesh Comparison & Selection:** Compare `_raw` vs `_lowpoly` in Blender and pick the approved mesh. *(Note: for **static props**, skip Rigging steps 9–12 and jump straight to step 13).*
9. **Prepare Rigging Source (Characters & Creatures Only):** Use `raw_task_id` for unedited `_raw`, or `upload_file()` -> `file_token` directly for `_lowpoly`/edited meshes (using `import_model` only as SDK fallback).
10. **Rig Check:** Run `rig_check()` / `check_riggable()` and confirm `output.riggable == True` and `output.rig_type`.
11. **Auto-Rig (Authorized):** Present cost, get approval, and run `rig_model` (`v3.0-20260909` + `spec="mixamo"` for humanoids; `v2.5-20260210` without `spec` for creatures; `out_format="fbx"` to keep quads).
12. **Safe `_rigged` Download:** Save `<asset>_rigged` in `assets/import/work/tripo/` with its true extension.
13. **Blender Visual & Technical Validation:** Inspect mesh, materials, pivot, and (if rigged) skeleton/weights with front/side/back screenshots (`show_in_front = True`).
14. **Safe Fixes, Pose Tests & `.blend` Save:** Place feet/base at `Z = 0`, set prop pivots, run pose deformation tests (resetting to Rest Pose afterward), and save `assets/import/work/<asset_name>.blend` preserving quads.
15. **Final Export & Round-Trip Check:** Backup any prior export, export final `.glb`/`.fbx` to `assets/models/`, and re-import into a clean Blender scene to verify integrity.
16. **Game Engine Integration:** Integrate and test in the target Game Engine, or record `ENGINE_VALIDATION` honestly with step-by-step user instructions.
17. **Final Financial Audit & Report:** Query `get_usage()`, record exact `Decimal` credit usage per step, log all task IDs/`file_token`s, and write the report in `assets/import/reports/<asset_name>_report.md`.

### 7.2 Pipeline States
`WAITING_FOR_REFERENCES` -> `VALID_REFERENCES` -> `WAITING_TRIPO_AUTH` -> `GENERATING_3D` -> `GEOMETRY_INSPECTION` -> `LOWPOLY_OPTIMIZATION` *(optional)* -> `LOWPOLY_IMPORT` *(if applicable)* -> `RIG_CHECK` *(riggable only)* -> `RIGGING` *(riggable only)* -> `BLENDER_IMPORT` -> `DIAGNOSTICS` -> `FIXING` -> `RIG_VALIDATION` *(or `PROP_VALIDATION`)* -> `EXPORTING` -> `ENGINE_VALIDATION` -> `COMPLETED` | `FAILED`.

### 7.3 Mandatory Technical Reports
Generate `assets/import/reports/<asset_name>_report.md` containing: asset name/category, target engine, input mode (Single-View 1 image vs Multi-View N images), exact SDK/model/texture/lowpoly/rig versions, all `task_id` and `file_token` identifiers, exact `Decimal` credits consumed per task (or `unavailable` + `get_usage()`), quad vs triangulated vertex/face/bone counts, real file paths, issues detected/fixed, diagnostic screenshots, and honest Engine validation status.

---

# 8. FINAL QUALITY STANDARD

Every delivered asset must honor the project's art direction, clean silhouette, consistent metric scale, intact PBR materials with zero broken textures, proper gameplay pivot, species-appropriate topology/rigging, and verified Game Engine compatibility.

**GOLDEN RULE:** Automate aggressively to accelerate game development, **never sacrificing art direction, credit safety, or runtime gameplay integrity**. Never declare an asset, rig, or engine test complete without real verification.
