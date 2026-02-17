# System Architecture 🏗️

This project implements a **Hybrid-Cloud / Local-First** architecture designed to maximize security and privacy while maintaining accessibility.

## Network Topology

```mermaid
graph TD
    subgraph "Public Internet 🌍"
        User([User 📱])
        WhatsApp([WhatsApp API 💬])
    end

    subgraph "Cloud Layer (VPS) ☁️"
        Gateway[OpenClaw Gateway]
        Tailscale1[Tailscale Interface 🔒]
    end

    subgraph "Private Mesh Network (Zero Trust) 🛡️"
        Tunnel[Encrypted WireGuard Tunnel]
    end

    subgraph "Local Layer (Home Network) 🏠"
        Tailscale2[Tailscale Interface 🔒]
        Laptop[Local Server (Ryzen 7)]
        Ollama[Ollama Inference Engine 🦙]
    end

    User -->|Msg| WhatsApp
    WhatsApp -->|Webhook| Gateway
    Gateway -->|HTTP (100.x.y.z)| Tailscale1
    Tailscale1 <-->|WireGuard| Tunnel
    Tunnel <-->|WireGuard| Tailscale2
    Tailscale2 -->|JSON| Ollama
```

## Design Principles

### 1. Zero-Trust Networking

We utilize **Tailscale** (built on WireGuard) to create a private, encrypted mesh network.

- **No Open Ports**: The local laptop does not expose port 11434 (Ollama) to the public internet.
- **Authenticated Peers**: Only devices authenticated to the specific Tailscale tailnet can communicate.
- **NAT Traversal**: Allows the VPS to talk to the laptop behind a residential rigorous NAT without port forwarding.

### 2. Local Data Sovereignty

All "thinking" happens locally.

- **Privacy**: User messages are processed on the local laptop. No text is sent to third-party AI APIs (like OpenAI) for inference.
- **Cost**: Leveraging existing consumer hardware (AMD Ryzen 7 5825U) avoids per-token API costs.

### 3. Asynchronous Processing

Due to the latency of CPU inference (~18-30s), the system validates the webhook immediately (200 OK) to satisfy WhatsApp's timeout requirements, while the OpenClaw gateway manages the long-running inference task asynchronously.
