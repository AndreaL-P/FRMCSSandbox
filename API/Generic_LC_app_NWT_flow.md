# Generic LC app flow - inter-RBC (cross border with make-before-break constraint 1 RM 1 MCx client) 
# Network Transition occuring after Hand Over

## High-level flow
## Session recoveries might be handled at GW level (but not only option if something exists)
```mermaid

sequenceDiagram
participant A as OB DATA app
participant O as OB FRMCS
participant M1 as MCX1
participant M2 as MCX2
participant T1 as TS GW1
participant T2 as TS GW2
participant B1 as TS DATA app 1
participant B2 as TS DATA app 2

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
Note over A,B1: session (S1 OB side, S3 TS side)  OB DATA app ↔ TS DATA app 1 (IPe ↔ IPr1) over Domain 1
% establishment of session to B2
Note over A:	HO_Request_With_Make_Before_Brake_Contraint (TS DATA app 2)

A->>O:	POST /sessions/{DynamicID=D1} (localAppIPAddress=IPe,recipients=[TS DATA app 2])
par Initial answer
	O->>A:	200 OK [initial answer] (SessionID=S2)
and Session establishment via Interconnection (as per FU 7120 11.4.4.11 for ATP or 11.5.4.10 for ATO)
	Note over O,T2:	MCdata OB FRMCS ↔ MCX1<br/>↳ MCX2 ↔ TS GW2 <br/> via Interconnection (as per FU 7120 11.4.4.11 for ATP or 11.5.4.10 for ATO)
	T2-->>B2:	SSE Incoming Session (SessionID=S4)
	B2->>T2:	PUT /sessions/{DynamicID=D3}/{SessionID=S4} (localAppIPAddress=IPr2)
	T2->>B2:	200 OK
end
O-->>A:	SSE Final answer (SessionID=S2,remoteAppIPAddress=IPr2)	
Note over A,B2: establishment of OB DATA app 1 ↔ TS DATA app 2 connection (IPe ↔ IPr2)
Note over A,B1: ← end connection with TS DATA app 1
A->>O:	DELETE /sessions/{DynamicID=D1}/{SessionID=S1}
par ack to OB DATA app 1
	O->>A:	200 OK
and closure towards TS
	Note over O,T1:	MCdata termination [TS DATA app 1]
	T1-->>B1: SSE closure (SessionID=S3)
end
Note over A,B1: → ended connection with TS DATA app 1
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
# Generic LC app flow - inter-RBC (cross border with make-before-break constraint 1 RM 1 MCx client) 
# Network Transition occuring before Hand Over

## High-level flow
## Session recoveries might be handled at GW level (but not only option if something exists)
```mermaid

sequenceDiagram
participant A as OB DATA app
participant O as OB FRMCS
participant M1 as MCX1
participant M2 as MCX2
participant T1 as TS GW1
participant T2 as TS GW2
participant B1 as TS DATA app 1
participant B2 as TS DATA app 2

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
Note over A,B1: session (S1 OB side, S3 TS side)  OB DATA app ↔ TS DATA app 1 (IPe ↔ IPr1) over Domain 1
% establishment of session to B2
% NTT to move to domain 2
Note over O: NTT to move to Domain 2
% Domain Change
Note over O,M2: Acquisition of Domain 2
% SSE FSD_AVL
O-->>A: SSE FSD_AVL[Domain 2]
critical Recovery of Session (S1 OB side S3 TS side)
	Note over O,T1:	MCdata OB FRMCS ↔ MCX2<br/>↳ MCX1 ↔ TS GW1 <br/> via Interconnection (as per FU 7120 11.4.4.11 for ATP or 11.5.4.10 for ATO)
	O-->>T1:	Session Replacement (OldSessionID=S3)
	T1-->>O:	200 OK
	Note over O,T1: Replacement of Bearer for Session S1 OB side ↔ S1 TS
end

Note over A:	HO_Request_With_Make_Before_Brake_Contraint (TS DATA app 2)

A->>O:	POST /sessions/{DynamicID=D1} (localAppIPAddress=IPe,recipients=[TS DATA app 2])
par Initial answer
	O->>A:	200 OK [initial answer] (SessionID=S2)
