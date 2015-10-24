/**
MonoPrice dual relay multi endpoint Zwave switch
*
Author Chad Torkkola based of of Matt Frank based on the work of chrisb for AEON Power Strip
*
Date Created: 6/26/2014
Last Modified: 10/22/2015
*
*/
// for the UI
metadata {
definition (name: "MonoPrice Dual Relay", namespace: "Monoprice", author: "Chad Torkkola") {
capability "Switch"
capability "Polling"
capability "Configuration"
capability "Refresh"
attribute "switch1", "string"
attribute "switch2", "string"


command "on1"
command "off1"
command "on2"
command "off2"

    fingerprint deviceId: "0x1001", inClusters:"0x25, 0x27, 0x60, 0x72, 0x86"
}

simulator {
status "on": "command: 2003, payload: FF"
status "off": "command: 2003, payload: 00"

    // reply messages
    reply "2001FF,delay 100,2502": "command: 2503, payload: FF"
    reply "200100,delay 100,2502": "command: 2503, payload: 00"
}

tiles {

standardTile("switch1", "device.switch1",canChangeIcon: true) {
                    state "on", label: "switch1", action: "off1", icon: "st.switches.switch.on", backgroundColor: "#79b821"
                    state "off", label: "switch1", action: "on1", icon: "st.switches.switch.off", backgroundColor: "#ffffff"
            }
standardTile("switch2", "device.switch2",canChangeIcon: true) {
                    state "on", label: "switch2", action: "off2", icon: "st.switches.switch.on", backgroundColor: "#79b821"
                    state "off", label: "switch2", action: "on2", icon: "st.switches.switch.off", backgroundColor: "#ffffff"
            }
    standardTile("refresh", "device.switch", inactiveLabel: false, decoration: "flat") {
                    state "default", label:"", action:"refresh.refresh", icon:"st.secondary.refresh"
            }

    standardTile("configure", "device.switch", inactiveLabel: false, decoration: "flat") {
            state "default", label:"", action:"configure", icon:"st.secondary.configure"
            }


    main(["switch1", "switch2"])
    details(["switch1","switch2","refresh","configure"])
}
}

// 0x25 0x32 0x27 0x70 0x85 0x72 0x86 0x60 0xEF 0x82

// 0x25: switch binary
// 0x32: meter --- no meter
// 0x27: switch all
// 0x70: configuration
// 0x85: association
// 0x86: version
// 0x60: multi-channel
// 0xEF: mark
// 0x82: hail

// parse events into attributes
def parse(String description) {
log.debug "Parsing desc => '${description}'"

def result = null
def cmd = zwave.parse(description, [0x60:3, 0x25:1,  ])
if (cmd) {
    result = createEvent(zwaveEvent(cmd))
}

return result
}

//Reports

def zwaveEvent(physicalgraph.zwave.commands.basicv1.BasicReport cmd) {
//log.debug "basic"
[name: "switch", value: cmd.value ? "on" : "off", type: "physical"]
}

def zwaveEvent(physicalgraph.zwave.commands.switchbinaryv1.SwitchBinaryReport cmd) 
{
//log.debug "SwitchBinary"
[name: "switch", value: cmd.value ? "on" : "off", type: "digital"]
}

def zwaveEvent(physicalgraph.zwave.commands.multichannelv3.MultiChannelCapabilityReport cmd) 
{
// log.debug "multichannelv3.MultiChannelCapabilityReport $cmd"
if (cmd.endPoint == 2 ) 
{
def currstate = device.currentState("switch2").getValue()
// log.debug "$currstate";
if (currstate == "on")
sendEvent(name: "switch2", value: "off", isStateChange: true, display: false)
else if (currstate == "off")
sendEvent(name: "switch2", value: "on", isStateChange: true, display: false)
}
else if (cmd.endPoint == 1 ) 
{
def currstate = device.currentState("switch1").getValue()
// log.debug "$currstate";
if (currstate == "on")
sendEvent(name: "switch1", value: "off", isStateChange: true, display: false)
else if (currstate == "off")
sendEvent(name: "switch1", value: "on", isStateChange: true, display: false)
}

}

