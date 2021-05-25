# Case Study: OTA Firmware Update for Wireless Sensor Network

I worked with a small but excellent company called The Detection Group to create and deploy a OTA system.  
This was a challenging and interesting project because of the following factors:
- Battery Powered Sensors, ie. they are asleep most of the time
- 2-level hierarchical network of routers and sensors, ie. multi-hop multi-step process
- Large scale network, 1000's of devices
- Critical performance limits.  Naive implementation or user error could sever or brick the entire network.
- 4 different device types to be upgradeable over the same transport and protocol.


Some background, I developed the firmware for their product in 2015, and they launched in 2016.  
Of course OTA was asked for.  I prototyped a system but when analyzing the system at scale I noticed
there would be significant risks and significant user interaction required.  

So we came up with a halfway compromise and shipped without true OTA.  
  
Fast forward to 2020, the company was acquired and there was a lot of demand for a true OTA system.  
With enough buy-in and willingness to work with me together, I knew it could be done and jumped at the opportunity.  
  
Here are some of the numbers and outcomes:

- Upgraded an entire network of 1000+ devices in 4 days without any downtime in service.  
- No person physically present at the customer site.  
- Various failure modes tested by pulling out batteries and disconnecting antennae.  
- Stress tests performed in ideal conditions with zero failure rate.
- UI developed for singular and network wide updates
- Point-to-Point upgrade feature also implemented for faster latency.  
- Scheduled upgrades possible, allowing the image transferring/verification to be a separate step from
  the copy+reset ie. switchover operation.

* By using OTA for the routers, the client was able to extend their infrastructure to support
  more devices types, allowing third parties to integrate with existing infrastructure.
* No bootloader required



