![image](https://github.com/user-attachments/assets/9fbc2e32-1cf9-46dd-9816-538d24d71e88)# How-does-ePDG-work-
How does ePDG work

The ePDG is responsible for interworking between the EPC and untrusted non-3GPP networks that require secure access, such as a WiFi, LTE metro, and femtocell access networks. 
eNodeB: The eNodeB (evolved Node B) is the termination point for all radio-related protocols. As a network, E-UTRAN is simply a mesh of eNodeBs connected to neighboring eNodeBs via the X2 interface. 
MME : The Cisco MME (Mobility Management Entity) is the key control node for the LTE access network. It works in conjunction with the eNodeB and the Cisco S-GW to control bearer activation and deactivation. The MME is typically responsible for selecting the Cisco P-GW for the UEs to access the PDN, but for secure access from untrusted non- 3GPP IP access networks, the ePDG is responsible for selecting the P-GW.
S-GW : The Cisco S-GW (Serving Gateway) routes and forwards data packets from the 3GPP UEs and acts as the mobility anchor during inter-eNodeB handovers. The S-GW receives signals from the MME that control the data traffic. Every 3GPP UE accessing the EPC is associated with a single S-GW. 
P-GW : The Cisco P-GW (Packet Data Network Gateway) is the network node that terminates the SGi interface towards the PDN. The P-GW provides connectivity to external PDNs for the subscriber UEs by being the point of entry and exit for all subscriber UE traffic. A subscriber UE may have simultaneous connectivity with more than one P-GW for accessing multiple PDNs. The P-GW performs policy enforcement, packet filtering, charging support, lawful interception, and packet screening. The P-GW is the mobility anchor for both trusted and untrusted non-3GPP IP access networks. For PMIP-based S2a and S2b interfaces, the P-GW hosts the LMA (Local Mobility Anchor) function. 
3GPP AAA Server : The 3GPP AAA (Authentication, Authorization, and Accounting) server provides UE authentication via the EAP-AKA (Extensible Authentication Protocol - Authentication and Key Agreement) authentication method. 
HSS : The HSS (Home Subscriber Server), is the master user database that supports the IMS (IP Multimedia Subsystem) network entities. It contains subscriber profiles, performs subscriber authentication and authorization, and provides information about the subscriber's location and IP information. 
PCRF : The PCRF (Policy and Charging Rules Function) determines policy rules in the IMS network. The PCRF operates in the network core, accesses subscriber databases and charging systems, and makes intelligent policy decisions for subscribers. 


The Cisco® ePDG (evolved Packet Data Gateway) enables mobile operators to provide secure access to the 3GPP E- UTRAN/EPC (Evolved UTRAN/Evolved Packet Core) network from untrusted non-3GPP IP access networks. The ePDG functions as a security gateway to provide network security and internet working control via IPSec tunnel establishment based on information obtained during 3GPP AAA (Authentication, Authorization, and Accounting). The ePDG enables mobile operators to extend wireless service coverage, reduce the load on the macro wireless network, and make use of existing backhaul infrastructure to reduce the cost of carrying wireless calls. 
The ePDG has the following key features: 
	•   Support for the IPSec/IKEv2-based SWu interface between the ePDG and the WLAN (Wireless LAN) UEs. 
	•   Routing of packets between the WLAN UEs and the Cisco P-GW (Packet Data Network Gateway) over the S2b interface via GTPv2 or PMIPv6 (Proxy Mobile IP version 6) protocol. 
	•   P-GW selection via DNS client functionality to provide PDN (Packet Data Network) connectivity to the WLAN UEs. 
	•   Support for passing assigned IPv4/IPv6 address configurations from the P-GW to the WLAN UEs. 
	•   Support for the Diameter-based SWm interface between the ePDG and the external 3GPP AAA server. 
	•   Tunnel authentication and authorization for IPSec/PMIPv6/GTPv2 tunnels using the EAP-AKA (Extensible Authentication Protocol - Authentication and Key Agreement) authentication method between the 3GPP AAA server and the WLAN UEs. 
	•   Encapsulation and decapsulation of packets sent over the IPSec/PMIPv6/GTPv2 tunnels. 
	•   Hosts a MAG (Mobile Access Gateway) function, which acts as a proxy mobility agent in the E-UTRAN/EPC network and uses PMIPv6 signaling to provide network-based mobility management on behalf of the WLAN UEs attached to the network. 

EPDG Call
=========
1. IKE_SA_INIT/34 from syfer to EPDG context  			// ikev2 and ipsek transform set for negotiation
2. IKE_SA_INIT/34 from EPDG to syfer    			// return back with its own ikev2 and ipsek transform set for negotiation
3. IKE_AUTH/35 syfer to EPDG   			 // syfer accepts the ipsec and ikev2 negotiations 
	a. this time there is a payload with 0.0.0.0 IP requesting for IP for the subs from ip pool 
4. Diameter-EAP-Request from EPDG to AAA to authenticate subsriber that came from syfer 
	a. subs info like IMSI, apn name, msk, ki, op should match the one coming from syfer and in AAA
	b. this can be found in syfer.conf and aaa.conf in test tools
5. AAA responds with Diameter-EAP-Answer 
	a. if it can find a subs in its database matching the subs imsi and apn and all other info then it sends out an authorization authenticate
	b. along with that subs bndwidth UL and DL speed is also sent out 
6. IKE_AUTH/35 is sent from EPDG to Syfer with IKEV2_EAP_PAYLOAD_CODE_REQUEST/1 
7. IKE_AUTH/35 is returned back from Syfer to EPDG with IKEV2_EAP_PAYLOAD_CODE_RESPONSE/2
8. Another Diameter-EAP-Request goes out from EPDG to AAA 
9. AAA repnds with Diameter-EAP-Answer
10. EPDG sends a IKEV2_EAP_PAYLOAD_CODE_SUCCESS/3  to Syfer 
11. Syfer sends out a SHARED_KEY_MESSAGE_INTEGRITY_CODE/2 to EPDG 
12. EPDG now sends out a EGTP_CREATE_SESSION_REQUEST to PGW to get an IP Address for the UE. s2b > s5-PGW
	a. with apn that UE wants to attach to. If APN is not present then session wont be creted
	b. Also with UL and DL speed authorized to UE from AAA 
	c. Interface is ePDG S2b DATA
13.  PGW responds with EGTP_CREATE_SESSION_RESPONSE
	a. if everything matches and there is an apn and pool then EGTP_CAUSE_REQ_ACCEPTED  with an IP Address for UE 
	b. if not then it is delicined and session is terminated and message is sent to Syfer that it got declined from PGW 
	c. check apn name and make sure everything is correct. 
14.  EPDG sends a IKEV2_CFG_ATTRIBUTE_INTERNAL_IP4_ADDRESS/1 to Syfer with the IP address for UE sent from PGW 
15. Session comes up. Check in DI - show subs all 



