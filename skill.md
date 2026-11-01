### MANDATORY ARCHITECTURAL, VISUAL & UI RULES (`marine-cad-digital-twin-builder v2.1.0`)
1. **Single Self-Contained HTML File (`< 1 MB`)**:
   - Output a single standalone HTML file using Three.js (`r160` via ES module importmap or `r128`) and Tailwind CSS CDN. Zero external 3D asset files (`.glb`/`.obj`). Build all geometry procedurally in true marine millimetre/metre scale.
2. **Unified Light-Studio Environment & Plinth (Diesel Generator / VCB Standard)**:
   - Scene background: `new THREE.Color(0xf1f3f6)` and CSS `#canvas-container` radial gradient `radial-gradient(circle at 50% 45%, #f8fafc 0%, #eaedf1 100%)`.
   - Two-tier architectural concrete studio plinth at `y = 0.0` (upper slab `#f8fafc` at `y = -0.04 m`, lower curb `#e2e8f0` at `y = -0.09 m`) with a `ShadowMaterial({ opacity: 0.16 })` plane at `y = 0.001 m`. **Never** add a wireframe `GridHelper` on the floor.
   - Lighting: `HemisphereLight(0xffffff, 0xdbe2ea, 1.25)`, key `DirectionalLight(0xfffdf9, 1.35)` at `(8, 14, 7)` with `2048x2048` `PCFSoftShadowMap`, fill light `(0xe2e8f0, 0.85)`, rim light `(0xffffff, 0.55)`, and `ACESFilmicToneMapping` (`toneMappingExposure = 1.12`).
3. **Soft Industrial CAD Material Palette (No Harsh Jet-Black)**:
   - Cabinet / Enclosure Steel: `#c9d2db` (`roughness: 0.42, metalness: 0.16`)
   - Structural Trim / Frames: `#9aa8b6` (`roughness: 0.45, metalness: 0.22`)
   - Heavy Carriage / Castings: `#8796a5` (`roughness: 0.44, metalness: 0.28`)
   - Bezels / Base Channels / Conduits: `#738291` (`roughness: 0.46, metalness: 0.26`)
   - Marine Machinery Sage Enamel: `#3b6254` (`roughness: 0.44, metalness: 0.22`)
   - Cast Epoxy Resin / HV Coils: `#a6534c` (`roughness: 0.36, metalness: 0.10`)
   - Copper Busbars & Windings: `#d47a42` (`roughness: 0.28, metalness: 0.80`)
   - Nickel-Aluminium-Bronze (NAB) / Brass: `#d8b25c` (`roughness: 0.28, metalness: 0.78`)
   - Polished Shaft Steel / Silver Contacts: `#e2e8f0` (`roughness: 0.20, metalness: 0.85`)
4. **Native-Color Ghost-Opacity Subsystem Isolation (`opacity: 0.14`)**:
   - In `prepareSubsystemMaterials()`, clone every mesh's material per subsystem into **both** `obj.userData.origMaterial` AND `obj.userData.ghostMaterial = origClone.clone()` with `transparent: true, opacity: 0.14, depthWrite: false`.
   - **Never** use a single shared grey or yellow `matGhost` material! When a subsystem is isolated, non-selected subsystems must use their own `obj.userData.ghostMaterial` so every ghosted part retains its native color at low opacity (`0.14`).
   - Any dynamically colored status flags/indicators must mutate `mesh.userData.origMaterial.color` and `mesh.userData.ghostMaterial.color` on that specific mesh only.
5. **Floor-Safe Exploded Teardown (`y >= 0`) & Dashed Centerlines**:
   - Every subsystem explosion vector `explodeVectors[key]` must keep all geometry strictly above the studio plinth (`y >= 0.0`) at `0%–100%` explosion. Never explode bottom subsystems downward (`-Y`).
   - Draw dynamic `THREE.LineDashedMaterial` (`color: 0xc25b2a`) guide lines when `explodeFactor > 0.02`.
6. **Smooth, Non-Jumping Custom Wheel Zoom & Unlocked 360° Orbit**:
   - Set `controls.minAzimuthAngle = -Infinity`, `controls.maxAzimuthAngle = Infinity`, `controls.maxPolarAngle = Math.PI * 0.495`.
   - Set `controls.enableZoom = false;` (to avoid Three.js `r160` `devicePixelRatio | 0` jump bugs) and attach this exact custom wheel handler on `renderer.domElement`:
     ```js
     renderer.domElement.addEventListener('wheel', (e) => {
       e.preventDefault();
       e.stopPropagation();
       if (e.deltaY === 0) return;
       isCameraTweening = false;
       const offset = new THREE.Vector3().subVectors(camera.position, controls.target);
       const dist = offset.length();
       if (dist < 0.001) return;
       const dir = e.deltaY > 0 ? 1 : -1;
       const nextDist = Math.max(controls.minDistance, Math.min(controls.maxDistance, dist * (1 + dir * 0.042)));
       offset.normalize().multiplyScalar(nextDist);
       camera.position.copy(controls.target).add(offset);
       controls.update();
     }, { passive: false });
     ```
7. **Left & Right Divided Callout Columns with Long SVG Leader Lines (Zero Overlap)**:
   - Divide the 7 callout labels into a **Left column (`side: 'left'`, `transform: translate(-100%, -50%)`)** and a **Right column (`side: 'right'`, `transform: translate(0%, -50%)`)** positioned well outside the 3D equipment footprint (`baseLeftColX` and `baseRightColX`).
   - Run a vertical anti-overlap solver (`resolveVerticalSpacing(items, 92, window.innerHeight - 76, 48)`) on each column every frame so labels never overlap each other.
   - Render an `<svg>` layer inside `#callouts-container` drawing long `<polyline>` leader lines and `<circle r="2.6">` anchor dots from each pill's inner edge to the projected 3D part coordinate `(anchorX, anchorY)`.
8. **Consistent Minimal Light-Studio UI Layout**:
   - **Left Sidebar (`280px` fixed white panel):** Domain pill, Equipment title & IEC/SOLAS spec subtitle, `Equipment` header with `Show all`, 7 subsystem isolation buttons with 2-letter mono badges, and bottom `2x2` live telemetry strip.
   - **Top-Left Floating Pill Bar:** `← Vessel view` (`href="../index.html"` in the same tab) and `Kinematic & IEC proof` modal button.
   - **Right Floating Inspector Card (`330px` translucent white card):** Active BOM code (`ASSY-01`), title & description, `View` toggle pills (`Labels`, `Cutaway X-Ray`, `Telemetry`), Camera preset dropdown, operational sliders + `Exploded teardown` slider (`0%` / `100%`), two live HTML5 2D telemetry canvases, and `[ Reset view ] [ + ] [ - ]` footer.