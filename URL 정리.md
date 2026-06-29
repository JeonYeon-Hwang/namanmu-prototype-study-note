**기본 골격 구현 사항**  
```
http://localhost:5173/requirements
http://localhost:5173/builds/result
http://localhost:5173/builds/change-part
http://localhost:5173/my-quotes
http://localhost:5173/parts
http://localhost:5173/support
http://localhost:5173/support/tickets/sample
http://localhost:5173/login
http://localhost:5173/signup
``` 
<br>  

**테이블 관계 파악하기: 스키마**  
```mermaid
erDiagram
  parts ||--o{ price_snapshots : "part_id"
  parts ||--o{ price_alerts : "part_id"
  parts ||--o{ benchmark_summaries : "part_id"

  users ||--o{ price_alerts : "user_id"
  users ||--o{ price_jobs : "requested_by"

  parts {
    bigint id PK
    uuid public_id
    varchar category
    varchar name
    varchar manufacturer
    integer price
    varchar status
    jsonb attributes
  }

  price_snapshots {
    bigint id PK
    bigint part_id FK
    integer price
    varchar source
    timestamptz collected_at
    jsonb raw_payload
  }

  price_alerts {
    bigint id PK
    bigint user_id FK
    bigint part_id FK
    integer target_price
    varchar status
    timestamptz triggered_at
  }

  price_jobs {
    bigint id PK
    uuid public_id
    bigint requested_by FK
    varchar status
    timestamptz started_at
    timestamptz finished_at
    text error_summary
  }

  compatibility_rules {
    bigint id PK
    varchar rule_key
    varchar category
    jsonb condition
    varchar result_status
    text message
  }

  benchmark_summaries {
    bigint id PK
    bigint part_id FK
    varchar benchmark_key
    text summary
    numeric score
    jsonb metadata
  }
```
<br>

**검증 Tool 엔진**  
| 항목 | 직접 구현 가치 |
| :--- | :--- |
| 소켓 호환성 검사 | CPU socket ↔ Motherboard socket |
| RAM 규격 검사 | DDR4/DDR5 호환 |
| GPU 길이 검사 | GPU length ↔ Case max GPU length |
| PSU 전력 검사 | CPU/GPU 소비전력 + 여유율 |
| PASS/WARN/FAIL 판정 | 룰 기반 결과 생성 |
| evidence 생성 | 어떤 룰로 판단했는지 반환 |

<br>  

**소켓 호환성 검사**  

