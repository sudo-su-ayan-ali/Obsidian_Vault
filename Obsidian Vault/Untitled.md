### 1. Setup CMake Project & Integrate FTXUI via vcpkg  
  
 • Project Structure: Set up header/source separation and package manifest for FTXUI dependencies.  
 • CMake Build Configuration: Enforce C++20 standard, import FTXUI modules (Screen, DOM, Component), and configure executable targets.  
 • Toolchain Bootstrapping: Ensure build generation and compilation execute cleanly without missing symbols.  
 ──────  
### 2. Implement Tab Layout Architecture  
  
 Build the core application layout containing four primary functional tabs:  
  
 • Tab Header Component: Interactive navigation bar supporting directional inputs/mouse focus.  
 • View Container Assembly:  
     • Proxy View: Interception controls & traffic status indicators.  
     • History View: Traffic request list + detail inspection split pane.  
     • Repeater View: Manual HTTP request editor & response viewer.  
     • Decoder View: Text manipulation & transform buffer controls.  
 • Viewport Layout: Fixed header title/tabs with a dynamically expanding content body.  
 ──────  
### 3. Build History Tab Dummy List & Split-Pane Layout  
  
 • Synthetic Data Model: Define HttpTransaction structs and populate an in-memory collection with dummy HTTP traffic.  
 • Master List View: Render a scrollable list component for browsing transaction entries (Verb, Path, Status).  
 • Detail Inspector Pane: Render a dynamic inspector showing selected request/response headers and payloads.  
 • Dual-Pane Layout: Horizontal split-pane combining fixed-width list view and flexible detail pane.  
 ──────  
### 4. Implement Global & Contextual Keyboard Navigation  
  
 • Global Keybindings:  
     • Tab / Shift+Tab: Cycle through main tabs (Proxy, History, Repeater, Decoder).  
     • q / Ctrl+C: Gracefully exit the application event loop.  
 • Contextual Focus Navigation:  
     • j / k or Up / Down: Navigate transaction list.  
     • h / l or Left / Right: Switch focus between Master List and Detail Pane.  
  
 ──────  
### Summary Checklist for Phase 1  
  
 [ ] CMake project links against FTXUI cleanly using modern C++20.  
 [ ] Root view renders title header, tab bar, and main viewport.  
 [ ] Active tab switches smoothly via tab bar or keybindings (Tab / Shift+Tab).  
 [ ] History tab displays synthetic data in a scrollable master list.  
 [ ] Detail pane updates dynamically as list selection changes.  
 [ ] Keyboard navigation (j/k, h/l, q) functions seamlessly across panes.