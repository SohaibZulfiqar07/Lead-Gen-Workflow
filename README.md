# Automated Lead Generation & AI Outreach Workflow

This n8n workflow automates the process of sourcing B2B leads from job boards (Greenhouse, Lever) and Product Hunt, standardizing the data, writing highly personalized cold outreach drafts using AI (Groq/LLaMA 3.1), and storing the final data in Google Sheets. 

## 📊 Workflow Architecture

```mermaid
flowchart TD
    %% Trigger
    Trigger([🕒 Schedule Trigger: 4 AM Daily])

    %% Product Hunt Branch
    Trigger --> PH_API[📦 Fetch Product Hunt Posts]
    PH_API --> PH_FMT[⚙️ Format Maker Data]

    %% ATS Job Board Branch
    Trigger --> GSheet_Settings[📄 Read Settings & Targets via Google Sheets]
    GSheet_Settings --> Split[✂️ Split Target Rows]
    Split --> If_ATS{ATS Type?}
    
    If_ATS -->|Greenhouse| GH_API[🏢 Greenhouse API]
    If_ATS -->|Lever| LV_API[🏢 Lever API]
    
    GH_API --> Job_FMT[⚙️ Format Job Data]
    LV_API --> Job_FMT
    
    %% Merging & Processing
    Job_FMT --> Merge((🔄 Merge Data Streams))
    PH_FMT --> Merge
    
    Merge --> Dedup[🗑️ Remove Duplicates by URL]
    Dedup --> Limit[🛑 Apply Daily Limit from Settings]
    
    %% AI Outreach & Filtering
    Limit --> Wait[⏳ Wait 1s ]
    Wait --> AI[🧠 Generate Outreach Draft via Groq LLaMA-3.1]
    Limit --> MergeAI((Combine with Drafts))
    AI --> MergeAI
    
    MergeAI --> Read_Leads[📊 Read Existing Leads]
    Read_Leads --> Filter_Existing[🚫 Filter Out Existing URLs]
    Filter_Existing --> Set_Fields[📝 Set Default Fields & Status]
    Set_Fields --> Append[(💾 Append to Google Sheets)]

    classDef sheet fill:#0f9d58,stroke:#fff,stroke-width:2px,color:#fff;
    classDef api fill:#4285f4,stroke:#fff,stroke-width:2px,color:#fff;
    classDef ai fill:#ab47bc,stroke:#fff,stroke-width:2px,color:#fff;
    classDef core fill:#f4b400,stroke:#fff,stroke-width:2px,color:#000;
    
    class GSheet_Settings,Read_Leads,Append sheet;
    class PH_API,GH_API,LV_API api;
    class AI ai;
    class Trigger,Merge,MergeAI,Dedup,Limit core;
