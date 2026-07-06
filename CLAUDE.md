# WordMesh — Execution Spec (CLAUDE.md)

Read this whole file before touching code. Strategy is LOCKED (2026-07-05). Do not reopen "Locked decisions" — execute them.

## What this is
A live, walkable 3D network of the Superhero.com (aeternity) word-token economy. WebXR, one link, runs on desktop / mobile touch / Quest VR / AR passthrough. Word-tokens are glowing nodes (size/brightness = Trendminer price/volume). Creators and `.chain` users are nodes. Real buys/tips travel as light-pulses down lines between nodes. Erlang aesthetic: message-passing mesh that self-heals when nodes drop.

## Locked decisions — do not reopen
1. Live network floor, not ambient drift. The mesh visibly reacts to trade events. Nodes flare, dim, lines reroute.
2. No audio mechanics. No calls. Default silent. Erlang is expressed structurally, never as telephony.
3. Two distinct verbs, never conflated: BUY a word (Trendminer bonding-curve market) vs TIP a post/creator (tipping contract, social gift). Words are not tipped. Buys thicken/brighten word-nodes; tips draw person→person support-lines.
4. Non-custodial, reads-first. Rendering runs on free reads (REST/WS/node). Writes (buy/tip) ONLY via Superhero Wallet handoff (QR to phone / deep-link). Never hold keys. Writes are phase 3.
5. Data-adapter seam. All data flows through a `DataSource` interface. `MockSource` and `SuperheroSource` interchangeable. Engine chain-agnostic.
6. Replay layer. Real live volume is tiny (<$1k/day). Under the live WS feed, run a time-compressed replay of the last ~7 days of real trades, clearly labeled (dimmer/cooler pulses). The room must always breathe.
7. Presence privacy line (phase 2): other users appear as callsign (.chain name) + words they explicitly claim. NEVER visualize holdings/portfolio.
8. Single-file phase 0. One `index.html`, Three.js from CDN, GitHub Pages. No build step until it hurts.
9. Human nodes are embodied and mobile. `.chain` users = pill-shaped avatars with cut-sphere faces (Nostr XR stage style) + floating callsign. Locomotion: walk/run on an implied plane AND free flight (fly default in a 3D mesh; smooth locomotion + snap-turn in VR, WASD+mouse desktop, touch+gyro mobile).
10. Free to roam, paid to interact (phase 4). Seeing/walking/flying/discovering always free. Economic interactions cost AE, settle on-chain non-custodially: payload-tagged spend txs to dev `.chain` wallet (client verifies via middleware read; chain is the store, no backend accounts), plus one small Sophia split-contract for avatar gifts (~95% recipient / ~5% dev). Never paywall movement or visibility.

## Architecture
- Client: static, GitHub Pages. Three.js + WebXR. Mode switcher: desktop (orbit/WASD) / mobile (touch+gyro) / VR (Quest) / AR passthrough. Use webxr-threejs + threejs-visual-polish skills if available (guidance, not deps).
- Backend: none until forced. Railway proxy ONLY if CORS check fails. Secrets never in client.
- Data endpoints (from superhero-com/superhero config, verified 2026-07): SUPERHERO_API_URL (Trendminer REST), SUPERHERO_WS_URL (live trades), NODE_URL / MIDDLEWARE_URL (.chain resolution, on-chain reads), WALLET_URL (handoff), PROFILE_REGISTRY_CONTRACT_ADDRESS. Phase 1 first task: capture concrete URLs + CORS headers from superhero.com devtools.

### DataSource interface (keep this exact seam)
```js
// getSnapshot(): Promise<{tokens: [{word, price, volume24h, holders?, candles?}]}>
// subscribe(cb): live events {type: 'buy'|'sell'|'tip', word?, from?, to?, amount, ts}
// getReplay(days): Promise<event[]>
```
MockSource generates plausible data + Poisson-timed events. SuperheroSource implements the same against REST/WS. Swap via `?data=mock|live`.

## Performance budget (Quest = floor)
72fps Quest. Word-nodes via InstancedMesh (one draw call), cap ~300 visible, LOD: distant nodes = point sprites, no labels. All lines in one LineSegments buffer, rebuilt at most 1x/s; pulses are shader/uv animations. Labels: billboarded canvas textures, only within ~8m, pooled. No per-frame allocations in the render loop.

## Phases + acceptance
### Phase 0 — mock mesh demo (ship same-day)
Single index.html on Pages. MockSource. ~150–300 word-nodes in a 3D constellation around spawn, related-word lines, event pulses every few seconds, node flare/dim, and the Erlang beat: every ~60s one node dies and the mesh visibly reroutes. HUD: project name + "MOCK DATA" badge + mode switcher.
Done when: loads from Pages link; 72fps in Quest browser; usable on desktop mouse + phone touch; a 30s capture looks alive.
### Phase 1 — real data
Devtools probe → record real endpoints + CORS verdict here. SuperheroSource (REST snapshot → sizes; WS → live pulses; history → replay). Railway proxy only if CORS forces. Live pulses distinct from replay.
### Phase 2 — identity & presence
.chain resolve → embodied avatar (pill body, cut-sphere face, callsign, walk/run/fly), creator nodes, bot-presence then LiveKit behind the seam. Respect privacy line.
### Phase 3 — writes
Select node → QR → Superhero mobile app deep-link → buy/tip. Client never signs. Confirmation → pulse.
### Phase 4 — in-world economy
Free-to-roam absolute. Paid: cosmetics (`wm:glow:7d`), beacons (`wm:beacon:<word>:<ttl>`), orbit slots, avatar gifts (Sophia 95/5). Moderation on user text. Chain is the store.
### Phase 4+ revenue
B2B placements, weekly auctions, market-night events, semantic ad inventory (affiliate, Zesty programmatic, direct placements.json, brand-owned words with GLTF gated to verified owners). Ad integrity: every ad labeled; ad money never affects node size/brightness/position; no ads on avatars/.chain identities; billboards excluded from pulse grammar.