def zwaveEvent(physicalgraph.zwave.commands.multichannelv3.MultiChannelCmdEncap cmd) {
// log.debug "MultiChannelCmdEncap $cmd"

def map = [ name: "switch$cmd.sourceEndPoint" ]
if (cmd.commandClass == 37){
  if (cmd.parameter == [0]) {
      map.value = "off"
    }
    if (cmd.parameter == [255]) {
        map.value = "on"
    }
    map
}
/*else if (cmd.commandClass == 50) {
    // bitAddress: false, command: 2, commandClass: 50, destinationEndPoint: 1, parameter: [33, 100, 0, 0, 0, 0, 0, 94, 0, 0, 0, 0], res01: false, sourceEndPoint: 1
    def hex1 = { n -> String.format("%02X", n) }
    def desc = "command: ${hex1(cmd.commandClass)}${hex1(cmd.command)}, payload: " + cmd.parameter.collect{hex1(it)}.join(" ")
    // Re-assign source end point 3-6 to 1-4 and 1-2 to 5-6 to sync up with the switch end points. 
    // Source end point in the message refers to always-on sockets.
 }   zwaveEvent((cmd.sourceEndPoint > 2) ? (cmd.sourceEndPoint-2) : (cmd.sourceEndPoint+4), zwave.parse(desc, [ 0x60:3, 0x25:1, 0x32:1, 0x70:1 ]))
*/
}
def zwaveEvent(physicalgraph.zwave.Command cmd) {
// Handles all Z-Wave commands we aren't interested in
[:]
// log.debug "Capture All $cmd"
}

// handle commands

def refresh() {

for ( i in 1..3 )
cmds << zwave.multiChannelV3.multiChannelCmdEncap(sourceEndPoint:1, destinationEndPoint:i, commandClass:37, command:2).format()
cmds << zwave.multiChannelV3.multiChannelCmdEncap(sourceEndPoint:2, destinationEndPoint:i, commandClass:37, command:2).format()

delayBetween(cmds)
}

def poll() {
delayBetween([
zwave.switchBinaryV1.switchBinaryGet().format(),
zwave.manufacturerSpecificV1.manufacturerSpecificGet().format()
])
}

def configure() {
// log.debug "Executing 'configure'"
delayBetween([
zwave.configurationV1.configurationSet(parameterNumber:4, configurationValue: [0]).format()	// Report reguarly
])
}
//255 is ALL the way on and 0 is off
def on1() {
delayBetween([
zwave.multiChannelV3.multiChannelCmdEncap(sourceEndPoint:1, destinationEndPoint:1, commandClass:37, command:1, parameter:[255]).format(),
zwave.multiChannelV3.multiChannelCmdEncap(sourceEndPoint:1, destinationEndPoint:1, commandClass:37, command:2).format()
// zwave.multiChannelV3.multiChannelCmdEncap(sourceEndPoint:2, destinationEndPoint:2, commandClass:37, command:2).format(),
])
}

def off1() {
delayBetween([
zwave.multiChannelV3.multiChannelCmdEncap(sourceEndPoint:1, destinationEndPoint:1, commandClass:37, command:1, parameter:[0]).format(),
zwave.multiChannelV3.multiChannelCmdEncap(sourceEndPoint:1, destinationEndPoint:1, commandClass:37, command:2).format()
// zwave.multiChannelV3.multiChannelCmdEncap(sourceEndPoint:2, destinationEndPoint:2, commandClass:37, command:2).format(),

])
}

def on2() {
delayBetween([
zwave.multiChannelV3.multiChannelCmdEncap(sourceEndPoint:2, destinationEndPoint:2, commandClass:37, command:1, parameter:[255]).format(),
zwave.multiChannelV3.multiChannelCmdEncap(sourceEndPoint:2, destinationEndPoint:2, commandClass:37, command:2).format()
//zwave.multiChannelV3.multiChannelCmdEncap(sourceEndPoint:1, destinationEndPoint:1, commandClass:37, command:2).format()
])
}

def off2() {
delayBetween([
zwave.multiChannelV3.multiChannelCmdEncap(sourceEndPoint:2, destinationEndPoint:2, commandClass:37, command:1, parameter:[0]).format(),
zwave.multiChannelV3.multiChannelCmdEncap(sourceEndPoint:2, destinationEndPoint:2, commandClass:37, command:2).format()
// zwave.multiChannelV3.multiChannelCmdEncap(sourceEndPoint:1, destinationEndPoint:1, commandClass:37, command:2).format()
])
}
