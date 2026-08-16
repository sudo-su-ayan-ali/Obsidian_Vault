
# Phase1
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
 
 
# done
 
---

### Phase 2 Overview: Plain HTTP Proxy (Weeks 2–3)

  Phase 2 focuses on building the networking backend that handles live HTTP traffic, relays messages between
  client browsers and upstream target servers, and feeds traffic data safely into the FTXUI interface.
  ──────
  ### Core Objectives & Steps

  1. Boost.Asio & llhttp Integration
      • Separate threading model: Main thread runs the FTXUI event loop; a background thread runs the Boost.Asio
      io_context event loop.
      • Embed llhttp parser callbacks to inspect HTTP headers, methods, paths, and URLs on the fly.
  2. ProxyServer & Session Handler
      • ProxyServer Listener: Binds a TCP acceptor to a configurable port (default 8080) and accepts incoming
      connections.
      • Session Dual-Socket Handler: Manages two sockets per connection:
          • Client → Proxy
          • Proxy → Target Server
      • Parses incoming request headers to extract the target Host, asynchronously resolves DNS, and streams
      data bidirectionally.
  3. Thread-Safe Queue (TsQueue) for UI Updates
      • Safe producer-consumer architecture bridging networking threads and the UI main thread.
      • Emits custom FTXUI screen events whenever a new HttpTransaction is logged to refresh UI state without
      thread race conditions or freezing.
  4. HTTP Keep-Alive & Robust Error Handling
      • Evaluates Connection: keep-alive headers to reuse existing TCP connections and resets llhttp parser
      states per request.
      • Includes Asio timer-based idle connection timeouts and generates 502 Bad Gateway / 504 Gateway Timeout
      responses when upstream targets fail.
  5. Live History Tab Wiring
      • Replaces the synthetic Phase 1 dummy data with dynamic live traffic logs.
      • Updates the FTXUI list view dynamically as new HTTP requests and responses flow through the proxy.

  ──────
  ### Phase 2 Completion Checklist

  [ ] Proxy server binds to configurable port and accepts TCP connections.
  [ ] HTTP requests are parsed and forwarded to upstream servers.
  [ ] Responses are relayed back to the browser correctly.
  [ ] Keep-alive connections handle multiple sequential requests over the same socket.
  [ ] Captured traffic streams into the History tab in real time.
  [ ] Shutdown cleanly terminates io_context and active sessions.

---

# Phase 3: HTTPS MITM (Weeks 4–6)