## Session workflow
- Sonnet phase 0–1; Opus for Quest perf / phase-2 presence / stubborn data. Strategy settled.
- Read this file first, check Status log, do the next unchecked criterion, update the log. Verify on-device claims honestly.

## Status log
- 2026-07-05: Strategy locked. Docs created. Repo: github.com/developerofwebxr-oss/wordmesh (Pages enabled). Endpoints unprobed. Phase 0 not started.
- 2026-07-05: Phase 0 shipped. Single index.html: MockSource (Poisson trades + ~60s death/respawn beat), 220 InstancedMesh word-nodes with k-NN related-word edges (single LineSegments buffer), pooled InstancedMesh pulses, node flare/dim, LOD (near = instanced spheres, far = point sprites), pooled billboarded labels (~8m), starfield backdrop, ACES tone mapping + bloom (flat only) + fog, desktop hold-drag+WASD, mobile touch+joystick+gyro, VR (smooth move + 30° snap-turn) and AR (passthrough, shell suppressed) via raw WebXR session lifecycle with sessionend cleanup. Verified in a Chrome tab via extension scripting: scene graph populates correctly, mesh/edges/pulses/flares render correctly, HUD (title, MOCK DATA badge, mode switcher, Fly/Free-look toggles) renders correctly. Fixed two real bugs found during verification: (1) AR sessions weren't clearing scene.background/fog, which would have blocked passthrough; (2) the far-LOD point layer's unclamped additive blending could blow out under many overlapping distant nodes — clamped and switched to normal blending. Untested/unverifiable this session: real Quest fps (no headset available), real phone touch/gyro (no device available). The automation browser tab used for self-verification runs backgrounded (`document.visibilityState: 'hidden'`), which freezes `requestAnimationFrame` — screenshots were produced by forcing one real synchronous render per capture rather than reading the live animation loop; this is a testing-harness limitation, not app behavior, and is worth reconfirming once on a normal foregrounded tab/device.

- 2026-07-06: Phase 0 movement/legibility polish, from on-device feedback ("movement feels dingy and off", "mesh reads as anonymous dots"). Movement root causes fixed: no velocity smoothing (instant snap start/stop instead of glide) — added frame-rate-independent exponential smoothing (`1-exp(-dt/0.15)`); base/sprint speed too slow for the field scale (1.4/2.8 → 5.0/11.0, VR 3.6/7.5); digital WASD diagonal moved at sqrt(2)x speed — clamped combined input to unit magnitude; joystick had no deadzone/curve, so tiny touch jitter caused drift — added a 0.12 deadzone + quadratic ease-in applied radially; gyro free-look set yaw/pitch from *absolute* compass heading/tilt, which snapped the view to an unrelated orientation the instant it was enabled and drifted independently of how far you'd actually turned — rewritten to calibrate a baseline on enable and apply *delta* rotation only (up/down sign is a best-effort guess, unverified on real hardware); there was no dedicated vertical axis at all (only pitch-and-fly-forward) — added Space/Ctrl on desktop and two on-screen ascend/descend buttons on mobile, decoupled from look direction. Labels: added a second, larger, persistent label pool (20 slots) for the top-volume words, ranked/reassigned every 2.5s (not per frame), scaling up with distance (clamped) so they stay legible across the mesh and fading in/out near the camera and near the fog line; smaller nodes keep the original ~8m proximity-only pool, now skipping any node already carrying a persistent label. Verified via Chrome extension scripting (same backgrounded-tab caveat as above — forced renders, not the live loop): top labels render correctly ranked by volume with distance-based scale/fade, mobile joystick + new vertical buttons lay out without overlap. Movement feel itself (smoothing, speed, deadzone) verified by re-deriving the shipped formulas and checking them at representative inputs, not by feel — genuinely confirming "glide" quality needs an on-device pass.

## Facts that shape tone/claims (verified 2026-07)
- Business model: monthly sponsorship, NOT grants/bounties. Aeternity/Superhero sponsor #1. Pitch after phase 0 ships. Never grant/bounty language.
- Each phase doubles as a monthly deliverable. Superhero case exclusively. AE ≈ $0.006–0.008, volume <$1k/day (hence replay). Superhero mobile app newly launched; QR handoff stars it; store links carry attribution.

## Sponsor-reporting instrumentation
- Phase 0+: tag outbound store/app links `?ref=wordmesh`; privacy-light visit counter. Phase 2: register project `.chain`. Phase 4: economy revenue report. Every phase ships a 20–30s clip.
