# FAQ on SIM Card

### **Does the SIM card support hot swap?**

- Official website WIKI hot swap API support:

   BC25/EG810M series do not support it;

   Except for the LA/LE/GA series in EC600M/EC800M, other series do not support it;

   The EC600U/EC200U/EG912/EC800G series will not save even if powered off after setting as hot swap. 

   600N/200A/BG95 Series do support it;

- In addition to the official website API method, you can also detect SIM card plug-in/out through interrupts and switch between CFUN0/1.

### **How to investigate and solve the problem of invisible SIM card?**
- Use API to detect the status of the SIM card:  If it returns 1 after calling sim.getStatus(), which means it is normal. For other situations, please refer to [sim](../../api_reference/iotlib/sim.md) in detail. 
- Check whether the SIM card is inserted backwards, damaged or poor contact.
- Remove the capacitors and ESD components from the SIM card circuit to prevent the SIM card from not being recognized due to incorrect welding capacitance values or incorrect welding of ESD components.
- Test the pin impedance via the diode stall in multimeter with the red probe connected to ground and the black probe connected to the pin.
- Use an oscilloscope to measure the waveform of each pin of the module to see if the module recognizes the card.

### **Whether does it support dual SIM dual standby?**
- Does not support DSDS(Dual SIM Dual Standby)
- EC600M/EC800M series support DSSS (Dual SIM Single Standby). However, the EC800MCNGA does not support.

### **Is it available to read the mobile phone number of the SIM card?**

 You can query the mobile phone number by calling the sim.getPhoneNumber() interface. The IoT card does not have a mobile phone number. It will return -1 when calling this interface.

### **Whether ESIM and VSIM are supported and how to use them?**
- ESIM is only supported by part of modules, please contact Quectel technical support for details.
- Do not support VSIM.

### **How to determine the operator of the SIM card?**
- Usually there is an operator logo on the front of the SIM card
- Call the net.operatorName() interface to query, and the returned value contains operator information.
- Call the dataCall.getPDPContext(profileID) interface to view the apn. The apn of each operator is different. Thus, it can be discriminate by APN. 

