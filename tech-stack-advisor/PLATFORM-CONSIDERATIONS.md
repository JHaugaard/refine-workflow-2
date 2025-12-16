# Platform Considerations

Reference material for tech-stack-advisor when project requirements suggest non-web platforms.

**Scope:** Apple ecosystem only, personal device deployment (no App Store)

---

## Detection Triggers

Surface platform considerations when the brief contains these signals:

| Signal | Implication |
|--------|-------------|
| "mobile app", "phone", "on the go" | Mobile platform likely relevant |
| "iOS", "iPhone", "iPad", "Apple Watch" | Native Apple (SwiftUI) |
| "desktop app", "Mac app", "menubar" | Desktop platform consideration |
| "offline", "works without internet" | PWA, native, or desktop with local storage |
| "notifications", "push alerts" | Platform-specific capabilities matter |
| "camera", "GPS", "sensors", "health data" | Native APIs likely needed |
| "system tray", "background process" | Desktop application (Tauri or native) |
| "installable", "feels like an app" | PWA or native |

**No triggers detected?** Default to web-first recommendation. Web covers the broadest use cases and aligns with the user's existing skill trajectory.

---

## Platform Decision Matrix

### Web-First (Default)

**Recommend when:**
- No specific platform signals in brief
- "Accessible from anywhere" mentioned
- Learning web technologies is a goal
- MVP or prototype phase
- Single-user, private access (workflow default)

**Options spectrum:**
- Static site (Astro, HTML/CSS/JS) — simplest
- SPA (React, Vue, Svelte) — interactive
- Full-stack (SvelteKit, Next.js, Remix) — SSR, routing, API
- PWA enhancement — adds installability, offline capability

**Learning opportunity:** Foundation skills transferable everywhere.

---

### PWA (Progressive Web App)

**Recommend when:**
- Mobile/desktop presence desired without native complexity
- Offline capability needed but full native APIs not required
- "Installable" experience valued
- Avoiding platform-specific development

**What it provides:**
- Install to home screen / dock
- Offline support via service workers
- Push notifications (with limitations)
- Runs in own window (no browser chrome)

**Trade-offs:**
- Limited device API access compared to native
- iOS has some PWA limitations (no push on older iOS)
- Still fundamentally a web app

**When to upgrade to native:** When you need HealthKit, HomeKit, Shortcuts, widgets, or deep OS integration.

---

### Native iOS/iPadOS (Swift/SwiftUI)

**Recommend when:**
- Brief explicitly mentions iPhone/iPad use
- Device capabilities required (HealthKit, HomeKit, camera with advanced features)
- Best-in-class iOS UX is priority
- Learning Apple development is a goal
- Touch-first interaction patterns needed

**Deployment:** Xcode → device (USB/Wi-Fi). No App Store.

**Trade-offs:**
- Apple-only (but that's in scope)
- Xcode/macOS required for development
- New language (Swift) — but Claude Code handles implementation

**Learning opportunity:** SwiftUI skills transfer to macOS, watchOS, tvOS.

---

### Native macOS (SwiftUI/AppKit)

**Recommend when:**
- Brief mentions Mac app, menubar app
- Deep macOS integration (menu bar, services, Finder extensions)
- System-level access needed (file system, background processes)
- Apple silicon optimization matters

**Deployment:** Run directly from Xcode, or build and run locally.

**Trade-offs:**
- Mac-only
- Same learning curve as iOS (but skills transfer)

---

### Tauri (Cross-Platform Desktop)

**Recommend when:**
- Desktop app needed
- Web skills should transfer
- Windows/Mac/Linux coverage desired
- Smaller bundle size valued over maximum ecosystem

**What it is:** Desktop app framework using system webview + Rust backend. Build your UI with web technologies (HTML/CSS/JS, React, Svelte, etc.), get native desktop capabilities.

**Trade-offs:**
- Rust backend (Claude Code can handle, but different from pure web)
- System webview means slight rendering differences across platforms
- Smaller ecosystem than Electron

**When to choose Tauri over native macOS:** When you want cross-platform, or prefer web UI development.

---

## Platform in Recommendation Output

When platform considerations are relevant, include in the recommendation:

1. **Primary platform** with rationale tied to brief requirements
2. **Platform alternatives** with trade-offs
3. **Expansion path** — how the choice enables or constrains future platforms
4. **Learning framing** — what platform skills this builds toward

Example framing:
> "For a personal finance tracker used primarily on your phone with offline access, I recommend **SwiftUI for iOS**. This gives you native performance, offline-first with SwiftData, and deep iOS integration.
>
> **Alternative:** A PWA would work across all devices and avoid Xcode setup, but the mobile experience would be less polished and offline support more complex.
>
> **Expansion path:** SwiftUI skills transfer directly to macOS and watchOS."

---

## Platform Field in JSON Handoff

When platform is relevant, include in tech-stack-decision.json:

```json
"platform": {
  "primary": "web | pwa | ios | macos | tauri",
  "rationale": "Why this platform fits the project",
  "alternatives_noted": ["pwa", "web"],
  "native_capabilities_needed": ["offline", "healthkit"] // or empty array
}
```

If no platform signals detected, omit the platform field entirely (web is assumed default).

---

## What This Document Does NOT Cover

- **Deployment specifics** — Device deployment walkthrough (deploy-guide's domain)
- **Tool installation/setup** — Xcode setup, Tauri CLI (project-spinup's domain)
- **Implementation details** — How to use SwiftUI, Tauri APIs (Claude Code knows this)
- **Android** — Out of scope (Apple ecosystem only)
- **App Store** — Out of scope (personal device deployment only)

This document informs the **recommendation decision**, not the implementation.
