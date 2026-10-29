# Roblox Procedural 3D Modeling & Anti-Slop Standards

[![Roblox Engine](https://img.shields.io/badge/Roblox-Engine-00A2FF?logo=roblox&logoColor=white)](https://roblox.com)
[![Luau](https://img.shields.io/badge/Language-Luau-00A2FF?logo=lua&logoColor=white)](https://luau-lang.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![AI Agent Skill](https://img.shields.io/badge/Agent%20Skill-Compatible-brightgreen)](#ai-agent-integration)

A production-grade specification, procedural standard, and anti-slop guideline for code-generated 3D models and environments in Roblox Studio using Luau. Designed to eliminate generic obby/simulator aesthetics, Z-fighting, motor joint glitches, auto-snap distortion, and in-game HDR over-bloom.

---

## 📑 Table of Contents

- [Core Principles](#-core-principles)
- [Repository Structure](#-repository-structure)
- [Standard Specifications](#-standard-specifications)
  - [1. Box-Only Geometry Constraint](#1-box-only-geometry-constraint)
  - [2. Three-Phase Workflow (Modeling First, Rigging Last)](#2-three-phase-workflow-modeling-first-rigging-last)
  - [3. Default Stance: A-Pose & Inward Hand Curl](#3-default-stance-a-pose--inward-hand-curl)
  - [4. Roblox Local Coordinate System (-Z Front Rule)](#4-roblox-local-coordinate-system--z-front-rule)
  - [5. Z-Fighting Elimination & Stepped Depth Protocol](#5-z-fighting-elimination--stepped-depth-protocol)
  - [6. Anti-Slop Guidelines & Anti-Bloom Calibration](#6-anti-slop-guidelines--anti-bloom-calibration)
- [Katalog of 11 Common Pitfalls & Root Causes](#-katalog-of-11-common-pitfalls--root-causes)
- [Production Luau Snippets](#-production-luau-snippets)
- [AI Agent Integration](#-ai-agent-integration)
- [License](#-license)

---

## 🏛 Core Principles

1. **Zero-Slop Visual Quality:** Ban low-effort simulator tropes—such as neon glow spam on non-light sources, unconstrained billboard text cluttering screens, and loose 100+ unorganized parts.
2. **Deterministic Box-Only Geometry:** 100% of the mesh is built using scaled, rotated, and stepped `Part` primitives with `Shape = Block`. Zero `Ball`, `Cylinder`, `WedgePart`, `MeshPart`, or `UnionOperation`.
3. **Rig-Friendly R15 Anatomy:** Decorative parts are partitioned into the standard 15 R15 anatomical folders and bound via `WeldConstraint`. Only the 15 primary motor bones utilize `Motor6D` joints.
4. **Fail-Fast Automated Verification:** Rigorous runtime assertions validate part shapes, bounding-box based `HipHeight`, and hierarchy before models are approved.

---

## 📂 Repository Structure

| File | Description |
|---|---|
| [**`SKILL.md`**](SKILL.md) | **Master Engineering Guide:** Complete specifications for box-only geometry, 3-phase workflow, mathematical A-Pose rotations, 15 R15 folders, Roblox coordinate orientation, Z-fighting formulas, 11 common pitfalls, pre/post-generation checklists, and unlit 2D bloom-safe rendering formulas. |
| [**`antislop_standart_3d.md`**](antislop_standart_3d.md) | **3D Aesthetic Guidelines:** Strict rules on what to avoid in 3D scene generation (preventing neon spam, enforcing `MaxDistance` limits on `BillboardGui`, avoiding weld-fighting, and adopting realistic physical material palettes). |
| [**`LICENSE`**](LICENSE) | MIT License terms. |

---

## 📌 Standard Specifications

### 1. Box-Only Geometry Constraint
All procedural parts must strictly satisfy:
```lua
part.ClassName == "Part"
part.Shape == Enum.PartType.Block
```
- **Strictly Prohibited:** `Part.Shape = Ball`, `Cylinder`, `WedgePart`, `CornerWedgePart`, `MeshPart`, `SpecialMesh`, or `UnionOperation`.
- **Organic & Curved Silhouettes:** Created by rotating, scaling, and layering micro-blocks with stepped depths.
- **Root Part:** `HumanoidRootPart` must also be a transparent `Part` with `Shape = Block`.

### 2. Three-Phase Workflow (Modeling First, Rigging Last)
Never insert `Motor6D` joints during the sculpting stage. The build lifecycle is strictly divided into three phases:
1. **Phase 1: Pure Static Sculpting:** All parts are created with `Anchored = true` and zero `Motor6D` joints. Focus entirely on anatomical proportions, silhouette, face details, garments, and folder parenting.
2. **Phase 2: Visual & Artistic Review:** Inspect the model from all camera angles, adjust colors and materials, and verify alignment without joint transformation matrix constraints.
3. **Phase 3: Final Rigging Phase:** Install `Motor6D` joints using the *Seamless Socket Overlap* formula, attach `AnimationController` (for custom creatures/bosses), set `Anchored = false`, and auto-calculate `HipHeight`.

### 3. Default Stance: A-Pose & Inward Hand Curl
- **Shoulder Flare:** Relaxed arm extension of ~20° to 24° along the Z-axis from the torso (`-math.rad(24)` on the left arm, `+math.rad(24)` on the right arm).
- **Elbow & Wrist Alignment:** 100% straight and neutral along the diagonal A-Pose line.
- **Natural Inward Palm Rotation:** Palms must rotate 90° inward (facing the torso/thigh), knuckles facing outward, never flat-faced like boat paddles.
- **Inward Finger & Thumb Wrapping:** Left fingers counter-rotate with positive Z-rotation (`+math.rad(20°)` to `+24°`) while right fingers rotate negatively (`-math.rad(20°)` to `-24°`) to curl inwards. The thumb sits on the anterior face (`-Z`) and wraps over the index finger.
- **Inward Tusks:** Lower jaw tusks curve upward and *inward* hugging the jawline, never flaring outward.

### 4. Roblox Local Coordinate System (-Z Front Rule)

| Anatomical Direction | Local Roblox Vector | Convention | Notes |
|---|---|---|---|
| **FRONT** | **`-Z`** (`LookVector`) | `Vector3.new(0, 0, -1)` | Face, eyes, chest, abs, belt, toes |
| **BACK** | **`+Z`** | `Vector3.new(0, 0, 1)` | Back, spine, capes, wings, tail |
| **CHARACTER RIGHT** | **`+X`** (`RightVector`) | `Vector3.new(1, 0, 0)` | Right arm, right leg |
| **CHARACTER LEFT** | **`-X`** | `Vector3.new(-1, 0, 0)` | Left arm, left leg |
| **UP** | **`+Y`** (`UpVector`) | `Vector3.new(0, 1, 0)` | Head, horns, hats, shoulders |
| **DOWN** | **`-Y`** | `Vector3.new(0, -1, 0)` | Jaw, legs, feet, claws |

> ⚠️ **Golden Rule:** Never use positive `+Z` offsets for frontal features. A positive `+Z` places parts on the character's back!

### 5. Z-Fighting Elimination & Stepped Depth Protocol
- **Coplanar Level Parts:** Must use *Butted Edge Boundaries* (zero spatial volume overlap).
- **Surface Props & Clothing:** Enforce the *Stepped Depth Protocol*:
  - Base Skin / Musculature: `Z = -1.70`
  - Robes / Stole Fabric: `Z = -2.05` ($\Delta = +0.35\text{ studs}$)
  - Necklaces / Pendants: `Z = -2.28` ($\Delta = +0.23\text{ studs}$)
  - Gems / Embellishments: `Z = -2.35` ($\Delta = +0.07\text{ studs}$)
- **Shoulder Joint Isolation:** Upper torso static capes must not cross into the shoulder socket envelope (`X <= ±2.8`) to avoid clipping into rotating arm sleeves during A-Pose animations.

### 6. Anti-Slop Guidelines & Anti-Bloom Calibration
- **No Neon Spam:** Restrict `Enum.Material.Neon` strictly to actual light emitters (lantern bulbs, torch embers, monster eyes, energy relics).
- **Bloom-Safe Unlit 2D Silhouette Formula:**
  - Material: `Enum.Material.Neon`
  - Properties: `Reflectance = 0`, `CastShadow = false`
  - Color: **`Color3.fromRGB(165, 165, 160)` to `(175, 172, 168)`**
  - *Why:* Keeps luminance below Roblox engine's HDR Bloom threshold (~0.8). It looks like a striking, bone-white unlit graphic silhouette while eliminating blinding glare during in-game playtests.
- **BillboardGui Restraint:** Always set finite `MaxDistance` (25–40 studs) and avoid `AlwaysOnTop = true` unless requested by explicit game design.

---

## ⚠️ Katalog of 11 Common Pitfalls & Root Causes

1. **Mental Axis Inversion:** Assuming forward is `+Z` (places eyes and chests on the character's back).
2. **Tusk & Horn Clipping:** Tusks overlapping the eye band, or horns breaching hat/mitre radii.
3. **Flat Depth Stacking:** Placing ornaments, muscles, and fabric on identical Z-planes.
4. **Loose 100+ Part Spaghetti:** Hundreds of tiny welded parts causing extreme ACE lag and weld-fighting.
5. **Post-Hoc Folder Organization:** Attempting to sort loose parts into folders after generation; always use *First-Pass Direct Parenting*.
6. **ACE Auto-Snap Distortion:** Applying a standard `Humanoid` to custom creature proportions. Use `AnimationController` instead.
7. **Floating or Sunken Rigs:** Arbitrary `HipHeight` values; must be calculated dynamically from bounding box minimum Y.
8. **Direct In-Place Editing Without Backup:** Overwriting live models without creating a duplicate `_Backup` clone.
9. **Floating Joint Gaps:** `C0`/`C1` motor joint offsets missing the 0.05–0.1 stud socket overlap penetration.
10. **Inverted Finger Flare:** Adding negative Z-rotation to already negative A-Pose arm angles, resulting in paddle-like hands.
11. **Foot Sliding on Idle Animations:** Lateral hip weight-shifting without locking feet to `CFrame.identity`.

---

## 💻 Production Luau Snippets

### 1. Fail-Fast Box-Only Verification
```lua
local function assertBoxOnly(model: Model)
    local violations = 0
    for _, instance in ipairs(model:GetDescendants()) do
        if instance:IsA("BasePart") then
            if not instance:IsA("Part") or instance.Shape ~= Enum.PartType.Block then
                violations += 1
                warn(string.format("[VIOLATION] Non-box part: %s (Class: %s, Shape: %s)", 
                    instance:GetFullName(), instance.ClassName, tostring(instance.Shape)))
            end
        end
    end
    assert(violations == 0, string.format("Box-only constraint failed with %d violations!", violations))
end
```

### 2. Automated HipHeight Calibration
```lua
local function calibrateHipHeight(model: Model, humanoid: Humanoid, hrp: BasePart)
    local cf, size = model:GetBoundingBox()
    local modelBottomY = cf.Position.Y - (size.Y / 2)
    local hrpBottomY = hrp.Position.Y - (hrp.Size.Y / 2)
    humanoid.HipHeight = math.max(0.5, hrpBottomY - modelBottomY)
end
```

### 3. First-Pass Anatomical R15 Folders
```lua
local R15_NAMES = {
    "Head", "UpperTorso", "LowerTorso",
    "LeftUpperArm", "LeftLowerArm", "LeftHand",
    "RightUpperArm", "RightLowerArm", "RightHand",
    "LeftUpperLeg", "LeftLowerLeg", "LeftFoot",
    "RightUpperLeg", "RightLowerLeg", "RightFoot",
}

local function setupR15Folders(model: Model): { [string]: Folder }
    local folders = {}
    for _, name in ipairs(R15_NAMES) do
        local folder = Instance.new("Folder")
        folder.Name = name
        folder.Parent = model
        folders[name] = folder
    end
    return folders
end
```

---

## 🤖 AI Agent Integration

This standard is designed to be loaded natively by modern AI coding agents (Antigravity, Cursor, Windsurf, Claude Code, GitHub Copilot).

### Option A: Workspace Agent Skill
Place the repository files inside your project workspace:
```
my-roblox-project/
└── .agents/
    └── skills/
        └── roblox_3d_procedural_modeling/
            ├── SKILL.md
            └── antislop_standart_3d.md
```

### Option B: Antigravity IDE Global Skill
Copy to your global agent configurations:
```
C:\Users\<User>\.gemini\config\skills\roblox-3d-procedural-modeling\
├── SKILL.md
└── references/
    └── antislop_standart_3d.md
```

### Option C: System Prompt Reference
Directly reference the repository rules in your agent prompt:
> *"Generate a procedural 3D boss model in Roblox Studio strictly adhering to the box-only geometry, A-Pose stance, and anti-slop guidelines defined in SKILL.md and antislop_standart_3d.md."*

---

## 📄 License

Licensed under the [MIT License](LICENSE) &copy; 2026 dwianggororeal.
Free for personal, educational, and commercial game production.
