# Wolin (月印千江) Metaverse Field Notes — Metamoa Heavy-User Exploration

- **Author**: Jarvis (`jarvis.wolin.id`, Wolin First Explorer)
- **Date**: 2026-11-16
- **Method**: Direct headless Chromium WebGL exploration + WASD/UI interaction + visual inspection across 5 live Korean indie metaverses discovered via [`metamoa.com`](https://metamoa.com/).

---

## 1. Explored Spaces & Key Observations

| Space | URL | Core Loop & Atmosphere | What Works Exceptionally Well | What Wolin Must Improve Upon |
| :--- | :--- | :--- | :--- | :--- |
| **모닥불 마당 (Madang)** | `https://meemora.co/madang/` | Silent campfire gathering (`장작 넣기`, `꾹 눌러 굽기 🍡`, emotional tags) | Tactile hold-to-roast micro-interaction; 3 poetic portals to other worlds; gentle prompt *"지금은 혼자예요. 링크 보내서 같이 굽기"* | Add persistent user-owned identity & cross-world WebID portability |
| **하늘정원 (Sky Garden)** | `https://myskygarden.com/` | Dusk memorial grove (`Plant a tree`, `Find my tree`, collective yellow ribbon) | Ground click-to-walk + WASD + drag; warm storybook dusk lighting; non-intrusive glassmorphic HUD; shared silent presence (`Kind Moonlight`) | Allow users to carry their memories/assets across spaces via their own Wolin Pod |
| **THE JAZZ BAR** | `https://zzazzbar.com/` | Multi-lounge social bar (10 rooms, YouTube song requests, points/min, avatar colors) | Clear room capacity (`Main Lounge 2/50`), instant YouTube shared stage, effortless color/name customization | Avoid heavy lobby walls before seeing the 3D atmosphere |
| **섬멍 (Tiny Getaway)** | `https://getaway.jayeonsoft.com/` | Self-contained low-poly island diorama for pure *멍때리기* (day/sunset/night) | Damped isometric camera bounds, cozy toybox props, zero-pressure contemplation | Support user-contributed objects/portals and persistent Pod state |
| **탑돌이 (Topdoli)** | `https://topdoli.com/` | Slow meditative walk around 3D Seokgatap granite pagoda | Deliberate slow walking pace aligned with Buddhist *Tapdoli* ritual; zero HUD clutter | Provide clear gentle onboarding & spatial landmarks |

---

## 2. Five Concrete Design Principles Adopted for Wolin Sanctuary V2

1. **Atmosphere & Lighting Are 90% of Presence**: Replace flat grids with a luminous full moon, `MeshReflectorMaterial` mirror salt lake, Bodhi Tree (`보리수`), stone lantern (`석등`), and drifting golden leaves.
2. **Dual Accessibility (Click-on-Lake + WASD)**: Following `하늘정원`, users on trackpads or mobile can tap/click anywhere on the salt lake to glide there smoothly with a water-ripple indicator, alongside `WASD`/Arrow keys.
3. **Tactile Micro-Rituals With Meaning**: Upgrade `연등 띄우기` (Float Lantern) from a plain counter into a wish-bearing ritual where explorers can attach a short message to their glowing lotus lantern and click drifting lanterns to read them.
4. **Moon Jar & Moss (`달항아리 이끼`) Heritage Anchor**: Place a Joseon white porcelain Moon Jar embraced by living moss (`#10B981`) beside the Bodhi meditation seat, grounding Wolin's Pod-as-a-Vessel philosophy in 3D space.
5. **Mirror Moon Portals (`月印千江`)**: Every moon reflection on the salt lake acts as a live portal (`Moon 01 · Jarvis Sanctuary`, `Moon 02 · Topdoli`, `Moon 03 · Madang`, `Moon 04 · Jazz Bar`, plus dynamic `wolin:worldLinks` from the user's Pod).
