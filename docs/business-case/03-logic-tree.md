# Logic Tree

```text
Service Quality Performance
|-- Is the issue driven by delivery performance?
|   |-- On-time delivery
|   |-- Transit delay
|   `-- SLA compliance
|-- Is the issue driven by network operations?
|   |-- Origin checkpoint
|   |-- Gateway checkpoint
|   `-- Destination checkpoint
|-- Is the issue driven by operational incidents?
|   |-- Gateway congestion
|   |-- Customs delay
|   |-- Capacity shortage
|   |-- Data entry error
|   `-- Weather / force majeure
|-- Is customer experience affected?
|   |-- CSI score
|   `-- Delay impact on CSI
`-- Are resources and service mix aligned?
    |-- Cost per move
    |-- Service level
    `-- Destination region
```

This issue tree turns the business problem into testable analytical hypotheses. Each branch guides SQL investigation and Power BI dashboard design.
