```mermaid
sequenceDiagram
    actor AdminUser as 👤 System Administrator
    
    box "🛡️ Target Node"
        participant client as 🖥️ Host client (Production Server)
    end
    
    box "🚀 Central Unix Jump Server"
        participant admin as 🛠️ Jump Server admin (Runs cx)
        participant Log1 as 📝 user-map.log (Audit)
    end
    
    box "⚙️ Dedicated Gateway Host"
        participant gateway as 🛡️ Gateway Server gateway (Runs c)
        participant Log2 as 📝 prompts.log (Clean Logs)
    end
    
    participant Cloud as ☁️ Red Hat Cloud Engine
    participant Term as 💻 System Terminal

    AdminUser->>admin: 1. Logged into admin, executes:<br>ssh -q client "tail -20 /var/log/messages" | cx
    activate admin
    
    admin->>client: 2. Requests log snippet (Executes tail)
    client-->>admin: Streams log lines back to stdin
    
    admin->>Log1: 3. Logs admin context & prompt_hash
    admin->>gateway: 4. Redirects data to native binary /usr/bin/c (TCP/18080)
    deactivate admin
    
    activate gateway
    Note over gateway: 5a. Enforces 'tier0-readonly' check
    
    alt Payload Contains Blacklisted Activities
        gateway-->>Term: 5c. Blocks execution & outputs Policy Warning
    else Payload Passes Validation
        Note over gateway: 5b. Runs Regex Filters (Scrubs Host/IP/Keys)
        gateway->>Log2: 6. Commits localized telemetry log write
        gateway->>Cloud: 7. Forwards clean telemetry prompt (mTLS 1.3)
        deactivate gateway
        
        activate Cloud
        Cloud->>Term: 8. Returns response payload
        deactivate Cloud
    end
```
