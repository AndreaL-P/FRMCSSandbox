```mermaid
sequenceDiagram
    Title Transport Domain Change with active MCx session
    %%{ init: { 'themeVariables':{ 'sequenceNumberFontSize':'13' } } }%%
    %% === Phase 1 : Établissement session MCx via HPLMN A ===
    participant UE_A as UE A
    participant gNB_A as gNodeB A (HPLMN A)
    participant AMF_A as AMF (HPLMN A)
    participant SMF_A as SMF (HPLMN A)
    participant IMS as IMS/SIP Proxy (HPLMN A)
    participant SCC_AS as SCC AS
    participant MCX_AS as MCx Server
    participant UE_B as UE B

    Note over UE_A,UE_B: **Phase 1: Establishment of MCx session via HPLMN A**

    %% RAN - Radio Connection + 5G Access
    UE_A->>gNB_A: RRC Connection Request/Setup/Complete (TS 38.331)
    gNB_A->>AMF_A: NGAP Initial UE Message (TS 38.413)

    %% NAS - Authentication and Registration
    UE_A->>AMF_A: Registration Request (NAS, TS 24.501 5.5.1.2)
    AMF_A->>UE_A: Authentication, Security (NAS)
    AMF_A->>UE_A: Registration Accept

    %% NAS - PDU Session for Data
    UE_A->>AMF_A: PDU Session Establishment Req (NAS, TS 24.501 6.4.1.2)
    AMF_A->>SMF_A: Create Session (Nsmf-)
    SMF_A->>gNB_A: Path Setup

    %% RAN
    gNB_A->>UE_A: PDU Session Activation

    %% IMS/SIP - Establishment of MCx Voice Service
    UE_A->>IMS: SIP REGISTER (TS 24.229, TS 33.203)
    IMS->>UE_A: SIP 200 OK
    UE_A->>MCX_AS: SIP INVITE (via IMS, TS 23.280 Annex B)
    MCX_AS->>UE_B: SIP INVITE
    UE_B->>MCX_AS: SIP 200 OK
    MCX_AS->>UE_A: SIP 200 OK
    Note over UE_A,MCX_AS: **MCx session active, anchored in SCC AS (TS 23.237 5.3.1)**

    UE_A->>SCC_AS: Establishment of SCC AS anchor (via IMS)

    %% === Phase 2: Inter-HPLMN HO to HPLMN B, maintaining MCx session ===

    Note over UE_A,MCX_AS: **Phase 2: Inter-HPLMN HO, maintaining MCx session**

    %% RAN - Measurement Report and HO Decision
    UE_A->>gNB_A: Measurement Report (TS 38.331 5.5.5)
    gNB_A->>AMF_A: Handover Required (NGAP)

    %% Between operators (AMF/SMF)
    AMF_A->>AMF_B: Handover Request (TS 23.502 4.9.1, inter-AMF/PLMN)
    AMF_B->>gNB_B: Resource Prep Req

    gNB_B->>AMF_B: Resource Prep Response
    AMF_B->>AMF_A: Handover Request Ack
    AMF_A->>gNB_A: Handover Command

    %% RAN - HO Execution
    gNB_A->>UE_A: Mobility Control Info / HO Command
    UE_A->>gNB_B: RRC Reconfiguration Complete

    %% NAS - PDU Session Update
    gNB_B->>AMF_B: Handover Notify / Registration Update
    UE_A->>AMF_B: Registration Update (with old GUTI)
    AMF_B->>SMF_A: InterPLMN Mobility (PDU session update, TS 24.501 6.4.2.2)
    SMF_A->>SMF_B: PDU Session Anchor Move (optional)

    %% IMS/SIP - Session Transfer via STI
    UE_A->>IMS: SIP REGISTER (IMS HPLMN B)
    IMS->>UE_A: SIP 200 OK
    UE_A->>SCC_AS: SIP INVITE with STI (Session Transfer Identifier, TS 23.237 6.3.2.1)
    SCC_AS->>MCX_AS: Mapping update MCx flows

    %% Release of Original Resources
    SCC_AS->>SCC_AS: Identification of session to transfer (TS 23.237 6.3.2.1.4)
    SCC_AS->>gNB_A: Release resources

    %% === Phase 3: UE_A connected to HPLMN B, MCx session maintained ===

    Note over UE_A,UE_B: **Phase 3: UE A connected to HPLMN B, MCx session active**

    %% RAN
    UE_A->>gNB_B: Data/Media flows (without interruption)

    %% NAS
    UE_A->>AMF_B: Stable connection, PDU session maintained (TS 24.501 6.4.2.2)

    %% IMS/SIP/MCx
    UE_A->>SCC_AS: SIP session still anchored, continuous media via MCx Server (TS 23.280 Annex B)
    MCX_AS->>UE_B: Continuous media flows

    Note over UE_A,UE_B: End-to-end MCx service session between UE_A and UE_B maintained, with re-anchoring on HPLMN B.
