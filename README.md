```mermaid
flowchart LR
    subgraph G1[CUDA Graph A (captured)]
        C1[Upstream GPU compute] --> I1[Intra-node phase 1<br/>(GPU)]
        I1 --> M1[Record marker<br/>cudaEventRecord or stream write-value]
    end

    M1 --> CPU[CPU-driven network phase<br/>(UCX/IB/Ethernet or SHARP)<br/>NOT capturable]

    CPU --> M2[Inject resume marker<br/>cudaEventRecord / stream signal]

    subgraph G2[CUDA Graph B (captured)]
        W[Wait on marker<br/>cudaStreamWaitEvent / wait-value] --> I2[Intra-node phase 2<br/>(GPU)]
        I2 --> C2[Downstream GPU compute]
    end

    M2 --> W

```
