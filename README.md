# dweetio-sample

This is a very simple sample application that uses the dweetio connector. It is composed of a single decision table (scriptr.io) script.

The decision table receives a numeric value that corresponds to a pressure that was applied on a "smart" door lock (the device). 
Based on the value of the measured pressure and optionally the number of times a pressure was applied on the loc, the decision table determines if this corresponds to a intrusion attempt or not.

The decision table was modified by adding code to its **pre-script** and **post-script** sections:
- The **pre-script** increments a counter by 1 everytime a pressure measure is received within 5 seconds. If the elapsed time between two measured pressures is higher than 5 seconds, the pre-script section resets the counter to 1.
```
if (!storage.local.pressure){ // initialize if this is the first time the script is invoked
  init(); 
}else {
  
  // if detected pressure occurred less than 5 seconds before the first event, increment counter
  var now = new Date().getTime();  
  if (now - storage.local.pressure.time < 5000) {
    storage.local.pressure.count = storage.local.pressure.count + 1;
  }else {
    // if pressure was detected more than 5 seconds after first event, re-initialize
    init();
  }  
}
... // more
```
- The ** post-script** section evaluates the "decision" taken by the decision table. If "decision.intrusion" is true, it dweets a new message on behalf of the "main_entrance_door" device towards all other devices.
```
//Variable "decision" is the object returned by the decision table execution.
//Variable "decision" format is [{"action1": "value1", "action2":  "value2"}].
if (decision.intrusion) {
  var dweetio = require("dweet");
  var dweeter = new dweetio.Dweet("main_entrance_door");
  dweeter.postDweet({"msg":"instrusion"});
}
return decision;
```
