
```mermaid
sequenceDiagram
    participant Dev as Developer
    participant GH as GitHub Actions
    participant Blue as Blue Env (Production)
    participant Green as Green Env (Staging)
    participant DNS as DNS/Load Balancer  
    participant Users as End Users
    participant Monitor as Monitoring
    
    Note over Blue: Currently serving 100% traffic
    Note over Green: Idle environment
    
    activate Blue 
    %%% Blue is active as current production
    
    Dev->>GH: Push to main branch (v2.0)
    activate GH 
    %%% GH Actions becomes active
    GH->>Green: 1. Deploy new version to Green
    activate Green 
    %%% Green becomes active for deployment
    Green-->>GH: Deployment successful
    
    GH->>Green: 2. Run automated tests
    Green-->>GH: All tests pass (checkmark)
    
    GH->>Green: 3. Run smoke tests
    Green-->>GH: Smoke tests pass (checkmark)
    
    GH->>Monitor: 4. Health check Green environment
    Monitor-->>GH: Green environment healthy (checkmark)
    
    Note over GH: Pre-switch validation complete
    
    GH->>DNS: 5. Switch 10% traffic to Green
    activate DNS 
    %%% DNS/Load Balancer becomes active
    DNS->>Users: Route 10% requests to Green
    Users->>Green: Limited production traffic
    
    GH->>Monitor: 6. Monitor Green with real traffic
    Monitor-->>GH: Metrics normal (checkmark)
    
    GH->>DNS: 7. Gradually increase to 50%
    DNS->>Users: Route 50% requests to Green
    
    GH->>Monitor: 8. Continue monitoring
    Monitor-->>GH: Performance stable (checkmark)
    
    GH->>DNS: 9. Switch 100% traffic to Green
    deactivate Blue 
    %%% Blue is now inactive for traffic
    activate Green 
    %%% Green becomes fully active for all production traffic
    deactivate DNS 
    %%% DNS/Load Balancer's main routing task is done
    
    Note over Green: Green is now Production
    Note over Blue: Blue becomes new Staging
    
    GH->>Blue: 10. Update Blue for next deployment
    deactivate GH 
    %%% GH Actions completes its main task

    Users->>Green: All production traffic
    Green-->>Users: v2.0 responses
```
