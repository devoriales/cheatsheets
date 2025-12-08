# Your Blog Post Title

Your content here...

## Visualizing the Relationship

```mermaid
graph TB
    CRD[Custom Resource Definition]
    CR[Custom Resource Instance]
    Controller[Controller/Operator]
    
    CRD -->|defines| CR
    CR -->|watched by| Controller
```

