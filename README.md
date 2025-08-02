```mermaid
---
config:
  layout: dagre
  theme: redux
  look: handDrawn
---
flowchart TB
 subgraph PackerZone["📦 Packer 作業ゾーン / Packer Zone"]
    direction TB
        B["ピッキング待ち<br>Pending"]
        A(["Shopify 新規注文<br>New Order"])
        C["ピッキング<br>Picking"]
        D[/"ピッキング完了<br>Picked"/]
        F["梱包開始<br>Start Packing"]
        F1["寸法と重量を入力<br>Enter Dimensions &amp; Weight"]
        F2["PlusShipping/B2で伝票発行<br>Create Label"]
        PE1["🚨 Packer: Exception処理<br>Packer Exception Handling"]
        PC1["🛑 Packer: タスク中止<br>Packer Task Cancellation"]
  end
 subgraph WebhookSystem[" "]
    direction TB
        WH2["タスク状態を自動更新<br>Updates Task Status"]
        WH1["Webhook取得運単番号<br>Webhook Gets Tracking Number"]
  end
 subgraph CorrectionLoop["修正ループ / Correction Loop"]
    direction LR
        L["修正モードへ<br>Enter Correction Mode"]
        J{"検査結果<br>Inspection Result"}
        M{"修正内容判断<br>Determine Issue"}
        N["再ピッキング<br>Repick"]
        O_decision{"送料に影響？<br>Affects Cost?"}
        O_void["古い伝票を廃棄<br>Void Old Label"]
        O_repack["再梱包・再発行<br>Repack &amp; Relabel"]
        O_reuse["再梱包（伝票再利用）<br>Repack (Reuse Label)"]
        P["送り状番号入力<br>Record Shipping Label"]
        Q["検査スキップ<br>Skip Inspection"]
        SL["封緘・伝票貼付<br>Final Seal &amp; Labeling"]
  end
 subgraph InspectorZone["🔎 Inspector 作業ゾーン / Inspector Zone"]
    direction TB
        G[/"Packing 完了<br>Packed"/]
        I["検査中<br>Inspecting"]
        H("検査開始<br>Start Inspection")
        K(["出荷完了<br>Completed"])
        CorrectionLoop
        IE1["🚨 Inspector: Exception処理<br>Inspector Exception Handling"]
        IC1["🛑 Inspector: タスク中止<br>Inspector Task Cancellation"]
  end
 subgraph ExceptionHandling["🚨 例外処理<br>Exception Handling"]
    direction TB
        E3["修正不可<br>Unfixable"]
        T["Inspector: 例外処理<br>Exception Process"]
        U["タスクプールに戻す<br>Return to Pending"]
        Cancelled["Cancelled<br>キャンセル済み"]
  end
    A --> B
    B --> C & PE1 & PC1
    C --> D & PE1 & PC1
    D --> F
    F --> F1 & PE1 & PC1
    F1 --> F2
    WH1 --> WH2
    H --> I & IE1 & IC1
    I --> J & IE1 & IC1
    J -- ✅ 合格 / Pass --> SL
    SL --> K & IC1
    J -- ❌ 修正必要<br>Needs Correction --> L
    L --> M & IE1 & IC1
    M -- ピッキングエラー<br>Picking Error --> N
    M -- 梱包エラー<br>Packing Error --> O_decision
    O_decision -- あり / Yes --> O_void
    O_void --> O_repack
    O_decision -- なし / No --> O_reuse
    N --> O_decision
    O_repack --> P
    O_reuse --> P
    P --> Q
    Q --> SL
    M -- 修正不可<br>Unfixable --> E3
    E3 --> T
    T -- 問題解決 / Resolved --> U
    T -- キャンセル / Cancel --> Cancelled
    U --> B
    F2 --> WH1
    WH2 --> G
    G --> H & PC1
    F2 -.-> V["🔓 Packer 次の作業へ<br>Packer Free to Start Next Task"]
    PE1 --> E3
    PE1 -.-> V
    PC1 --> Cancelled
    PC1 -.-> V
    IE1 --> E3
    IE1 -.-> V3["🔓 Inspector 次の作業へ<br>Inspector Free to Process Next Task"]
    IC1 --> Cancelled
    IC1 -.-> V3
     B:::pending
     C:::process
     D:::picked
     F:::process
     F1:::process
     F2:::process
     PE1:::except
     PC1:::cancelled
     WH2:::webhook
     WH1:::webhook
     L:::correction
     J:::process
     M:::correction
     N:::picked
     O_decision:::correction
     O_void:::correction
     O_repack:::picked
     O_reuse:::picked
     P:::barcode
     Q:::correction
     SL:::process
     G:::packed
     I:::process
     H:::process
     K:::completed
     IE1:::except
     IC1:::cancelled
     E3:::except
     T:::process
     U:::pending
     Cancelled:::cancelled
     V:::unlock
     V3:::unlock
    classDef pending fill:#fff2cc,stroke:#d6b656,stroke-width:2px
    classDef picked fill:#d5e8d4,stroke:#82b666,stroke-width:2px
    classDef packed fill:#dae8fc,stroke:#6c8ebf,stroke-width:2px
    classDef inspecting fill:#e1d5e7,stroke:#9673a6,stroke-width:2px
    classDef completed fill:#d4edda,stroke:#28a745,stroke-width:3px
    classDef except fill:#f8d7da,stroke:#dc3545,stroke-width:2px
    classDef process fill:#f0f0f0,stroke:#666666,stroke-width:1px
    classDef unlock fill:#fff3cd,stroke:#ffc107,stroke-width:2px,stroke-dasharray: 5 5
    classDef correction fill:#ffd6cc,stroke:#ff6600,stroke-width:2px
    classDef barcode fill:#e6ffe6,stroke:#009900,stroke-width:2px
    classDef webhook fill:#e8f5e8,stroke:#2e7d32,stroke-width:2px
    classDef cancelled fill:#e9ecef,stroke:#6c757d,stroke-width:2px
    style WebhookSystem stroke:none,fill:transparent
```