and Session establishment
	Note over O,T2:	MCdata OB FRMCS ↔ MCX2 ↔ TS GW2
	T2-->>B2:	SSE Incoming Session (SessionID=S4)
	B2->>T2:	PUT /sessions/{DynamicID=D3}/{SessionID=S4} (localAppIPAddress=IPr2)
	T2->>B2:	200 OK
end
O-->>A:	SSE Final answer (SessionID=S2,remoteAppIPAddress=IPr2)	
Note over A,B2: establishment of OB DATA app 1 ↔ TS DATA app 2 connection (IPe ↔ IPr2)
Note over A,B1: ← end connection with TS DATA app 1
A->>O:	DELETE /sessions/{DynamicID=D1}/{SessionID=S1}
par ack to OB DATA app 1
	O->>A:	200 OK
and closure towards TS
	Note over O,T1:	MCdata termination [TS DATA app 1]
	T1-->>B1: SSE closure (SessionID=S3)
end
Note over A,B1: → ended connection with TS DATA app 1
```

# Generic LC app flow - inter-RBC (cross border with make-before-break constraint 1 RM 1 MCx client) 
# Network Transition occuring during Hand Over

## High-level flow
## Session recoveries might be handled at GW level (but not only option if something exists)
```mermaid

sequenceDiagram
participant A as OB DATA app
participant O as OB FRMCS
participant M1 as MCX1
participant M2 as MCX2
participant T1 as TS GW1
participant T2 as TS GW2
participant B1 as TS DATA app 1
participant B2 as TS DATA app 2

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
Note over A,B1: session (S1 OB side, S3 TS side)  OB DATA app ↔ TS DATA app 1 (IPe ↔ IPr1) over Domain 1
% establishment of session to B2
Note over A:	HO_Request_With_Make_Before_Brake_Contraint (TS DATA app 2)

A->>O:	POST /sessions/{DynamicID=D1} (localAppIPAddress=IPe,recipients=[TS DATA app 2])
par Initial answer
	O->>A:	200 OK [initial answer] (SessionID=S2)
and Session establishment via Interconnection (as per FU 7120 11.4.4.11 for ATP or 11.5.4.10 for ATO)
	Note over O,T2:	MCdata OB FRMCS ↔ MCX1<br/>↳ MCX2 ↔ TS GW2 <br/> via Interconnection (as per FU 7120 11.4.4.11 for ATP or 11.5.4.10 for ATO)
	T2-->>B2:	SSE Incoming Session (SessionID=S4)
	B2->>T2:	PUT /sessions/{DynamicID=D3}/{SessionID=S4} (localAppIPAddress=IPr2)
	T2->>B2:	200 OK
end
O-->>A:	SSE Final answer (SessionID=S2,remoteAppIPAddress=IPr2)	
Note over A,B2: establishment of OB DATA app 1 ↔ TS DATA app 2 connection (IPe ↔ IPr2)
% NTT to move to domain 2
Note over O: NTT to move to Domain 2
% Domain Change
Note over O,M2: Acquisition of Domain 2
% SSE FSD_AVL
O-->>A: SSE FSD_AVL[Domain 2]
critical Recovery of Session (S1 and S3 OB side S2 and S4 TS side)
	par S1 OB Side and S2 TS Side
		Note over O,T1:	MCdata OB FRMCS ↔ MCX2 ↔ MCX1 ↔ TS GW1 <br/> via Interconnection (as per FU 7120 11.4.4.11 for ATP or 11.5.4.10 for ATO)
		O-->>T2:	Session Replacement (OldSessionID=S3)
		T2-->>O:	200 OK
		Note over O,T2: Replacement of Bearer for Session S1 OB side ↔ S3 TS
	and S3 OB side and S4 TS side
		Note over O,T2:	MCdata OB FRMCS ↔ MCX2 ↔ TS GW2
		O-->>T2:	Session Replacement (OldSessionID=S4)
		T2-->>O:	200 OK
		Note over O,T2: Replacement of Bearer for Session S2 OB side ↔ S4 TS
        end
end
Note over A,B1: ← end connection with TS DATA app 1
A->>O:	DELETE /sessions/{DynamicID=D1}/{SessionID=S1}
par ack to OB DATA app 1
	O->>A:	200 OK
and closure towards TS
	Note over O,T1:	MCdata termination [TS DATA app 1]
	T1-->>B1: SSE closure (SessionID=S3)
end
Note over A,B1: → ended connection with TS DATA app 1
```
