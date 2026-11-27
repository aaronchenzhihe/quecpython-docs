# FAQ on Internet dialup

### **Investigation on rejected network attachment **
- When the error message is "No suitable cells in tracking area", the SIM card may be in debt or there may be occasional network exception.

- When the error message is "EPS services not allowed in this PLMN", it may camp to a network excluded from the main operators.

- When the error is "Tracking area not allowed", it may be that the cell does not allow UE to roam and camp.

- When the error is "illegal UE or illegal ME", it may be caused by incorrect IMEI of the module or illegal terminal.

### **Investigation on rejected PDN activation **
- When the error is "missing or unknown APN", it is caused by the incorrect APN configured in the PDN activation request.

- When the error is "requested service option not subscribed", that means the network does not support this type of PDN activation request, or the UE does not subscribe to the service. For example, the VoLTE service is not activated in SIM card while the VoLTE PDN activation request is initiated.

- When the error is "multiple PDN connections for a given APN not allowed", that means the network only supports single default load and does not support multiple loads.

### <span id="label1">**In automatic mode, if LTE, 3G, and 2G cannot find a network, what is the module's processing logic, such as time interval and attempt counts? **</span>
- Scenario 1: The network deregisters from the network registration state and enters a non-coverage area or no signal. At this time, it will enter the OOS stage (out of service)
  Solution: Search the network every 30 seconds for 10 times in the first stage, ; if it fails, search the network every 45 seconds for 10 time again; if it still does not work in second stage, search the network every 60 seconds without stop.
  
- Scenario 2: Search the Internet after initiating
Solution: After the module is turned on, it will generate a list of available PLMN + RAT (Modes, that is 4G, 3G and 2G). Then, it will search and register the network according to the list. If it still fails to register to the network under the circumstance that all items in PLMN list runs out, it will enter limited service state.  (i.e. it camps on the network for emergency call)

### **Is the network search mechanism same as that in automatic mode after locking network mode?**
These two modes ultimately affect the content of the available PLMN + RAT (standard, 4G, 3G, 2G) list. After confirming PLMN and RAT, the subsequent network attachment is same for two modes.

### **After the module is successfully connected to the Internet, what is the process logic of the module if it disconnects (LTE and 3G) all of sudden?**

When there is a signal, the logic is the same as searching the network and registering the network after initiating, which is quite similar with Scenario 2 aforementioned.

### **After the module registers with the 3G network successfully, it will search for LTE. What is the time interval and attempt count?**
The logic of cell reselection is as follows:

- RRC idle state: The system messages (SIBs) issued by the 3G base station will contain neighbor cell information, such as 2G, 3G, and 4G neighbor cell information/combinations.

     After reaching the measurement threshold, the terminal will measure the LTE neighbor cells. After S criterion and R criterion are both met (which neighbor cell is reselected mainly based on cell priority and signal quality), the LTE neighbor cell will be reselected.

     Therefore, if there is no LTE neighbor cell information in the 3G system information; or it does not meet the S criterion & R criterion even if it owns the neighbor cell info, the terminal will still not be reselected to LTE.

     The time strategy for the 3G base station to deliver system messages is determined by itself, the module has no choice but accept. 

     Normally, the module will monitor and measure the serving cell and neighbor cells consecutively for sake of the mobility of the device (the terminal needs to detect the network at all times to ensure that it can receive the services provided by the network and avoid disconnection) .

- RRC connection state: If the 3G base station issues an LTE neighbor cell measurement instruction to the terminal (The Measurement Control signaling, which contains information such as the LTE neighbor cell frequency), the 3G base station will decide which neighboring cell the terminal will switch to after the terminal completes the measurement and reports.

     If the 3G base station does not issue LTE neighbor cell measurement instructions, the terminal will stay in 3G forever and will not switch to LTE.

> RRC is always in the connected state, which will occur generally when business is being performed (such as data transmission) all the time.

### **Investigation on failed or slow network registration and network attachment. **

- Call the [`sim.getStatus()`](../../api_reference/iotlib/sim.md#sim.getStatus) to check whether the SIM card is identified.
- Check whether the SIM card is in debt and whether the flow runs out [The 4G module will be registered to 2G network instead of 4G network]. [For other situations, please consult the SIM card operator]
- Confirm that the antenna is matched.
- Check whether the antenna is connected, whether the interface of the antenna socket is correct, and whether the antenna is damaged [confirm by replacing the antenna].
- Do comparison test by changing SIM card and EVB to ensure that normal hardware and SIM cards are used in test.
- Make sure the module supports the local network modes: Summary of international network bands
- Network coverage in remote areas may be incomplete, or the antenna RF in module is unqualified. Try carry out comparison test based on mobile-phone; During the test, please make sure the network mode in mobile phone is as same as that in module. Meanwhile, the Cellular-Z can also be applied to confirm . For details, refer to: How to judge the 4G/2G signal reception performance of the main board.
- Environmental factors (garage, basement, elevator, etc.).
- Confirm whether all devices behave consistently at the same location. It may be caused by pseudo base stations.
- Whether the card is a private card and whether the machine is bound to the card. It is needed to set correct APN for private network card. While the machine and card is unbound and out of service, please contact corresponding operator and rebind it.