[](https://github.com/sudo-su-ayan-ali/BurpTUI/blob/main/phases/phase3.md#phase-3-https-mitm-weeks-46)

## Goal

[](https://github.com/sudo-su-ayan-ali/BurpTUI/blob/main/phases/phase3.md#goal)

The goal of this phase is to intercept and decrypt encrypted HTTPS traffic using a Man-in-the-Middle (MITM) architecture. This requires generating on-the-fly SSL/TLS certificates, managing secure socket layers, establishing dynamic trust bridges between the client and the target server, and tunneling the decrypted traffic through the HTTP proxy pipeline.

## Prerequisites

[](https://github.com/sudo-su-ayan-ali/BurpTUI/blob/main/phases/phase3.md#prerequisites)

Depends on Phase 2. The plain HTTP proxy must be fully operational with the ProxyServer, Session handler, and TsQueue pipeline working end-to-end.

---

## 1. Integrate OpenSSL

[](https://github.com/sudo-su-ayan-ali/BurpTUI/blob/main/phases/phase3.md#1-integrate-openssl)

To handle robust cryptographic operations, certificate generation, and secure TLS handshakes, OpenSSL must be integrated into the build environment.

### Step 1.1: Dependency Management

[](https://github.com/sudo-su-ayan-ali/BurpTUI/blob/main/phases/phase3.md#step-11-dependency-management)

- Update the package manifest (e.g., vcpkg.json) to include OpenSSL as a core dependency.
- Modify the CMake configuration to discover the OpenSSL libraries using standard package finder mechanisms and securely link them against the primary executable targets.
- Ensure that the asynchronous networking framework (Boost.Asio) is configured with its SSL extensions enabled, allowing it to seamlessly wrap raw sockets into TLS streams.

### Step 1.2: Cryptographic Initialization Strategy

[](https://github.com/sudo-su-ayan-ali/BurpTUI/blob/main/phases/phase3.md#step-12-cryptographic-initialization-strategy)

- Implement an application-wide initialization routine that configures the OpenSSL cryptographic algorithms and thread-safety locks at application startup.
- Prepare a dedicated directory on the local filesystem to securely store the generated Root Certificate Authority (CA) keypair and the dynamically minted per-host certificates.

---

## 2. Implement CertGenerator (Root CA & Per-Host Certs)

[](https://github.com/sudo-su-ayan-ali/BurpTUI/blob/main/phases/phase3.md#2-implement-certgenerator-root-ca--per-host-certs)

A MITM proxy requires a Root Certificate Authority installed on the user's OS or browser. The proxy then uses this Root CA to mint trusted certificates dynamically for any domain the client attempts to visit.

### Step 2.1: Root CA Generation and Setup

[](https://github.com/sudo-su-ayan-ali/BurpTUI/blob/main/phases/phase3.md#step-21-root-ca-generation-and-setup)

- Implement a utility routine within the CertGenerator module using OpenSSL programmatic APIs (or an external setup script) to generate a high-entropy RSA or ECDSA private key.
- Generate a self-signed Root CA X.509 certificate using this private key. Ensure it is explicitly marked with the CA:TRUE Basic Constraint.
- The Root CA must be explicitly trusted by the user's system. Detailed installation procedures vary by environment:
    - Linux: Copy the certificate to /usr/local/share/ca-certificates/ (with a .crt extension) and execute the update-ca-certificates command.
    - macOS: Add the certificate to the system Keychain using the security add-trusted-cert command or import it manually via the Keychain Access GUI, marking it as Always Trust.
    - Windows: Import the certificate into the Trusted Root Certification Authorities store using the command certutil -addstore.
    - Firefox: Since Firefox maintains its own certificate store independent of the OS, navigate to Settings > Privacy & Security > Certificates > View Certificates > Import to manually add the Root CA.

### Step 2.2: Dynamic Per-Host Certificate Minting

[](https://github.com/sudo-su-ayan-ali/BurpTUI/blob/main/phases/phase3.md#step-22-dynamic-per-host-certificate-minting)

- Utilize the CertGenerator module to forge server certificates on demand.
- When a client requests a secure connection to a specific domain (e.g., example.com), generate a new private key specifically for that domain.
- Construct a new X.509 certificate, setting the Common Name (CN) and the Subject Alternative Name (SAN) extension to match the requested domain exactly, ensuring browsers accept it.
- Sign this newly minted domain certificate using the local Root CA's private key.
- Serialize both the minted certificate and its corresponding private key so they can be loaded rapidly into an ssl::context.

---

## 3. Build Thread-Safe CertCache

[](https://github.com/sudo-su-ayan-ali/BurpTUI/blob/main/phases/phase3.md#3-build-thread-safe-certcache)

Generating RSA/ECDSA keys and signing certificates is a computationally expensive operation. To prevent severe latency on concurrent HTTPS requests, dynamically generated certificates must be cached and reused.

### Step 3.1: Cache Data Structure Design

[](https://github.com/sudo-su-ayan-ali/BurpTUI/blob/main/phases/phase3.md#step-31-cache-data-structure-design)

- Design an in-memory cache structure (like a Hash Map or Dictionary) where the key is the requested domain string and the value is a structure containing the corresponding ssl::context or the certificate/key pair.
- Implement thread-safe access to this structure using standard synchronization primitives (e.g., read-write locks). This allows multiple session threads to read from the CertCache concurrently, but enforces exclusive locking only when a new certificate is actively being minted and written to the map.

### Step 3.2: Cache Lookup and Generation Workflow

[](https://github.com/sudo-su-ayan-ali/BurpTUI/blob/main/phases/phase3.md#step-32-cache-lookup-and-generation-workflow)

- Before establishing a server-side SSL handshake with the client, query the CertCache with the target domain.
- If the certificate exists (a cache hit), immediately retrieve and apply it to the session's ssl::context.
- If the certificate does not exist (a cache miss), lock the generation pathway, invoke the CertGenerator to mint it, store the new certificate in the CertCache, and then apply it to the session's ssl::context.

---

## 4. Implement MitmSession (CONNECT & ALPN HTTP/1.1)

[](https://github.com/sudo-su-ayan-ali/BurpTUI/blob/main/phases/phase3.md#4-implement-mitmsession-connect--alpn-http11)

The proxy must intercept the initial connection request, establish an encrypted tunnel, negotiate the application-layer protocol, and extract the decrypted traffic stream.

### Step 4.1: Handling the HTTP CONNECT Method

[](https://github.com/sudo-su-ayan-ali/BurpTUI/blob/main/phases/phase3.md#step-41-handling-the-http-connect-method)

- Update the HTTP parser in your session handler to detect the CONNECT method. The CONNECT method indicates the client wants to establish an opaque TCP tunnel to a destination.
- Extract the target hostname and port from the URI field.
- Halt the standard HTTP/1.1 parsing flow. Immediately respond to the client with an HTTP/1.1 200 Connection Established message, signaling to the client that the tunnel is ready for TLS negotiation.

### Step 4.2: TLS Handshake, ALPN Negotiation, and Error Handling

[](https://github.com/sudo-su-ayan-ali/BurpTUI/blob/main/phases/phase3.md#step-42-tls-handshake-alpn-negotiation-and-error-handling)

- Upgrade the client-side TCP socket into an SSL/TLS stream within the MitmSession.
- Utilize the extracted hostname to fetch the correct forged certificate from the CertCache and configure the ssl::context.
- Configure the ssl::context to use Application-Layer Protocol Negotiation (ALPN). Explicitly advertise support only for http/1.1 to force the client to downgrade from HTTP/2 or HTTP/3, simplifying the proxy's parsing requirements while maintaining compatibility.
- Initiate the asynchronous TLS handshake with the client.
- Error Handling: Handshakes may fail due to various reasons, such as the client rejecting the dynamically minted certificate (untrusted Root CA) or a cipher suite mismatch. The MitmSession must handle these TLS handshake failures gracefully. In the event of a failure, the session should cleanly terminate the connection, gracefully degrade where appropriate, log the event with specific failure details, and ensure all resources are safely deallocated without crashing the proxy.

### Step 4.3: Upstream Connection and Decrypted Forwarding

[](https://github.com/sudo-su-ayan-ali/BurpTUI/blob/main/phases/phase3.md#step-43-upstream-connection-and-decrypted-forwarding)

- Simultaneously establish a secure TLS connection to the actual upstream server using a client-side ssl::context.
- Once both the client-facing and server-facing TLS handshakes are complete, the MitmSession effectively holds two decrypted streams in memory.
- Re-attach the HTTP parsers to the decrypted data flowing out of the client stream.
- Implement the asynchronous read/write forwarding loop: read decrypted data from the client stream, parse it for the TUI History tab events, and write it seamlessly into the encrypted upstream socket (and perform the exact inverse for the server's response).

---

## Cross-Cutting Concerns

[](https://github.com/sudo-su-ayan-ali/BurpTUI/blob/main/phases/phase3.md#cross-cutting-concerns)

- **Error Handling**: The system must anticipate and gracefully handle failures during cryptographic operations. This includes certificate generation failures (e.g., OpenSSL internal errors) and TLS handshake failures (e.g., client rejects cert, cipher mismatch). Robust error logging and connection termination strategies are critical to prevent crashes.
- **Security Considerations**: The Root CA private key is highly sensitive. If compromised, it allows attackers to transparently intercept any encrypted communication trusted by the system. The application must store this key securely with restrictive filesystem permissions. Furthermore, users should be explicitly warned about the systemic security implications of installing a MITM Root CA on their machines.

---

## Completion Checklist

[](https://github.com/sudo-su-ayan-ali/BurpTUI/blob/main/phases/phase3.md#completion-checklist)

- [ ]  Root CA certificate and private key are generated and stored locally
- [ ]  Per-host certificates are dynamically minted and signed by the Root CA
- [ ]  CertCache prevents redundant certificate generation across concurrent requests
- [ ]  CONNECT tunnels are established and the proxy responds with 200 Connection Established
- [ ]  ALPN negotiation forces HTTP/1.1 downgrade
- [ ]  Decrypted HTTPS traffic appears in the History tab identically to HTTP traffic
- [ ]  TLS handshake failures are handled gracefully without crashing