# Universal 3D Game Asset Pipeline — Claude + Tripo SDK (v3) + Blender MCP

> 🇺🇸 **English:** `README.md` | 🇧🇷 **Português:** [README.pt-BR.md](./README.pt-BR.md)

A production-grade specification ([`CLAUDE.md`](./CLAUDE.md)) that turns **Claude** (Claude Code / Claude Desktop) into a **3D Production Engineer** capable of generating 3D assets from 2D references, preserving clean *quad* topology, applying species-aware *Auto-Rigging*, inspecting/fixing meshes inside **Blender** via MCP, and exporting game-ready `.glb` / `.fbx` assets for **Godot 4**, **Unity**, or **Unreal Engine 5**.

---

## Why This Pipeline Exists (vs. Basic MCP Demos)

Most AI-to-3D setups stop at *"generate a raw mesh from a text prompt via MCP."* In real game production, that leads to wasted API credits, broken triangles on joints, baked shadows clashing with engine lights, and twisted rigs. This pipeline enforces battle-tested production rules:

- **Works for Any Game & Art Style:** Stylized (*hand-painted*, *claymation*, *anime/cel-shaded*, *low-poly*) or realistic PBR, with explicit `delight` control (`False` to keep painted gradients, `True` to strip baked lighting for dynamic engine lights).
- **Flexible Input (1 or 4 Images from Any Source):**
  - **1 Image (`Single-View`):** Fast workflow for static props, pickups, and symmetrical objects (`image_to_model`).
  - **4 Images (`Multi-View` — Front, Left, Back, Right):** High anatomical accuracy for characters, creatures, and asymmetrical designs (`multiview_to_model`).
  - **Zero 2D Tool Lock-in:** Use hand-drawn concept art (Photoshop, Procreate, Krita, Blender Grease Pencil) or any image generator (Midjourney, Gemini, ChatGPT, Flux, Stable Diffusion / ComfyUI, Leonardo AI, etc.).
- **Strict API Credit Safety (`Decimal`):** Queries balance first, requires **explicit human approval** before every paid Tripo API call, prevents double-billing on polling timeouts (`get_task` recovery), and tracks credits using Python `Decimal` (zero floating-point rounding bugs).
- **Official Python SDK (`tripo3d` v3) Isolation:** Separates Tripo's `.venv` from Blender's internal `bpy` Python—avoiding environment conflicts and exposing v3 features (`P2` native quads, `file_token` rigging, `v3.5` textures) often missing from third-party wrappers.
- **True Quad Preservation:** Understands `P2` native quad `.fbx` rules and `Smart Low Poly` constraints, keeping your master `.blend` editable in quads and triangulating only on final `.glb` export.
- **Species-Aware Rigging + Static Prop Bypass:** Automatically selects `v3.0` (`spec="mixamo"`) for humanoids and `v2.5` (omitting `spec`) for birds/quadrupeds/creatures—and automatically skips rigging on static props to save credits.
- **Visual Viewport Inspection & Round-Trip Check:** Verifies UVs, normals, bone alignment (`show_in_front = True`), pose deformations, and re-imports the final `.glb` into a clean Blender scene before engine handoff.

---

## Prerequisites

1. **Python 3.10+** (for the isolated Tripo SDK virtual environment)
2. **[Tripo 3D](https://platform.tripo3d.ai/) Account & API Key** set as the `TRIPO_API_KEY` environment variable
3. **Blender 3.6 / 4.x** + **[Blender MCP](https://github.com/ahujasid/blender-mcp)** connected to your Claude client
4. **Claude Code** or **Claude Desktop**
5. *(Optional)* **Godot 4**, **Unity**, or **Unreal Engine 5**

---

## Quick Setup

### 1. Copy `CLAUDE.md` to your game project root
Copy [`CLAUDE.md`](./CLAUDE.md) into the root directory of your game repository (so Claude Code loads it automatically) and optionally fill in the **Project Configuration** block at the top.

### 2. Create the isolated Tripo SDK virtual environment
From your game project root:

**Windows (PowerShell):**
```powershell
python -m venv tools/pipeline_3d/tripo/.venv
.\tools\pipeline_3d\tripo\.venv\Scripts\pip.exe install "tripo3d>=1.0.0,<2.0.0"
```

**Linux / macOS:**
```bash
python3 -m venv tools/pipeline_3d/tripo/.venv
./tools/pipeline_3d/tripo/.venv/bin/pip install "tripo3d>=1.0.0,<2.0.0"
```

### 3. Set your `TRIPO_API_KEY` environment variable
Never paste your API key into chat or commit it to Git!

**Windows (PowerShell — user environment):**
```powershell
[Environment]::SetEnvironmentVariable("TRIPO_API_KEY", "tsk_your_api_key_here", "User")
```

**Linux / macOS (`~/.bashrc` or `~/.zshrc`):**
```bash
export TRIPO_API_KEY="tsk_your_api_key_here"
```

---

## Mandatory Step Before Generating: Add 1 or 4 Images!

Before asking Claude to generate a 3D asset, create a folder under `assets/import/references/<asset_name>/` and drop in:

- **Option A — 1 Image (Single-View):**
  - Example: `assets/import/references/wooden_chest/front.png`
  - Best for static props, simple items, or symmetrical objects.
- **Option B — 4 Images (Multi-View — Recommended for Characters & Creatures):**
  - `assets/import/references/knight/front.png`
  - `assets/import/references/knight/left.png`
  - `assets/import/references/knight/back.png`
  - `assets/import/references/knight/right.png`
  - For humanoids intended for Auto-Rigging, prefer a clean **T-Pose** on a solid neutral background with visible shoulder and waist/pelvis joints.

> [!NOTE]
> If the folder is empty, Claude will automatically pause at `WAITING_FOR_REFERENCES` and remind you where to place your 1 or 4 images before spending a single credit.

---

## Example Prompts for Claude

With Blender open (and the Blender MCP addon running):

> *"Follow `CLAUDE.md` and process the character in `assets/import/references/knight/` using the P2 quad model for Godot 4."*

Or for a static prop using a single reference image:

> *"Follow `CLAUDE.md` and generate the static prop in `assets/import/references/wooden_chest/` (1 image, no rig)."*

---

## License

Released under the MIT License. Feel free to use, adapt, and contribute!
