# Introduction

The present page provides an illustration of the OBapp / TSapp API in call flows in the context of ETCS.

# ETCS flow - connection to RBC

## High-level flow

```mermaid

sequenceDiagram
participant A as EVC
participant O as OB FRMCS
participant M as MCX
participant T as TS GW
participant B as RBC

Note over O: Start of Operation
par App Local Binding and subscriptions to general events
	%Note over LCAppOB,OBfrmcs: OBapp local binding
	A->>O: POST /registrations
	O->>A: 200 OK (DynamicID=D1)
	A->>O: GET /notifications/{DynamicID}/general
	O->>A: 200 OK
	B->>T: POST /registrations
	T->>B: 200 OK (DynamicID=D2)
	B->>T: GET /notifications/{DynamicID}/general
	T->>B: 200 OK
and MC registration for App
	Note over O,M: MCregistration
	O-->>A: SSE FSD_AVL
	Note over T,M: MCregistration
end

%open session
A->>O:	POST /sessions/{DynamicID=D1} (localAppIPAddress=IPe,recipients=[RBC])
par Initial answer
	O->>A:	200 OK [initial answer] (SessionID=S1)
and Session establishment
	Note over O,T: MCdata
	T-->>B:	SSE Incoming Session (SessionID=S3)
	B->>T:	PUT /sessions/{DynamicID}/{SessionID=S3} (localAppIPAddress=IPr)
	T->>B: 200 OK
end
O-->>A:	SSE Final answer (SessionID=S1)	
Note over A,B: establishment of EVC ↔ RBC connection between IPe and IPr

```

## Details

```mermaid

sequenceDiagram
participant A as EVC
participant O as OB FRMCS
participant M as MCX
participant T as TS GW
participant B as RBC

Note over O: Start of Operation
par App Local Binding and subscriptions to general events
	%Note over LCAppOB,OBfrmcs: OBapp local binding
	A->>O: POST /registrations
	O->>A: 200 OK (DynamicID=D1)
	A->>O: GET /notifications/{DynamicID}/general
	O->>A: 200 OK
	B->>T: POST /registrations
	T->>B: 200 OK (DynamicID=D2)
	B->>T: GET /notifications/{DynamicID}/general
	T->>B: 200 OK
and MC registration for App
	Note over O,M: MCregistration
	O-->>A: SSE FSD_AVL
	Note over T,M: MCregistration
end

%open session
A->>O:	POST /sessions/{DynamicID=D1} (localAppIPAddress=IPe,recipients=[RBC])
par Initial answer
	O->>A:	200 OK [initial answer] (SessionID=S1)
and Session establishment
  O->>M:  MCData IPcon P2P request
  M->>T:  MCData IPcon P2P request
	T-->>B:	SSE Incoming Session (SessionID=S3)
	B->>T:	PUT /sessions/{DynamicID}/{SessionID=S3} (localAppIPAddress=IPr)
	T->>B: 200 OK
  T->>M: MCData IPcon P2P response (AnyExt(remoteAppIPAddress=IPr))
  M->>O: MCData IPcon P2P response (AnyExt(remoteAppIPAddress=IPr))
end
O-->>A:	SSE Final answer (SessionID=S1, remoteAppIPAddress=IPr)	

Note over A,B: establishment of EVC ↔ RBC connection between IPe and IPr
Note over A,B: TCP establishment
A->>B:  SYN
B->>A:  SYN ACK
A->>B:  ACK
Note over A,B: TLS establishment
A->>B: Client Hello
B->>A: Server Hello
A->>B: Client Finished
B->>A: Server Finished
Note over A,B: ALE
A->>B:  AU1
B->>A:  AU2
```

# ETCS flow - inter-RBC (national)

## High-level flow

```mermaid

sequenceDiagram
participant A as EVC
participant O as OB FRMCS
participant M as MCX
participant T as TS GW
participant B1 as RBC1
participant B2 as RBC2

par OB setup
	Note over O: Start of Operation 
	Note over A,O: OBapp local binding (DynamicID=D1)
	Note over O,M:	MC registration
and TS setup
	Note over B1,T: TSapp local binding (DynamicID=D2)
	Note over B2,T: TSapp local binding (DynamicID=D3)
	Note over T,M:	MC registration
end

% existing session with B1
Note over A,B1: session (S1 OB side, S3 TS side)  EVC ↔ RBC1 (IPe ↔ IPr1)

% establishment of session to B2
Note over A:	SBG (RBC2)

A->>O:	POST /sessions/{DynamicID=D1} (localAppIPAddress=IPe,recipients=[RBC2])
par Initial answer
	O->>A:	200 OK [initial answer] (SessionID=S2)
and Session establishment
	Note over O,T: MCdata
	T-->>B2: SSE Incoming Session (SessionID=S4)
	B2->>T:	PUT /sessions/{DynamicID=D3}/{SessionID=S4} (localAppIPAddress=IPr2)
	T->>B2: 200 OK
end
O-->>A:	SSE Final answer (SessionID=S2,remoteAppIPAddress=IPr2)	
Note over A,B2: establishment of EVC ↔ RBC2 connection (IPe ↔ IPr2)
Note over A,B2: ← end connection with RBC1
A->>O:	DELETE /sessions/{DynamicID=D1}/{SessionID=S1}
par ack to EVC
	O->>A:	200 OK
and closure towards TS
	Note over O,T:	MCdata termination [RBC1]
	T-->>B1: SSE closure (SessionID=S3)
end
Note over A,B2: → ended connection with RBC1

```
# ETCS flow - inter-RBC (cross border standard case) 

