# ARParticlesKit

> A modular, high-performance particles framework for visionOS (RealityKit/RealityView) and SwiftUI 2D overlays. Built around four core parts—**Detection → Trigger → Emissor → Controller**—with a mergeable **Design System** of presets.

---

## Table of Contents

* [Vision & Objectives](#vision--objectives)
* [Architecture Overview](#architecture-overview)
* [Projects & Targets](#projects--targets)
* [Folder Structure](#folder-structure)
* [Quickstart (5 minutes)](#quickstart-5-minutes)
* [Core Concepts](#core-concepts)

  * [Detection](#detection)
  * [Trigger](#trigger)
  * [Emissor (Builder + Runtimes)](#emissor-builder--runtimes)
  * [Controller & Public API](#controller--public-api)
  * [Design System](#design-system)
* [Configuration](#configuration)

  * [Scopes](#scopes)
  * [Budgets & Priorities](#budgets--priorities)
  * [Accessibility](#accessibility)
* [Rendering Paths](#rendering-paths)
* [Performance](#performance)
* [Showcases](#showcases)
* [Testing](#testing)
* [Roadmap & Milestones](#roadmap--milestones)
* [Contributing](#contributing)
* [License](#license)

---

## Vision & Objectives

* **Make immersive particles simple**: a single SwiftUI entry point with a fluent builder, sensible defaults, and great performance out-of-the-box.
* **Bridge ML signals to visuals**: map **SoundAnalysis** and **Vision/CoreML** outputs to particle effects via declarative triggers.
* **Be flexible and extensible**: plug in **external/custom detectors** (async closures/streams), add **custom particle runtimes**, and **merge** user catalogs with the built-in Design System.
* **Ship delightful demos**: provide showcase apps for sound-driven rain/sparkles and multimedia (brightness→sparkles, dusty→dust).

---

## Architecture Overview

```
+-------------------+     +-------------------+     +-------------------+     +------------------+
|   1) Detection    | --> |   2) Trigger      | --> |   3) Emissor       | --> | 4) Controller    |
|  (Sound/Vision/   |     |(Rules/Conditions) |     | (Particle Builder) |     | (Orchestration)  |
|   External Fn)    |     |                   |     |   + Animator       |     |  + Budgeting     |
+-------------------+     +-------------------+     +-------------------+     +------------------+
         |                         |                         |                         |
         | Async Events            | Evaluated Conditions    | Spawn/Update/Recycle    | Life-cycle, Scopes,
         v                         v                         v                         v
     EventBus  ─────────────────────────────────────────────────────────────────>  Immersive State
```

* **Detection**: adapters for SoundAnalysis, Vision/CoreML, and **custom async sources**. Emit normalized **DetectionEvents**.
* **Trigger**: **TriggerPredicates** with confidence thresholds, combinators (AND/OR/NOT), and per-rule rate-limit/debounce.
* **Emissor**: builder/DSL creates **ParticleConfig** and delegates to **ParticleRuntime** implementations (e.g., Rain/Dust/Sparkle/Glow) updated by a single **Animator** and pooled.
* **Controller**: central orchestration (scopes, budget/priorities, routing Detection→Trigger→Actions, lifecycle). SwiftUI modifiers configure it.

---

## Projects & Targets

* **ARParticlesKit** (library): the framework sources.
* **ARParticlesShowcase** (demos): sound & multimedia showcases, plus controls for scopes/intensity/accessibility.
* **ARParticlesKitTests** (tests): unit & smoke tests for core, detection, trigger, emissor, and rendering.

> Build platforms: **visionOS** for RealityView; **iOS 17+** for 2D overlays (SwiftUI Canvas).

---

## Folder Structure

```
ARParticlesKit/
├─ Package.swift
├─ Sources/
│  ├─ ARParticlesKit/
│  │  ├─ Core/
│  │  │  ├─ EventBus.swift
│  │  │  ├─ ParticlesController.swift
│  │  │  ├─ ParticleBudget.swift
│  │  │  ├─ AccessibilityPolicy.swift
│  │  │  └─ (Routing, Scopes, AdaptivePolicy, Backpressure)
│  │  ├─ Detection/
│  │  │  ├─ DetectionSource.swift
│  │  │  ├─ SoundDetection.swift
│  │  │  ├─ VisionDetection.swift
│  │  │  └─ CustomDetection.swift
│  │  ├─ Trigger/
│  │  │  ├─ Trigger.swift
│  │  │  ├─ RuleEngine.swift
│  │  │  └─ TriggerCombinators.swift
│  │  ├─ Emissor/
│  │  │  ├─ Particle.swift
│  │  │  ├─ ParticleBuilder.swift
│  │  │  ├─ ParticleSystems/
│  │  │  │  ├─ RainParticle.swift
│  │  │  │  ├─ DustParticle.swift
│  │  │  │  ├─ SparkleParticle.swift
│  │  │  │  └─ GlowParticle.swift
│  │  │  ├─ Animator.swift
│  │  │  └─ Pooling.swift
│  │  ├─ Render/
│  │  │  ├─ RealityViewAdaptor.swift
│  │  │  └─ Particle2DView.swift
│  │  ├─ DesignSystem/
│  │  │  ├─ ParticleCatalog.swift
│  │  │  ├─ Categories.swift
│  │  │  └─ JSONLoader.swift
│  │  └─ PublicAPI/
│  │     ├─ ParticleImmersiveView.swift
│  │     └─ Modifiers.swift
│  └─ ARParticlesShowcase/
│     ├─ SoundDemo/
│     ├─ MultimediaDemo/
│     └─ Common/Controls.swift
└─ Tests/
   └─ ARParticlesKitTests/
      ├─ CoreTests.swift
      ├─ DetectionTests.swift
      ├─ TriggerTests.swift
      ├─ EmissorTests.swift
      └─ RenderTests.swift
```

---

## Quickstart (5 minutes)

### 1) Install via SPM

Add this package URL in Xcode (File → Add Packages…) and select **ARParticlesKit**.

### 2) Minimal Immersive View (visionOS)

```swift
import SwiftUI
import ARParticlesKit

struct DemoImmersiveView: View {
    var body: some View {
        ParticleImmersiveView()
            .scope([.sound, .multimedia])
            .designSystem(.default) // can be merged later with custom
            .particle("Rain") { b in
                b.style(.rain)
                 .spawnRate(120)
                 .lifetime(2.0...3.0)
                 .opacity(0.6)
                 .trigger(.sound(.class("rain"), minConfidence: 0.8))
            }
            .onAppear { ARParticles.current.controller.start() }
            .onDisappear { ARParticles.current.controller.stop() }
    }
}
```

### 3) 2D Overlay (iOS or visionOS overlays)

```swift
import SwiftUI
import ARParticlesKit

struct HUD: View {
    var body: some View {
        Particle2DView()
            .scope([.sound])
            .particle("Dust") { $0.style(.dust()).spawnRate(20).opacity(0.5) }
    }
}
```

> `ARParticles.current` is a small convenience façade used in the demos to access the shared `ParticlesController`.

---

## Core Concepts

### Detection

* **Built-in**: `SoundDetection` (SoundAnalysis), `VisionDetection` (brightness/tags).
* **Custom**: provide an async source that emits `DetectionEvent`s.

```swift
// Register an external/custom detector (e.g., weather)
ARParticles.current.detectors.register(
    source: .custom(id: "externalWeather") {
        // emit events when appropriate
        .label("rain", confidence: 0.9)
    }
)
```

### Trigger

* Describe **when** to act: `.soundClass("rain", minConfidence: 0.8)`, `.imageTag("bright", minConfidence: 0.7)`, `.feature("loudness", >, 0.6)`, or `.custom(...)`.
* Combine with **AND/OR/NOT** and apply **per-rule debounce/rate limits**.

### Emissor (Builder + Runtimes)

* Fluent **builder** to configure: `spawnRate`, `speed`, `lifetime`, `opacity`, `angle`, `depth`, `priority`, `categories`.
* **Runtimes**: `RainParticle`, `DustParticle`, `SparkleParticle`, `GlowParticle` (or create your own by conforming to `ParticleRuntime`).
* **Animator**: single tick actor; **Pooling**: avoids per-frame allocations.

### Controller & Public API

* The **orchestrator**: manages **scopes**, **budgets/priorities**, routing, lifecycle (`start/stop`), and manual control.
* SwiftUI **modifiers** configure behavior: `.scope()`, `.designSystem()`, `.budget()`, `.accessibility()`.

### Design System

* `ParticleCatalog.default` ships presets with documented defaults.
* **Merge** with user catalogs: `.default + .myCustomSet`.
* Optional **JSON/asset loader** to import presets at runtime.

---

## Configuration

### Scopes

Activate only the inputs you want the app to listen to (e.g., sound-only environments):

```swift
ParticleImmersiveView().scope([.sound]) // image/video triggers won’t fire
```

### Budgets & Priorities

Keep performance stable with global/per-category caps; preempt low-priority effects first.

```swift
.budget(.init(policy: .adaptive(cpuTarget: 0.6),
              perCategoryCaps: [.natural: 400, .magical: 200]))
```

### Accessibility

* **Auto-contrast** adapt particle materials against scene luminance.
* **Reduce Motion** scales spawn rate/speed for sensitive users.

```swift
.accessibility(.autoContrast(minRatio: 4.5))
```

---

## Rendering Paths

* **RealityViewAdaptor** (visionOS): bridges particle runtimes to RealityKit `Entity` trees; camera-relative emitters; depth bands.
* **Particle2DView** (SwiftUI): Canvas + TimelineView for HUDs and non-immersive overlays.

---

## Performance

* **Single Animator Tick**: one scheduler updates all particles using delta time.
* **Pooling & Batching**: avoid entity churn; reuse memory and node graphs.
* **Adaptive Policy**: throttles spawn/lifetime to meet `cpuTarget` with hysteresis to avoid oscillations.
* **Backpressure**: per-rule debounce and global rate limiting to prevent action storms.

---

## Showcases

* **Sound Immersive**: `rain` → RainParticle; `ambient_music` → SparkleParticle.
* **Multimedia**: `bright` → SparkleParticle; `dusty` → DustParticle.
* **Custom Detection**: external weather feed emits `rain` label.
* **Controls**: UI toggles for scopes, intensity, and reduce-motion.

---

## Testing

* **Core**: RuleEngine, EventBus, Budget (high coverage; deterministic seeds).
* **Detection**: fixtures for labels/tags; confidence thresholds and sampling.
* **Trigger**: truth tables for AND/OR/NOT; debounce/rate-limit behavior.
* **Emissor**: deterministic sequences under seeded randomness.
* **Render**: smoke tests for spawn/update; snapshot checks for 2D when feasible.

---

## Roadmap & Milestones

* **M1 — MVP Engine + Sound Demo**

  * EventBus, Events, RuleEngine, Builder, Animator, Pooling, Budget
  * RealityView adaptor, Rain preset, SoundDetection & permissions
  * Triggers + mapping, Controller lifecycle, SwiftUI entry points, Quickstart
* **M2 — Multimedia + 2D + Adaptive Perf**

  * VisionDetection + frame pipeline, Sparkle preset, Particle2DView
  * Batching + telemetry, perf harness, adaptive throttling
* **M3 — Design System + Extensibility + Polish**

  * Catalog presets & categories, Merge API, JSON loader
  * CustomDetection demo, Accessibility polish, CI & docs

> Detailed, Notion-ready CSV backlogs are available per area (Core, Detection, Trigger, Render, Controller, DesignSystem, etc.).

---

## Contributing

1. Fork and create a feature branch (`feature/area-shortname`).
2. Add tests and docs for any new feature.
3. Run CI locally (build + tests + coverage).
4. Open a PR with a concise summary and screenshots/clips for visual changes.

Code style: Swift 6 concurrency-first; avoid per-frame allocations; favor value types for configs.

---

## License

TBD. Include license text here (e.g., MIT/Apache-2.0).
