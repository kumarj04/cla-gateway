```mermaid
graph TD
    %% Styling definitions
    classDef user fill:#2ecc71,stroke:#27ae60,stroke-width:2px,color:#fff;
    classDef local fill:#3498db,stroke:#2980b9,stroke-width:2px,color:#fff;
    classDef daemon fill:#e67e22,stroke:#d35400,stroke-width:2px,color:#fff;
    classDef subEngine fill:#f39c12,stroke:#d35400,stroke-width:1px,color:#fff;
    classDef cloud fill:#9b59b6,stroke:#8e44ad,stroke-width:2px,color:#fff;
    classDef terminal fill:#34495e,stroke:#2c3e50,stroke-width:2px,color:#fff;
    classDef log fill:#95a5a6,stroke:#7f8c8d,stroke-width:1px,color:#fff,stroke-dasharray: 5 5;

    %% Workflow Nodes
    Admin[👤 System Administrator]:::user
    
    subgraph LocalHost ["🏡 Local Hardened Host Environment"]
        %% Main execution blocks grouped together down the center
        Wrapper["🛠️ /usr/local/bin/cx<br>(Hardened Wrapper Script)"]:::local
        Daemon["⚙️ Local Gateway Daemon<br>(127.0.0.1:18080)"]:::daemon
        Check1["🔒 4a. Enforces 'tier0-readonly' check"]:::subEngine
        Check2["🧹 4b. Runs localized Regex filters"]:::subEngine
        
        %% Side logging containers to break rendering dependency
        UserLog[("📝 user-map.log<br>(UID, TTY, Hash)")]:::log
        PromptLog[("📝 prompts.log<br>(Sanitized Telemetry)")]:::log
    end

    subgraph RedHat ["☁️ Red Hat Infrastructure"]
        RHCloud["Red Hat Hybrid Cloud Console<br>API Engine"]:::cloud
    end

    Response["💻 System Terminal Response<br>(Verified Linux KB)"]:::terminal

    %% Pipeline Processing
    Admin -->|1. Executes cx 'how do I find...'| Wrapper
    Wrapper -->|3. Redirects to /usr/bin/c| Daemon
    Daemon --> Check1
    Check1 --> Check2
    
    %% Logs split onto separate sides to prevent overlapping connections
    Wrapper --->|2a. Generates prompt_hash| UserLog
    Wrapper --->|2b. Logs user context| UserLog
    Check2 --->|5. Commits sanitized telemetry| PromptLog
    
    %% Outbound delivery path
    Check2 -->|6. Forwards clean prompt via Satellite| RHCloud
    RHCloud -->|7. Resolves query against KB| Response

    %% Explicit rendering constraints to align nodes left-to-right perfectly
    UserLog ~~~ Daemon
    Check1 ~~~ PromptLog
    PromptLog ~~~ RHCloud
```