## High-level flow

```mermaid

sequenceDiagram
participant A as EVC
participant O as OB FRMCS
participant M1 as MCX1
participant M2 as MCX2
participant T1 as TS GW1
participant T2 as TS GW2
participant B1 as RBC1
participant B2 as RBC2

par OB setup
	Note over O: Start of Operation 
	Note over A,O: OBapp local binding (DynamicID=D1)
	Note over O,M1:	MC1 registration
and TS1 setup
	Note over B1,T1: TSapp local binding (DynamicID=D2)
	Note over T1,M1:	MC1 registration
and TS2 setup
	Note over B2,T2: TSapp local binding (DynamicID=D3)
	Note over T2,M2:	MC2 registration
end

% existing session with B1
Note over A,B1: session (S1 OB side, S3 TS side)  EVC ↔ RBC1 (IPe ↔ IPr1) over Domain 1
% NTT to move to domain 2
Note over O: NTT to move to Domain 2
% Domain Change
Note over O,M2: Acquisition of Domain 2
% SSE FSD_AVL
O-->>A: SSE FSD_AVL[Domain 2]
% establishment of session to B2
Note over A:	SBG (RBC2)

A->>O:	POST /sessions/{DynamicID=D1} (localAppIPAddress=IPe,recipients=[RBC2])
par Initial answer
	O->>A:	200 OK [initial answer] (SessionID=S2)
and Session establishment
	Note over O,T2: MCdata OB FRMCS ↔ MCX2 ↔ TS GW2
	T2-->>B2: SSE Incoming Session (SessionID=S4)
	B2->>T2:	PUT /sessions/{DynamicID=D3}/{SessionID=S4} (localAppIPAddress=IPr2)
	T2->>B2: 200 OK
end
O-->>A:	SSE Final answer (SessionID=S2,remoteAppIPAddress=IPr2)	
Note over A,B2: establishment of EVC ↔ RBC2 connection (IPe ↔ IPr2)
Note over A,B2: ← end connection with RBC1
A->>O:	DELETE /sessions/{DynamicID=D1}/{SessionID=S1}
par ack to EVC
	O->>A:	200 OK
and closure towards TS
	Note over O,T1:	MCdata termination [RBC1]
	T1-->>B1: SSE closure (SessionID=S3)
end
Note over A,B1: → ended connection with RBC1

```
# ETCS flow - inter-RBC (cross border RBC HO occurs before NTT) 

## High-level flow

```mermaid

sequenceDiagram
participant A as EVC
participant O as OB FRMCS
participant M1 as MCX1
participant M2 as MCX2
participant T1 as TS GW1
participant T2 as TS GW2
participant B1 as RBC1
participant B2 as RBC2

par OB setup
	Note over O: Start of Operation 
	Note over A,O: OBapp local binding (DynamicID=D1)
	Note over O,M1:	MC1 registration
and TS1 setup
	Note over B1,T1: TSapp local binding (DynamicID=D2)
	Note over T1,M1:	MC1 registration
and TS2 setup
	Note over B2,T2: TSapp local binding (DynamicID=D3)
	Note over T2,M2:	MC2 registration

end

% existing session with B1
Note over A,B1: session (S1 OB side, S3 TS side)  EVC ↔ RBC1 (IPe ↔ IPr1) over Domain 1
% establishment of session to B2
Note over A:	SBG (RBC2)

A->>O:	POST /sessions/{DynamicID=D1} (localAppIPAddress=IPe,recipients=[RBC2])
par Initial answer
	O->>A:	200 OK [initial answer] (SessionID=S2)
and Session establishment
	Note over O,T2:	MCdata OB FRMCS ↔ MCX1<br/>↳ MCX2 ↔ TS GW2
	T2-->>B2:	SSE Incoming Session (SessionID=S4)
	B2->>T2:	PUT /sessions/{DynamicID=D3}/{SessionID=S4} (localAppIPAddress=IPr2)
	T2->>B2:	200 OK
end
O-->>A:	SSE Final answer (SessionID=S2,remoteAppIPAddress=IPr2)	
Note over A,B2: establishment of EVC ↔ RBC2 connection (IPe ↔ IPr2)
Note over A,B2: ← end connection with RBC1
A->>O:	DELETE /sessions/{DynamicID=D1}/{SessionID=S1}
par ack to EVC
	O->>A:	200 OK
and closure towards TS
	Note over O,T1:	MCdata termination [RBC1]
	T1-->>B1: SSE closure (SessionID=S3)
end
Note over A,B1: → ended connection with RBC1
% NTT to move to domain 2
Note over O: NTT to move to Domain 2
% Domain Change
Note over O,M2: Acquisition of Domain 2
% SSE FSD_AVL
O-->>A: SSE FSD_AVL[Domain 2]
critical Recovery of Session (S3 OB side S4 TS side)
	Note over O,T2:	MCdata OB FRMCS ↔ MCX2 ↔ TS GW2
	O-->>T2:	Session Replacement (OldSessionID=S4)
	T2-->>O:	200 OK
	Note over O,T2: Replacement of Bearer for Session S2 OB side ↔ S4 TS
end
```
