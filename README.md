```mermaid
sequenceDiagram
    autonumber
    participant App as App Thread
    participant CL as CL/HIER Progress
    participant Intra as Intra TL (CUDA/CPU)
    participant Net as Inter TL (UCP/SHARP)
    participant S as CUDA Stream (EE)
    participant E as CUDA Event

    App->>CL: ucc_collective_post(req)
    CL->>Intra: post(Intra Phase 1)
    Intra->>S: enqueue memcpyAsync/kernels
    Intra->>E: cudaEventRecord(E_intra_done, S)

    loop progress()
        CL->>E: cudaEventQuery(E_intra_done)
        alt not ready
            CL-->>App: return INPROGRESS
        else ready
            CL->>Net: post(Inter-node AllReduce)
            break
        end
    end

    loop progress()
        CL->>Net: test/progress network
        alt not ready
            CL-->>App: return INPROGRESS
        else ready
            CL->>Intra: post(Intra Phase 2)
            Intra->>S: enqueue memcpyAsync/kernels
            Intra->>E: cudaEventRecord(E_final_done, S)
            break
        end
    end

    loop progress()
        CL->>E: cudaEventQuery(E_final_done)
        alt not ready
            CL-->>App: return INPROGRESS
        else ready
            CL-->>App: complete (UCC_OK)
        end
    end

```
