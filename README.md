﻿# Cisco/Juniper bandwidth monitor
Using Grafana/Prometheus/snmpexporter to monitor CPU/bandwidth of Cisco interfaces
## Download and install Prometheus  
```
wget https://github.com/prometheus/prometheus/releases/download/v2.15.2/prometheus-2.15.2.linux-amd64.tar.gz
```
## Untar the gz file
```
mkdir /home/prometheus
tar -xvf prometheus-2.15.2.linux-amd64.tar.gz
```
## Download snmpexporter
```
wget https://github.com/prometheus/snmp_exporter/releases/download/v0.16.1/snmp_exporter-0.16.1.linux-amd64.tar.gz
```
## Untar the snmpexporter file
```
mkdir /home/prometheus/snmp_exporter
tar -xvf snmp_exporter-0.16.1.linux-amd64.tar.gz
```
## Configure the prometheus.yml file
```
# Global config
global:
  scrape_interval:     240s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 240s # Evaluate rules every 15 seconds. The default is every 1 minute.
  scrape_timeout: 120s  # scrape_timeout is set to the global default (10s).

# A scrape configuration containing exactly one endpoint to scrape:# Here it's Prometheus itself.
scrape_configs:
  - job_name: 'snmp'
    static_configs:
    - targets: ['cisco1']
    - targets: ['cisco2']
    metrics_path: /snmp
    params:
      module: ['ciscosw']
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: localhost:9116
  - job_name: 'snmpj'
    static_configs:
    - targets: ['juniper1']
    metrics_path: /snmp
    params:
      module: ['junipersw']
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: localhost:9116
```
## Configure the snmp.yml file
Add the ciscosw and junipersw snmp yml for Input and Output bandwidth monitor

```
apcups:
  walk:
  - 1.3.6.1.2.1.1.3
  - 1.3.6.1.2.1.2
  - 1.3.6.1.4.1.318.1.1.1.12
  - 1.3.6.1.4.1.318.1.1.1.2
  - 1.3.6.1.4.1.318.1.1.1.3
  - 1.3.6.1.4.1.318.1.1.1.4
  - 1.3.6.1.4.1.318.1.1.1.7.2
  - 1.3.6.1.4.1.318.1.1.1.8.1
  - 1.3.6.1.4.1.318.1.1.10.2.3.2
  metrics:
  - name: sysUpTime
    oid: 1.3.6.1.2.1.1.3
    type: gauge
    help: The time (in hundredths of a second) since the network management portion
      of the system was last re-initialized. - 1.3.6.1.2.1.1.3
  - name: ifNumber
    oid: 1.3.6.1.2.1.2.1
    type: gauge
    help: The number of network interfaces (regardless of their current state) present
      on this system. - 1.3.6.1.2.1.2.1
  - name: ifIndex
    oid: 1.3.6.1.2.1.2.2.1.1
    type: gauge
    help: A unique value, greater than zero, for each interface - 1.3.6.1.2.1.2.2.1.1
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifDescr
    oid: 1.3.6.1.2.1.2.2.1.2
    type: DisplayString
    help: A textual string containing information about the interface - 1.3.6.1.2.1.2.2.1.2
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifType
    oid: 1.3.6.1.2.1.2.2.1.3
    type: gauge
    help: The type of interface - 1.3.6.1.2.1.2.2.1.3
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifMtu
    oid: 1.3.6.1.2.1.2.2.1.4
    type: gauge
    help: The size of the largest packet which can be sent/received on the interface,
      specified in octets - 1.3.6.1.2.1.2.2.1.4
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifSpeed
    oid: 1.3.6.1.2.1.2.2.1.5
    type: gauge
    help: An estimate of the interface's current bandwidth in bits per second - 1.3.6.1.2.1.2.2.1.5
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifPhysAddress
    oid: 1.3.6.1.2.1.2.2.1.6
    type: PhysAddress48
    help: The interface's address at its protocol sub-layer - 1.3.6.1.2.1.2.2.1.6
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifAdminStatus
    oid: 1.3.6.1.2.1.2.2.1.7
    type: gauge
    help: The desired state of the interface - 1.3.6.1.2.1.2.2.1.7
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOperStatus
    oid: 1.3.6.1.2.1.2.2.1.8
    type: gauge
    help: The current operational state of the interface - 1.3.6.1.2.1.2.2.1.8
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifLastChange
    oid: 1.3.6.1.2.1.2.2.1.9
    type: gauge
    help: The value of sysUpTime at the time the interface entered its current operational
      state - 1.3.6.1.2.1.2.2.1.9
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInOctets
    oid: 1.3.6.1.2.1.2.2.1.10
    type: counter
    help: The total number of octets received on the interface, including framing
      characters - 1.3.6.1.2.1.2.2.1.10
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.11
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were not addressed to a multicast or broadcast address at this sub-layer
      - 1.3.6.1.2.1.2.2.1.11
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInNUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.12
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a multicast or broadcast address at this sub-layer -
      1.3.6.1.2.1.2.2.1.12
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInDiscards
    oid: 1.3.6.1.2.1.2.2.1.13
    type: counter
    help: The number of inbound packets which were chosen to be discarded even though
      no errors had been detected to prevent their being deliverable to a higher-layer
      protocol - 1.3.6.1.2.1.2.2.1.13
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInErrors
    oid: 1.3.6.1.2.1.2.2.1.14
    type: counter
    help: For packet-oriented interfaces, the number of inbound packets that contained
      errors preventing them from being deliverable to a higher-layer protocol - 1.3.6.1.2.1.2.2.1.14
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInUnknownProtos
    oid: 1.3.6.1.2.1.2.2.1.15
    type: counter
    help: For packet-oriented interfaces, the number of packets received via the interface
      which were discarded because of an unknown or unsupported protocol - 1.3.6.1.2.1.2.2.1.15
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutOctets
    oid: 1.3.6.1.2.1.2.2.1.16
    type: counter
    help: The total number of octets transmitted out of the interface, including framing
      characters - 1.3.6.1.2.1.2.2.1.16
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.17
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were not addressed to a multicast or broadcast address at this sub-layer,
      including those that were discarded or not sent - 1.3.6.1.2.1.2.2.1.17
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutNUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.18
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a multicast or broadcast address at this sub-layer,
      including those that were discarded or not sent - 1.3.6.1.2.1.2.2.1.18
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutDiscards
    oid: 1.3.6.1.2.1.2.2.1.19
    type: counter
    help: The number of outbound packets which were chosen to be discarded even though
      no errors had been detected to prevent their being transmitted - 1.3.6.1.2.1.2.2.1.19
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutErrors
    oid: 1.3.6.1.2.1.2.2.1.20
    type: counter
    help: For packet-oriented interfaces, the number of outbound packets that could
      not be transmitted because of errors - 1.3.6.1.2.1.2.2.1.20
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutQLen
    oid: 1.3.6.1.2.1.2.2.1.21
    type: gauge
    help: The length of the output packet queue (in packets). - 1.3.6.1.2.1.2.2.1.21
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: upsOutletGroupStatusTableSize
    oid: 1.3.6.1.4.1.318.1.1.1.12.1.1
    type: gauge
    help: The number of outlet groups for the UPS. - 1.3.6.1.4.1.318.1.1.1.12.1.1
  - name: upsOutletGroupStatusIndex
    oid: 1.3.6.1.4.1.318.1.1.1.12.1.2.1.1
    type: gauge
    help: The index to the outlet group entry. - 1.3.6.1.4.1.318.1.1.1.12.1.2.1.1
    indexes:
    - labelname: upsOutletGroupStatusName
      type: gauge
    lookups:
    - labels:
      - upsOutletGroupStatusName
      labelname: upsOutletGroupStatusName
      oid: 1.3.6.1.4.1.318.1.1.1.12.1.2.1.2
      type: OctetString
  - name: upsOutletGroupStatusName
    oid: 1.3.6.1.4.1.318.1.1.1.12.1.2.1.2
    type: OctetString
    help: The name of the outlet group - 1.3.6.1.4.1.318.1.1.1.12.1.2.1.2
    indexes:
    - labelname: upsOutletGroupStatusName
      type: gauge
    lookups:
    - labels:
      - upsOutletGroupStatusName
      labelname: upsOutletGroupStatusName
      oid: 1.3.6.1.4.1.318.1.1.1.12.1.2.1.2
      type: OctetString
  - name: upsOutletGroupStatusGroupState
    oid: 1.3.6.1.4.1.318.1.1.1.12.1.2.1.3
    type: gauge
    help: Getting this variable will return the outlet group state - 1.3.6.1.4.1.318.1.1.1.12.1.2.1.3
    indexes:
    - labelname: upsOutletGroupStatusName
      type: gauge
    lookups:
    - labels:
      - upsOutletGroupStatusName
      labelname: upsOutletGroupStatusName
      oid: 1.3.6.1.4.1.318.1.1.1.12.1.2.1.2
      type: OctetString
  - name: upsOutletGroupStatusCommandPending
    oid: 1.3.6.1.4.1.318.1.1.1.12.1.2.1.4
    type: gauge
    help: Getting this variable will return the command pending state of the outlet
      group - 1.3.6.1.4.1.318.1.1.1.12.1.2.1.4
    indexes:
    - labelname: upsOutletGroupStatusName
      type: gauge
    lookups:
    - labels:
      - upsOutletGroupStatusName
      labelname: upsOutletGroupStatusName
      oid: 1.3.6.1.4.1.318.1.1.1.12.1.2.1.2
      type: OctetString
  - name: upsOutletGroupStatusOutletType
    oid: 1.3.6.1.4.1.318.1.1.1.12.1.2.1.5
    type: gauge
    help: Getting this variable will return the type of outlet group - 1.3.6.1.4.1.318.1.1.1.12.1.2.1.5
    indexes:
    - labelname: upsOutletGroupStatusName
      type: gauge
    lookups:
    - labels:
      - upsOutletGroupStatusName
      labelname: upsOutletGroupStatusName
      oid: 1.3.6.1.4.1.318.1.1.1.12.1.2.1.2
      type: OctetString
  - name: upsOutletGroupStatusGroupFullState
    oid: 1.3.6.1.4.1.318.1.1.1.12.1.2.1.6
    type: OctetString
    help: An ASCII string containing the 32 flags representing the current state(s)
      of the outlet group - 1.3.6.1.4.1.318.1.1.1.12.1.2.1.6
    indexes:
    - labelname: upsOutletGroupStatusName
      type: gauge
    lookups:
    - labels:
      - upsOutletGroupStatusName
      labelname: upsOutletGroupStatusName
      oid: 1.3.6.1.4.1.318.1.1.1.12.1.2.1.2
      type: OctetString
  - name: upsOutletGroupConfigTableSize
    oid: 1.3.6.1.4.1.318.1.1.1.12.2.1
    type: gauge
    help: The number of outlet groups for the UPS. - 1.3.6.1.4.1.318.1.1.1.12.2.1
  - name: upsOutletGroupConfigIndex
    oid: 1.3.6.1.4.1.318.1.1.1.12.2.2.1.1
    type: gauge
    help: The index to the outlet group entry. - 1.3.6.1.4.1.318.1.1.1.12.2.2.1.1
    indexes:
    - labelname: upsOutletGroupConfigIndex
      type: gauge
  - name: upsOutletGroupConfigName
    oid: 1.3.6.1.4.1.318.1.1.1.12.2.2.1.2
    type: OctetString
    help: The name of the outlet group. - 1.3.6.1.4.1.318.1.1.1.12.2.2.1.2
    indexes:
    - labelname: upsOutletGroupConfigIndex
      type: gauge
  - name: upsOutletGroupConfigPowerOnDelay
    oid: 1.3.6.1.4.1.318.1.1.1.12.2.2.1.3
    type: gauge
    help: The amount of time (seconds) the outlet group will delay powering on when
      the delayed on, reboot, or shutdown command is applied - 1.3.6.1.4.1.318.1.1.1.12.2.2.1.3
    indexes:
    - labelname: upsOutletGroupConfigIndex
      type: gauge
  - name: upsOutletGroupConfigPowerOffDelay
    oid: 1.3.6.1.4.1.318.1.1.1.12.2.2.1.4
    type: gauge
    help: The amount of time (seconds) the outlet group will delay powering off when
      the delayed off, reboot, or shutdown command is applied - 1.3.6.1.4.1.318.1.1.1.12.2.2.1.4
    indexes:
    - labelname: upsOutletGroupConfigIndex
      type: gauge
  - name: upsOutletGroupConfigRebootDuration
    oid: 1.3.6.1.4.1.318.1.1.1.12.2.2.1.5
    type: gauge
    help: During a reboot sequence, power is turned off and then back on - 1.3.6.1.4.1.318.1.1.1.12.2.2.1.5
    indexes:
    - labelname: upsOutletGroupConfigIndex
      type: gauge
  - name: upsOutletGroupConfigMinReturnRuntime
    oid: 1.3.6.1.4.1.318.1.1.1.12.2.2.1.6
    type: gauge
    help: In an Outlet Group shutdown sequence, the Outlet Group cycles power off
      then on - 1.3.6.1.4.1.318.1.1.1.12.2.2.1.6
    indexes:
    - labelname: upsOutletGroupConfigIndex
      type: gauge
  - name: upsOutletGroupConfigOutletType
    oid: 1.3.6.1.4.1.318.1.1.1.12.2.2.1.7
    type: gauge
    help: Getting this variable will return the type of outlet group - 1.3.6.1.4.1.318.1.1.1.12.2.2.1.7
    indexes:
    - labelname: upsOutletGroupConfigIndex
      type: gauge
  - name: upsOutletGroupConfigLoadShedControlSkipOffDelay
    oid: 1.3.6.1.4.1.318.1.1.1.12.2.2.1.8
    type: gauge
    help: Configures whether the outlet group's off delay setting (upsOutletGroupConfigPowerOffDelay)
      will be used in a load shedding situation, where applicable. - 1.3.6.1.4.1.318.1.1.1.12.2.2.1.8
    indexes:
    - labelname: upsOutletGroupConfigIndex
      type: gauge
  - name: upsOutletGroupConfigLoadShedControlAutoRestart
    oid: 1.3.6.1.4.1.318.1.1.1.12.2.2.1.9
    type: gauge
    help: Configures whether the outlet group will automatically restart after a load
      shedding situation, where applicable. - 1.3.6.1.4.1.318.1.1.1.12.2.2.1.9
    indexes:
    - labelname: upsOutletGroupConfigIndex
      type: gauge
  - name: upsOutletGroupConfigLoadShedControlTimeOnBattery
    oid: 1.3.6.1.4.1.318.1.1.1.12.2.2.1.10
    type: gauge
    help: Configures whether the outlet group will load shed (turn off) after the
      UPS's time on battery has exceeded the upsOutletGroupConfigLoadShedTimeOnBattery
      OID setting - 1.3.6.1.4.1.318.1.1.1.12.2.2.1.10
    indexes:
    - labelname: upsOutletGroupConfigIndex
      type: gauge
  - name: upsOutletGroupConfigLoadShedControlRuntimeRemaining
    oid: 1.3.6.1.4.1.318.1.1.1.12.2.2.1.11
    type: gauge
    help: Configures whether the outlet group will load shed (turn off) when the UPS
      is on battery and the remaining runtime is less than the upsOutletGroupConfigLoadShedRuntimeRemaining
      OID setting - 1.3.6.1.4.1.318.1.1.1.12.2.2.1.11
    indexes:
    - labelname: upsOutletGroupConfigIndex
      type: gauge
  - name: upsOutletGroupConfigLoadShedControlInOverload
    oid: 1.3.6.1.4.1.318.1.1.1.12.2.2.1.12
    type: gauge
    help: Configures whether the outlet group will load shed (turn off) when the UPS
      is in an overload condition - 1.3.6.1.4.1.318.1.1.1.12.2.2.1.12
    indexes:
    - labelname: upsOutletGroupConfigIndex
      type: gauge
  - name: upsOutletGroupConfigLoadShedTimeOnBattery
    oid: 1.3.6.1.4.1.318.1.1.1.12.2.2.1.13
    type: gauge
    help: When the UPS has been on battery for more time than this value, the outlet
      group will turn off if this condition is enabled by the upsOutletGroupConfigLoadShedControlTimeOnBattery
      OID - 1.3.6.1.4.1.318.1.1.1.12.2.2.1.13
    indexes:
    - labelname: upsOutletGroupConfigIndex
      type: gauge
  - name: upsOutletGroupConfigLoadShedRuntimeRemaining
    oid: 1.3.6.1.4.1.318.1.1.1.12.2.2.1.14
    type: gauge
    help: When the runtime remaining is less than this value, the outlet group will
      turn off if this condition is enabled by the upsOutletGroupConfigLoadShedControlRuntimeRemaining
      OID - 1.3.6.1.4.1.318.1.1.1.12.2.2.1.14
    indexes:
    - labelname: upsOutletGroupConfigIndex
      type: gauge
  - name: upsOutletGroupControlTableSize
    oid: 1.3.6.1.4.1.318.1.1.1.12.3.1
    type: gauge
    help: The number of outlet groups for the UPS. - 1.3.6.1.4.1.318.1.1.1.12.3.1
  - name: upsOutletGroupControlIndex
    oid: 1.3.6.1.4.1.318.1.1.1.12.3.2.1.1
    type: gauge
    help: The index to the outlet group entry. - 1.3.6.1.4.1.318.1.1.1.12.3.2.1.1
    indexes:
    - labelname: upsOutletGroupControlIndex
      type: gauge
  - name: upsOutletGroupControlName
    oid: 1.3.6.1.4.1.318.1.1.1.12.3.2.1.2
    type: OctetString
    help: The name of the outlet group - 1.3.6.1.4.1.318.1.1.1.12.3.2.1.2
    indexes:
    - labelname: upsOutletGroupControlIndex
      type: gauge
  - name: upsOutletGroupControlCommand
    oid: 1.3.6.1.4.1.318.1.1.1.12.3.2.1.3
    type: gauge
    help: Getting this variable will return the outlet group state - 1.3.6.1.4.1.318.1.1.1.12.3.2.1.3
    indexes:
    - labelname: upsOutletGroupControlIndex
      type: gauge
  - name: upsOutletGroupControlOutletType
    oid: 1.3.6.1.4.1.318.1.1.1.12.3.2.1.4
    type: gauge
    help: Getting this variable will return the type of outlet group - 1.3.6.1.4.1.318.1.1.1.12.3.2.1.4
    indexes:
    - labelname: upsOutletGroupControlIndex
      type: gauge
  - name: upsBasicBatteryStatus
    oid: 1.3.6.1.4.1.318.1.1.1.2.1.1
    type: gauge
    help: The status of the UPS batteries - 1.3.6.1.4.1.318.1.1.1.2.1.1
  - name: upsBasicBatteryTimeOnBattery
    oid: 1.3.6.1.4.1.318.1.1.1.2.1.2
    type: gauge
    help: The elapsed time since the UPS has switched to battery power. - 1.3.6.1.4.1.318.1.1.1.2.1.2
  - name: upsBasicBatteryLastReplaceDate
    oid: 1.3.6.1.4.1.318.1.1.1.2.1.3
    type: OctetString
    help: The date when the UPS system's batteries were last replaced in mm/dd/yy
      (or yyyy) format - 1.3.6.1.4.1.318.1.1.1.2.1.3
  - name: upsAdvBatteryCapacity
    oid: 1.3.6.1.4.1.318.1.1.1.2.2.1
    type: gauge
    help: The remaining battery capacity expressed in percent of full capacity. -
      1.3.6.1.4.1.318.1.1.1.2.2.1
  - name: upsAdvBatteryTemperature
    oid: 1.3.6.1.4.1.318.1.1.1.2.2.2
    type: gauge
    help: The current internal UPS temperature expressed in Celsius. - 1.3.6.1.4.1.318.1.1.1.2.2.2
  - name: upsAdvBatteryRunTimeRemaining
    oid: 1.3.6.1.4.1.318.1.1.1.2.2.3
    type: gauge
    help: The UPS battery run time remaining before battery exhaustion. - 1.3.6.1.4.1.318.1.1.1.2.2.3
  - name: upsAdvBatteryReplaceIndicator
    oid: 1.3.6.1.4.1.318.1.1.1.2.2.4
    type: gauge
    help: Indicates whether the UPS batteries need replacing. - 1.3.6.1.4.1.318.1.1.1.2.2.4
  - name: upsAdvBatteryNumOfBattPacks
    oid: 1.3.6.1.4.1.318.1.1.1.2.2.5
    type: gauge
    help: The number of external battery packs connected to the UPS - 1.3.6.1.4.1.318.1.1.1.2.2.5
  - name: upsAdvBatteryNumOfBadBattPacks
    oid: 1.3.6.1.4.1.318.1.1.1.2.2.6
    type: gauge
    help: The number of external battery packs connected to the UPS that are defective
      - 1.3.6.1.4.1.318.1.1.1.2.2.6
  - name: upsAdvBatteryNominalVoltage
    oid: 1.3.6.1.4.1.318.1.1.1.2.2.7
    type: gauge
    help: The nominal battery voltage in Volts. - 1.3.6.1.4.1.318.1.1.1.2.2.7
  - name: upsAdvBatteryActualVoltage
    oid: 1.3.6.1.4.1.318.1.1.1.2.2.8
    type: gauge
    help: The actual battery bus voltage in Volts. - 1.3.6.1.4.1.318.1.1.1.2.2.8
  - name: upsAdvBatteryCurrent
    oid: 1.3.6.1.4.1.318.1.1.1.2.2.9
    type: gauge
    help: The battery current in Amps. - 1.3.6.1.4.1.318.1.1.1.2.2.9
  - name: upsAdvTotalDCCurrent
    oid: 1.3.6.1.4.1.318.1.1.1.2.2.10
    type: gauge
    help: The total DC current in Amps. - 1.3.6.1.4.1.318.1.1.1.2.2.10
  - name: upsAdvBatteryFullCapacity
    oid: 1.3.6.1.4.1.318.1.1.1.2.2.11
    type: gauge
    help: The full chargeable capacity of the battery which is expressed in percentage.
      - 1.3.6.1.4.1.318.1.1.1.2.2.11
  - name: upsAdvBatteryActualVoltageTableIndex
    oid: 1.3.6.1.4.1.318.1.1.1.2.2.12.1.1
    type: gauge
    help: The Battery Frame identifier - 1.3.6.1.4.1.318.1.1.1.2.2.12.1.1
    indexes:
    - labelname: upsAdvBatteryActualVoltageTableIndex
      type: gauge
  - name: upsAdvBatteryActualVoltagePolarity
    oid: 1.3.6.1.4.1.318.1.1.1.2.2.12.1.2
    type: gauge
    help: The selected Battery Voltage Polarity - 1.3.6.1.4.1.318.1.1.1.2.2.12.1.2
    indexes:
    - labelname: upsAdvBatteryActualVoltageTableIndex
      type: gauge
  - name: upsAdvBatteryFrameActualVoltage
    oid: 1.3.6.1.4.1.318.1.1.1.2.2.12.1.3
    type: gauge
    help: The actual battery bus voltage in Volts. - 1.3.6.1.4.1.318.1.1.1.2.2.12.1.3
    indexes:
    - labelname: upsAdvBatteryActualVoltageTableIndex
      type: gauge
  - name: upsAdvTotalDCCurrentTableIndex
    oid: 1.3.6.1.4.1.318.1.1.1.2.2.13.1.1
    type: gauge
    help: The Battery Frame identifier - 1.3.6.1.4.1.318.1.1.1.2.2.13.1.1
    indexes:
    - labelname: upsAdvTotalDCCurrentTableIndex
      type: gauge
  - name: upsAdvTotalDCCurrentPolarity
    oid: 1.3.6.1.4.1.318.1.1.1.2.2.13.1.2
    type: gauge
    help: The selected Battery Current Polarity - 1.3.6.1.4.1.318.1.1.1.2.2.13.1.2
    indexes:
    - labelname: upsAdvTotalDCCurrentTableIndex
      type: gauge
  - name: upsAdvTotalFrameDCCurrent
    oid: 1.3.6.1.4.1.318.1.1.1.2.2.13.1.3
    type: gauge
    help: The Total DC Current of battery in Amperes. - 1.3.6.1.4.1.318.1.1.1.2.2.13.1.3
    indexes:
    - labelname: upsAdvTotalDCCurrentTableIndex
      type: gauge
  - name: upsAdvBatteryCurrentTableIndex
    oid: 1.3.6.1.4.1.318.1.1.1.2.2.14.1.1
    type: gauge
    help: The Battery Frame identifier - 1.3.6.1.4.1.318.1.1.1.2.2.14.1.1
    indexes:
    - labelname: upsAdvBatteryCurrentTableIndex
      type: gauge
    - labelname: upsAdvBatteryCurrentIndex
      type: gauge
  - name: upsAdvBatteryCurrentIndex
    oid: 1.3.6.1.4.1.318.1.1.1.2.2.14.1.2
    type: gauge
    help: The battery frame identifier. - 1.3.6.1.4.1.318.1.1.1.2.2.14.1.2
    indexes:
    - labelname: upsAdvBatteryCurrentTableIndex
      type: gauge
    - labelname: upsAdvBatteryCurrentIndex
      type: gauge
  - name: upsAdvBatteryCurrentPolarity
    oid: 1.3.6.1.4.1.318.1.1.1.2.2.14.1.3
    type: gauge
    help: The selected Battery current polarity - 1.3.6.1.4.1.318.1.1.1.2.2.14.1.3
    indexes:
    - labelname: upsAdvBatteryCurrentTableIndex
      type: gauge
    - labelname: upsAdvBatteryCurrentIndex
      type: gauge
  - name: upsAdvBatteryFrameCurrent
    oid: 1.3.6.1.4.1.318.1.1.1.2.2.14.1.4
    type: gauge
    help: The Battery current measured in Amperes - 1.3.6.1.4.1.318.1.1.1.2.2.14.1.4
    indexes:
    - labelname: upsAdvBatteryCurrentTableIndex
      type: gauge
    - labelname: upsAdvBatteryCurrentIndex
      type: gauge
  - name: upsAdvBatteryEstimatedChargeTime
    oid: 1.3.6.1.4.1.318.1.1.1.2.2.15
    type: gauge
    help: The estimated remaining time required to charge the UPS to a full state
      of charge. - 1.3.6.1.4.1.318.1.1.1.2.2.15
  - name: upsAdvBatteryPower
    oid: 1.3.6.1.4.1.318.1.1.1.2.2.16
    type: gauge
    help: The measured battery power in kW. - 1.3.6.1.4.1.318.1.1.1.2.2.16
  - name: upsAdvBatteryChargerStatus
    oid: 1.3.6.1.4.1.318.1.1.1.2.2.17
    type: gauge
    help: The status of the battery charger - 1.3.6.1.4.1.318.1.1.1.2.2.17
  - name: upsAdvBatteryInternalSKU
    oid: 1.3.6.1.4.1.318.1.1.1.2.2.19
    type: OctetString
    help: The SKU of the internal battery. - 1.3.6.1.4.1.318.1.1.1.2.2.19
  - name: upsAdvBatteryExternalSKU
    oid: 1.3.6.1.4.1.318.1.1.1.2.2.20
    type: OctetString
    help: The SKU of the external battery. - 1.3.6.1.4.1.318.1.1.1.2.2.20
  - name: upsAdvBatteryRecommendedReplaceDate
    oid: 1.3.6.1.4.1.318.1.1.1.2.2.21
    type: OctetString
    help: The recommended replacement date for the battery based on the UPS internal
      battery life algorithm. - 1.3.6.1.4.1.318.1.1.1.2.2.21
  - name: upsHighPrecBatteryCapacity
    oid: 1.3.6.1.4.1.318.1.1.1.2.3.1
    type: gauge
    help: The remaining battery capacity expressed in tenths of percent of full capacity.
      - 1.3.6.1.4.1.318.1.1.1.2.3.1
  - name: upsHighPrecBatteryTemperature
    oid: 1.3.6.1.4.1.318.1.1.1.2.3.2
    type: gauge
    help: The current internal UPS temperature expressed in tenths of degrees Celsius.
      - 1.3.6.1.4.1.318.1.1.1.2.3.2
  - name: upsHighPrecBatteryNominalVoltage
    oid: 1.3.6.1.4.1.318.1.1.1.2.3.3
    type: gauge
    help: The nominal battery voltage in tenths of Volts. - 1.3.6.1.4.1.318.1.1.1.2.3.3
  - name: upsHighPrecBatteryActualVoltage
    oid: 1.3.6.1.4.1.318.1.1.1.2.3.4
    type: gauge
    help: The actual battery bus voltage in tenths of Volts. - 1.3.6.1.4.1.318.1.1.1.2.3.4
  - name: upsHighPrecBatteryCurrent
    oid: 1.3.6.1.4.1.318.1.1.1.2.3.5
    type: gauge
    help: The battery current in tenths of Amps. - 1.3.6.1.4.1.318.1.1.1.2.3.5
  - name: upsHighPrecTotalDCCurrent
    oid: 1.3.6.1.4.1.318.1.1.1.2.3.6
    type: gauge
    help: The total DC current in tenths of Amps. - 1.3.6.1.4.1.318.1.1.1.2.3.6
  - name: upsHighPrecBatteryActualVoltageTableIndex
    oid: 1.3.6.1.4.1.318.1.1.1.2.3.7.1.1
    type: gauge
    help: The Battery Frame identifier - 1.3.6.1.4.1.318.1.1.1.2.3.7.1.1
    indexes:
    - labelname: upsHighPrecBatteryActualVoltageTableIndex
      type: gauge
  - name: upsHighPrecBatteryActualVoltagePolarity
    oid: 1.3.6.1.4.1.318.1.1.1.2.3.7.1.2
    type: gauge
    help: The selected Battery Voltage polarity - 1.3.6.1.4.1.318.1.1.1.2.3.7.1.2
    indexes:
    - labelname: upsHighPrecBatteryActualVoltageTableIndex
      type: gauge
  - name: upsHighPrecBatteryVoltage
    oid: 1.3.6.1.4.1.318.1.1.1.2.3.7.1.3
    type: gauge
    help: The actual battery bus voltage expressed as tenths of Volts. - 1.3.6.1.4.1.318.1.1.1.2.3.7.1.3
    indexes:
    - labelname: upsHighPrecBatteryActualVoltageTableIndex
      type: gauge
  - name: upsHighPrecTotalDCCurrentTableIndex
    oid: 1.3.6.1.4.1.318.1.1.1.2.3.8.1.1
    type: gauge
    help: The Battery Frame identifier - 1.3.6.1.4.1.318.1.1.1.2.3.8.1.1
    indexes:
    - labelname: upsHighPrecTotalDCCurrentTableIndex
      type: gauge
  - name: upsHighPrecTotalDCCurrentPolarity
    oid: 1.3.6.1.4.1.318.1.1.1.2.3.8.1.2
    type: gauge
    help: The selected Battery Current polarity - 1.3.6.1.4.1.318.1.1.1.2.3.8.1.2
    indexes:
    - labelname: upsHighPrecTotalDCCurrentTableIndex
      type: gauge
  - name: upsHighPrecTotalDCFrameCurrent
    oid: 1.3.6.1.4.1.318.1.1.1.2.3.8.1.3
    type: gauge
    help: The total DC current in tenths of Amperes. - 1.3.6.1.4.1.318.1.1.1.2.3.8.1.3
    indexes:
    - labelname: upsHighPrecTotalDCCurrentTableIndex
      type: gauge
  - name: upsHighPrecBatteryCurrentTableIndex
    oid: 1.3.6.1.4.1.318.1.1.1.2.3.9.1.1
    type: gauge
    help: The Battery frame identifier - 1.3.6.1.4.1.318.1.1.1.2.3.9.1.1
    indexes:
    - labelname: upsHighPrecBatteryCurrentTableIndex
      type: gauge
    - labelname: upsHighPrecBatteryCurrentIndex
      type: gauge
  - name: upsHighPrecBatteryCurrentIndex
    oid: 1.3.6.1.4.1.318.1.1.1.2.3.9.1.2
    type: gauge
    help: The Battery frame identifier - 1.3.6.1.4.1.318.1.1.1.2.3.9.1.2
    indexes:
    - labelname: upsHighPrecBatteryCurrentTableIndex
      type: gauge
    - labelname: upsHighPrecBatteryCurrentIndex
      type: gauge
  - name: upsHighPrecBatteryCurrentPolarity
    oid: 1.3.6.1.4.1.318.1.1.1.2.3.9.1.3
    type: gauge
    help: The selected Battery Current polarity - 1.3.6.1.4.1.318.1.1.1.2.3.9.1.3
    indexes:
    - labelname: upsHighPrecBatteryCurrentTableIndex
      type: gauge
    - labelname: upsHighPrecBatteryCurrentIndex
      type: gauge
  - name: upsHighPrecBatteryFrameCurrent
    oid: 1.3.6.1.4.1.318.1.1.1.2.3.9.1.4
    type: gauge
    help: The Battery current in tenths of Amperes. - 1.3.6.1.4.1.318.1.1.1.2.3.9.1.4
    indexes:
    - labelname: upsHighPrecBatteryCurrentTableIndex
      type: gauge
    - labelname: upsHighPrecBatteryCurrentIndex
      type: gauge
  - name: upsHighPrecBatteryPackTableSize
    oid: 1.3.6.1.4.1.318.1.1.1.2.3.10.1
    type: gauge
    help: The number of entries in the upsHighPrecBatteryPacks. - 1.3.6.1.4.1.318.1.1.1.2.3.10.1
  - name: upsHighPrecBatteryPackIndex
    oid: 1.3.6.1.4.1.318.1.1.1.2.3.10.2.1.1
    type: gauge
    help: The battery pack identifier. - 1.3.6.1.4.1.318.1.1.1.2.3.10.2.1.1
    indexes:
    - labelname: upsHighPrecBatteryPackIndex
      type: gauge
    - labelname: upsHighPrecBatteryCartridgeIndex
      type: gauge
  - name: upsHighPrecBatteryCartridgeIndex
    oid: 1.3.6.1.4.1.318.1.1.1.2.3.10.2.1.2
    type: gauge
    help: The battery cartridge identifier. - 1.3.6.1.4.1.318.1.1.1.2.3.10.2.1.2
    indexes:
    - labelname: upsHighPrecBatteryPackIndex
      type: gauge
    - labelname: upsHighPrecBatteryCartridgeIndex
      type: gauge
  - name: upsHighPrecBatteryPackFirmwareRevision
    oid: 1.3.6.1.4.1.318.1.1.1.2.3.10.2.1.3
    type: OctetString
    help: The battery pack firmware revision. - 1.3.6.1.4.1.318.1.1.1.2.3.10.2.1.3
    indexes:
    - labelname: upsHighPrecBatteryPackIndex
      type: gauge
    - labelname: upsHighPrecBatteryCartridgeIndex
      type: gauge
  - name: upsHighPrecBatteryPackSerialNumber
    oid: 1.3.6.1.4.1.318.1.1.1.2.3.10.2.1.4
    type: OctetString
    help: The battery pack serial number. - 1.3.6.1.4.1.318.1.1.1.2.3.10.2.1.4
    indexes:
    - labelname: upsHighPrecBatteryPackIndex
      type: gauge
    - labelname: upsHighPrecBatteryCartridgeIndex
      type: gauge
  - name: upsHighPrecBatteryPackTemperature
    oid: 1.3.6.1.4.1.318.1.1.1.2.3.10.2.1.5
    type: gauge
    help: The battery pack temperature measured in 10ths of degree Celcius - 1.3.6.1.4.1.318.1.1.1.2.3.10.2.1.5
    indexes:
    - labelname: upsHighPrecBatteryPackIndex
      type: gauge
    - labelname: upsHighPrecBatteryCartridgeIndex
      type: gauge
  - name: upsHighPrecBatteryPackStatus
    oid: 1.3.6.1.4.1.318.1.1.1.2.3.10.2.1.6
    type: OctetString
    help: The battery status for the pack only - 1.3.6.1.4.1.318.1.1.1.2.3.10.2.1.6
    indexes:
    - labelname: upsHighPrecBatteryPackIndex
      type: gauge
    - labelname: upsHighPrecBatteryCartridgeIndex
      type: gauge
  - name: upsHighPrecBatteryPackCartridgeHealth
    oid: 1.3.6.1.4.1.318.1.1.1.2.3.10.2.1.7
    type: OctetString
    help: The battery cartridge health - 1.3.6.1.4.1.318.1.1.1.2.3.10.2.1.7
    indexes:
    - labelname: upsHighPrecBatteryPackIndex
      type: gauge
    - labelname: upsHighPrecBatteryCartridgeIndex
      type: gauge
  - name: upsHighPrecBatteryPackCartridgeReplaceDate
    oid: 1.3.6.1.4.1.318.1.1.1.2.3.10.2.1.8
    type: OctetString
    help: The battery cartridge estimated battery replace date. - 1.3.6.1.4.1.318.1.1.1.2.3.10.2.1.8
    indexes:
    - labelname: upsHighPrecBatteryPackIndex
      type: gauge
    - labelname: upsHighPrecBatteryCartridgeIndex
      type: gauge
  - name: upsHighPrecBatteryPackCartridgeInstallDate
    oid: 1.3.6.1.4.1.318.1.1.1.2.3.10.2.1.9
    type: OctetString
    help: The battery cartridge install date. - 1.3.6.1.4.1.318.1.1.1.2.3.10.2.1.9
    indexes:
    - labelname: upsHighPrecBatteryPackIndex
      type: gauge
    - labelname: upsHighPrecBatteryCartridgeIndex
      type: gauge
  - name: upsHighPrecBatteryPackCartridgeStatus
    oid: 1.3.6.1.4.1.318.1.1.1.2.3.10.2.1.10
    type: OctetString
    help: The battery cartridge status - 1.3.6.1.4.1.318.1.1.1.2.3.10.2.1.10
    indexes:
    - labelname: upsHighPrecBatteryPackIndex
      type: gauge
    - labelname: upsHighPrecBatteryCartridgeIndex
      type: gauge
  - name: upsHighPrecBatteryHealth
    oid: 1.3.6.1.4.1.318.1.1.1.2.3.11
    type: OctetString
    help: The battery health - 1.3.6.1.4.1.318.1.1.1.2.3.11
  - name: upsBasicInputPhase
    oid: 1.3.6.1.4.1.318.1.1.1.3.1.1
    type: gauge
    help: The current AC input phase. - 1.3.6.1.4.1.318.1.1.1.3.1.1
  - name: upsAdvInputLineVoltage
    oid: 1.3.6.1.4.1.318.1.1.1.3.2.1
    type: gauge
    help: The current utility line voltage in VAC. - 1.3.6.1.4.1.318.1.1.1.3.2.1
  - name: upsAdvInputMaxLineVoltage
    oid: 1.3.6.1.4.1.318.1.1.1.3.2.2
    type: gauge
    help: The maximum utility line voltage in VAC over the previous 1 minute period.
      - 1.3.6.1.4.1.318.1.1.1.3.2.2
  - name: upsAdvInputMinLineVoltage
    oid: 1.3.6.1.4.1.318.1.1.1.3.2.3
    type: gauge
    help: The minimum utility line voltage in VAC over the previous 1 minute period.
      - 1.3.6.1.4.1.318.1.1.1.3.2.3
  - name: upsAdvInputFrequency
    oid: 1.3.6.1.4.1.318.1.1.1.3.2.4
    type: gauge
    help: The current input frequency to the UPS system in Hz. - 1.3.6.1.4.1.318.1.1.1.3.2.4
  - name: upsAdvInputLineFailCause
    oid: 1.3.6.1.4.1.318.1.1.1.3.2.5
    type: gauge
    help: The reason for the occurrence of the last transfer to UPS battery power
      - 1.3.6.1.4.1.318.1.1.1.3.2.5
  - name: upsAdvInputNominalFrequency
    oid: 1.3.6.1.4.1.318.1.1.1.3.2.6
    type: gauge
    help: The nominal input frequency of the UPS system in Hz. - 1.3.6.1.4.1.318.1.1.1.3.2.6
  - name: upsAdvInputNominalVoltage
    oid: 1.3.6.1.4.1.318.1.1.1.3.2.7
    type: gauge
    help: The nominal input voltage of the UPS system in VAC. - 1.3.6.1.4.1.318.1.1.1.3.2.7
  - name: upsAdvInputBypassNominalFrequency
    oid: 1.3.6.1.4.1.318.1.1.1.3.2.8
    type: gauge
    help: The nominal bypass input frequency of the UPS system in Hz. - 1.3.6.1.4.1.318.1.1.1.3.2.8
  - name: upsAdvInputBypassNominalVoltage
    oid: 1.3.6.1.4.1.318.1.1.1.3.2.9
    type: gauge
    help: The nominal bypass input voltage of the UPS system in VAC. - 1.3.6.1.4.1.318.1.1.1.3.2.9
  - name: upsAdvInputStatisticsIndex
    oid: 1.3.6.1.4.1.318.1.1.1.3.2.10.1.1
    type: gauge
    help: The input identifier. - 1.3.6.1.4.1.318.1.1.1.3.2.10.1.1
    indexes:
    - labelname: upsAdvInputStatisticsIndex
      type: gauge
  - name: upsAdvInputApparentPower
    oid: 1.3.6.1.4.1.318.1.1.1.3.2.10.1.2
    type: gauge
    help: The input apparent power(sum of all of the three phases) measured in kVA.
      - 1.3.6.1.4.1.318.1.1.1.3.2.10.1.2
    indexes:
    - labelname: upsAdvInputStatisticsIndex
      type: gauge
  - name: upsAdvInputVoltageTHD
    oid: 1.3.6.1.4.1.318.1.1.1.3.2.10.1.3
    type: gauge
    help: The input voltage Total Harmonic Distortion in percent. - 1.3.6.1.4.1.318.1.1.1.3.2.10.1.3
    indexes:
    - labelname: upsAdvInputStatisticsIndex
      type: gauge
  - name: upsAdvInputBypassVoltageTHD
    oid: 1.3.6.1.4.1.318.1.1.1.3.2.10.1.4
    type: gauge
    help: The bypass input voltage Total Harmonic Distortion in percent. - 1.3.6.1.4.1.318.1.1.1.3.2.10.1.4
    indexes:
    - labelname: upsAdvInputStatisticsIndex
      type: gauge
  - name: upsAdvInputPeakCurrent
    oid: 1.3.6.1.4.1.318.1.1.1.3.2.10.1.5
    type: gauge
    help: The input peak current in Amperes. - 1.3.6.1.4.1.318.1.1.1.3.2.10.1.5
    indexes:
    - labelname: upsAdvInputStatisticsIndex
      type: gauge
  - name: upsAdvInputBypassPeakCurrent
    oid: 1.3.6.1.4.1.318.1.1.1.3.2.10.1.6
    type: gauge
    help: The bypass input peak current in Amperes. - 1.3.6.1.4.1.318.1.1.1.3.2.10.1.6
    indexes:
    - labelname: upsAdvInputStatisticsIndex
      type: gauge
  - name: upsAdvInputActivePower
    oid: 1.3.6.1.4.1.318.1.1.1.3.2.10.1.7
    type: gauge
    help: The input active power measured in kW. - 1.3.6.1.4.1.318.1.1.1.3.2.10.1.7
    indexes:
    - labelname: upsAdvInputStatisticsIndex
      type: gauge
  - name: upsAdvInputTotalApparentPower
    oid: 1.3.6.1.4.1.318.1.1.1.3.2.11
    type: gauge
    help: The input total apparent power(sum of all three phases) of the UPS system
      in kVA. - 1.3.6.1.4.1.318.1.1.1.3.2.11
  - name: upsAdvInputTotalActivePower
    oid: 1.3.6.1.4.1.318.1.1.1.3.2.12
    type: gauge
    help: The input total active power(sum of all three phases) of the UPS system
      in kW. - 1.3.6.1.4.1.318.1.1.1.3.2.12
  - name: upsAdvInputBypassTotalApparentPower
    oid: 1.3.6.1.4.1.318.1.1.1.3.2.13
    type: gauge
    help: The input bypass total apparent power(sum of all three phases) of the UPS
      system in kVA. - 1.3.6.1.4.1.318.1.1.1.3.2.13
  - name: upsAdvInputBypassTotalActivePower
    oid: 1.3.6.1.4.1.318.1.1.1.3.2.14
    type: gauge
    help: The input bypass total active power(sum of all three phases) of the UPS
      system in kW. - 1.3.6.1.4.1.318.1.1.1.3.2.14
  - name: upsAdvInputEnergyUsage
    oid: 1.3.6.1.4.1.318.1.1.1.3.2.15
    type: gauge
    help: The input energy usage of the UPS in kWh. - 1.3.6.1.4.1.318.1.1.1.3.2.15
  - name: upsHighPrecInputLineVoltage
    oid: 1.3.6.1.4.1.318.1.1.1.3.3.1
    type: gauge
    help: The current utility line voltage in tenths of VAC. - 1.3.6.1.4.1.318.1.1.1.3.3.1
  - name: upsHighPrecInputMaxLineVoltage
    oid: 1.3.6.1.4.1.318.1.1.1.3.3.2
    type: gauge
    help: The maximum utility line voltage in tenths of VAC over the previous 1 minute
      period. - 1.3.6.1.4.1.318.1.1.1.3.3.2
  - name: upsHighPrecInputMinLineVoltage
    oid: 1.3.6.1.4.1.318.1.1.1.3.3.3
    type: gauge
    help: The minimum utility line voltage in tenths of VAC over the previous 1 minute
      period. - 1.3.6.1.4.1.318.1.1.1.3.3.3
  - name: upsHighPrecInputFrequency
    oid: 1.3.6.1.4.1.318.1.1.1.3.3.4
    type: gauge
    help: The current input frequency to the UPS system in tenths of Hz. - 1.3.6.1.4.1.318.1.1.1.3.3.4
  - name: upsHighPrecInputEnergyUsage
    oid: 1.3.6.1.4.1.318.1.1.1.3.3.5
    type: gauge
    help: The input energy usage of the UPS in hundredths of kWh. - 1.3.6.1.4.1.318.1.1.1.3.3.5
  - name: upsHighPrecInputBypassVoltage
    oid: 1.3.6.1.4.1.318.1.1.1.3.3.6
    type: gauge
    help: The current bypass line voltage in tenths of VAC. - 1.3.6.1.4.1.318.1.1.1.3.3.6
  - name: upsHighPrecInputBypassFrequency
    oid: 1.3.6.1.4.1.318.1.1.1.3.3.7
    type: gauge
    help: The current input bypass frequency to the UPS system in tenths of Hz. -
      1.3.6.1.4.1.318.1.1.1.3.3.7
  - name: upsBasicOutputStatus
    oid: 1.3.6.1.4.1.318.1.1.1.4.1.1
    type: gauge
    help: The current state of the UPS - 1.3.6.1.4.1.318.1.1.1.4.1.1
  - name: upsBasicOutputPhase
    oid: 1.3.6.1.4.1.318.1.1.1.4.1.2
    type: gauge
    help: The current output phase. - 1.3.6.1.4.1.318.1.1.1.4.1.2
  - name: upsBasicSystemStatus
    oid: 1.3.6.1.4.1.318.1.1.1.4.1.3
    type: gauge
    help: Current state for the whole system (UPS and surrounding breakers) - 1.3.6.1.4.1.318.1.1.1.4.1.3
  - name: upsBasicSystemInternalTemperature
    oid: 1.3.6.1.4.1.318.1.1.1.4.1.4
    type: gauge
    help: The actual internal temperature of the UPS system in Celsius. - 1.3.6.1.4.1.318.1.1.1.4.1.4
  - name: upsBasicSystemInverterStatus
    oid: 1.3.6.1.4.1.318.1.1.1.4.1.5
    type: gauge
    help: The current state of the UPS inverter - 1.3.6.1.4.1.318.1.1.1.4.1.5
  - name: upsBasicSystemPFCStatus
    oid: 1.3.6.1.4.1.318.1.1.1.4.1.6
    type: gauge
    help: The general status of the power factor correction (AC input stage of the
      UPS) - 1.3.6.1.4.1.318.1.1.1.4.1.6
  - name: upsBasicOutputACwiringConfiguration
    oid: 1.3.6.1.4.1.318.1.1.1.4.1.7
    type: gauge
    help: Indicates if neutral wire on output side of the UPS is used (load wired
      line to neutral) - 1.3.6.1.4.1.318.1.1.1.4.1.7
  - name: upsAdvOutputVoltage
    oid: 1.3.6.1.4.1.318.1.1.1.4.2.1
    type: gauge
    help: The output voltage of the UPS system in VAC. - 1.3.6.1.4.1.318.1.1.1.4.2.1
  - name: upsAdvOutputFrequency
    oid: 1.3.6.1.4.1.318.1.1.1.4.2.2
    type: gauge
    help: The current output frequency of the UPS system in Hz. - 1.3.6.1.4.1.318.1.1.1.4.2.2
  - name: upsAdvOutputLoad
    oid: 1.3.6.1.4.1.318.1.1.1.4.2.3
    type: gauge
    help: The current UPS load expressed in percent of rated capacity. - 1.3.6.1.4.1.318.1.1.1.4.2.3
  - name: upsAdvOutputCurrent
    oid: 1.3.6.1.4.1.318.1.1.1.4.2.4
    type: gauge
    help: The current in amperes drawn by the load on the UPS. - 1.3.6.1.4.1.318.1.1.1.4.2.4
  - name: upsAdvOutputRedundancy
    oid: 1.3.6.1.4.1.318.1.1.1.4.2.5
    type: gauge
    help: The number of power modules which can fail or be removed without causing
      the UPS to generate a Minimum Redundancy Lost event. - 1.3.6.1.4.1.318.1.1.1.4.2.5
  - name: upsAdvOutputKVACapacity
    oid: 1.3.6.1.4.1.318.1.1.1.4.2.6
    type: gauge
    help: The maximum load that this UPS can support. - 1.3.6.1.4.1.318.1.1.1.4.2.6
  - name: upsAdvOutputNominalFrequency
    oid: 1.3.6.1.4.1.318.1.1.1.4.2.7
    type: gauge
    help: The nominal output frequency of the UPS system in Hz. - 1.3.6.1.4.1.318.1.1.1.4.2.7
  - name: upsAdvOutputActivePower
    oid: 1.3.6.1.4.1.318.1.1.1.4.2.8
    type: gauge
    help: The total output active power of the UPS system in W - 1.3.6.1.4.1.318.1.1.1.4.2.8
  - name: upsAdvOutputApparentPower
    oid: 1.3.6.1.4.1.318.1.1.1.4.2.9
    type: gauge
    help: The total output apparent power of all phases of the UPS system in VA. -
      1.3.6.1.4.1.318.1.1.1.4.2.9
  - name: upsAdvOutputStatisticsIndex
    oid: 1.3.6.1.4.1.318.1.1.1.4.2.10.1.1
    type: gauge
    help: The output identifier. - 1.3.6.1.4.1.318.1.1.1.4.2.10.1.1
    indexes:
    - labelname: upsAdvOutputStatisticsIndex
      type: gauge
  - name: upsAdvOutputPeakCurrent
    oid: 1.3.6.1.4.1.318.1.1.1.4.2.10.1.2
    type: gauge
    help: The output peak current in Amperes. - 1.3.6.1.4.1.318.1.1.1.4.2.10.1.2
    indexes:
    - labelname: upsAdvOutputStatisticsIndex
      type: gauge
  - name: upsAdvOutputCurrentTHD
    oid: 1.3.6.1.4.1.318.1.1.1.4.2.10.1.3
    type: gauge
    help: The output current Total Harmonic Distortion in percent. - 1.3.6.1.4.1.318.1.1.1.4.2.10.1.3
    indexes:
    - labelname: upsAdvOutputStatisticsIndex
      type: gauge
  - name: upsAdvOutputCrestFactor
    oid: 1.3.6.1.4.1.318.1.1.1.4.2.10.1.4
    type: gauge
    help: The output current crest factor expressed in tenths. - 1.3.6.1.4.1.318.1.1.1.4.2.10.1.4
    indexes:
    - labelname: upsAdvOutputStatisticsIndex
      type: gauge
  - name: upsAdvOutputNeutralCurrent
    oid: 1.3.6.1.4.1.318.1.1.1.4.2.11
    type: gauge
    help: The neutral current in amperes. - 1.3.6.1.4.1.318.1.1.1.4.2.11
  - name: upsAdvOutputEnergyUsage
    oid: 1.3.6.1.4.1.318.1.1.1.4.2.12
    type: gauge
    help: The output energy usage of the UPS in kWh. - 1.3.6.1.4.1.318.1.1.1.4.2.12
  - name: upsHighPrecOutputVoltage
    oid: 1.3.6.1.4.1.318.1.1.1.4.3.1
    type: gauge
    help: The output voltage of the UPS system in tenths of VAC. - 1.3.6.1.4.1.318.1.1.1.4.3.1
  - name: upsHighPrecOutputFrequency
    oid: 1.3.6.1.4.1.318.1.1.1.4.3.2
    type: gauge
    help: The current output frequency of the UPS system in tenths of Hz. - 1.3.6.1.4.1.318.1.1.1.4.3.2
  - name: upsHighPrecOutputLoad
    oid: 1.3.6.1.4.1.318.1.1.1.4.3.3
    type: gauge
    help: The current UPS load expressed in tenths of percent of rated capacity. -
      1.3.6.1.4.1.318.1.1.1.4.3.3
  - name: upsHighPrecOutputCurrent
    oid: 1.3.6.1.4.1.318.1.1.1.4.3.4
    type: gauge
    help: The current in tenths of amperes drawn by the load on the UPS. - 1.3.6.1.4.1.318.1.1.1.4.3.4
  - name: upsHighPrecOutputEfficiency
    oid: 1.3.6.1.4.1.318.1.1.1.4.3.5
    type: gauge
    help: The positive values represent efficiency of the UPS in tenths of percent
      - 1.3.6.1.4.1.318.1.1.1.4.3.5
  - name: upsHighPrecOutputEnergyUsage
    oid: 1.3.6.1.4.1.318.1.1.1.4.3.6
    type: gauge
    help: The output energy usage of the UPS in hundredths of kWh. - 1.3.6.1.4.1.318.1.1.1.4.3.6
  - name: upsAdvTestDiagnosticSchedule
    oid: 1.3.6.1.4.1.318.1.1.1.7.2.1
    type: gauge
    help: The UPS system's automatic battery test schedule. - 1.3.6.1.4.1.318.1.1.1.7.2.1
  - name: upsAdvTestDiagnostics
    oid: 1.3.6.1.4.1.318.1.1.1.7.2.2
    type: gauge
    help: Setting this variable to testDiagnostics(2) causes the UPS to perform a
      diagnostic self test - 1.3.6.1.4.1.318.1.1.1.7.2.2
  - name: upsAdvTestDiagnosticsResults
    oid: 1.3.6.1.4.1.318.1.1.1.7.2.3
    type: gauge
    help: The results of the last UPS diagnostics test performed. - 1.3.6.1.4.1.318.1.1.1.7.2.3
  - name: upsAdvTestLastDiagnosticsDate
    oid: 1.3.6.1.4.1.318.1.1.1.7.2.4
    type: OctetString
    help: The date the last UPS diagnostics test was performed in mm/dd/yy format.
      - 1.3.6.1.4.1.318.1.1.1.7.2.4
  - name: upsAdvTestRuntimeCalibration
    oid: 1.3.6.1.4.1.318.1.1.1.7.2.5
    type: gauge
    help: Setting this variable to performCalibration(2) causes the UPS to discharge
      to calibrate the UPS - 1.3.6.1.4.1.318.1.1.1.7.2.5
  - name: upsAdvTestCalibrationResults
    oid: 1.3.6.1.4.1.318.1.1.1.7.2.6
    type: gauge
    help: The results of the last runtime calibration - 1.3.6.1.4.1.318.1.1.1.7.2.6
  - name: upsAdvTestCalibrationDate
    oid: 1.3.6.1.4.1.318.1.1.1.7.2.7
    type: OctetString
    help: The date the last UPS runtime calibration was performed in mm/dd/yy format.
      - 1.3.6.1.4.1.318.1.1.1.7.2.7
  - name: upsAdvTestDiagnosticTime
    oid: 1.3.6.1.4.1.318.1.1.1.7.2.8
    type: OctetString
    help: The time of the day to perform automatic battery test - 1.3.6.1.4.1.318.1.1.1.7.2.8
  - name: upsAdvTestDiagnosticDay
    oid: 1.3.6.1.4.1.318.1.1.1.7.2.9
    type: gauge
    help: The day of the week to perform automatic battery test. - 1.3.6.1.4.1.318.1.1.1.7.2.9
  - name: upsCommStatus
    oid: 1.3.6.1.4.1.318.1.1.1.8.1
    type: gauge
    help: The status of agent's communication with UPS. - 1.3.6.1.4.1.318.1.1.1.8.1
  - name: iemStatusProbeNumber
    oid: 1.3.6.1.4.1.318.1.1.10.2.3.2.1.1
    type: gauge
    help: The index of the probe. - 1.3.6.1.4.1.318.1.1.10.2.3.2.1.1
    indexes:
    - labelname: iemStatusProbeNumber
      type: gauge
  - name: iemStatusProbeName
    oid: 1.3.6.1.4.1.318.1.1.10.2.3.2.1.2
    type: OctetString
    help: A descriptive name for the probe set by the user. - 1.3.6.1.4.1.318.1.1.10.2.3.2.1.2
    indexes:
    - labelname: iemStatusProbeNumber
      type: gauge
  - name: iemStatusProbeStatus
    oid: 1.3.6.1.4.1.318.1.1.10.2.3.2.1.3
    type: gauge
    help: The connected status of the probe, either disconnected(1) or connected(2).
      - 1.3.6.1.4.1.318.1.1.10.2.3.2.1.3
    indexes:
    - labelname: iemStatusProbeNumber
      type: gauge
  - name: iemStatusProbeCurrentTemp
    oid: 1.3.6.1.4.1.318.1.1.10.2.3.2.1.4
    type: gauge
    help: The current temperature reading from the probe displayed in the units shown
      in the 'iemStatusProbeTempUnits' OID (Celsius or Fahrenheit). - 1.3.6.1.4.1.318.1.1.10.2.3.2.1.4
    indexes:
    - labelname: iemStatusProbeNumber
      type: gauge
  - name: iemStatusProbeTempUnits
    oid: 1.3.6.1.4.1.318.1.1.10.2.3.2.1.5
    type: gauge
    help: The temperature scale used to display the temperature thresholds of the
      probe, Celsius(1) or Fahrenheit(2) - 1.3.6.1.4.1.318.1.1.10.2.3.2.1.5
    indexes:
    - labelname: iemStatusProbeNumber
      type: gauge
  - name: iemStatusProbeCurrentHumid
    oid: 1.3.6.1.4.1.318.1.1.10.2.3.2.1.6
    type: gauge
    help: The current humidity reading from the probe in percent relative humidity.
      - 1.3.6.1.4.1.318.1.1.10.2.3.2.1.6
    indexes:
    - labelname: iemStatusProbeNumber
      type: gauge
  - name: iemStatusProbeHighTempViolation
    oid: 1.3.6.1.4.1.318.1.1.10.2.3.2.1.7
    type: gauge
    help: The high temperature violation status of the probe temperature reading -
      1.3.6.1.4.1.318.1.1.10.2.3.2.1.7
    indexes:
    - labelname: iemStatusProbeNumber
      type: gauge
  - name: iemStatusProbeLowTempViolation
    oid: 1.3.6.1.4.1.318.1.1.10.2.3.2.1.8
    type: gauge
    help: The high temperature violation status of the probe temperature reading -
      1.3.6.1.4.1.318.1.1.10.2.3.2.1.8
    indexes:
    - labelname: iemStatusProbeNumber
      type: gauge
  - name: iemStatusProbeHighHumidViolation
    oid: 1.3.6.1.4.1.318.1.1.10.2.3.2.1.9
    type: gauge
    help: The high humidity violation status of the probe humidity reading - 1.3.6.1.4.1.318.1.1.10.2.3.2.1.9
    indexes:
    - labelname: iemStatusProbeNumber
      type: gauge
  - name: iemStatusProbeLowHumidViolation
    oid: 1.3.6.1.4.1.318.1.1.10.2.3.2.1.10
    type: gauge
    help: The low humidity violation status of the probe humidity reading - 1.3.6.1.4.1.318.1.1.10.2.3.2.1.10
    indexes:
    - labelname: iemStatusProbeNumber
      type: gauge
  - name: iemStatusProbeMaxTempViolation
    oid: 1.3.6.1.4.1.318.1.1.10.2.3.2.1.11
    type: gauge
    help: The max temperature violation status of the probe temperature reading -
      1.3.6.1.4.1.318.1.1.10.2.3.2.1.11
    indexes:
    - labelname: iemStatusProbeNumber
      type: gauge
  - name: iemStatusProbeMinTempViolation
    oid: 1.3.6.1.4.1.318.1.1.10.2.3.2.1.12
    type: gauge
    help: The minimum temperature violation status of the probe temperature reading
      - 1.3.6.1.4.1.318.1.1.10.2.3.2.1.12
    indexes:
    - labelname: iemStatusProbeNumber
      type: gauge
  - name: iemStatusProbeMaxHumidViolation
    oid: 1.3.6.1.4.1.318.1.1.10.2.3.2.1.13
    type: gauge
    help: The maximum humidity violation status of the probe humidity reading - 1.3.6.1.4.1.318.1.1.10.2.3.2.1.13
    indexes:
    - labelname: iemStatusProbeNumber
      type: gauge
  - name: iemStatusProbeMinHumidViolation
    oid: 1.3.6.1.4.1.318.1.1.10.2.3.2.1.14
    type: gauge
    help: The minimum humidity violation status of the probe humidity reading - 1.3.6.1.4.1.318.1.1.10.2.3.2.1.14
    indexes:
    - labelname: iemStatusProbeNumber
      type: gauge
  - name: iemStatusProbeLocation
    oid: 1.3.6.1.4.1.318.1.1.10.2.3.2.1.15
    type: OctetString
    help: A descriptive location for the probe set by the user. - 1.3.6.1.4.1.318.1.1.10.2.3.2.1.15
    indexes:
    - labelname: iemStatusProbeNumber
      type: gauge
  version: 1
arista_sw:
  walk:
  - 1.3.6.1.2.1.1.3
  - 1.3.6.1.2.1.2
  - 1.3.6.1.2.1.25.2.3.1.6
  - 1.3.6.1.2.1.25.3.3.1.2
  - 1.3.6.1.2.1.31.1.1
  - 1.3.6.1.4.1.30065.3.1.1
  metrics:
  - name: sysUpTime
    oid: 1.3.6.1.2.1.1.3
    type: gauge
    help: The time (in hundredths of a second) since the network management portion
      of the system was last re-initialized. - 1.3.6.1.2.1.1.3
  - name: ifNumber
    oid: 1.3.6.1.2.1.2.1
    type: gauge
    help: The number of network interfaces (regardless of their current state) present
      on this system. - 1.3.6.1.2.1.2.1
  - name: ifIndex
    oid: 1.3.6.1.2.1.2.2.1.1
    type: gauge
    help: A unique value, greater than zero, for each interface - 1.3.6.1.2.1.2.2.1.1
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifDescr
    oid: 1.3.6.1.2.1.2.2.1.2
    type: DisplayString
    help: A textual string containing information about the interface - 1.3.6.1.2.1.2.2.1.2
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifType
    oid: 1.3.6.1.2.1.2.2.1.3
    type: gauge
    help: The type of interface - 1.3.6.1.2.1.2.2.1.3
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifMtu
    oid: 1.3.6.1.2.1.2.2.1.4
    type: gauge
    help: The size of the largest packet which can be sent/received on the interface,
      specified in octets - 1.3.6.1.2.1.2.2.1.4
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifSpeed
    oid: 1.3.6.1.2.1.2.2.1.5
    type: gauge
    help: An estimate of the interface's current bandwidth in bits per second - 1.3.6.1.2.1.2.2.1.5
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifPhysAddress
    oid: 1.3.6.1.2.1.2.2.1.6
    type: PhysAddress48
    help: The interface's address at its protocol sub-layer - 1.3.6.1.2.1.2.2.1.6
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifAdminStatus
    oid: 1.3.6.1.2.1.2.2.1.7
    type: gauge
    help: The desired state of the interface - 1.3.6.1.2.1.2.2.1.7
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOperStatus
    oid: 1.3.6.1.2.1.2.2.1.8
    type: gauge
    help: The current operational state of the interface - 1.3.6.1.2.1.2.2.1.8
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifLastChange
    oid: 1.3.6.1.2.1.2.2.1.9
    type: gauge
    help: The value of sysUpTime at the time the interface entered its current operational
      state - 1.3.6.1.2.1.2.2.1.9
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInOctets
    oid: 1.3.6.1.2.1.2.2.1.10
    type: counter
    help: The total number of octets received on the interface, including framing
      characters - 1.3.6.1.2.1.2.2.1.10
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.11
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were not addressed to a multicast or broadcast address at this sub-layer
      - 1.3.6.1.2.1.2.2.1.11
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInNUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.12
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a multicast or broadcast address at this sub-layer -
      1.3.6.1.2.1.2.2.1.12
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInDiscards
    oid: 1.3.6.1.2.1.2.2.1.13
    type: counter
    help: The number of inbound packets which were chosen to be discarded even though
      no errors had been detected to prevent their being deliverable to a higher-layer
      protocol - 1.3.6.1.2.1.2.2.1.13
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInErrors
    oid: 1.3.6.1.2.1.2.2.1.14
    type: counter
    help: For packet-oriented interfaces, the number of inbound packets that contained
      errors preventing them from being deliverable to a higher-layer protocol - 1.3.6.1.2.1.2.2.1.14
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInUnknownProtos
    oid: 1.3.6.1.2.1.2.2.1.15
    type: counter
    help: For packet-oriented interfaces, the number of packets received via the interface
      which were discarded because of an unknown or unsupported protocol - 1.3.6.1.2.1.2.2.1.15
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutOctets
    oid: 1.3.6.1.2.1.2.2.1.16
    type: counter
    help: The total number of octets transmitted out of the interface, including framing
      characters - 1.3.6.1.2.1.2.2.1.16
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.17
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were not addressed to a multicast or broadcast address at this sub-layer,
      including those that were discarded or not sent - 1.3.6.1.2.1.2.2.1.17
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutNUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.18
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a multicast or broadcast address at this sub-layer,
      including those that were discarded or not sent - 1.3.6.1.2.1.2.2.1.18
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutDiscards
    oid: 1.3.6.1.2.1.2.2.1.19
    type: counter
    help: The number of outbound packets which were chosen to be discarded even though
      no errors had been detected to prevent their being transmitted - 1.3.6.1.2.1.2.2.1.19
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutErrors
    oid: 1.3.6.1.2.1.2.2.1.20
    type: counter
    help: For packet-oriented interfaces, the number of outbound packets that could
      not be transmitted because of errors - 1.3.6.1.2.1.2.2.1.20
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutQLen
    oid: 1.3.6.1.2.1.2.2.1.21
    type: gauge
    help: The length of the output packet queue (in packets). - 1.3.6.1.2.1.2.2.1.21
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: hrStorageUsed
    oid: 1.3.6.1.2.1.25.2.3.1.6
    type: gauge
    help: The amount of the storage represented by this entry that is allocated, in
      units of hrStorageAllocationUnits. - 1.3.6.1.2.1.25.2.3.1.6
    indexes:
    - labelname: hrStorageIndex
      type: gauge
  - name: hrProcessorLoad
    oid: 1.3.6.1.2.1.25.3.3.1.2
    type: gauge
    help: The average, over the last minute, of the percentage of time that this processor
      was not idle - 1.3.6.1.2.1.25.3.3.1.2
    indexes:
    - labelname: hrDeviceIndex
      type: gauge
  - name: ifName
    oid: 1.3.6.1.2.1.31.1.1.1.1
    type: DisplayString
    help: The textual name of the interface - 1.3.6.1.2.1.31.1.1.1.1
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.2
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a multicast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.2
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.3
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a broadcast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.3
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.4
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a multicast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.4
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.5
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a broadcast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.5
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifHCInOctets
    oid: 1.3.6.1.2.1.31.1.1.1.6
    type: counter
    help: The total number of octets received on the interface, including framing
      characters - 1.3.6.1.2.1.31.1.1.1.6
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifHCInUcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.7
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were not addressed to a multicast or broadcast address at this sub-layer
      - 1.3.6.1.2.1.31.1.1.1.7
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifHCInMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.8
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a multicast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.8
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifHCInBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.9
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a broadcast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.9
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifHCOutOctets
    oid: 1.3.6.1.2.1.31.1.1.1.10
    type: counter
    help: The total number of octets transmitted out of the interface, including framing
      characters - 1.3.6.1.2.1.31.1.1.1.10
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifHCOutUcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.11
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were not addressed to a multicast or broadcast address at this sub-layer,
      including those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.11
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifHCOutMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.12
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a multicast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.12
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifHCOutBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.13
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a broadcast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.13
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifLinkUpDownTrapEnable
    oid: 1.3.6.1.2.1.31.1.1.1.14
    type: gauge
    help: Indicates whether linkUp/linkDown traps should be generated for this interface
      - 1.3.6.1.2.1.31.1.1.1.14
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifHighSpeed
    oid: 1.3.6.1.2.1.31.1.1.1.15
    type: gauge
    help: An estimate of the interface's current bandwidth in units of 1,000,000 bits
      per second - 1.3.6.1.2.1.31.1.1.1.15
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifPromiscuousMode
    oid: 1.3.6.1.2.1.31.1.1.1.16
    type: gauge
    help: This object has a value of false(2) if this interface only accepts packets/frames
      that are addressed to this station - 1.3.6.1.2.1.31.1.1.1.16
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifConnectorPresent
    oid: 1.3.6.1.2.1.31.1.1.1.17
    type: gauge
    help: This object has the value 'true(1)' if the interface sublayer has a physical
      connector and the value 'false(2)' otherwise. - 1.3.6.1.2.1.31.1.1.1.17
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifAlias
    oid: 1.3.6.1.2.1.31.1.1.1.18
    type: DisplayString
    help: This object is an 'alias' name for the interface as specified by a network
      manager, and provides a non-volatile 'handle' for the interface - 1.3.6.1.2.1.31.1.1.1.18
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifCounterDiscontinuityTime
    oid: 1.3.6.1.2.1.31.1.1.1.19
    type: gauge
    help: The value of sysUpTime on the most recent occasion at which any one or more
      of this interface's counters suffered a discontinuity - 1.3.6.1.2.1.31.1.1.1.19
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: aristaSwFwdIpStatsIPVersion
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.1
    type: gauge
    help: The IP version of this row. - 1.3.6.1.4.1.30065.3.1.1.1.1.1
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsInReceives
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.3
    type: counter
    help: The total number of input IP datagrams received in software, including those
      received in error - 1.3.6.1.4.1.30065.3.1.1.1.1.3
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsHCInReceives
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.4
    type: counter
    help: The total number of input IP datagrams received in software, including those
      received in error - 1.3.6.1.4.1.30065.3.1.1.1.1.4
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsInOctets
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.5
    type: counter
    help: The total number of octets received in software in input IP datagrams, including
      those received in error - 1.3.6.1.4.1.30065.3.1.1.1.1.5
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsHCInOctets
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.6
    type: counter
    help: The total number of octets received in software in input IP datagrams, including
      those received in error - 1.3.6.1.4.1.30065.3.1.1.1.1.6
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsInHdrErrors
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.7
    type: counter
    help: The number of input IP datagrams discarded in software due to errors in
      their IP headers, including version number mismatch, other format errors, hop
      count exceeded, errors discovered in processing their IP options, etc - 1.3.6.1.4.1.30065.3.1.1.1.1.7
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsInNoRoutes
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.8
    type: counter
    help: The number of input IP datagrams discarded in software because no route
      could be found to transmit them to their destination - 1.3.6.1.4.1.30065.3.1.1.1.1.8
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsInAddrErrors
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.9
    type: counter
    help: The number of input IP datagrams discarded in software because the IP address
      in their IP header's destination field was not a valid address to be received
      at this entity - 1.3.6.1.4.1.30065.3.1.1.1.1.9
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsInUnknownProtos
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.10
    type: counter
    help: The number of locally-addressed IP datagrams received successfully in software
      but discarded because of an unknown or unsupported protocol - 1.3.6.1.4.1.30065.3.1.1.1.1.10
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsInTruncatedPkts
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.11
    type: counter
    help: The number of input IP datagrams discarded in software because the datagram
      frame didn't carry enough data - 1.3.6.1.4.1.30065.3.1.1.1.1.11
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsInForwDatagrams
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.12
    type: counter
    help: The number of input datagrams for which this entity was not their final
      IP destination and for which this entity attempted in software to find a route
      to forward them to that final destination - 1.3.6.1.4.1.30065.3.1.1.1.1.12
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsHCInForwDatagrams
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.13
    type: counter
    help: The number of input datagrams for which this entity was not their final
      IP destination and for which this entity attempted in software to find a route
      to forward them to that final destination - 1.3.6.1.4.1.30065.3.1.1.1.1.13
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsReasmReqds
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.14
    type: counter
    help: The number of IP fragments received that needed to be reassembled at this
      interface - 1.3.6.1.4.1.30065.3.1.1.1.1.14
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsReasmOKs
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.15
    type: counter
    help: The number of IP datagrams successfully reassembled - 1.3.6.1.4.1.30065.3.1.1.1.1.15
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsReasmFails
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.16
    type: counter
    help: 'The number of failures detected by the IP re-assembly algorithm (for whatever
      reason: timed out, errors, etc.) - 1.3.6.1.4.1.30065.3.1.1.1.1.16'
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsInDiscards
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.17
    type: counter
    help: The number of input IP datagrams received in software for which no problems
      were encountered to prevent their continued processing, but were discarded (e.g.,
      for lack of buffer space) - 1.3.6.1.4.1.30065.3.1.1.1.1.17
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsInDelivers
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.18
    type: counter
    help: The total number of datagrams successfully delivered to IP user-protocols
      (including ICMP) - 1.3.6.1.4.1.30065.3.1.1.1.1.18
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsHCInDelivers
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.19
    type: counter
    help: The total number of datagrams successfully delivered to IP user-protocols
      (including ICMP) - 1.3.6.1.4.1.30065.3.1.1.1.1.19
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsOutRequests
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.20
    type: counter
    help: The total number of IP datagrams that local IP user- protocols (including
      ICMP) supplied to IP in requests for transmission - 1.3.6.1.4.1.30065.3.1.1.1.1.20
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsHCOutRequests
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.21
    type: counter
    help: The total number of IP datagrams that local IP user- protocols (including
      ICMP) supplied to IP in requests for transmission - 1.3.6.1.4.1.30065.3.1.1.1.1.21
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsOutNoRoutes
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.22
    type: counter
    help: The number of locally generated IP datagrams discarded because no route
      could be found to transmit them to their destination - 1.3.6.1.4.1.30065.3.1.1.1.1.22
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsOutForwDatagrams
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.23
    type: counter
    help: The number of datagrams for which this entity was not their final IP destination
      and for which it was successful in finding a path to their final destination
      in software - 1.3.6.1.4.1.30065.3.1.1.1.1.23
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsHCOutForwDatagrams
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.24
    type: counter
    help: The number of datagrams for which this entity was not their final IP destination
      and for which it was successful in finding a path to their final destination
      in software - 1.3.6.1.4.1.30065.3.1.1.1.1.24
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsOutDiscards
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.25
    type: counter
    help: The number of output IP datagrams for which no problem was encountered to
      prevent their transmission to their destination, but were discarded in software
      (e.g., for lack of buffer space) - 1.3.6.1.4.1.30065.3.1.1.1.1.25
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsOutFragReqds
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.26
    type: counter
    help: The number of IP datagrams that would require fragmentation in order to
      be transmitted - 1.3.6.1.4.1.30065.3.1.1.1.1.26
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsOutFragOKs
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.27
    type: counter
    help: The number of IP datagrams that have been successfully fragmented - 1.3.6.1.4.1.30065.3.1.1.1.1.27
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsOutFragFails
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.28
    type: counter
    help: The number of IP datagrams that have been discarded because they needed
      to be fragmented but could not be - 1.3.6.1.4.1.30065.3.1.1.1.1.28
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsOutFragCreates
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.29
    type: counter
    help: The number of output datagram fragments that have been generated as a result
      of IP fragmentation - 1.3.6.1.4.1.30065.3.1.1.1.1.29
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsOutTransmits
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.30
    type: counter
    help: The total number of IP datagrams that this entity supplied by software to
      the lower layers for transmission - 1.3.6.1.4.1.30065.3.1.1.1.1.30
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsHCOutTransmits
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.31
    type: counter
    help: The total number of IP datagrams that this entity supplied by software to
      the lower layers for transmission - 1.3.6.1.4.1.30065.3.1.1.1.1.31
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsOutOctets
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.32
    type: counter
    help: The total number of octets in IP datagrams delivered by software to the
      lower layers for transmission - 1.3.6.1.4.1.30065.3.1.1.1.1.32
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsHCOutOctets
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.33
    type: counter
    help: The total number of octets in IP datagrams delivered by software to the
      lower layers for transmission - 1.3.6.1.4.1.30065.3.1.1.1.1.33
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsInMcastPkts
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.34
    type: counter
    help: The number of IP multicast datagrams received by software - 1.3.6.1.4.1.30065.3.1.1.1.1.34
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsHCInMcastPkts
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.35
    type: counter
    help: The number of IP multicast datagrams received by software - 1.3.6.1.4.1.30065.3.1.1.1.1.35
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsInMcastOctets
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.36
    type: counter
    help: The total number of octets received by software in IP multicast datagrams
      - 1.3.6.1.4.1.30065.3.1.1.1.1.36
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsHCInMcastOctets
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.37
    type: counter
    help: The total number of octets received by software in IP multicast datagrams
      - 1.3.6.1.4.1.30065.3.1.1.1.1.37
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsOutMcastPkts
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.38
    type: counter
    help: The number of IP multicast datagrams transmitted by software - 1.3.6.1.4.1.30065.3.1.1.1.1.38
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsHCOutMcastPkts
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.39
    type: counter
    help: The number of IP multicast datagrams transmitted by software - 1.3.6.1.4.1.30065.3.1.1.1.1.39
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsOutMcastOctets
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.40
    type: counter
    help: The total number of octets transmitted by software in IP multicast datagrams
      - 1.3.6.1.4.1.30065.3.1.1.1.1.40
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsHCOutMcastOctets
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.41
    type: counter
    help: The total number of octets transmitted by software in IP multicast datagrams
      - 1.3.6.1.4.1.30065.3.1.1.1.1.41
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsInBcastPkts
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.42
    type: counter
    help: The number of IP broadcast datagrams received by software - 1.3.6.1.4.1.30065.3.1.1.1.1.42
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsHCInBcastPkts
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.43
    type: counter
    help: The number of IP broadcast datagrams received by software - 1.3.6.1.4.1.30065.3.1.1.1.1.43
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsOutBcastPkts
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.44
    type: counter
    help: The number of IP broadcast datagrams transmitted by software - 1.3.6.1.4.1.30065.3.1.1.1.1.44
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsHCOutBcastPkts
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.45
    type: counter
    help: The number of IP broadcast datagrams transmitted by software - 1.3.6.1.4.1.30065.3.1.1.1.1.45
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsDiscontinuityTime
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.46
    type: gauge
    help: The value of sysUpTime on the most recent occasion at which any one or more
      of this entry's counters suffered a discontinuity - 1.3.6.1.4.1.30065.3.1.1.1.1.46
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
  - name: aristaSwFwdIpStatsRefreshRate
    oid: 1.3.6.1.4.1.30065.3.1.1.1.1.47
    type: gauge
    help: The minimum reasonable polling interval for this entry - 1.3.6.1.4.1.30065.3.1.1.1.1.47
    indexes:
    - labelname: aristaSwFwdIpStatsIPVersion
      type: gauge
cisco_wlc:
  walk:
  - 1.3.6.1.2.1.2
  - 1.3.6.1.2.1.31.1.1
  - 1.3.6.1.4.1.14179.2.1.1.1.2
  - 1.3.6.1.4.1.14179.2.1.1.1.38
  - 1.3.6.1.4.1.14179.2.2.1.1.3
  - 1.3.6.1.4.1.14179.2.2.13.1.3
  - 1.3.6.1.4.1.14179.2.2.15.1.21
  - 1.3.6.1.4.1.14179.2.2.2.1.15
  - 1.3.6.1.4.1.14179.2.2.2.1.2
  - 1.3.6.1.4.1.14179.2.2.2.1.4
  - 1.3.6.1.4.1.14179.2.2.6.1
  metrics:
  - name: ifNumber
    oid: 1.3.6.1.2.1.2.1
    type: gauge
    help: The number of network interfaces (regardless of their current state) present
      on this system. - 1.3.6.1.2.1.2.1
  - name: ifIndex
    oid: 1.3.6.1.2.1.2.2.1.1
    type: gauge
    help: A unique value, greater than zero, for each interface - 1.3.6.1.2.1.2.2.1.1
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifDescr
    oid: 1.3.6.1.2.1.2.2.1.2
    type: DisplayString
    help: A textual string containing information about the interface - 1.3.6.1.2.1.2.2.1.2
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifType
    oid: 1.3.6.1.2.1.2.2.1.3
    type: gauge
    help: The type of interface - 1.3.6.1.2.1.2.2.1.3
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifMtu
    oid: 1.3.6.1.2.1.2.2.1.4
    type: gauge
    help: The size of the largest packet which can be sent/received on the interface,
      specified in octets - 1.3.6.1.2.1.2.2.1.4
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifSpeed
    oid: 1.3.6.1.2.1.2.2.1.5
    type: gauge
    help: An estimate of the interface's current bandwidth in bits per second - 1.3.6.1.2.1.2.2.1.5
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifPhysAddress
    oid: 1.3.6.1.2.1.2.2.1.6
    type: PhysAddress48
    help: The interface's address at its protocol sub-layer - 1.3.6.1.2.1.2.2.1.6
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifAdminStatus
    oid: 1.3.6.1.2.1.2.2.1.7
    type: gauge
    help: The desired state of the interface - 1.3.6.1.2.1.2.2.1.7
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOperStatus
    oid: 1.3.6.1.2.1.2.2.1.8
    type: gauge
    help: The current operational state of the interface - 1.3.6.1.2.1.2.2.1.8
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifLastChange
    oid: 1.3.6.1.2.1.2.2.1.9
    type: gauge
    help: The value of sysUpTime at the time the interface entered its current operational
      state - 1.3.6.1.2.1.2.2.1.9
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInOctets
    oid: 1.3.6.1.2.1.2.2.1.10
    type: counter
    help: The total number of octets received on the interface, including framing
      characters - 1.3.6.1.2.1.2.2.1.10
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.11
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were not addressed to a multicast or broadcast address at this sub-layer
      - 1.3.6.1.2.1.2.2.1.11
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInNUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.12
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a multicast or broadcast address at this sub-layer -
      1.3.6.1.2.1.2.2.1.12
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInDiscards
    oid: 1.3.6.1.2.1.2.2.1.13
    type: counter
    help: The number of inbound packets which were chosen to be discarded even though
      no errors had been detected to prevent their being deliverable to a higher-layer
      protocol - 1.3.6.1.2.1.2.2.1.13
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInErrors
    oid: 1.3.6.1.2.1.2.2.1.14
    type: counter
    help: For packet-oriented interfaces, the number of inbound packets that contained
      errors preventing them from being deliverable to a higher-layer protocol - 1.3.6.1.2.1.2.2.1.14
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInUnknownProtos
    oid: 1.3.6.1.2.1.2.2.1.15
    type: counter
    help: For packet-oriented interfaces, the number of packets received via the interface
      which were discarded because of an unknown or unsupported protocol - 1.3.6.1.2.1.2.2.1.15
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutOctets
    oid: 1.3.6.1.2.1.2.2.1.16
    type: counter
    help: The total number of octets transmitted out of the interface, including framing
      characters - 1.3.6.1.2.1.2.2.1.16
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.17
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were not addressed to a multicast or broadcast address at this sub-layer,
      including those that were discarded or not sent - 1.3.6.1.2.1.2.2.1.17
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutNUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.18
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a multicast or broadcast address at this sub-layer,
      including those that were discarded or not sent - 1.3.6.1.2.1.2.2.1.18
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutDiscards
    oid: 1.3.6.1.2.1.2.2.1.19
    type: counter
    help: The number of outbound packets which were chosen to be discarded even though
      no errors had been detected to prevent their being transmitted - 1.3.6.1.2.1.2.2.1.19
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutErrors
    oid: 1.3.6.1.2.1.2.2.1.20
    type: counter
    help: For packet-oriented interfaces, the number of outbound packets that could
      not be transmitted because of errors - 1.3.6.1.2.1.2.2.1.20
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutQLen
    oid: 1.3.6.1.2.1.2.2.1.21
    type: gauge
    help: The length of the output packet queue (in packets). - 1.3.6.1.2.1.2.2.1.21
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifName
    oid: 1.3.6.1.2.1.31.1.1.1.1
    type: DisplayString
    help: The textual name of the interface - 1.3.6.1.2.1.31.1.1.1.1
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.2
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a multicast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.2
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.3
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a broadcast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.3
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.4
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a multicast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.4
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.5
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a broadcast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.5
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifHCInOctets
    oid: 1.3.6.1.2.1.31.1.1.1.6
    type: counter
    help: The total number of octets received on the interface, including framing
      characters - 1.3.6.1.2.1.31.1.1.1.6
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifHCInUcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.7
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were not addressed to a multicast or broadcast address at this sub-layer
      - 1.3.6.1.2.1.31.1.1.1.7
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifHCInMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.8
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a multicast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.8
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifHCInBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.9
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a broadcast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.9
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifHCOutOctets
    oid: 1.3.6.1.2.1.31.1.1.1.10
    type: counter
    help: The total number of octets transmitted out of the interface, including framing
      characters - 1.3.6.1.2.1.31.1.1.1.10
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifHCOutUcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.11
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were not addressed to a multicast or broadcast address at this sub-layer,
      including those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.11
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifHCOutMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.12
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a multicast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.12
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifHCOutBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.13
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a broadcast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.13
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifLinkUpDownTrapEnable
    oid: 1.3.6.1.2.1.31.1.1.1.14
    type: gauge
    help: Indicates whether linkUp/linkDown traps should be generated for this interface
      - 1.3.6.1.2.1.31.1.1.1.14
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifHighSpeed
    oid: 1.3.6.1.2.1.31.1.1.1.15
    type: gauge
    help: An estimate of the interface's current bandwidth in units of 1,000,000 bits
      per second - 1.3.6.1.2.1.31.1.1.1.15
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifPromiscuousMode
    oid: 1.3.6.1.2.1.31.1.1.1.16
    type: gauge
    help: This object has a value of false(2) if this interface only accepts packets/frames
      that are addressed to this station - 1.3.6.1.2.1.31.1.1.1.16
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifConnectorPresent
    oid: 1.3.6.1.2.1.31.1.1.1.17
    type: gauge
    help: This object has the value 'true(1)' if the interface sublayer has a physical
      connector and the value 'false(2)' otherwise. - 1.3.6.1.2.1.31.1.1.1.17
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifAlias
    oid: 1.3.6.1.2.1.31.1.1.1.18
    type: DisplayString
    help: This object is an 'alias' name for the interface as specified by a network
      manager, and provides a non-volatile 'handle' for the interface - 1.3.6.1.2.1.31.1.1.1.18
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifCounterDiscontinuityTime
    oid: 1.3.6.1.2.1.31.1.1.1.19
    type: gauge
    help: The value of sysUpTime on the most recent occasion at which any one or more
      of this interface's counters suffered a discontinuity - 1.3.6.1.2.1.31.1.1.1.19
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: bsnDot11EssNumberOfMobileStations
    oid: 1.3.6.1.4.1.14179.2.1.1.1.38
    type: counter
    help: No of Mobile Stations currently associated with the WLAN. - 1.3.6.1.4.1.14179.2.1.1.1.38
    indexes:
    - labelname: bsnDot11EssSsid
      type: gauge
    lookups:
    - labels:
      - bsnDot11EssSsid
      labelname: bsnDot11EssSsid
      oid: 1.3.6.1.4.1.14179.2.1.1.1.2
      type: DisplayString
  - name: bsnAPIfLoadChannelUtilization
    oid: 1.3.6.1.4.1.14179.2.2.13.1.3
    type: gauge
    help: Channel Utilization - 1.3.6.1.4.1.14179.2.2.13.1.3
    indexes:
    - labelname: bsnAPName
      type: PhysAddress48
    - labelname: bsnAPIfSlotId
      type: gauge
    lookups:
    - labels:
      - bsnAPName
      labelname: bsnAPName
      oid: 1.3.6.1.4.1.14179.2.2.1.1.3
      type: OctetString
  - name: bsnAPIfDBNoisePower
    oid: 1.3.6.1.4.1.14179.2.2.15.1.21
    type: gauge
    help: This is the average noise power in dBm on each channel that is available
      to Airespace AP - 1.3.6.1.4.1.14179.2.2.15.1.21
    indexes:
    - labelname: bsnAPName
      type: PhysAddress48
    - labelname: bsnAPIfSlotId
      type: gauge
    - labelname: bsnAPIfNoiseChannelNo
      type: gauge
    lookups:
    - labels:
      - bsnAPName
      labelname: bsnAPName
      oid: 1.3.6.1.4.1.14179.2.2.1.1.3
      type: OctetString
  - name: bsnApIfNoOfUsers
    oid: 1.3.6.1.4.1.14179.2.2.2.1.15
    type: counter
    help: No of Users associated with this radio. - 1.3.6.1.4.1.14179.2.2.2.1.15
    indexes:
    - labelname: bsnAPName
      type: PhysAddress48
    - labelname: bsnAPIfSlotId
      type: gauge
    lookups:
    - labels:
      - bsnAPName
      labelname: bsnAPName
      oid: 1.3.6.1.4.1.14179.2.2.1.1.3
      type: OctetString
  - name: bsnAPIfType
    oid: 1.3.6.1.4.1.14179.2.2.2.1.2
    type: gauge
    help: The type of this interface - 1.3.6.1.4.1.14179.2.2.2.1.2
    indexes:
    - labelname: bsnAPName
      type: PhysAddress48
    - labelname: bsnAPIfSlotId
      type: gauge
    lookups:
    - labels:
      - bsnAPName
      labelname: bsnAPName
      oid: 1.3.6.1.4.1.14179.2.2.1.1.3
      type: OctetString
  - name: bsnAPIfPhyChannelNumber
    oid: 1.3.6.1.4.1.14179.2.2.2.1.4
    type: gauge
    help: Current channel number of the AP Interface - 1.3.6.1.4.1.14179.2.2.2.1.4
    indexes:
    - labelname: bsnAPName
      type: PhysAddress48
    - labelname: bsnAPIfSlotId
      type: gauge
    lookups:
    - labels:
      - bsnAPName
      labelname: bsnAPName
      oid: 1.3.6.1.4.1.14179.2.2.1.1.3
      type: OctetString
  - name: bsnAPIfDot11TransmittedFragmentCount
    oid: 1.3.6.1.4.1.14179.2.2.6.1.1
    type: counter
    help: This counter shall be incremented for an acknowledged MPDU with an individual
      address in the address 1 field or an MPDU with a multicast address in the address
      1 field of type Data or Management. - 1.3.6.1.4.1.14179.2.2.6.1.1
    indexes:
    - labelname: bsnAPName
      type: PhysAddress48
    - labelname: bsnAPIfSlotId
      type: gauge
    lookups:
    - labels:
      - bsnAPName
      labelname: bsnAPName
      oid: 1.3.6.1.4.1.14179.2.2.1.1.3
      type: OctetString
  - name: bsnAPIfDot11MulticastTransmittedFrameCount
    oid: 1.3.6.1.4.1.14179.2.2.6.1.2
    type: counter
    help: This counter shall increment only when the multicast bit is set in the destination
      MAC address of a successfully transmitted MSDU - 1.3.6.1.4.1.14179.2.2.6.1.2
    indexes:
    - labelname: bsnAPName
      type: PhysAddress48
    - labelname: bsnAPIfSlotId
      type: gauge
    lookups:
    - labels:
      - bsnAPName
      labelname: bsnAPName
      oid: 1.3.6.1.4.1.14179.2.2.1.1.3
      type: OctetString
  - name: bsnAPIfDot11RetryCount
    oid: 1.3.6.1.4.1.14179.2.2.6.1.3
    type: counter
    help: This counter shall increment when an MSDU is successfully transmitted after
      one or more retransmissions. - 1.3.6.1.4.1.14179.2.2.6.1.3
    indexes:
    - labelname: bsnAPName
      type: PhysAddress48
    - labelname: bsnAPIfSlotId
      type: gauge
    lookups:
    - labels:
      - bsnAPName
      labelname: bsnAPName
      oid: 1.3.6.1.4.1.14179.2.2.1.1.3
      type: OctetString
  - name: bsnAPIfDot11MultipleRetryCount
    oid: 1.3.6.1.4.1.14179.2.2.6.1.4
    type: counter
    help: This counter shall increment when an MSDU is successfully transmitted after
      more than one retransmission. - 1.3.6.1.4.1.14179.2.2.6.1.4
    indexes:
    - labelname: bsnAPName
      type: PhysAddress48
    - labelname: bsnAPIfSlotId
      type: gauge
    lookups:
    - labels:
      - bsnAPName
      labelname: bsnAPName
      oid: 1.3.6.1.4.1.14179.2.2.1.1.3
      type: OctetString
  - name: bsnAPIfDot11FrameDuplicateCount
    oid: 1.3.6.1.4.1.14179.2.2.6.1.5
    type: counter
    help: This counter shall increment when a frame is received that the Sequence
      Control field indicates is a duplicate. - 1.3.6.1.4.1.14179.2.2.6.1.5
    indexes:
    - labelname: bsnAPName
      type: PhysAddress48
    - labelname: bsnAPIfSlotId
      type: gauge
    lookups:
    - labels:
      - bsnAPName
      labelname: bsnAPName
      oid: 1.3.6.1.4.1.14179.2.2.1.1.3
      type: OctetString
  - name: bsnAPIfDot11RTSSuccessCount
    oid: 1.3.6.1.4.1.14179.2.2.6.1.6
    type: counter
    help: This counter shall increment when a CTS is received in response to an RTS.
      - 1.3.6.1.4.1.14179.2.2.6.1.6
    indexes:
    - labelname: bsnAPName
      type: PhysAddress48
    - labelname: bsnAPIfSlotId
      type: gauge
    lookups:
    - labels:
      - bsnAPName
      labelname: bsnAPName
      oid: 1.3.6.1.4.1.14179.2.2.1.1.3
      type: OctetString
  - name: bsnAPIfDot11RTSFailureCount
    oid: 1.3.6.1.4.1.14179.2.2.6.1.7
    type: counter
    help: This counter shall increment when a CTS is not received in response to an
      RTS. - 1.3.6.1.4.1.14179.2.2.6.1.7
    indexes:
    - labelname: bsnAPName
      type: PhysAddress48
    - labelname: bsnAPIfSlotId
      type: gauge
    lookups:
    - labels:
      - bsnAPName
      labelname: bsnAPName
      oid: 1.3.6.1.4.1.14179.2.2.1.1.3
      type: OctetString
  - name: bsnAPIfDot11ACKFailureCount
    oid: 1.3.6.1.4.1.14179.2.2.6.1.8
    type: counter
    help: This counter shall increment when an ACK is not received when expected.
      - 1.3.6.1.4.1.14179.2.2.6.1.8
    indexes:
    - labelname: bsnAPName
      type: PhysAddress48
    - labelname: bsnAPIfSlotId
      type: gauge
    lookups:
    - labels:
      - bsnAPName
      labelname: bsnAPName
      oid: 1.3.6.1.4.1.14179.2.2.1.1.3
      type: OctetString
  - name: bsnAPIfDot11ReceivedFragmentCount
    oid: 1.3.6.1.4.1.14179.2.2.6.1.9
    type: counter
    help: This counter shall be incremented for each successfully received MPDU of
      type Data or Management. - 1.3.6.1.4.1.14179.2.2.6.1.9
    indexes:
    - labelname: bsnAPName
      type: PhysAddress48
    - labelname: bsnAPIfSlotId
      type: gauge
    lookups:
    - labels:
      - bsnAPName
      labelname: bsnAPName
      oid: 1.3.6.1.4.1.14179.2.2.1.1.3
      type: OctetString
  - name: bsnAPIfDot11MulticastReceivedFrameCount
    oid: 1.3.6.1.4.1.14179.2.2.6.1.10
    type: counter
    help: This counter shall increment when a MSDU is received with the multicast
      bit set in the destination MAC address. - 1.3.6.1.4.1.14179.2.2.6.1.10
    indexes:
    - labelname: bsnAPName
      type: PhysAddress48
    - labelname: bsnAPIfSlotId
      type: gauge
    lookups:
    - labels:
      - bsnAPName
      labelname: bsnAPName
      oid: 1.3.6.1.4.1.14179.2.2.1.1.3
      type: OctetString
  - name: bsnAPIfDot11FCSErrorCount
    oid: 1.3.6.1.4.1.14179.2.2.6.1.11
    type: counter
    help: This counter shall increment when an FCS error is detected in a received
      MPDU. - 1.3.6.1.4.1.14179.2.2.6.1.11
    indexes:
    - labelname: bsnAPName
      type: PhysAddress48
    - labelname: bsnAPIfSlotId
      type: gauge
    lookups:
    - labels:
      - bsnAPName
      labelname: bsnAPName
      oid: 1.3.6.1.4.1.14179.2.2.1.1.3
      type: OctetString
  - name: bsnAPIfDot11TransmittedFrameCount
    oid: 1.3.6.1.4.1.14179.2.2.6.1.12
    type: counter
    help: This counter shall increment for each successfully transmitted MSDU. - 1.3.6.1.4.1.14179.2.2.6.1.12
    indexes:
    - labelname: bsnAPName
      type: PhysAddress48
    - labelname: bsnAPIfSlotId
      type: gauge
    lookups:
    - labels:
      - bsnAPName
      labelname: bsnAPName
      oid: 1.3.6.1.4.1.14179.2.2.1.1.3
      type: OctetString
  - name: bsnAPIfDot11WEPUndecryptableCount
    oid: 1.3.6.1.4.1.14179.2.2.6.1.13
    type: counter
    help: This counter shall increment when a frame is received with the WEP subfield
      of the Frame Control field set to one and the WEPOn value for the key mapped
      to the TA's MAC address indicates that the frame should not have been encrypted
      or that frame is discarded due to the receiving STA not implementing the privacy
      option. - 1.3.6.1.4.1.14179.2.2.6.1.13
    indexes:
    - labelname: bsnAPName
      type: PhysAddress48
    - labelname: bsnAPIfSlotId
      type: gauge
    lookups:
    - labels:
      - bsnAPName
      labelname: bsnAPName
      oid: 1.3.6.1.4.1.14179.2.2.1.1.3
      type: OctetString
  - name: bsnAPIfDot11FailedCount
    oid: 1.3.6.1.4.1.14179.2.2.6.1.33
    type: counter
    help: This counter shall increment when an MSDU is not transmitted successfully
      due to the number of transmit attempts exceeding either the bsnAPIfDot11ShortRetryLimit
      or dot11LongRetryLimit. - 1.3.6.1.4.1.14179.2.2.6.1.33
    indexes:
    - labelname: bsnAPName
      type: PhysAddress48
    - labelname: bsnAPIfSlotId
      type: gauge
    lookups:
    - labels:
      - bsnAPName
      labelname: bsnAPName
      oid: 1.3.6.1.4.1.14179.2.2.1.1.3
      type: OctetString
ddwrt:
  walk:
  - 1.3.6.1.2.1.1.3
  - 1.3.6.1.2.1.2
  - 1.3.6.1.2.1.25.2
  - 1.3.6.1.2.1.31.1.1
  - 1.3.6.1.4.1.2021.10.1.1
  - 1.3.6.1.4.1.2021.10.1.2
  - 1.3.6.1.4.1.2021.10.1.5
  - 1.3.6.1.4.1.2021.11
  - 1.3.6.1.4.1.2021.4
  metrics:
  - name: sysUpTime
    oid: 1.3.6.1.2.1.1.3
    type: gauge
    help: The time (in hundredths of a second) since the network management portion
      of the system was last re-initialized. - 1.3.6.1.2.1.1.3
  - name: ifNumber
    oid: 1.3.6.1.2.1.2.1
    type: gauge
    help: The number of network interfaces (regardless of their current state) present
      on this system. - 1.3.6.1.2.1.2.1
  - name: ifIndex
    oid: 1.3.6.1.2.1.2.2.1.1
    type: gauge
    help: A unique value, greater than zero, for each interface - 1.3.6.1.2.1.2.2.1.1
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifDescr
    oid: 1.3.6.1.2.1.2.2.1.2
    type: DisplayString
    help: A textual string containing information about the interface - 1.3.6.1.2.1.2.2.1.2
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifType
    oid: 1.3.6.1.2.1.2.2.1.3
    type: gauge
    help: The type of interface - 1.3.6.1.2.1.2.2.1.3
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifMtu
    oid: 1.3.6.1.2.1.2.2.1.4
    type: gauge
    help: The size of the largest packet which can be sent/received on the interface,
      specified in octets - 1.3.6.1.2.1.2.2.1.4
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifSpeed
    oid: 1.3.6.1.2.1.2.2.1.5
    type: gauge
    help: An estimate of the interface's current bandwidth in bits per second - 1.3.6.1.2.1.2.2.1.5
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifPhysAddress
    oid: 1.3.6.1.2.1.2.2.1.6
    type: PhysAddress48
    help: The interface's address at its protocol sub-layer - 1.3.6.1.2.1.2.2.1.6
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifAdminStatus
    oid: 1.3.6.1.2.1.2.2.1.7
    type: gauge
    help: The desired state of the interface - 1.3.6.1.2.1.2.2.1.7
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifOperStatus
    oid: 1.3.6.1.2.1.2.2.1.8
    type: gauge
    help: The current operational state of the interface - 1.3.6.1.2.1.2.2.1.8
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifLastChange
    oid: 1.3.6.1.2.1.2.2.1.9
    type: gauge
    help: The value of sysUpTime at the time the interface entered its current operational
      state - 1.3.6.1.2.1.2.2.1.9
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifInOctets
    oid: 1.3.6.1.2.1.2.2.1.10
    type: counter
    help: The total number of octets received on the interface, including framing
      characters - 1.3.6.1.2.1.2.2.1.10
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifInUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.11
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were not addressed to a multicast or broadcast address at this sub-layer
      - 1.3.6.1.2.1.2.2.1.11
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifInNUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.12
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a multicast or broadcast address at this sub-layer -
      1.3.6.1.2.1.2.2.1.12
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifInDiscards
    oid: 1.3.6.1.2.1.2.2.1.13
    type: counter
    help: The number of inbound packets which were chosen to be discarded even though
      no errors had been detected to prevent their being deliverable to a higher-layer
      protocol - 1.3.6.1.2.1.2.2.1.13
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifInErrors
    oid: 1.3.6.1.2.1.2.2.1.14
    type: counter
    help: For packet-oriented interfaces, the number of inbound packets that contained
      errors preventing them from being deliverable to a higher-layer protocol - 1.3.6.1.2.1.2.2.1.14
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifInUnknownProtos
    oid: 1.3.6.1.2.1.2.2.1.15
    type: counter
    help: For packet-oriented interfaces, the number of packets received via the interface
      which were discarded because of an unknown or unsupported protocol - 1.3.6.1.2.1.2.2.1.15
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifOutOctets
    oid: 1.3.6.1.2.1.2.2.1.16
    type: counter
    help: The total number of octets transmitted out of the interface, including framing
      characters - 1.3.6.1.2.1.2.2.1.16
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifOutUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.17
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were not addressed to a multicast or broadcast address at this sub-layer,
      including those that were discarded or not sent - 1.3.6.1.2.1.2.2.1.17
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifOutNUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.18
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a multicast or broadcast address at this sub-layer,
      including those that were discarded or not sent - 1.3.6.1.2.1.2.2.1.18
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifOutDiscards
    oid: 1.3.6.1.2.1.2.2.1.19
    type: counter
    help: The number of outbound packets which were chosen to be discarded even though
      no errors had been detected to prevent their being transmitted - 1.3.6.1.2.1.2.2.1.19
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifOutErrors
    oid: 1.3.6.1.2.1.2.2.1.20
    type: counter
    help: For packet-oriented interfaces, the number of outbound packets that could
      not be transmitted because of errors - 1.3.6.1.2.1.2.2.1.20
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifOutQLen
    oid: 1.3.6.1.2.1.2.2.1.21
    type: gauge
    help: The length of the output packet queue (in packets). - 1.3.6.1.2.1.2.2.1.21
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: hrMemorySize
    oid: 1.3.6.1.2.1.25.2.2
    type: gauge
    help: The amount of physical read-write main memory, typically RAM, contained
      by the host. - 1.3.6.1.2.1.25.2.2
  - name: hrStorageIndex
    oid: 1.3.6.1.2.1.25.2.3.1.1
    type: gauge
    help: A unique value for each logical storage area contained by the host. - 1.3.6.1.2.1.25.2.3.1.1
    indexes:
    - labelname: hrStorageIndex
      type: gauge
  - name: hrStorageDescr
    oid: 1.3.6.1.2.1.25.2.3.1.3
    type: DisplayString
    help: A description of the type and instance of the storage described by this
      entry. - 1.3.6.1.2.1.25.2.3.1.3
    indexes:
    - labelname: hrStorageIndex
      type: gauge
  - name: hrStorageAllocationUnits
    oid: 1.3.6.1.2.1.25.2.3.1.4
    type: gauge
    help: The size, in bytes, of the data objects allocated from this pool - 1.3.6.1.2.1.25.2.3.1.4
    indexes:
    - labelname: hrStorageIndex
      type: gauge
  - name: hrStorageSize
    oid: 1.3.6.1.2.1.25.2.3.1.5
    type: gauge
    help: The size of the storage represented by this entry, in units of hrStorageAllocationUnits
      - 1.3.6.1.2.1.25.2.3.1.5
    indexes:
    - labelname: hrStorageIndex
      type: gauge
  - name: hrStorageUsed
    oid: 1.3.6.1.2.1.25.2.3.1.6
    type: gauge
    help: The amount of the storage represented by this entry that is allocated, in
      units of hrStorageAllocationUnits. - 1.3.6.1.2.1.25.2.3.1.6
    indexes:
    - labelname: hrStorageIndex
      type: gauge
  - name: hrStorageAllocationFailures
    oid: 1.3.6.1.2.1.25.2.3.1.7
    type: counter
    help: The number of requests for storage represented by this entry that could
      not be honored due to not enough storage - 1.3.6.1.2.1.25.2.3.1.7
    indexes:
    - labelname: hrStorageIndex
      type: gauge
  - name: ifName
    oid: 1.3.6.1.2.1.31.1.1.1.1
    type: DisplayString
    help: The textual name of the interface - 1.3.6.1.2.1.31.1.1.1.1
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifInMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.2
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a multicast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.2
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifInBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.3
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a broadcast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.3
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifOutMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.4
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a multicast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.4
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifOutBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.5
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a broadcast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.5
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifHCInOctets
    oid: 1.3.6.1.2.1.31.1.1.1.6
    type: counter
    help: The total number of octets received on the interface, including framing
      characters - 1.3.6.1.2.1.31.1.1.1.6
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifHCInUcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.7
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were not addressed to a multicast or broadcast address at this sub-layer
      - 1.3.6.1.2.1.31.1.1.1.7
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifHCInMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.8
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a multicast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.8
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifHCInBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.9
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a broadcast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.9
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifHCOutOctets
    oid: 1.3.6.1.2.1.31.1.1.1.10
    type: counter
    help: The total number of octets transmitted out of the interface, including framing
      characters - 1.3.6.1.2.1.31.1.1.1.10
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifHCOutUcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.11
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were not addressed to a multicast or broadcast address at this sub-layer,
      including those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.11
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifHCOutMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.12
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a multicast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.12
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifHCOutBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.13
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a broadcast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.13
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifLinkUpDownTrapEnable
    oid: 1.3.6.1.2.1.31.1.1.1.14
    type: gauge
    help: Indicates whether linkUp/linkDown traps should be generated for this interface
      - 1.3.6.1.2.1.31.1.1.1.14
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifHighSpeed
    oid: 1.3.6.1.2.1.31.1.1.1.15
    type: gauge
    help: An estimate of the interface's current bandwidth in units of 1,000,000 bits
      per second - 1.3.6.1.2.1.31.1.1.1.15
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifPromiscuousMode
    oid: 1.3.6.1.2.1.31.1.1.1.16
    type: gauge
    help: This object has a value of false(2) if this interface only accepts packets/frames
      that are addressed to this station - 1.3.6.1.2.1.31.1.1.1.16
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifConnectorPresent
    oid: 1.3.6.1.2.1.31.1.1.1.17
    type: gauge
    help: This object has the value 'true(1)' if the interface sublayer has a physical
      connector and the value 'false(2)' otherwise. - 1.3.6.1.2.1.31.1.1.1.17
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifAlias
    oid: 1.3.6.1.2.1.31.1.1.1.18
    type: DisplayString
    help: This object is an 'alias' name for the interface as specified by a network
      manager, and provides a non-volatile 'handle' for the interface - 1.3.6.1.2.1.31.1.1.1.18
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifCounterDiscontinuityTime
    oid: 1.3.6.1.2.1.31.1.1.1.19
    type: gauge
    help: The value of sysUpTime on the most recent occasion at which any one or more
      of this interface's counters suffered a discontinuity - 1.3.6.1.2.1.31.1.1.1.19
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: laIndex
    oid: 1.3.6.1.4.1.2021.10.1.1
    type: gauge
    help: reference index/row number for each observed loadave. - 1.3.6.1.4.1.2021.10.1.1
    indexes:
    - labelname: laNames
      type: gauge
    lookups:
    - labels:
      - laNames
      labelname: laNames
      oid: 1.3.6.1.4.1.2021.10.1.2
      type: DisplayString
  - name: laNames
    oid: 1.3.6.1.4.1.2021.10.1.2
    type: DisplayString
    help: The list of loadave names we're watching. - 1.3.6.1.4.1.2021.10.1.2
    indexes:
    - labelname: laNames
      type: gauge
    lookups:
    - labels:
      - laNames
      labelname: laNames
      oid: 1.3.6.1.4.1.2021.10.1.2
      type: DisplayString
  - name: laLoadInt
    oid: 1.3.6.1.4.1.2021.10.1.5
    type: gauge
    help: The 1,5 and 15 minute load averages as an integer - 1.3.6.1.4.1.2021.10.1.5
    indexes:
    - labelname: laNames
      type: gauge
    lookups:
    - labels:
      - laNames
      labelname: laNames
      oid: 1.3.6.1.4.1.2021.10.1.2
      type: DisplayString
  - name: ssIndex
    oid: 1.3.6.1.4.1.2021.11.1
    type: gauge
    help: Bogus Index - 1.3.6.1.4.1.2021.11.1
  - name: ssErrorName
    oid: 1.3.6.1.4.1.2021.11.2
    type: DisplayString
    help: Bogus Name - 1.3.6.1.4.1.2021.11.2
  - name: ssSwapIn
    oid: 1.3.6.1.4.1.2021.11.3
    type: gauge
    help: The average amount of memory swapped in from disk, calculated over the last
      minute. - 1.3.6.1.4.1.2021.11.3
  - name: ssSwapOut
    oid: 1.3.6.1.4.1.2021.11.4
    type: gauge
    help: The average amount of memory swapped out to disk, calculated over the last
      minute. - 1.3.6.1.4.1.2021.11.4
  - name: ssIOSent
    oid: 1.3.6.1.4.1.2021.11.5
    type: gauge
    help: The average amount of data written to disk or other block device, calculated
      over the last minute - 1.3.6.1.4.1.2021.11.5
  - name: ssIOReceive
    oid: 1.3.6.1.4.1.2021.11.6
    type: gauge
    help: The average amount of data read from disk or other block device, calculated
      over the last minute - 1.3.6.1.4.1.2021.11.6
  - name: ssSysInterrupts
    oid: 1.3.6.1.4.1.2021.11.7
    type: gauge
    help: The average rate of interrupts processed (including the clock) calculated
      over the last minute - 1.3.6.1.4.1.2021.11.7
  - name: ssSysContext
    oid: 1.3.6.1.4.1.2021.11.8
    type: gauge
    help: The average rate of context switches, calculated over the last minute -
      1.3.6.1.4.1.2021.11.8
  - name: ssCpuUser
    oid: 1.3.6.1.4.1.2021.11.9
    type: gauge
    help: The percentage of CPU time spent processing user-level code, calculated
      over the last minute - 1.3.6.1.4.1.2021.11.9
  - name: ssCpuSystem
    oid: 1.3.6.1.4.1.2021.11.10
    type: gauge
    help: The percentage of CPU time spent processing system-level code, calculated
      over the last minute - 1.3.6.1.4.1.2021.11.10
  - name: ssCpuIdle
    oid: 1.3.6.1.4.1.2021.11.11
    type: gauge
    help: The percentage of processor time spent idle, calculated over the last minute
      - 1.3.6.1.4.1.2021.11.11
  - name: ssCpuRawUser
    oid: 1.3.6.1.4.1.2021.11.50
    type: counter
    help: The number of 'ticks' (typically 1/100s) spent processing user-level code
      - 1.3.6.1.4.1.2021.11.50
  - name: ssCpuRawNice
    oid: 1.3.6.1.4.1.2021.11.51
    type: counter
    help: The number of 'ticks' (typically 1/100s) spent processing reduced-priority
      code - 1.3.6.1.4.1.2021.11.51
  - name: ssCpuRawSystem
    oid: 1.3.6.1.4.1.2021.11.52
    type: counter
    help: The number of 'ticks' (typically 1/100s) spent processing system-level code
      - 1.3.6.1.4.1.2021.11.52
  - name: ssCpuRawIdle
    oid: 1.3.6.1.4.1.2021.11.53
    type: counter
    help: The number of 'ticks' (typically 1/100s) spent idle - 1.3.6.1.4.1.2021.11.53
  - name: ssCpuRawWait
    oid: 1.3.6.1.4.1.2021.11.54
    type: counter
    help: The number of 'ticks' (typically 1/100s) spent waiting for IO - 1.3.6.1.4.1.2021.11.54
  - name: ssCpuRawKernel
    oid: 1.3.6.1.4.1.2021.11.55
    type: counter
    help: The number of 'ticks' (typically 1/100s) spent processing kernel-level code
      - 1.3.6.1.4.1.2021.11.55
  - name: ssCpuRawInterrupt
    oid: 1.3.6.1.4.1.2021.11.56
    type: counter
    help: The number of 'ticks' (typically 1/100s) spent processing hardware interrupts
      - 1.3.6.1.4.1.2021.11.56
  - name: ssIORawSent
    oid: 1.3.6.1.4.1.2021.11.57
    type: counter
    help: Number of blocks sent to a block device - 1.3.6.1.4.1.2021.11.57
  - name: ssIORawReceived
    oid: 1.3.6.1.4.1.2021.11.58
    type: counter
    help: Number of blocks received from a block device - 1.3.6.1.4.1.2021.11.58
  - name: ssRawInterrupts
    oid: 1.3.6.1.4.1.2021.11.59
    type: counter
    help: Number of interrupts processed - 1.3.6.1.4.1.2021.11.59
  - name: ssRawContexts
    oid: 1.3.6.1.4.1.2021.11.60
    type: counter
    help: Number of context switches - 1.3.6.1.4.1.2021.11.60
  - name: ssCpuRawSoftIRQ
    oid: 1.3.6.1.4.1.2021.11.61
    type: counter
    help: The number of 'ticks' (typically 1/100s) spent processing software interrupts
      - 1.3.6.1.4.1.2021.11.61
  - name: ssRawSwapIn
    oid: 1.3.6.1.4.1.2021.11.62
    type: counter
    help: Number of blocks swapped in - 1.3.6.1.4.1.2021.11.62
  - name: ssRawSwapOut
    oid: 1.3.6.1.4.1.2021.11.63
    type: counter
    help: Number of blocks swapped out - 1.3.6.1.4.1.2021.11.63
  - name: ssCpuRawSteal
    oid: 1.3.6.1.4.1.2021.11.64
    type: counter
    help: The number of 'ticks' (typically 1/100s) spent by the hypervisor code to
      run other VMs even though the CPU in the current VM had something runnable -
      1.3.6.1.4.1.2021.11.64
  - name: ssCpuRawGuest
    oid: 1.3.6.1.4.1.2021.11.65
    type: counter
    help: The number of 'ticks' (typically 1/100s) spent by the CPU to run a virtual
      CPU (guest) - 1.3.6.1.4.1.2021.11.65
  - name: ssCpuRawGuestNice
    oid: 1.3.6.1.4.1.2021.11.66
    type: counter
    help: The number of 'ticks' (typically 1/100s) spent by the CPU to run a niced
      virtual CPU (guest) - 1.3.6.1.4.1.2021.11.66
  - name: ssCpuNumCpus
    oid: 1.3.6.1.4.1.2021.11.67
    type: gauge
    help: The number of processors, as counted by the agent - 1.3.6.1.4.1.2021.11.67
  - name: memIndex
    oid: 1.3.6.1.4.1.2021.4.1
    type: gauge
    help: Bogus Index - 1.3.6.1.4.1.2021.4.1
  - name: memErrorName
    oid: 1.3.6.1.4.1.2021.4.2
    type: DisplayString
    help: Bogus Name - 1.3.6.1.4.1.2021.4.2
  - name: memTotalSwap
    oid: 1.3.6.1.4.1.2021.4.3
    type: gauge
    help: The total amount of swap space configured for this host. - 1.3.6.1.4.1.2021.4.3
  - name: memAvailSwap
    oid: 1.3.6.1.4.1.2021.4.4
    type: gauge
    help: The amount of swap space currently unused or available. - 1.3.6.1.4.1.2021.4.4
  - name: memTotalReal
    oid: 1.3.6.1.4.1.2021.4.5
    type: gauge
    help: The total amount of real/physical memory installed on this host. - 1.3.6.1.4.1.2021.4.5
  - name: memAvailReal
    oid: 1.3.6.1.4.1.2021.4.6
    type: gauge
    help: The amount of real/physical memory currently unused or available. - 1.3.6.1.4.1.2021.4.6
  - name: memTotalSwapTXT
    oid: 1.3.6.1.4.1.2021.4.7
    type: gauge
    help: The total amount of swap space or virtual memory allocated for text pages
      on this host - 1.3.6.1.4.1.2021.4.7
  - name: memAvailSwapTXT
    oid: 1.3.6.1.4.1.2021.4.8
    type: gauge
    help: The amount of swap space or virtual memory currently being used by text
      pages on this host - 1.3.6.1.4.1.2021.4.8
  - name: memTotalRealTXT
    oid: 1.3.6.1.4.1.2021.4.9
    type: gauge
    help: The total amount of real/physical memory allocated for text pages on this
      host - 1.3.6.1.4.1.2021.4.9
  - name: memAvailRealTXT
    oid: 1.3.6.1.4.1.2021.4.10
    type: gauge
    help: The amount of real/physical memory currently being used by text pages on
      this host - 1.3.6.1.4.1.2021.4.10
  - name: memTotalFree
    oid: 1.3.6.1.4.1.2021.4.11
    type: gauge
    help: The total amount of memory free or available for use on this host - 1.3.6.1.4.1.2021.4.11
  - name: memMinimumSwap
    oid: 1.3.6.1.4.1.2021.4.12
    type: gauge
    help: The minimum amount of swap space expected to be kept free or available during
      normal operation of this host - 1.3.6.1.4.1.2021.4.12
  - name: memShared
    oid: 1.3.6.1.4.1.2021.4.13
    type: gauge
    help: The total amount of real or virtual memory currently allocated for use as
      shared memory - 1.3.6.1.4.1.2021.4.13
  - name: memBuffer
    oid: 1.3.6.1.4.1.2021.4.14
    type: gauge
    help: The total amount of real or virtual memory currently allocated for use as
      memory buffers - 1.3.6.1.4.1.2021.4.14
  - name: memCached
    oid: 1.3.6.1.4.1.2021.4.15
    type: gauge
    help: The total amount of real or virtual memory currently allocated for use as
      cached memory - 1.3.6.1.4.1.2021.4.15
  - name: memUsedSwapTXT
    oid: 1.3.6.1.4.1.2021.4.16
    type: gauge
    help: The amount of swap space or virtual memory currently being used by text
      pages on this host - 1.3.6.1.4.1.2021.4.16
  - name: memUsedRealTXT
    oid: 1.3.6.1.4.1.2021.4.17
    type: gauge
    help: The amount of real/physical memory currently being used by text pages on
      this host - 1.3.6.1.4.1.2021.4.17
  - name: memSwapError
    oid: 1.3.6.1.4.1.2021.4.100
    type: gauge
    help: Indicates whether the amount of available swap space (as reported by 'memAvailSwap(4)'),
      is less than the desired minimum (specified by 'memMinimumSwap(12)'). - 1.3.6.1.4.1.2021.4.100
  - name: memSwapErrorMsg
    oid: 1.3.6.1.4.1.2021.4.101
    type: DisplayString
    help: Describes whether the amount of available swap space (as reported by 'memAvailSwap(4)'),
      is less than the desired minimum (specified by 'memMinimumSwap(12)'). - 1.3.6.1.4.1.2021.4.101
if_mib:
  walk:
  - 1.3.6.1.2.1.1.3
  - 1.3.6.1.2.1.2
  - 1.3.6.1.2.1.31.1.1
  metrics:
  - name: sysUpTime
    oid: 1.3.6.1.2.1.1.3
    type: gauge
    help: The time (in hundredths of a second) since the network management portion
      of the system was last re-initialized. - 1.3.6.1.2.1.1.3
  - name: ifNumber
    oid: 1.3.6.1.2.1.2.1
    type: gauge
    help: The number of network interfaces (regardless of their current state) present
      on this system. - 1.3.6.1.2.1.2.1
  - name: ifIndex
    oid: 1.3.6.1.2.1.2.2.1.1
    type: gauge
    help: A unique value, greater than zero, for each interface - 1.3.6.1.2.1.2.2.1.1
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifDescr
    oid: 1.3.6.1.2.1.2.2.1.2
    type: DisplayString
    help: A textual string containing information about the interface - 1.3.6.1.2.1.2.2.1.2
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifType
    oid: 1.3.6.1.2.1.2.2.1.3
    type: gauge
    help: The type of interface - 1.3.6.1.2.1.2.2.1.3
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifMtu
    oid: 1.3.6.1.2.1.2.2.1.4
    type: gauge
    help: The size of the largest packet which can be sent/received on the interface,
      specified in octets - 1.3.6.1.2.1.2.2.1.4
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifSpeed
    oid: 1.3.6.1.2.1.2.2.1.5
    type: gauge
    help: An estimate of the interface's current bandwidth in bits per second - 1.3.6.1.2.1.2.2.1.5
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifPhysAddress
    oid: 1.3.6.1.2.1.2.2.1.6
    type: PhysAddress48
    help: The interface's address at its protocol sub-layer - 1.3.6.1.2.1.2.2.1.6
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifAdminStatus
    oid: 1.3.6.1.2.1.2.2.1.7
    type: gauge
    help: The desired state of the interface - 1.3.6.1.2.1.2.2.1.7
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOperStatus
    oid: 1.3.6.1.2.1.2.2.1.8
    type: gauge
    help: The current operational state of the interface - 1.3.6.1.2.1.2.2.1.8
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifLastChange
    oid: 1.3.6.1.2.1.2.2.1.9
    type: gauge
    help: The value of sysUpTime at the time the interface entered its current operational
      state - 1.3.6.1.2.1.2.2.1.9
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInOctets
    oid: 1.3.6.1.2.1.2.2.1.10
    type: counter
    help: The total number of octets received on the interface, including framing
      characters - 1.3.6.1.2.1.2.2.1.10
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.11
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were not addressed to a multicast or broadcast address at this sub-layer
      - 1.3.6.1.2.1.2.2.1.11
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInNUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.12
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a multicast or broadcast address at this sub-layer -
      1.3.6.1.2.1.2.2.1.12
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInDiscards
    oid: 1.3.6.1.2.1.2.2.1.13
    type: counter
    help: The number of inbound packets which were chosen to be discarded even though
      no errors had been detected to prevent their being deliverable to a higher-layer
      protocol - 1.3.6.1.2.1.2.2.1.13
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInErrors
    oid: 1.3.6.1.2.1.2.2.1.14
    type: counter
    help: For packet-oriented interfaces, the number of inbound packets that contained
      errors preventing them from being deliverable to a higher-layer protocol - 1.3.6.1.2.1.2.2.1.14
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInUnknownProtos
    oid: 1.3.6.1.2.1.2.2.1.15
    type: counter
    help: For packet-oriented interfaces, the number of packets received via the interface
      which were discarded because of an unknown or unsupported protocol - 1.3.6.1.2.1.2.2.1.15
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutOctets
    oid: 1.3.6.1.2.1.2.2.1.16
    type: counter
    help: The total number of octets transmitted out of the interface, including framing
      characters - 1.3.6.1.2.1.2.2.1.16
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.17
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were not addressed to a multicast or broadcast address at this sub-layer,
      including those that were discarded or not sent - 1.3.6.1.2.1.2.2.1.17
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutNUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.18
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a multicast or broadcast address at this sub-layer,
      including those that were discarded or not sent - 1.3.6.1.2.1.2.2.1.18
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutDiscards
    oid: 1.3.6.1.2.1.2.2.1.19
    type: counter
    help: The number of outbound packets which were chosen to be discarded even though
      no errors had been detected to prevent their being transmitted - 1.3.6.1.2.1.2.2.1.19
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutErrors
    oid: 1.3.6.1.2.1.2.2.1.20
    type: counter
    help: For packet-oriented interfaces, the number of outbound packets that could
      not be transmitted because of errors - 1.3.6.1.2.1.2.2.1.20
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutQLen
    oid: 1.3.6.1.2.1.2.2.1.21
    type: gauge
    help: The length of the output packet queue (in packets). - 1.3.6.1.2.1.2.2.1.21
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifName
    oid: 1.3.6.1.2.1.31.1.1.1.1
    type: DisplayString
    help: The textual name of the interface - 1.3.6.1.2.1.31.1.1.1.1
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.2
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a multicast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.2
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.3
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a broadcast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.3
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.4
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a multicast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.4
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.5
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a broadcast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.5
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifHCInOctets
    oid: 1.3.6.1.2.1.31.1.1.1.6
    type: counter
    help: The total number of octets received on the interface, including framing
      characters - 1.3.6.1.2.1.31.1.1.1.6
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifHCInUcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.7
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were not addressed to a multicast or broadcast address at this sub-layer
      - 1.3.6.1.2.1.31.1.1.1.7
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifHCInMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.8
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a multicast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.8
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifHCInBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.9
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a broadcast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.9
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifHCOutOctets
    oid: 1.3.6.1.2.1.31.1.1.1.10
    type: counter
    help: The total number of octets transmitted out of the interface, including framing
      characters - 1.3.6.1.2.1.31.1.1.1.10
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifHCOutUcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.11
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were not addressed to a multicast or broadcast address at this sub-layer,
      including those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.11
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifHCOutMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.12
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a multicast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.12
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifHCOutBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.13
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a broadcast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.13
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifLinkUpDownTrapEnable
    oid: 1.3.6.1.2.1.31.1.1.1.14
    type: gauge
    help: Indicates whether linkUp/linkDown traps should be generated for this interface
      - 1.3.6.1.2.1.31.1.1.1.14
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifHighSpeed
    oid: 1.3.6.1.2.1.31.1.1.1.15
    type: gauge
    help: An estimate of the interface's current bandwidth in units of 1,000,000 bits
      per second - 1.3.6.1.2.1.31.1.1.1.15
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifPromiscuousMode
    oid: 1.3.6.1.2.1.31.1.1.1.16
    type: gauge
    help: This object has a value of false(2) if this interface only accepts packets/frames
      that are addressed to this station - 1.3.6.1.2.1.31.1.1.1.16
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifConnectorPresent
    oid: 1.3.6.1.2.1.31.1.1.1.17
    type: gauge
    help: This object has the value 'true(1)' if the interface sublayer has a physical
      connector and the value 'false(2)' otherwise. - 1.3.6.1.2.1.31.1.1.1.17
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifAlias
    oid: 1.3.6.1.2.1.31.1.1.1.18
    type: DisplayString
    help: This object is an 'alias' name for the interface as specified by a network
      manager, and provides a non-volatile 'handle' for the interface - 1.3.6.1.2.1.31.1.1.1.18
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifCounterDiscontinuityTime
    oid: 1.3.6.1.2.1.31.1.1.1.19
    type: gauge
    help: The value of sysUpTime on the most recent occasion at which any one or more
      of this interface's counters suffered a discontinuity - 1.3.6.1.2.1.31.1.1.1.19
    indexes:
    - labelname: ifIndex
      type: gauge
if_mib_ifalias:
  walk:
  - 1.3.6.1.2.1.1.3
  - 1.3.6.1.2.1.2
  - 1.3.6.1.2.1.31.1.1
  metrics:
  - name: sysUpTime
    oid: 1.3.6.1.2.1.1.3
    type: gauge
    help: The time (in hundredths of a second) since the network management portion
      of the system was last re-initialized. - 1.3.6.1.2.1.1.3
  - name: ifNumber
    oid: 1.3.6.1.2.1.2.1
    type: gauge
    help: The number of network interfaces (regardless of their current state) present
      on this system. - 1.3.6.1.2.1.2.1
  - name: ifIndex
    oid: 1.3.6.1.2.1.2.2.1.1
    type: gauge
    help: A unique value, greater than zero, for each interface - 1.3.6.1.2.1.2.2.1.1
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifDescr
    oid: 1.3.6.1.2.1.2.2.1.2
    type: DisplayString
    help: A textual string containing information about the interface - 1.3.6.1.2.1.2.2.1.2
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifType
    oid: 1.3.6.1.2.1.2.2.1.3
    type: gauge
    help: The type of interface - 1.3.6.1.2.1.2.2.1.3
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifMtu
    oid: 1.3.6.1.2.1.2.2.1.4
    type: gauge
    help: The size of the largest packet which can be sent/received on the interface,
      specified in octets - 1.3.6.1.2.1.2.2.1.4
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifSpeed
    oid: 1.3.6.1.2.1.2.2.1.5
    type: gauge
    help: An estimate of the interface's current bandwidth in bits per second - 1.3.6.1.2.1.2.2.1.5
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifPhysAddress
    oid: 1.3.6.1.2.1.2.2.1.6
    type: PhysAddress48
    help: The interface's address at its protocol sub-layer - 1.3.6.1.2.1.2.2.1.6
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifAdminStatus
    oid: 1.3.6.1.2.1.2.2.1.7
    type: gauge
    help: The desired state of the interface - 1.3.6.1.2.1.2.2.1.7
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifOperStatus
    oid: 1.3.6.1.2.1.2.2.1.8
    type: gauge
    help: The current operational state of the interface - 1.3.6.1.2.1.2.2.1.8
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifLastChange
    oid: 1.3.6.1.2.1.2.2.1.9
    type: gauge
    help: The value of sysUpTime at the time the interface entered its current operational
      state - 1.3.6.1.2.1.2.2.1.9
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifInOctets
    oid: 1.3.6.1.2.1.2.2.1.10
    type: counter
    help: The total number of octets received on the interface, including framing
      characters - 1.3.6.1.2.1.2.2.1.10
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifInUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.11
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were not addressed to a multicast or broadcast address at this sub-layer
      - 1.3.6.1.2.1.2.2.1.11
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifInNUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.12
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a multicast or broadcast address at this sub-layer -
      1.3.6.1.2.1.2.2.1.12
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifInDiscards
    oid: 1.3.6.1.2.1.2.2.1.13
    type: counter
    help: The number of inbound packets which were chosen to be discarded even though
      no errors had been detected to prevent their being deliverable to a higher-layer
      protocol - 1.3.6.1.2.1.2.2.1.13
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifInErrors
    oid: 1.3.6.1.2.1.2.2.1.14
    type: counter
    help: For packet-oriented interfaces, the number of inbound packets that contained
      errors preventing them from being deliverable to a higher-layer protocol - 1.3.6.1.2.1.2.2.1.14
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifInUnknownProtos
    oid: 1.3.6.1.2.1.2.2.1.15
    type: counter
    help: For packet-oriented interfaces, the number of packets received via the interface
      which were discarded because of an unknown or unsupported protocol - 1.3.6.1.2.1.2.2.1.15
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifOutOctets
    oid: 1.3.6.1.2.1.2.2.1.16
    type: counter
    help: The total number of octets transmitted out of the interface, including framing
      characters - 1.3.6.1.2.1.2.2.1.16
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifOutUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.17
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were not addressed to a multicast or broadcast address at this sub-layer,
      including those that were discarded or not sent - 1.3.6.1.2.1.2.2.1.17
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifOutNUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.18
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a multicast or broadcast address at this sub-layer,
      including those that were discarded or not sent - 1.3.6.1.2.1.2.2.1.18
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifOutDiscards
    oid: 1.3.6.1.2.1.2.2.1.19
    type: counter
    help: The number of outbound packets which were chosen to be discarded even though
      no errors had been detected to prevent their being transmitted - 1.3.6.1.2.1.2.2.1.19
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifOutErrors
    oid: 1.3.6.1.2.1.2.2.1.20
    type: counter
    help: For packet-oriented interfaces, the number of outbound packets that could
      not be transmitted because of errors - 1.3.6.1.2.1.2.2.1.20
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifOutQLen
    oid: 1.3.6.1.2.1.2.2.1.21
    type: gauge
    help: The length of the output packet queue (in packets). - 1.3.6.1.2.1.2.2.1.21
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifName
    oid: 1.3.6.1.2.1.31.1.1.1.1
    type: DisplayString
    help: The textual name of the interface - 1.3.6.1.2.1.31.1.1.1.1
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifInMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.2
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a multicast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.2
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifInBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.3
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a broadcast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.3
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifOutMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.4
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a multicast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.4
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifOutBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.5
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a broadcast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.5
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifHCInOctets
    oid: 1.3.6.1.2.1.31.1.1.1.6
    type: counter
    help: The total number of octets received on the interface, including framing
      characters - 1.3.6.1.2.1.31.1.1.1.6
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifHCInUcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.7
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were not addressed to a multicast or broadcast address at this sub-layer
      - 1.3.6.1.2.1.31.1.1.1.7
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifHCInMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.8
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a multicast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.8
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifHCInBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.9
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a broadcast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.9
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifHCOutOctets
    oid: 1.3.6.1.2.1.31.1.1.1.10
    type: counter
    help: The total number of octets transmitted out of the interface, including framing
      characters - 1.3.6.1.2.1.31.1.1.1.10
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifHCOutUcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.11
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were not addressed to a multicast or broadcast address at this sub-layer,
      including those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.11
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifHCOutMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.12
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a multicast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.12
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifHCOutBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.13
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a broadcast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.13
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifLinkUpDownTrapEnable
    oid: 1.3.6.1.2.1.31.1.1.1.14
    type: gauge
    help: Indicates whether linkUp/linkDown traps should be generated for this interface
      - 1.3.6.1.2.1.31.1.1.1.14
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifHighSpeed
    oid: 1.3.6.1.2.1.31.1.1.1.15
    type: gauge
    help: An estimate of the interface's current bandwidth in units of 1,000,000 bits
      per second - 1.3.6.1.2.1.31.1.1.1.15
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifPromiscuousMode
    oid: 1.3.6.1.2.1.31.1.1.1.16
    type: gauge
    help: This object has a value of false(2) if this interface only accepts packets/frames
      that are addressed to this station - 1.3.6.1.2.1.31.1.1.1.16
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifConnectorPresent
    oid: 1.3.6.1.2.1.31.1.1.1.17
    type: gauge
    help: This object has the value 'true(1)' if the interface sublayer has a physical
      connector and the value 'false(2)' otherwise. - 1.3.6.1.2.1.31.1.1.1.17
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifAlias
    oid: 1.3.6.1.2.1.31.1.1.1.18
    type: DisplayString
    help: This object is an 'alias' name for the interface as specified by a network
      manager, and provides a non-volatile 'handle' for the interface - 1.3.6.1.2.1.31.1.1.1.18
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
  - name: ifCounterDiscontinuityTime
    oid: 1.3.6.1.2.1.31.1.1.1.19
    type: gauge
    help: The value of sysUpTime on the most recent occasion at which any one or more
      of this interface's counters suffered a discontinuity - 1.3.6.1.2.1.31.1.1.1.19
    indexes:
    - labelname: ifAlias
      type: gauge
    lookups:
    - labels:
      - ifAlias
      labelname: ifAlias
      oid: 1.3.6.1.2.1.31.1.1.1.18
      type: DisplayString
if_mib_ifdescr:
  walk:
  - 1.3.6.1.2.1.1.3
  - 1.3.6.1.2.1.2
  - 1.3.6.1.2.1.31.1.1
  metrics:
  - name: sysUpTime
    oid: 1.3.6.1.2.1.1.3
    type: gauge
    help: The time (in hundredths of a second) since the network management portion
      of the system was last re-initialized. - 1.3.6.1.2.1.1.3
  - name: ifNumber
    oid: 1.3.6.1.2.1.2.1
    type: gauge
    help: The number of network interfaces (regardless of their current state) present
      on this system. - 1.3.6.1.2.1.2.1
  - name: ifIndex
    oid: 1.3.6.1.2.1.2.2.1.1
    type: gauge
    help: A unique value, greater than zero, for each interface - 1.3.6.1.2.1.2.2.1.1
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifDescr
    oid: 1.3.6.1.2.1.2.2.1.2
    type: DisplayString
    help: A textual string containing information about the interface - 1.3.6.1.2.1.2.2.1.2
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifType
    oid: 1.3.6.1.2.1.2.2.1.3
    type: gauge
    help: The type of interface - 1.3.6.1.2.1.2.2.1.3
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifMtu
    oid: 1.3.6.1.2.1.2.2.1.4
    type: gauge
    help: The size of the largest packet which can be sent/received on the interface,
      specified in octets - 1.3.6.1.2.1.2.2.1.4
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifSpeed
    oid: 1.3.6.1.2.1.2.2.1.5
    type: gauge
    help: An estimate of the interface's current bandwidth in bits per second - 1.3.6.1.2.1.2.2.1.5
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifPhysAddress
    oid: 1.3.6.1.2.1.2.2.1.6
    type: PhysAddress48
    help: The interface's address at its protocol sub-layer - 1.3.6.1.2.1.2.2.1.6
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifAdminStatus
    oid: 1.3.6.1.2.1.2.2.1.7
    type: gauge
    help: The desired state of the interface - 1.3.6.1.2.1.2.2.1.7
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifOperStatus
    oid: 1.3.6.1.2.1.2.2.1.8
    type: gauge
    help: The current operational state of the interface - 1.3.6.1.2.1.2.2.1.8
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifLastChange
    oid: 1.3.6.1.2.1.2.2.1.9
    type: gauge
    help: The value of sysUpTime at the time the interface entered its current operational
      state - 1.3.6.1.2.1.2.2.1.9
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifInOctets
    oid: 1.3.6.1.2.1.2.2.1.10
    type: counter
    help: The total number of octets received on the interface, including framing
      characters - 1.3.6.1.2.1.2.2.1.10
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifInUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.11
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were not addressed to a multicast or broadcast address at this sub-layer
      - 1.3.6.1.2.1.2.2.1.11
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifInNUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.12
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a multicast or broadcast address at this sub-layer -
      1.3.6.1.2.1.2.2.1.12
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifInDiscards
    oid: 1.3.6.1.2.1.2.2.1.13
    type: counter
    help: The number of inbound packets which were chosen to be discarded even though
      no errors had been detected to prevent their being deliverable to a higher-layer
      protocol - 1.3.6.1.2.1.2.2.1.13
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifInErrors
    oid: 1.3.6.1.2.1.2.2.1.14
    type: counter
    help: For packet-oriented interfaces, the number of inbound packets that contained
      errors preventing them from being deliverable to a higher-layer protocol - 1.3.6.1.2.1.2.2.1.14
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifInUnknownProtos
    oid: 1.3.6.1.2.1.2.2.1.15
    type: counter
    help: For packet-oriented interfaces, the number of packets received via the interface
      which were discarded because of an unknown or unsupported protocol - 1.3.6.1.2.1.2.2.1.15
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifOutOctets
    oid: 1.3.6.1.2.1.2.2.1.16
    type: counter
    help: The total number of octets transmitted out of the interface, including framing
      characters - 1.3.6.1.2.1.2.2.1.16
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifOutUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.17
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were not addressed to a multicast or broadcast address at this sub-layer,
      including those that were discarded or not sent - 1.3.6.1.2.1.2.2.1.17
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifOutNUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.18
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a multicast or broadcast address at this sub-layer,
      including those that were discarded or not sent - 1.3.6.1.2.1.2.2.1.18
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifOutDiscards
    oid: 1.3.6.1.2.1.2.2.1.19
    type: counter
    help: The number of outbound packets which were chosen to be discarded even though
      no errors had been detected to prevent their being transmitted - 1.3.6.1.2.1.2.2.1.19
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifOutErrors
    oid: 1.3.6.1.2.1.2.2.1.20
    type: counter
    help: For packet-oriented interfaces, the number of outbound packets that could
      not be transmitted because of errors - 1.3.6.1.2.1.2.2.1.20
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifOutQLen
    oid: 1.3.6.1.2.1.2.2.1.21
    type: gauge
    help: The length of the output packet queue (in packets). - 1.3.6.1.2.1.2.2.1.21
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifName
    oid: 1.3.6.1.2.1.31.1.1.1.1
    type: DisplayString
    help: The textual name of the interface - 1.3.6.1.2.1.31.1.1.1.1
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifInMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.2
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a multicast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.2
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifInBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.3
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a broadcast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.3
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifOutMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.4
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a multicast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.4
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifOutBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.5
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a broadcast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.5
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifHCInOctets
    oid: 1.3.6.1.2.1.31.1.1.1.6
    type: counter
    help: The total number of octets received on the interface, including framing
      characters - 1.3.6.1.2.1.31.1.1.1.6
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifHCInUcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.7
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were not addressed to a multicast or broadcast address at this sub-layer
      - 1.3.6.1.2.1.31.1.1.1.7
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifHCInMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.8
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a multicast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.8
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifHCInBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.9
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a broadcast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.9
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifHCOutOctets
    oid: 1.3.6.1.2.1.31.1.1.1.10
    type: counter
    help: The total number of octets transmitted out of the interface, including framing
      characters - 1.3.6.1.2.1.31.1.1.1.10
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifHCOutUcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.11
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were not addressed to a multicast or broadcast address at this sub-layer,
      including those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.11
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifHCOutMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.12
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a multicast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.12
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifHCOutBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.13
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a broadcast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.13
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifLinkUpDownTrapEnable
    oid: 1.3.6.1.2.1.31.1.1.1.14
    type: gauge
    help: Indicates whether linkUp/linkDown traps should be generated for this interface
      - 1.3.6.1.2.1.31.1.1.1.14
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifHighSpeed
    oid: 1.3.6.1.2.1.31.1.1.1.15
    type: gauge
    help: An estimate of the interface's current bandwidth in units of 1,000,000 bits
      per second - 1.3.6.1.2.1.31.1.1.1.15
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifPromiscuousMode
    oid: 1.3.6.1.2.1.31.1.1.1.16
    type: gauge
    help: This object has a value of false(2) if this interface only accepts packets/frames
      that are addressed to this station - 1.3.6.1.2.1.31.1.1.1.16
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifConnectorPresent
    oid: 1.3.6.1.2.1.31.1.1.1.17
    type: gauge
    help: This object has the value 'true(1)' if the interface sublayer has a physical
      connector and the value 'false(2)' otherwise. - 1.3.6.1.2.1.31.1.1.1.17
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifAlias
    oid: 1.3.6.1.2.1.31.1.1.1.18
    type: DisplayString
    help: This object is an 'alias' name for the interface as specified by a network
      manager, and provides a non-volatile 'handle' for the interface - 1.3.6.1.2.1.31.1.1.1.18
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
  - name: ifCounterDiscontinuityTime
    oid: 1.3.6.1.2.1.31.1.1.1.19
    type: gauge
    help: The value of sysUpTime on the most recent occasion at which any one or more
      of this interface's counters suffered a discontinuity - 1.3.6.1.2.1.31.1.1.1.19
    indexes:
    - labelname: ifDescr
      type: gauge
    lookups:
    - labels:
      - ifDescr
      labelname: ifDescr
      oid: 1.3.6.1.2.1.2.2.1.2
      type: DisplayString
if_mib_ifname:
  walk:
  - 1.3.6.1.2.1.1.3
  - 1.3.6.1.2.1.2
  - 1.3.6.1.2.1.31.1.1
  metrics:
  - name: sysUpTime
    oid: 1.3.6.1.2.1.1.3
    type: gauge
    help: The time (in hundredths of a second) since the network management portion
      of the system was last re-initialized. - 1.3.6.1.2.1.1.3
  - name: ifNumber
    oid: 1.3.6.1.2.1.2.1
    type: gauge
    help: The number of network interfaces (regardless of their current state) present
      on this system. - 1.3.6.1.2.1.2.1
  - name: ifIndex
    oid: 1.3.6.1.2.1.2.2.1.1
    type: gauge
    help: A unique value, greater than zero, for each interface - 1.3.6.1.2.1.2.2.1.1
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifDescr
    oid: 1.3.6.1.2.1.2.2.1.2
    type: DisplayString
    help: A textual string containing information about the interface - 1.3.6.1.2.1.2.2.1.2
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifType
    oid: 1.3.6.1.2.1.2.2.1.3
    type: gauge
    help: The type of interface - 1.3.6.1.2.1.2.2.1.3
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifMtu
    oid: 1.3.6.1.2.1.2.2.1.4
    type: gauge
    help: The size of the largest packet which can be sent/received on the interface,
      specified in octets - 1.3.6.1.2.1.2.2.1.4
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifSpeed
    oid: 1.3.6.1.2.1.2.2.1.5
    type: gauge
    help: An estimate of the interface's current bandwidth in bits per second - 1.3.6.1.2.1.2.2.1.5
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifPhysAddress
    oid: 1.3.6.1.2.1.2.2.1.6
    type: PhysAddress48
    help: The interface's address at its protocol sub-layer - 1.3.6.1.2.1.2.2.1.6
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifAdminStatus
    oid: 1.3.6.1.2.1.2.2.1.7
    type: gauge
    help: The desired state of the interface - 1.3.6.1.2.1.2.2.1.7
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifOperStatus
    oid: 1.3.6.1.2.1.2.2.1.8
    type: gauge
    help: The current operational state of the interface - 1.3.6.1.2.1.2.2.1.8
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifLastChange
    oid: 1.3.6.1.2.1.2.2.1.9
    type: gauge
    help: The value of sysUpTime at the time the interface entered its current operational
      state - 1.3.6.1.2.1.2.2.1.9
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifInOctets
    oid: 1.3.6.1.2.1.2.2.1.10
    type: counter
    help: The total number of octets received on the interface, including framing
      characters - 1.3.6.1.2.1.2.2.1.10
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifInUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.11
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were not addressed to a multicast or broadcast address at this sub-layer
      - 1.3.6.1.2.1.2.2.1.11
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifInNUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.12
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a multicast or broadcast address at this sub-layer -
      1.3.6.1.2.1.2.2.1.12
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifInDiscards
    oid: 1.3.6.1.2.1.2.2.1.13
    type: counter
    help: The number of inbound packets which were chosen to be discarded even though
      no errors had been detected to prevent their being deliverable to a higher-layer
      protocol - 1.3.6.1.2.1.2.2.1.13
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifInErrors
    oid: 1.3.6.1.2.1.2.2.1.14
    type: counter
    help: For packet-oriented interfaces, the number of inbound packets that contained
      errors preventing them from being deliverable to a higher-layer protocol - 1.3.6.1.2.1.2.2.1.14
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifInUnknownProtos
    oid: 1.3.6.1.2.1.2.2.1.15
    type: counter
    help: For packet-oriented interfaces, the number of packets received via the interface
      which were discarded because of an unknown or unsupported protocol - 1.3.6.1.2.1.2.2.1.15
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifOutOctets
    oid: 1.3.6.1.2.1.2.2.1.16
    type: counter
    help: The total number of octets transmitted out of the interface, including framing
      characters - 1.3.6.1.2.1.2.2.1.16
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifOutUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.17
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were not addressed to a multicast or broadcast address at this sub-layer,
      including those that were discarded or not sent - 1.3.6.1.2.1.2.2.1.17
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifOutNUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.18
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a multicast or broadcast address at this sub-layer,
      including those that were discarded or not sent - 1.3.6.1.2.1.2.2.1.18
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifOutDiscards
    oid: 1.3.6.1.2.1.2.2.1.19
    type: counter
    help: The number of outbound packets which were chosen to be discarded even though
      no errors had been detected to prevent their being transmitted - 1.3.6.1.2.1.2.2.1.19
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifOutErrors
    oid: 1.3.6.1.2.1.2.2.1.20
    type: counter
    help: For packet-oriented interfaces, the number of outbound packets that could
      not be transmitted because of errors - 1.3.6.1.2.1.2.2.1.20
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifOutQLen
    oid: 1.3.6.1.2.1.2.2.1.21
    type: gauge
    help: The length of the output packet queue (in packets). - 1.3.6.1.2.1.2.2.1.21
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifName
    oid: 1.3.6.1.2.1.31.1.1.1.1
    type: DisplayString
    help: The textual name of the interface - 1.3.6.1.2.1.31.1.1.1.1
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifInMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.2
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a multicast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.2
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifInBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.3
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a broadcast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.3
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifOutMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.4
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a multicast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.4
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifOutBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.5
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a broadcast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.5
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifHCInOctets
    oid: 1.3.6.1.2.1.31.1.1.1.6
    type: counter
    help: The total number of octets received on the interface, including framing
      characters - 1.3.6.1.2.1.31.1.1.1.6
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifHCInUcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.7
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were not addressed to a multicast or broadcast address at this sub-layer
      - 1.3.6.1.2.1.31.1.1.1.7
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifHCInMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.8
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a multicast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.8
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifHCInBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.9
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a broadcast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.9
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifHCOutOctets
    oid: 1.3.6.1.2.1.31.1.1.1.10
    type: counter
    help: The total number of octets transmitted out of the interface, including framing
      characters - 1.3.6.1.2.1.31.1.1.1.10
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifHCOutUcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.11
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were not addressed to a multicast or broadcast address at this sub-layer,
      including those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.11
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifHCOutMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.12
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a multicast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.12
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifHCOutBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.13
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a broadcast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.13
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifLinkUpDownTrapEnable
    oid: 1.3.6.1.2.1.31.1.1.1.14
    type: gauge
    help: Indicates whether linkUp/linkDown traps should be generated for this interface
      - 1.3.6.1.2.1.31.1.1.1.14
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifHighSpeed
    oid: 1.3.6.1.2.1.31.1.1.1.15
    type: gauge
    help: An estimate of the interface's current bandwidth in units of 1,000,000 bits
      per second - 1.3.6.1.2.1.31.1.1.1.15
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifPromiscuousMode
    oid: 1.3.6.1.2.1.31.1.1.1.16
    type: gauge
    help: This object has a value of false(2) if this interface only accepts packets/frames
      that are addressed to this station - 1.3.6.1.2.1.31.1.1.1.16
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifConnectorPresent
    oid: 1.3.6.1.2.1.31.1.1.1.17
    type: gauge
    help: This object has the value 'true(1)' if the interface sublayer has a physical
      connector and the value 'false(2)' otherwise. - 1.3.6.1.2.1.31.1.1.1.17
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifAlias
    oid: 1.3.6.1.2.1.31.1.1.1.18
    type: DisplayString
    help: This object is an 'alias' name for the interface as specified by a network
      manager, and provides a non-volatile 'handle' for the interface - 1.3.6.1.2.1.31.1.1.1.18
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifCounterDiscontinuityTime
    oid: 1.3.6.1.2.1.31.1.1.1.19
    type: gauge
    help: The value of sysUpTime on the most recent occasion at which any one or more
      of this interface's counters suffered a discontinuity - 1.3.6.1.2.1.31.1.1.1.19
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
paloalto_fw:
  walk:
  - 1.3.6.1.2.1.1.3
  - 1.3.6.1.2.1.2
  - 1.3.6.1.2.1.25.1
  - 1.3.6.1.2.1.25.2
  - 1.3.6.1.2.1.25.3
  - 1.3.6.1.4.1.25461.2.1.2.1
  - 1.3.6.1.4.1.25461.2.1.2.3
  - 1.3.6.1.4.1.25461.2.1.2.5
  metrics:
  - name: sysUpTime
    oid: 1.3.6.1.2.1.1.3
    type: gauge
    help: The time (in hundredths of a second) since the network management portion
      of the system was last re-initialized. - 1.3.6.1.2.1.1.3
  - name: ifNumber
    oid: 1.3.6.1.2.1.2.1
    type: gauge
    help: The number of network interfaces (regardless of their current state) present
      on this system. - 1.3.6.1.2.1.2.1
  - name: ifIndex
    oid: 1.3.6.1.2.1.2.2.1.1
    type: gauge
    help: A unique value, greater than zero, for each interface - 1.3.6.1.2.1.2.2.1.1
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifDescr
    oid: 1.3.6.1.2.1.2.2.1.2
    type: DisplayString
    help: A textual string containing information about the interface - 1.3.6.1.2.1.2.2.1.2
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifType
    oid: 1.3.6.1.2.1.2.2.1.3
    type: gauge
    help: The type of interface - 1.3.6.1.2.1.2.2.1.3
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifMtu
    oid: 1.3.6.1.2.1.2.2.1.4
    type: gauge
    help: The size of the largest packet which can be sent/received on the interface,
      specified in octets - 1.3.6.1.2.1.2.2.1.4
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifSpeed
    oid: 1.3.6.1.2.1.2.2.1.5
    type: gauge
    help: An estimate of the interface's current bandwidth in bits per second - 1.3.6.1.2.1.2.2.1.5
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifPhysAddress
    oid: 1.3.6.1.2.1.2.2.1.6
    type: PhysAddress48
    help: The interface's address at its protocol sub-layer - 1.3.6.1.2.1.2.2.1.6
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifAdminStatus
    oid: 1.3.6.1.2.1.2.2.1.7
    type: gauge
    help: The desired state of the interface - 1.3.6.1.2.1.2.2.1.7
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOperStatus
    oid: 1.3.6.1.2.1.2.2.1.8
    type: gauge
    help: The current operational state of the interface - 1.3.6.1.2.1.2.2.1.8
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifLastChange
    oid: 1.3.6.1.2.1.2.2.1.9
    type: gauge
    help: The value of sysUpTime at the time the interface entered its current operational
      state - 1.3.6.1.2.1.2.2.1.9
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInOctets
    oid: 1.3.6.1.2.1.2.2.1.10
    type: counter
    help: The total number of octets received on the interface, including framing
      characters - 1.3.6.1.2.1.2.2.1.10
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.11
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were not addressed to a multicast or broadcast address at this sub-layer
      - 1.3.6.1.2.1.2.2.1.11
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInNUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.12
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a multicast or broadcast address at this sub-layer -
      1.3.6.1.2.1.2.2.1.12
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInDiscards
    oid: 1.3.6.1.2.1.2.2.1.13
    type: counter
    help: The number of inbound packets which were chosen to be discarded even though
      no errors had been detected to prevent their being deliverable to a higher-layer
      protocol - 1.3.6.1.2.1.2.2.1.13
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInErrors
    oid: 1.3.6.1.2.1.2.2.1.14
    type: counter
    help: For packet-oriented interfaces, the number of inbound packets that contained
      errors preventing them from being deliverable to a higher-layer protocol - 1.3.6.1.2.1.2.2.1.14
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifInUnknownProtos
    oid: 1.3.6.1.2.1.2.2.1.15
    type: counter
    help: For packet-oriented interfaces, the number of packets received via the interface
      which were discarded because of an unknown or unsupported protocol - 1.3.6.1.2.1.2.2.1.15
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutOctets
    oid: 1.3.6.1.2.1.2.2.1.16
    type: counter
    help: The total number of octets transmitted out of the interface, including framing
      characters - 1.3.6.1.2.1.2.2.1.16
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.17
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were not addressed to a multicast or broadcast address at this sub-layer,
      including those that were discarded or not sent - 1.3.6.1.2.1.2.2.1.17
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutNUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.18
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a multicast or broadcast address at this sub-layer,
      including those that were discarded or not sent - 1.3.6.1.2.1.2.2.1.18
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutDiscards
    oid: 1.3.6.1.2.1.2.2.1.19
    type: counter
    help: The number of outbound packets which were chosen to be discarded even though
      no errors had been detected to prevent their being transmitted - 1.3.6.1.2.1.2.2.1.19
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutErrors
    oid: 1.3.6.1.2.1.2.2.1.20
    type: counter
    help: For packet-oriented interfaces, the number of outbound packets that could
      not be transmitted because of errors - 1.3.6.1.2.1.2.2.1.20
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifOutQLen
    oid: 1.3.6.1.2.1.2.2.1.21
    type: gauge
    help: The length of the output packet queue (in packets). - 1.3.6.1.2.1.2.2.1.21
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: hrSystemUptime
    oid: 1.3.6.1.2.1.25.1.1
    type: gauge
    help: The amount of time since this host was last initialized - 1.3.6.1.2.1.25.1.1
  - name: hrSystemDate
    oid: 1.3.6.1.2.1.25.1.2
    type: OctetString
    help: The host's notion of the local date and time of day. - 1.3.6.1.2.1.25.1.2
  - name: hrSystemInitialLoadDevice
    oid: 1.3.6.1.2.1.25.1.3
    type: gauge
    help: The index of the hrDeviceEntry for the device from which this host is configured
      to load its initial operating system configuration (i.e., which operating system
      code and/or boot parameters) - 1.3.6.1.2.1.25.1.3
  - name: hrSystemInitialLoadParameters
    oid: 1.3.6.1.2.1.25.1.4
    type: OctetString
    help: This object contains the parameters (e.g - 1.3.6.1.2.1.25.1.4
  - name: hrSystemNumUsers
    oid: 1.3.6.1.2.1.25.1.5
    type: gauge
    help: The number of user sessions for which this host is storing state information
      - 1.3.6.1.2.1.25.1.5
  - name: hrSystemProcesses
    oid: 1.3.6.1.2.1.25.1.6
    type: gauge
    help: The number of process contexts currently loaded or running on this system.
      - 1.3.6.1.2.1.25.1.6
  - name: hrSystemMaxProcesses
    oid: 1.3.6.1.2.1.25.1.7
    type: gauge
    help: The maximum number of process contexts this system can support - 1.3.6.1.2.1.25.1.7
  - name: hrMemorySize
    oid: 1.3.6.1.2.1.25.2.2
    type: gauge
    help: The amount of physical read-write main memory, typically RAM, contained
      by the host. - 1.3.6.1.2.1.25.2.2
  - name: hrStorageIndex
    oid: 1.3.6.1.2.1.25.2.3.1.1
    type: gauge
    help: A unique value for each logical storage area contained by the host. - 1.3.6.1.2.1.25.2.3.1.1
    indexes:
    - labelname: hrStorageIndex
      type: gauge
  - name: hrStorageDescr
    oid: 1.3.6.1.2.1.25.2.3.1.3
    type: DisplayString
    help: A description of the type and instance of the storage described by this
      entry. - 1.3.6.1.2.1.25.2.3.1.3
    indexes:
    - labelname: hrStorageIndex
      type: gauge
  - name: hrStorageAllocationUnits
    oid: 1.3.6.1.2.1.25.2.3.1.4
    type: gauge
    help: The size, in bytes, of the data objects allocated from this pool - 1.3.6.1.2.1.25.2.3.1.4
    indexes:
    - labelname: hrStorageIndex
      type: gauge
  - name: hrStorageSize
    oid: 1.3.6.1.2.1.25.2.3.1.5
    type: gauge
    help: The size of the storage represented by this entry, in units of hrStorageAllocationUnits
      - 1.3.6.1.2.1.25.2.3.1.5
    indexes:
    - labelname: hrStorageIndex
      type: gauge
  - name: hrStorageUsed
    oid: 1.3.6.1.2.1.25.2.3.1.6
    type: gauge
    help: The amount of the storage represented by this entry that is allocated, in
      units of hrStorageAllocationUnits. - 1.3.6.1.2.1.25.2.3.1.6
    indexes:
    - labelname: hrStorageIndex
      type: gauge
  - name: hrStorageAllocationFailures
    oid: 1.3.6.1.2.1.25.2.3.1.7
    type: counter
    help: The number of requests for storage represented by this entry that could
      not be honored due to not enough storage - 1.3.6.1.2.1.25.2.3.1.7
    indexes:
    - labelname: hrStorageIndex
      type: gauge
  - name: hrDeviceIndex
    oid: 1.3.6.1.2.1.25.3.2.1.1
    type: gauge
    help: A unique value for each device contained by the host - 1.3.6.1.2.1.25.3.2.1.1
    indexes:
    - labelname: hrDeviceIndex
      type: gauge
  - name: hrDeviceDescr
    oid: 1.3.6.1.2.1.25.3.2.1.3
    type: DisplayString
    help: A textual description of this device, including the device's manufacturer
      and revision, and optionally, its serial number. - 1.3.6.1.2.1.25.3.2.1.3
    indexes:
    - labelname: hrDeviceIndex
      type: gauge
  - name: hrDeviceStatus
    oid: 1.3.6.1.2.1.25.3.2.1.5
    type: gauge
    help: The current operational state of the device described by this row of the
      table - 1.3.6.1.2.1.25.3.2.1.5
    indexes:
    - labelname: hrDeviceIndex
      type: gauge
  - name: hrDeviceErrors
    oid: 1.3.6.1.2.1.25.3.2.1.6
    type: counter
    help: The number of errors detected on this device - 1.3.6.1.2.1.25.3.2.1.6
    indexes:
    - labelname: hrDeviceIndex
      type: gauge
  - name: hrProcessorLoad
    oid: 1.3.6.1.2.1.25.3.3.1.2
    type: gauge
    help: The average, over the last minute, of the percentage of time that this processor
      was not idle - 1.3.6.1.2.1.25.3.3.1.2
    indexes:
    - labelname: hrDeviceIndex
      type: gauge
  - name: hrNetworkIfIndex
    oid: 1.3.6.1.2.1.25.3.4.1.1
    type: gauge
    help: The value of ifIndex which corresponds to this network device - 1.3.6.1.2.1.25.3.4.1.1
    indexes:
    - labelname: hrDeviceIndex
      type: gauge
  - name: hrPrinterStatus
    oid: 1.3.6.1.2.1.25.3.5.1.1
    type: gauge
    help: The current status of this printer device. - 1.3.6.1.2.1.25.3.5.1.1
    indexes:
    - labelname: hrDeviceIndex
      type: gauge
  - name: hrPrinterDetectedErrorState
    oid: 1.3.6.1.2.1.25.3.5.1.2
    type: OctetString
    help: This object represents any error conditions detected by the printer - 1.3.6.1.2.1.25.3.5.1.2
    indexes:
    - labelname: hrDeviceIndex
      type: gauge
  - name: hrDiskStorageAccess
    oid: 1.3.6.1.2.1.25.3.6.1.1
    type: gauge
    help: An indication if this long-term storage device is readable and writable
      or only readable - 1.3.6.1.2.1.25.3.6.1.1
    indexes:
    - labelname: hrDeviceIndex
      type: gauge
  - name: hrDiskStorageMedia
    oid: 1.3.6.1.2.1.25.3.6.1.2
    type: gauge
    help: An indication of the type of media used in this long- term storage device.
      - 1.3.6.1.2.1.25.3.6.1.2
    indexes:
    - labelname: hrDeviceIndex
      type: gauge
  - name: hrDiskStorageRemoveble
    oid: 1.3.6.1.2.1.25.3.6.1.3
    type: gauge
    help: Denotes whether or not the disk media may be removed from the drive. - 1.3.6.1.2.1.25.3.6.1.3
    indexes:
    - labelname: hrDeviceIndex
      type: gauge
  - name: hrDiskStorageCapacity
    oid: 1.3.6.1.2.1.25.3.6.1.4
    type: gauge
    help: The total size for this long-term storage device - 1.3.6.1.2.1.25.3.6.1.4
    indexes:
    - labelname: hrDeviceIndex
      type: gauge
  - name: hrPartitionIndex
    oid: 1.3.6.1.2.1.25.3.7.1.1
    type: gauge
    help: A unique value for each partition on this long-term storage device - 1.3.6.1.2.1.25.3.7.1.1
    indexes:
    - labelname: hrDeviceIndex
      type: gauge
    - labelname: hrPartitionIndex
      type: gauge
  - name: hrPartitionLabel
    oid: 1.3.6.1.2.1.25.3.7.1.2
    type: OctetString
    help: A textual description of this partition. - 1.3.6.1.2.1.25.3.7.1.2
    indexes:
    - labelname: hrDeviceIndex
      type: gauge
    - labelname: hrPartitionIndex
      type: gauge
  - name: hrPartitionID
    oid: 1.3.6.1.2.1.25.3.7.1.3
    type: OctetString
    help: A descriptor which uniquely represents this partition to the responsible
      operating system - 1.3.6.1.2.1.25.3.7.1.3
    indexes:
    - labelname: hrDeviceIndex
      type: gauge
    - labelname: hrPartitionIndex
      type: gauge
  - name: hrPartitionSize
    oid: 1.3.6.1.2.1.25.3.7.1.4
    type: gauge
    help: The size of this partition. - 1.3.6.1.2.1.25.3.7.1.4
    indexes:
    - labelname: hrDeviceIndex
      type: gauge
    - labelname: hrPartitionIndex
      type: gauge
  - name: hrPartitionFSIndex
    oid: 1.3.6.1.2.1.25.3.7.1.5
    type: gauge
    help: The index of the file system mounted on this partition - 1.3.6.1.2.1.25.3.7.1.5
    indexes:
    - labelname: hrDeviceIndex
      type: gauge
    - labelname: hrPartitionIndex
      type: gauge
  - name: hrFSIndex
    oid: 1.3.6.1.2.1.25.3.8.1.1
    type: gauge
    help: A unique value for each file system local to this host - 1.3.6.1.2.1.25.3.8.1.1
    indexes:
    - labelname: hrFSIndex
      type: gauge
  - name: hrFSMountPoint
    oid: 1.3.6.1.2.1.25.3.8.1.2
    type: OctetString
    help: The path name of the root of this file system. - 1.3.6.1.2.1.25.3.8.1.2
    indexes:
    - labelname: hrFSIndex
      type: gauge
  - name: hrFSRemoteMountPoint
    oid: 1.3.6.1.2.1.25.3.8.1.3
    type: OctetString
    help: A description of the name and/or address of the server that this file system
      is mounted from - 1.3.6.1.2.1.25.3.8.1.3
    indexes:
    - labelname: hrFSIndex
      type: gauge
  - name: hrFSAccess
    oid: 1.3.6.1.2.1.25.3.8.1.5
    type: gauge
    help: An indication if this file system is logically configured by the operating
      system to be readable and writable or only readable - 1.3.6.1.2.1.25.3.8.1.5
    indexes:
    - labelname: hrFSIndex
      type: gauge
  - name: hrFSBootable
    oid: 1.3.6.1.2.1.25.3.8.1.6
    type: gauge
    help: A flag indicating whether this file system is bootable. - 1.3.6.1.2.1.25.3.8.1.6
    indexes:
    - labelname: hrFSIndex
      type: gauge
  - name: hrFSStorageIndex
    oid: 1.3.6.1.2.1.25.3.8.1.7
    type: gauge
    help: The index of the hrStorageEntry that represents information about this file
      system - 1.3.6.1.2.1.25.3.8.1.7
    indexes:
    - labelname: hrFSIndex
      type: gauge
  - name: hrFSLastFullBackupDate
    oid: 1.3.6.1.2.1.25.3.8.1.8
    type: OctetString
    help: The last date at which this complete file system was copied to another storage
      device for backup - 1.3.6.1.2.1.25.3.8.1.8
    indexes:
    - labelname: hrFSIndex
      type: gauge
  - name: hrFSLastPartialBackupDate
    oid: 1.3.6.1.2.1.25.3.8.1.9
    type: OctetString
    help: The last date at which a portion of this file system was copied to another
      storage device for backup - 1.3.6.1.2.1.25.3.8.1.9
    indexes:
    - labelname: hrFSIndex
      type: gauge
  - name: panAhoSw
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.1
    type: counter
    help: The total usage of software for AHO - 1.3.6.1.4.1.25461.2.1.2.1.19.1
  - name: panDfaSw
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.2
    type: counter
    help: The total number of dfa match using software - 1.3.6.1.4.1.25461.2.1.2.1.19.2
  - name: panFlowHostServiceAllow
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.3
    type: counter
    help: Device management session allowed - 1.3.6.1.4.1.25461.2.1.2.1.19.3
  - name: panHaPathmonSent
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.4
    type: counter
    help: HA path-monitoring packets sent - 1.3.6.1.4.1.25461.2.1.2.1.19.4
  - name: panAhoFpga
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.5
    type: counter
    help: The total requests to FPGA for AHO - 1.3.6.1.4.1.25461.2.1.2.1.19.5
  - name: panDfaFpga
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.6
    type: counter
    help: The total requests to FPGA for DFA - 1.3.6.1.4.1.25461.2.1.2.1.19.6
  - name: panFpgaPkt
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.7
    type: counter
    help: The packets held because of requests to FPGA - 1.3.6.1.4.1.25461.2.1.2.1.19.7
  - name: panFlowDosAgMaxSessLimit
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.1
    type: counter
    help: Session limit reached for aggregate profile, drop session - 1.3.6.1.4.1.25461.2.1.2.1.19.8.1
  - name: panFlowDosBlkNumEntries
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.2
    type: counter
    help: Number of entries in DOS block table - 1.3.6.1.4.1.25461.2.1.2.1.19.8.2
  - name: panFlowDosClMaxSessLimit
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.3
    type: counter
    help: Session limit reached for classified profile, drop session - 1.3.6.1.4.1.25461.2.1.2.1.19.8.3
  - name: panFlowDosClSyncookieAckErr
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.4
    type: counter
    help: 'TCP SYN cookies: Invalid ACKs received, classified profile - 1.3.6.1.4.1.25461.2.1.2.1.19.8.4'
  - name: panFlowDosClSyncookieAckRcv
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.5
    type: counter
    help: 'TCP SYN cookies: ACKs to cookies received, classified profile - 1.3.6.1.4.1.25461.2.1.2.1.19.8.5'
  - name: panFlowDosClSyncookieBlkDur
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.6
    type: counter
    help: 'Packets dropped: Flagged for blocking and under block duration for cl -
      1.3.6.1.4.1.25461.2.1.2.1.19.8.6'
  - name: panFlowDosClSyncookieMax
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.7
    type: counter
    help: 'Packet dropped: SYN cookies maximum threshold reached, classified pro -
      1.3.6.1.4.1.25461.2.1.2.1.19.8.7'
  - name: panFlowDosClSyncookieSent
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.8
    type: counter
    help: 'TCP SYN cookies: cookies sent, classified profile - 1.3.6.1.4.1.25461.2.1.2.1.19.8.8'
  - name: panFlowMeterVsysThrottle
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.9
    type: counter
    help: 'Session metering: sessions throttled by vsys configuration - 1.3.6.1.4.1.25461.2.1.2.1.19.8.9'
  - name: panFlowPolicyDeny
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.10
    type: counter
    help: 'Session setup: denied by policy - 1.3.6.1.4.1.25461.2.1.2.1.19.8.10'
  - name: panFlowPolicyNat
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.11
    type: counter
    help: 'Session setup: source NAT IP/port allocation error - 1.3.6.1.4.1.25461.2.1.2.1.19.8.11'
  - name: panFlowScanDrop
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.12
    type: counter
    help: 'Session setup: denied by scan detection - 1.3.6.1.4.1.25461.2.1.2.1.19.8.12'
  - name: panFlowDosDropIpBlocked
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.13
    type: counter
    help: 'Packets dropped: Flagged for blocking and under block duration by oth -
      1.3.6.1.4.1.25461.2.1.2.1.19.8.13'
  - name: panFlowDosRedIcmp
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.14
    type: counter
    help: 'Packets dropped: Zone protection protocol ''icmp'' RED - 1.3.6.1.4.1.25461.2.1.2.1.19.8.14'
  - name: panFlowDosRedIcmp6
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.15
    type: counter
    help: 'Packets dropped: Zone protection protocol ''icmpv6'' RED - 1.3.6.1.4.1.25461.2.1.2.1.19.8.15'
  - name: panFlowDosRedIp
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.16
    type: counter
    help: 'Packets dropped: Zone protection protocol ''other-ip'' RED - 1.3.6.1.4.1.25461.2.1.2.1.19.8.16'
  - name: panFlowDosRedTcp
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.17
    type: counter
    help: 'Packets dropped: Zone protection protocol ''tcp-syn'' RED - 1.3.6.1.4.1.25461.2.1.2.1.19.8.17'
  - name: panFlowDosRedUdp
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.18
    type: counter
    help: 'Packets dropped: Zone protection protocol ''udp'' RED - 1.3.6.1.4.1.25461.2.1.2.1.19.8.18'
  - name: panFlowDosRuleAgBlkDur
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.19
    type: counter
    help: 'Packets dropped: Flagged for blocking and under block duration for ag -
      1.3.6.1.4.1.25461.2.1.2.1.19.8.19'
  - name: panFlowDosRuleAgRedAct
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.20
    type: counter
    help: 'Packets dropped: Activate aggregate RED threshold reached, random ear -
      1.3.6.1.4.1.25461.2.1.2.1.19.8.20'
  - name: panFlowDosRuleAgRedMax
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.21
    type: counter
    help: 'Packets dropped: Maximal aggregate RED threshold reached - 1.3.6.1.4.1.25461.2.1.2.1.19.8.21'
  - name: panFlowDosRuleDeny
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.22
    type: counter
    help: 'Packets dropped: Denied action by DoS policy - 1.3.6.1.4.1.25461.2.1.2.1.19.8.22'
  - name: panFlowDosRuleDrop
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.23
    type: counter
    help: 'Packets dropped: Rate limited or IP blocked - 1.3.6.1.4.1.25461.2.1.2.1.19.8.23'
  - name: panFlowDosRuleDropAggr
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.24
    type: counter
    help: 'Packets dropped: due to aggregate rate limiting - 1.3.6.1.4.1.25461.2.1.2.1.19.8.24'
  - name: panFlowDosRuleDropClBlkDur
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.25
    type: counter
    help: 'Packets dropped: Flagged for blocking and under block duration for cl -
      1.3.6.1.4.1.25461.2.1.2.1.19.8.25'
  - name: panFlowDosRuleDropClRedAct
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.26
    type: counter
    help: 'Packets dropped: Activate classified RED threshold reached, random ea -
      1.3.6.1.4.1.25461.2.1.2.1.19.8.26'
  - name: panFlowDosRuleDropClRedMax
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.27
    type: counter
    help: 'Packets dropped: Maximal classified RED threshold reached - 1.3.6.1.4.1.25461.2.1.2.1.19.8.27'
  - name: panFlowDosRuleDropClassified
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.28
    type: counter
    help: 'Packets dropped: due to classified rate limiting - 1.3.6.1.4.1.25461.2.1.2.1.19.8.28'
  - name: panFlowDosSyncookieBlkDur
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.29
    type: counter
    help: 'Packets dropped: Flagged for blocking and under block duration for ag -
      1.3.6.1.4.1.25461.2.1.2.1.19.8.29'
  - name: panFlowDosSyncookieMax
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.30
    type: counter
    help: 'Packet dropped: SYN cookies maximum threshold reached, aggregate prof -
      1.3.6.1.4.1.25461.2.1.2.1.19.8.30'
  - name: panFlowDosZoneRedAct
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.31
    type: counter
    help: 'Packets dropped: Activate zone RED threshold reached, random early dr -
      1.3.6.1.4.1.25461.2.1.2.1.19.8.31'
  - name: panFlowDosZoneRedMax
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.8.32
    type: counter
    help: 'Packets dropped: Maximal zone RED threshold reached - 1.3.6.1.4.1.25461.2.1.2.1.19.8.32'
  - name: panFlowFwdL3TtlZero
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.9.1
    type: counter
    help: 'Packets dropped: IP TTL reaches zero - 1.3.6.1.4.1.25461.2.1.2.1.19.9.1'
  - name: panFlowMeterHostThrottle
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.9.2
    type: counter
    help: 'Session metering: sessions throttled by management session threshold -
      1.3.6.1.4.1.25461.2.1.2.1.19.9.2'
  - name: panFlowHostServiceDeny
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.9.3
    type: counter
    help: Device management session denied - 1.3.6.1.4.1.25461.2.1.2.1.19.9.3
  - name: panFlowHostServiceUnknown
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.9.4
    type: counter
    help: 'Session discarded: unknown application to control plane - 1.3.6.1.4.1.25461.2.1.2.1.19.9.4'
  - name: panPktAllocFailure
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.9.5
    type: counter
    help: Packet allocation error - 1.3.6.1.4.1.25461.2.1.2.1.19.9.5
  - name: panPktAllocFailureCos
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.9.6
    type: counter
    help: Packet allocation error due to QoS control - 1.3.6.1.4.1.25461.2.1.2.1.19.9.6
  - name: panSessionDiscard
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.9.7
    type: counter
    help: Session set to discard by security policy check - 1.3.6.1.4.1.25461.2.1.2.1.19.9.7
  - name: panFlowIpfragFragErr
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.10.1
    type: counter
    help: 'Packet dropped: IP fragmentation error - 1.3.6.1.4.1.25461.2.1.2.1.19.10.1'
  - name: panFlowIpfragRecv
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.10.2
    type: counter
    help: IP fragments received - 1.3.6.1.4.1.25461.2.1.2.1.19.10.2
  - name: panTcpAllocWqeFailed
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.11.1
    type: counter
    help: wqe allocation failure in tcp - 1.3.6.1.4.1.25461.2.1.2.1.19.11.1
  - name: panTcpDeny
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.11.2
    type: counter
    help: session denied because of failure in tcp reassembly - 1.3.6.1.4.1.25461.2.1.2.1.19.11.2
  - name: panTcpDropOutOfWnd
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.11.3
    type: counter
    help: out-of-window packets dropped - 1.3.6.1.4.1.25461.2.1.2.1.19.11.3
  - name: panTcpDropPacket
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.11.4
    type: counter
    help: packets dropped because of failure in tcp reassembly - 1.3.6.1.4.1.25461.2.1.2.1.19.11.4
  - name: panFlowActionClose
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.11.5
    type: counter
    help: TCP sessions closed via injecting RST - 1.3.6.1.4.1.25461.2.1.2.1.19.11.5
  - name: panFlowActionReset
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.11.6
    type: counter
    help: TCP clients reset via responding RST - 1.3.6.1.4.1.25461.2.1.2.1.19.11.6
  - name: panFlowTcpNonSyn
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.11.7
    type: counter
    help: Non-SYN TCP packets without session match - 1.3.6.1.4.1.25461.2.1.2.1.19.11.7
  - name: panTcpExceedSegLimit
    oid: 1.3.6.1.4.1.25461.2.1.2.1.19.11.8
    type: counter
    help: packets dropped due to the limitation on global tcp out-of-order pack -
      1.3.6.1.4.1.25461.2.1.2.1.19.11.8
  - name: panSysSwVersion
    oid: 1.3.6.1.4.1.25461.2.1.2.1.1
    type: DisplayString
    help: Full software version - 1.3.6.1.4.1.25461.2.1.2.1.1
  - name: panSysHwVersion
    oid: 1.3.6.1.4.1.25461.2.1.2.1.2
    type: DisplayString
    help: Hardware version of the unit. - 1.3.6.1.4.1.25461.2.1.2.1.2
  - name: panSysSerialNumber
    oid: 1.3.6.1.4.1.25461.2.1.2.1.3
    type: DisplayString
    help: The serial number of the unit - 1.3.6.1.4.1.25461.2.1.2.1.3
  - name: panSysTimeZoneOffset
    oid: 1.3.6.1.4.1.25461.2.1.2.1.4
    type: gauge
    help: The offset in seconds from UTC of the system's time zone - 1.3.6.1.4.1.25461.2.1.2.1.4
  - name: panSysDaylightSaving
    oid: 1.3.6.1.4.1.25461.2.1.2.1.5
    type: gauge
    help: Whether daylight savings are in currently in effect for the system's time
      zone. - 1.3.6.1.4.1.25461.2.1.2.1.5
  - name: panSysVpnClientVersion
    oid: 1.3.6.1.4.1.25461.2.1.2.1.6
    type: DisplayString
    help: Currently installed VPN client package version - 1.3.6.1.4.1.25461.2.1.2.1.6
  - name: panSysAppVersion
    oid: 1.3.6.1.4.1.25461.2.1.2.1.7
    type: DisplayString
    help: Currently installed application definition version - 1.3.6.1.4.1.25461.2.1.2.1.7
  - name: panSysAvVersion
    oid: 1.3.6.1.4.1.25461.2.1.2.1.8
    type: DisplayString
    help: Currently installed antivirus version - 1.3.6.1.4.1.25461.2.1.2.1.8
  - name: panSysThreatVersion
    oid: 1.3.6.1.4.1.25461.2.1.2.1.9
    type: DisplayString
    help: Currently installed threat definition version - 1.3.6.1.4.1.25461.2.1.2.1.9
  - name: panSysUrlFilteringVersion
    oid: 1.3.6.1.4.1.25461.2.1.2.1.10
    type: DisplayString
    help: Currently installed URL filtering version - 1.3.6.1.4.1.25461.2.1.2.1.10
  - name: panSysHAState
    oid: 1.3.6.1.4.1.25461.2.1.2.1.11
    type: DisplayString
    help: Current high-availability state. - 1.3.6.1.4.1.25461.2.1.2.1.11
  - name: panSysHAPeerState
    oid: 1.3.6.1.4.1.25461.2.1.2.1.12
    type: DisplayString
    help: Current peer high-availability state. - 1.3.6.1.4.1.25461.2.1.2.1.12
  - name: panSysHAMode
    oid: 1.3.6.1.4.1.25461.2.1.2.1.13
    type: DisplayString
    help: Current high-availability mode (disabled, active-passive, or active-active).
      - 1.3.6.1.4.1.25461.2.1.2.1.13
  - name: panSysUrlFilteringDatabase
    oid: 1.3.6.1.4.1.25461.2.1.2.1.14
    type: DisplayString
    help: Current installed URL filtering database (surfcontrol, brightcloud, etc)
      - 1.3.6.1.4.1.25461.2.1.2.1.14
  - name: panSysGlobalProtectClientVersion
    oid: 1.3.6.1.4.1.25461.2.1.2.1.15
    type: DisplayString
    help: Currently installed global-protect client package version - 1.3.6.1.4.1.25461.2.1.2.1.15
  - name: panSysOpswatDatafileVersion
    oid: 1.3.6.1.4.1.25461.2.1.2.1.16
    type: DisplayString
    help: Currently installed opswat database version - 1.3.6.1.4.1.25461.2.1.2.1.16
  - name: panSysWildfireVersion
    oid: 1.3.6.1.4.1.25461.2.1.2.1.17
    type: DisplayString
    help: Currently installed wildfire content version - 1.3.6.1.4.1.25461.2.1.2.1.17
  - name: panSysWildfirePrivateCloudVersion
    oid: 1.3.6.1.4.1.25461.2.1.2.1.18
    type: DisplayString
    help: Currently installed wildfire private cloud content version - 1.3.6.1.4.1.25461.2.1.2.1.18
  - name: panSessionUtilization
    oid: 1.3.6.1.4.1.25461.2.1.2.3.1
    type: gauge
    help: Session table utilization percentage - 1.3.6.1.4.1.25461.2.1.2.3.1
  - name: panSessionMax
    oid: 1.3.6.1.4.1.25461.2.1.2.3.2
    type: gauge
    help: Total number of sessions supported. - 1.3.6.1.4.1.25461.2.1.2.3.2
  - name: panSessionActive
    oid: 1.3.6.1.4.1.25461.2.1.2.3.3
    type: gauge
    help: Total number of active sessions. - 1.3.6.1.4.1.25461.2.1.2.3.3
  - name: panSessionActiveTcp
    oid: 1.3.6.1.4.1.25461.2.1.2.3.4
    type: gauge
    help: Total number of active TCP sessions. - 1.3.6.1.4.1.25461.2.1.2.3.4
  - name: panSessionActiveUdp
    oid: 1.3.6.1.4.1.25461.2.1.2.3.5
    type: gauge
    help: Total number of active UDP sessions. - 1.3.6.1.4.1.25461.2.1.2.3.5
  - name: panSessionActiveICMP
    oid: 1.3.6.1.4.1.25461.2.1.2.3.6
    type: gauge
    help: Total number of active ICMP sessions. - 1.3.6.1.4.1.25461.2.1.2.3.6
  - name: panSessionActiveSslProxy
    oid: 1.3.6.1.4.1.25461.2.1.2.3.7
    type: gauge
    help: Total number of active SSL proxy sessions. - 1.3.6.1.4.1.25461.2.1.2.3.7
  - name: panSessionSslProxyUtilization
    oid: 1.3.6.1.4.1.25461.2.1.2.3.8
    type: gauge
    help: SSL proxy Session utilization percentage - 1.3.6.1.4.1.25461.2.1.2.3.8
  - name: panVsysId
    oid: 1.3.6.1.4.1.25461.2.1.2.3.9.1.1
    type: gauge
    help: Vsys id - 1.3.6.1.4.1.25461.2.1.2.3.9.1.1
    indexes:
    - labelname: panVsysId
      type: gauge
  - name: panVsysName
    oid: 1.3.6.1.4.1.25461.2.1.2.3.9.1.2
    type: DisplayString
    help: User assigned vsys name (empty string if not available) - 1.3.6.1.4.1.25461.2.1.2.3.9.1.2
    indexes:
    - labelname: panVsysId
      type: gauge
  - name: panVsysSessionUtilizationPct
    oid: 1.3.6.1.4.1.25461.2.1.2.3.9.1.3
    type: gauge
    help: Vsys utilization percentage, if session limit is configured - 1.3.6.1.4.1.25461.2.1.2.3.9.1.3
    indexes:
    - labelname: panVsysId
      type: gauge
  - name: panVsysActiveSessions
    oid: 1.3.6.1.4.1.25461.2.1.2.3.9.1.4
    type: gauge
    help: Active sessions on this Vsys - 1.3.6.1.4.1.25461.2.1.2.3.9.1.4
    indexes:
    - labelname: panVsysId
      type: gauge
  - name: panVsysMaxSessions
    oid: 1.3.6.1.4.1.25461.2.1.2.3.9.1.5
    type: gauge
    help: Max sessions on this Vsys, if session limit is configured - 1.3.6.1.4.1.25461.2.1.2.3.9.1.5
    indexes:
    - labelname: panVsysId
      type: gauge
  - name: panGPGWUtilizationPct
    oid: 1.3.6.1.4.1.25461.2.1.2.5.1.1
    type: gauge
    help: GlobalProtect Gateway utilization percentage - 1.3.6.1.4.1.25461.2.1.2.5.1.1
  - name: panGPGWUtilizationMaxTunnels
    oid: 1.3.6.1.4.1.25461.2.1.2.5.1.2
    type: gauge
    help: Max tunnels allowed - 1.3.6.1.4.1.25461.2.1.2.5.1.2
  - name: panGPGWUtilizationActiveTunnels
    oid: 1.3.6.1.4.1.25461.2.1.2.5.1.3
    type: gauge
    help: Number of active tunnels - 1.3.6.1.4.1.25461.2.1.2.5.1.3
servertech_sentry3:
  walk:
  - 1.3.6.1.2.1.1.3
  - 1.3.6.1.4.1.1718.3.2.2
  - 1.3.6.1.4.1.1718.3.2.3
  metrics:
  - name: sysUpTime
    oid: 1.3.6.1.2.1.1.3
    type: gauge
    help: The time (in hundredths of a second) since the network management portion
      of the system was last re-initialized. - 1.3.6.1.2.1.1.3
  - name: infeedIndex
    oid: 1.3.6.1.4.1.1718.3.2.2.1.1
    type: gauge
    help: Index for the input feed table. - 1.3.6.1.4.1.1718.3.2.2.1.1
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
  - name: infeedID
    oid: 1.3.6.1.4.1.1718.3.2.2.1.2
    type: DisplayString
    help: The ID of the input feed. - 1.3.6.1.4.1.1718.3.2.2.1.2
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
  - name: infeedName
    oid: 1.3.6.1.4.1.1718.3.2.2.1.3
    type: DisplayString
    help: The name of the input feed. - 1.3.6.1.4.1.1718.3.2.2.1.3
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
  - name: infeedCapabilities
    oid: 1.3.6.1.4.1.1718.3.2.2.1.4
    type: OctetString
    help: The capabilities of the input feed. - 1.3.6.1.4.1.1718.3.2.2.1.4
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
  - name: infeedStatus
    oid: 1.3.6.1.4.1.1718.3.2.2.1.5
    type: gauge
    help: The status of the input feed line - 1.3.6.1.4.1.1718.3.2.2.1.5
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
  - name: infeedLoadStatus
    oid: 1.3.6.1.4.1.1718.3.2.2.1.6
    type: gauge
    help: The status of the load measured on the input feed line - 1.3.6.1.4.1.1718.3.2.2.1.6
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
  - name: infeedLoadValue
    oid: 1.3.6.1.4.1.1718.3.2.2.1.7
    type: gauge
    help: The load measured on the input feed line - 1.3.6.1.4.1.1718.3.2.2.1.7
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
  - name: infeedLoadHighThresh
    oid: 1.3.6.1.4.1.1718.3.2.2.1.8
    type: gauge
    help: The load high threshold value of the input feed line in Amps. - 1.3.6.1.4.1.1718.3.2.2.1.8
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
  - name: infeedOutletCount
    oid: 1.3.6.1.4.1.1718.3.2.2.1.9
    type: gauge
    help: The number of controlled and/or monitored outlets on the input feed. - 1.3.6.1.4.1.1718.3.2.2.1.9
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
  - name: infeedCapacity
    oid: 1.3.6.1.4.1.1718.3.2.2.1.10
    type: gauge
    help: The load capacity of the input feed line - 1.3.6.1.4.1.1718.3.2.2.1.10
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
  - name: infeedVoltage
    oid: 1.3.6.1.4.1.1718.3.2.2.1.11
    type: gauge
    help: The line-to-line voltage of the input feed - 1.3.6.1.4.1.1718.3.2.2.1.11
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
  - name: infeedPower
    oid: 1.3.6.1.4.1.1718.3.2.2.1.12
    type: gauge
    help: The active power consumption of the input feed phase - 1.3.6.1.4.1.1718.3.2.2.1.12
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
  - name: infeedApparentPower
    oid: 1.3.6.1.4.1.1718.3.2.2.1.13
    type: gauge
    help: The apparent power consumption of the input feed phase - 1.3.6.1.4.1.1718.3.2.2.1.13
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
  - name: infeedPowerFactor
    oid: 1.3.6.1.4.1.1718.3.2.2.1.14
    type: gauge
    help: The power factor of the input feed phase - 1.3.6.1.4.1.1718.3.2.2.1.14
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
  - name: infeedCrestFactor
    oid: 1.3.6.1.4.1.1718.3.2.2.1.15
    type: gauge
    help: The crest factor for the load of the input feed phase - 1.3.6.1.4.1.1718.3.2.2.1.15
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
  - name: infeedEnergy
    oid: 1.3.6.1.4.1.1718.3.2.2.1.16
    type: gauge
    help: The energy consumption of the input feed phase - 1.3.6.1.4.1.1718.3.2.2.1.16
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
  - name: infeedReactance
    oid: 1.3.6.1.4.1.1718.3.2.2.1.17
    type: gauge
    help: The characterization of the phase relation between the voltage and current
      of the input feed phase. - 1.3.6.1.4.1.1718.3.2.2.1.17
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
  - name: infeedPhaseVoltage
    oid: 1.3.6.1.4.1.1718.3.2.2.1.18
    type: gauge
    help: The voltage measured for the input feed phase - 1.3.6.1.4.1.1718.3.2.2.1.18
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
  - name: infeedPhaseCurrent
    oid: 1.3.6.1.4.1.1718.3.2.2.1.19
    type: gauge
    help: The current measured for the input feed phase - 1.3.6.1.4.1.1718.3.2.2.1.19
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
  - name: infeedCapacityUsed
    oid: 1.3.6.1.4.1.1718.3.2.2.1.20
    type: gauge
    help: The used percentage of the input feed line load capacity (infeedLoadValue
      / infeedCapacity x 100) - 1.3.6.1.4.1.1718.3.2.2.1.20
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
  - name: infeedLineID
    oid: 1.3.6.1.4.1.1718.3.2.2.1.21
    type: DisplayString
    help: The ID of the input feed line. - 1.3.6.1.4.1.1718.3.2.2.1.21
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
  - name: infeedLineToLineID
    oid: 1.3.6.1.4.1.1718.3.2.2.1.22
    type: DisplayString
    help: The line-to-line ID of the input feed. - 1.3.6.1.4.1.1718.3.2.2.1.22
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
  - name: infeedPhaseID
    oid: 1.3.6.1.4.1.1718.3.2.2.1.23
    type: DisplayString
    help: The ID of the input feed phase. - 1.3.6.1.4.1.1718.3.2.2.1.23
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
  - name: infeedVACapacity
    oid: 1.3.6.1.4.1.1718.3.2.2.1.24
    type: gauge
    help: The apparent power capacity of the input feed circuit - 1.3.6.1.4.1.1718.3.2.2.1.24
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
  - name: infeedVACapacityUsed
    oid: 1.3.6.1.4.1.1718.3.2.2.1.25
    type: gauge
    help: The used percentage of the input feed circuit apparent power capacity (infeedApparentPower
      / infeedVACapacity x 100) - 1.3.6.1.4.1.1718.3.2.2.1.25
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
  - name: outletIndex
    oid: 1.3.6.1.4.1.1718.3.2.3.1.1
    type: gauge
    help: Index for the outlet table. - 1.3.6.1.4.1.1718.3.2.3.1.1
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
    - labelname: outletIndex
      type: gauge
  - name: outletID
    oid: 1.3.6.1.4.1.1718.3.2.3.1.2
    type: DisplayString
    help: The ID of the outlet. - 1.3.6.1.4.1.1718.3.2.3.1.2
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
    - labelname: outletIndex
      type: gauge
  - name: outletName
    oid: 1.3.6.1.4.1.1718.3.2.3.1.3
    type: DisplayString
    help: The name of the outlet. - 1.3.6.1.4.1.1718.3.2.3.1.3
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
    - labelname: outletIndex
      type: gauge
  - name: outletCapabilities
    oid: 1.3.6.1.4.1.1718.3.2.3.1.4
    type: OctetString
    help: The capabilities of the outlet. - 1.3.6.1.4.1.1718.3.2.3.1.4
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
    - labelname: outletIndex
      type: gauge
  - name: outletStatus
    oid: 1.3.6.1.4.1.1718.3.2.3.1.5
    type: gauge
    help: The status of the outlet - 1.3.6.1.4.1.1718.3.2.3.1.5
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
    - labelname: outletIndex
      type: gauge
  - name: outletLoadStatus
    oid: 1.3.6.1.4.1.1718.3.2.3.1.6
    type: gauge
    help: The status of the load measured on the outlet - 1.3.6.1.4.1.1718.3.2.3.1.6
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
    - labelname: outletIndex
      type: gauge
  - name: outletLoadValue
    oid: 1.3.6.1.4.1.1718.3.2.3.1.7
    type: gauge
    help: The load measured on the outlet - 1.3.6.1.4.1.1718.3.2.3.1.7
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
    - labelname: outletIndex
      type: gauge
  - name: outletLoadLowThresh
    oid: 1.3.6.1.4.1.1718.3.2.3.1.8
    type: gauge
    help: The load low threshold value of the outlet in Amps. - 1.3.6.1.4.1.1718.3.2.3.1.8
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
    - labelname: outletIndex
      type: gauge
  - name: outletLoadHighThresh
    oid: 1.3.6.1.4.1.1718.3.2.3.1.9
    type: gauge
    help: The load high threshold value of the outlet in Amps. - 1.3.6.1.4.1.1718.3.2.3.1.9
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
    - labelname: outletIndex
      type: gauge
  - name: outletControlState
    oid: 1.3.6.1.4.1.1718.3.2.3.1.10
    type: gauge
    help: The control state of the outlet - 1.3.6.1.4.1.1718.3.2.3.1.10
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
    - labelname: outletIndex
      type: gauge
  - name: outletControlAction
    oid: 1.3.6.1.4.1.1718.3.2.3.1.11
    type: gauge
    help: An action to change the control state of the outlet - 1.3.6.1.4.1.1718.3.2.3.1.11
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
    - labelname: outletIndex
      type: gauge
  - name: outletCapacity
    oid: 1.3.6.1.4.1.1718.3.2.3.1.12
    type: gauge
    help: The load capacity of the outlet - 1.3.6.1.4.1.1718.3.2.3.1.12
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
    - labelname: outletIndex
      type: gauge
  - name: outletVoltage
    oid: 1.3.6.1.4.1.1718.3.2.3.1.13
    type: gauge
    help: The voltage of the outlet - 1.3.6.1.4.1.1718.3.2.3.1.13
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
    - labelname: outletIndex
      type: gauge
  - name: outletPower
    oid: 1.3.6.1.4.1.1718.3.2.3.1.14
    type: gauge
    help: The active power consumption of the device plugged into the outlet - 1.3.6.1.4.1.1718.3.2.3.1.14
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
    - labelname: outletIndex
      type: gauge
  - name: outletApparentPower
    oid: 1.3.6.1.4.1.1718.3.2.3.1.15
    type: gauge
    help: The apparent power consumption of the device plugged into the outlet - 1.3.6.1.4.1.1718.3.2.3.1.15
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
    - labelname: outletIndex
      type: gauge
  - name: outletPowerFactor
    oid: 1.3.6.1.4.1.1718.3.2.3.1.16
    type: gauge
    help: The power factor of the device plugged into the outlet - 1.3.6.1.4.1.1718.3.2.3.1.16
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
    - labelname: outletIndex
      type: gauge
  - name: outletCrestFactor
    oid: 1.3.6.1.4.1.1718.3.2.3.1.17
    type: gauge
    help: The crest factor for the load of the device plugged into the outlet - 1.3.6.1.4.1.1718.3.2.3.1.17
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
    - labelname: outletIndex
      type: gauge
  - name: outletEnergy
    oid: 1.3.6.1.4.1.1718.3.2.3.1.18
    type: gauge
    help: The energy consumption of the device plugged into the outlet - 1.3.6.1.4.1.1718.3.2.3.1.18
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
    - labelname: outletIndex
      type: gauge
  - name: outletWakeupState
    oid: 1.3.6.1.4.1.1718.3.2.3.1.19
    type: gauge
    help: The wakeup state of the outlet. - 1.3.6.1.4.1.1718.3.2.3.1.19
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
    - labelname: outletIndex
      type: gauge
  - name: outletPostOnDelay
    oid: 1.3.6.1.4.1.1718.3.2.3.1.20
    type: gauge
    help: The post-on delay of the outlet. - 1.3.6.1.4.1.1718.3.2.3.1.20
    indexes:
    - labelname: towerIndex
      type: gauge
    - labelname: infeedIndex
      type: gauge
    - labelname: outletIndex
      type: gauge
synology:
  walk:
  - 1.3.6.1.2.1.1.3
  - 1.3.6.1.2.1.2
  - 1.3.6.1.2.1.25.2
  - 1.3.6.1.2.1.31.1.1
  - 1.3.6.1.4.1.2021.10.1.2
  - 1.3.6.1.4.1.2021.10.1.5
  - 1.3.6.1.4.1.2021.11.10
  - 1.3.6.1.4.1.2021.11.11
  - 1.3.6.1.4.1.2021.11.9
  - 1.3.6.1.4.1.2021.4
  - 1.3.6.1.4.1.6574.1
  - 1.3.6.1.4.1.6574.101
  - 1.3.6.1.4.1.6574.102
  - 1.3.6.1.4.1.6574.104
  - 1.3.6.1.4.1.6574.2
  - 1.3.6.1.4.1.6574.3
  - 1.3.6.1.4.1.6574.4
  - 1.3.6.1.4.1.6574.5
  - 1.3.6.1.4.1.6574.6
  metrics:
  - name: sysUpTime
    oid: 1.3.6.1.2.1.1.3
    type: gauge
    help: The time (in hundredths of a second) since the network management portion
      of the system was last re-initialized. - 1.3.6.1.2.1.1.3
  - name: ifNumber
    oid: 1.3.6.1.2.1.2.1
    type: gauge
    help: The number of network interfaces (regardless of their current state) present
      on this system. - 1.3.6.1.2.1.2.1
  - name: ifIndex
    oid: 1.3.6.1.2.1.2.2.1.1
    type: gauge
    help: A unique value, greater than zero, for each interface - 1.3.6.1.2.1.2.2.1.1
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifDescr
    oid: 1.3.6.1.2.1.2.2.1.2
    type: DisplayString
    help: A textual string containing information about the interface - 1.3.6.1.2.1.2.2.1.2
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifType
    oid: 1.3.6.1.2.1.2.2.1.3
    type: gauge
    help: The type of interface - 1.3.6.1.2.1.2.2.1.3
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifMtu
    oid: 1.3.6.1.2.1.2.2.1.4
    type: gauge
    help: The size of the largest packet which can be sent/received on the interface,
      specified in octets - 1.3.6.1.2.1.2.2.1.4
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifSpeed
    oid: 1.3.6.1.2.1.2.2.1.5
    type: gauge
    help: An estimate of the interface's current bandwidth in bits per second - 1.3.6.1.2.1.2.2.1.5
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifPhysAddress
    oid: 1.3.6.1.2.1.2.2.1.6
    type: PhysAddress48
    help: The interface's address at its protocol sub-layer - 1.3.6.1.2.1.2.2.1.6
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifAdminStatus
    oid: 1.3.6.1.2.1.2.2.1.7
    type: gauge
    help: The desired state of the interface - 1.3.6.1.2.1.2.2.1.7
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifOperStatus
    oid: 1.3.6.1.2.1.2.2.1.8
    type: gauge
    help: The current operational state of the interface - 1.3.6.1.2.1.2.2.1.8
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifLastChange
    oid: 1.3.6.1.2.1.2.2.1.9
    type: gauge
    help: The value of sysUpTime at the time the interface entered its current operational
      state - 1.3.6.1.2.1.2.2.1.9
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifInOctets
    oid: 1.3.6.1.2.1.2.2.1.10
    type: counter
    help: The total number of octets received on the interface, including framing
      characters - 1.3.6.1.2.1.2.2.1.10
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifInUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.11
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were not addressed to a multicast or broadcast address at this sub-layer
      - 1.3.6.1.2.1.2.2.1.11
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifInNUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.12
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a multicast or broadcast address at this sub-layer -
      1.3.6.1.2.1.2.2.1.12
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifInDiscards
    oid: 1.3.6.1.2.1.2.2.1.13
    type: counter
    help: The number of inbound packets which were chosen to be discarded even though
      no errors had been detected to prevent their being deliverable to a higher-layer
      protocol - 1.3.6.1.2.1.2.2.1.13
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifInErrors
    oid: 1.3.6.1.2.1.2.2.1.14
    type: counter
    help: For packet-oriented interfaces, the number of inbound packets that contained
      errors preventing them from being deliverable to a higher-layer protocol - 1.3.6.1.2.1.2.2.1.14
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifInUnknownProtos
    oid: 1.3.6.1.2.1.2.2.1.15
    type: counter
    help: For packet-oriented interfaces, the number of packets received via the interface
      which were discarded because of an unknown or unsupported protocol - 1.3.6.1.2.1.2.2.1.15
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifOutOctets
    oid: 1.3.6.1.2.1.2.2.1.16
    type: counter
    help: The total number of octets transmitted out of the interface, including framing
      characters - 1.3.6.1.2.1.2.2.1.16
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifOutUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.17
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were not addressed to a multicast or broadcast address at this sub-layer,
      including those that were discarded or not sent - 1.3.6.1.2.1.2.2.1.17
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifOutNUcastPkts
    oid: 1.3.6.1.2.1.2.2.1.18
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a multicast or broadcast address at this sub-layer,
      including those that were discarded or not sent - 1.3.6.1.2.1.2.2.1.18
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifOutDiscards
    oid: 1.3.6.1.2.1.2.2.1.19
    type: counter
    help: The number of outbound packets which were chosen to be discarded even though
      no errors had been detected to prevent their being transmitted - 1.3.6.1.2.1.2.2.1.19
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifOutErrors
    oid: 1.3.6.1.2.1.2.2.1.20
    type: counter
    help: For packet-oriented interfaces, the number of outbound packets that could
      not be transmitted because of errors - 1.3.6.1.2.1.2.2.1.20
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifOutQLen
    oid: 1.3.6.1.2.1.2.2.1.21
    type: gauge
    help: The length of the output packet queue (in packets). - 1.3.6.1.2.1.2.2.1.21
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: hrMemorySize
    oid: 1.3.6.1.2.1.25.2.2
    type: gauge
    help: The amount of physical read-write main memory, typically RAM, contained
      by the host. - 1.3.6.1.2.1.25.2.2
  - name: hrStorageIndex
    oid: 1.3.6.1.2.1.25.2.3.1.1
    type: gauge
    help: A unique value for each logical storage area contained by the host. - 1.3.6.1.2.1.25.2.3.1.1
    indexes:
    - labelname: hrStorageDescr
      type: gauge
    lookups:
    - labels:
      - hrStorageDescr
      labelname: hrStorageDescr
      oid: 1.3.6.1.2.1.25.2.3.1.3
      type: DisplayString
  - name: hrStorageDescr
    oid: 1.3.6.1.2.1.25.2.3.1.3
    type: DisplayString
    help: A description of the type and instance of the storage described by this
      entry. - 1.3.6.1.2.1.25.2.3.1.3
    indexes:
    - labelname: hrStorageDescr
      type: gauge
    lookups:
    - labels:
      - hrStorageDescr
      labelname: hrStorageDescr
      oid: 1.3.6.1.2.1.25.2.3.1.3
      type: DisplayString
  - name: hrStorageAllocationUnits
    oid: 1.3.6.1.2.1.25.2.3.1.4
    type: gauge
    help: The size, in bytes, of the data objects allocated from this pool - 1.3.6.1.2.1.25.2.3.1.4
    indexes:
    - labelname: hrStorageDescr
      type: gauge
    lookups:
    - labels:
      - hrStorageDescr
      labelname: hrStorageDescr
      oid: 1.3.6.1.2.1.25.2.3.1.3
      type: DisplayString
  - name: hrStorageSize
    oid: 1.3.6.1.2.1.25.2.3.1.5
    type: gauge
    help: The size of the storage represented by this entry, in units of hrStorageAllocationUnits
      - 1.3.6.1.2.1.25.2.3.1.5
    indexes:
    - labelname: hrStorageDescr
      type: gauge
    lookups:
    - labels:
      - hrStorageDescr
      labelname: hrStorageDescr
      oid: 1.3.6.1.2.1.25.2.3.1.3
      type: DisplayString
  - name: hrStorageUsed
    oid: 1.3.6.1.2.1.25.2.3.1.6
    type: gauge
    help: The amount of the storage represented by this entry that is allocated, in
      units of hrStorageAllocationUnits. - 1.3.6.1.2.1.25.2.3.1.6
    indexes:
    - labelname: hrStorageDescr
      type: gauge
    lookups:
    - labels:
      - hrStorageDescr
      labelname: hrStorageDescr
      oid: 1.3.6.1.2.1.25.2.3.1.3
      type: DisplayString
  - name: hrStorageAllocationFailures
    oid: 1.3.6.1.2.1.25.2.3.1.7
    type: counter
    help: The number of requests for storage represented by this entry that could
      not be honored due to not enough storage - 1.3.6.1.2.1.25.2.3.1.7
    indexes:
    - labelname: hrStorageDescr
      type: gauge
    lookups:
    - labels:
      - hrStorageDescr
      labelname: hrStorageDescr
      oid: 1.3.6.1.2.1.25.2.3.1.3
      type: DisplayString
  - name: ifName
    oid: 1.3.6.1.2.1.31.1.1.1.1
    type: DisplayString
    help: The textual name of the interface - 1.3.6.1.2.1.31.1.1.1.1
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifInMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.2
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a multicast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.2
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifInBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.3
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a broadcast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.3
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifOutMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.4
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a multicast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.4
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifOutBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.5
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a broadcast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.5
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifHCInOctets
    oid: 1.3.6.1.2.1.31.1.1.1.6
    type: counter
    help: The total number of octets received on the interface, including framing
      characters - 1.3.6.1.2.1.31.1.1.1.6
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifHCInUcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.7
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were not addressed to a multicast or broadcast address at this sub-layer
      - 1.3.6.1.2.1.31.1.1.1.7
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifHCInMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.8
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a multicast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.8
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifHCInBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.9
    type: counter
    help: The number of packets, delivered by this sub-layer to a higher (sub-)layer,
      which were addressed to a broadcast address at this sub-layer - 1.3.6.1.2.1.31.1.1.1.9
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifHCOutOctets
    oid: 1.3.6.1.2.1.31.1.1.1.10
    type: counter
    help: The total number of octets transmitted out of the interface, including framing
      characters - 1.3.6.1.2.1.31.1.1.1.10
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifHCOutUcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.11
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were not addressed to a multicast or broadcast address at this sub-layer,
      including those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.11
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifHCOutMulticastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.12
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a multicast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.12
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifHCOutBroadcastPkts
    oid: 1.3.6.1.2.1.31.1.1.1.13
    type: counter
    help: The total number of packets that higher-level protocols requested be transmitted,
      and which were addressed to a broadcast address at this sub-layer, including
      those that were discarded or not sent - 1.3.6.1.2.1.31.1.1.1.13
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifLinkUpDownTrapEnable
    oid: 1.3.6.1.2.1.31.1.1.1.14
    type: gauge
    help: Indicates whether linkUp/linkDown traps should be generated for this interface
      - 1.3.6.1.2.1.31.1.1.1.14
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifHighSpeed
    oid: 1.3.6.1.2.1.31.1.1.1.15
    type: gauge
    help: An estimate of the interface's current bandwidth in units of 1,000,000 bits
      per second - 1.3.6.1.2.1.31.1.1.1.15
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifPromiscuousMode
    oid: 1.3.6.1.2.1.31.1.1.1.16
    type: gauge
    help: This object has a value of false(2) if this interface only accepts packets/frames
      that are addressed to this station - 1.3.6.1.2.1.31.1.1.1.16
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifConnectorPresent
    oid: 1.3.6.1.2.1.31.1.1.1.17
    type: gauge
    help: This object has the value 'true(1)' if the interface sublayer has a physical
      connector and the value 'false(2)' otherwise. - 1.3.6.1.2.1.31.1.1.1.17
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifAlias
    oid: 1.3.6.1.2.1.31.1.1.1.18
    type: DisplayString
    help: This object is an 'alias' name for the interface as specified by a network
      manager, and provides a non-volatile 'handle' for the interface - 1.3.6.1.2.1.31.1.1.1.18
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: ifCounterDiscontinuityTime
    oid: 1.3.6.1.2.1.31.1.1.1.19
    type: gauge
    help: The value of sysUpTime on the most recent occasion at which any one or more
      of this interface's counters suffered a discontinuity - 1.3.6.1.2.1.31.1.1.1.19
    indexes:
    - labelname: ifName
      type: gauge
    lookups:
    - labels:
      - ifName
      labelname: ifName
      oid: 1.3.6.1.2.1.31.1.1.1.1
      type: DisplayString
  - name: laNames
    oid: 1.3.6.1.4.1.2021.10.1.2
    type: DisplayString
    help: The list of loadave names we're watching. - 1.3.6.1.4.1.2021.10.1.2
    indexes:
    - labelname: laNames
      type: gauge
    lookups:
    - labels:
      - laNames
      labelname: laNames
      oid: 1.3.6.1.4.1.2021.10.1.2
      type: DisplayString
  - name: laLoadInt
    oid: 1.3.6.1.4.1.2021.10.1.5
    type: gauge
    help: The 1,5 and 15 minute load averages as an integer - 1.3.6.1.4.1.2021.10.1.5
    indexes:
    - labelname: laNames
      type: gauge
    lookups:
    - labels:
      - laNames
      labelname: laNames
      oid: 1.3.6.1.4.1.2021.10.1.2
      type: DisplayString
  - name: ssCpuSystem
    oid: 1.3.6.1.4.1.2021.11.10
    type: gauge
    help: The percentage of CPU time spent processing system-level code, calculated
      over the last minute - 1.3.6.1.4.1.2021.11.10
  - name: ssCpuIdle
    oid: 1.3.6.1.4.1.2021.11.11
    type: gauge
    help: The percentage of processor time spent idle, calculated over the last minute
      - 1.3.6.1.4.1.2021.11.11
  - name: ssCpuUser
    oid: 1.3.6.1.4.1.2021.11.9
    type: gauge
    help: The percentage of CPU time spent processing user-level code, calculated
      over the last minute - 1.3.6.1.4.1.2021.11.9
  - name: memIndex
    oid: 1.3.6.1.4.1.2021.4.1
    type: gauge
    help: Bogus Index - 1.3.6.1.4.1.2021.4.1
  - name: memErrorName
    oid: 1.3.6.1.4.1.2021.4.2
    type: DisplayString
    help: Bogus Name - 1.3.6.1.4.1.2021.4.2
  - name: memTotalSwap
    oid: 1.3.6.1.4.1.2021.4.3
    type: gauge
    help: The total amount of swap space configured for this host. - 1.3.6.1.4.1.2021.4.3
  - name: memAvailSwap
    oid: 1.3.6.1.4.1.2021.4.4
    type: gauge
    help: The amount of swap space currently unused or available. - 1.3.6.1.4.1.2021.4.4
  - name: memTotalReal
    oid: 1.3.6.1.4.1.2021.4.5
    type: gauge
    help: The total amount of real/physical memory installed on this host. - 1.3.6.1.4.1.2021.4.5
  - name: memAvailReal
    oid: 1.3.6.1.4.1.2021.4.6
    type: gauge
    help: The amount of real/physical memory currently unused or available. - 1.3.6.1.4.1.2021.4.6
  - name: memTotalSwapTXT
    oid: 1.3.6.1.4.1.2021.4.7
    type: gauge
    help: The total amount of swap space or virtual memory allocated for text pages
      on this host - 1.3.6.1.4.1.2021.4.7
  - name: memAvailSwapTXT
    oid: 1.3.6.1.4.1.2021.4.8
    type: gauge
    help: The amount of swap space or virtual memory currently being used by text
      pages on this host - 1.3.6.1.4.1.2021.4.8
  - name: memTotalRealTXT
    oid: 1.3.6.1.4.1.2021.4.9
    type: gauge
    help: The total amount of real/physical memory allocated for text pages on this
      host - 1.3.6.1.4.1.2021.4.9
  - name: memAvailRealTXT
    oid: 1.3.6.1.4.1.2021.4.10
    type: gauge
    help: The amount of real/physical memory currently being used by text pages on
      this host - 1.3.6.1.4.1.2021.4.10
  - name: memTotalFree
    oid: 1.3.6.1.4.1.2021.4.11
    type: gauge
    help: The total amount of memory free or available for use on this host - 1.3.6.1.4.1.2021.4.11
  - name: memMinimumSwap
    oid: 1.3.6.1.4.1.2021.4.12
    type: gauge
    help: The minimum amount of swap space expected to be kept free or available during
      normal operation of this host - 1.3.6.1.4.1.2021.4.12
  - name: memShared
    oid: 1.3.6.1.4.1.2021.4.13
    type: gauge
    help: The total amount of real or virtual memory currently allocated for use as
      shared memory - 1.3.6.1.4.1.2021.4.13
  - name: memBuffer
    oid: 1.3.6.1.4.1.2021.4.14
    type: gauge
    help: The total amount of real or virtual memory currently allocated for use as
      memory buffers - 1.3.6.1.4.1.2021.4.14
  - name: memCached
    oid: 1.3.6.1.4.1.2021.4.15
    type: gauge
    help: The total amount of real or virtual memory currently allocated for use as
      cached memory - 1.3.6.1.4.1.2021.4.15
  - name: memUsedSwapTXT
    oid: 1.3.6.1.4.1.2021.4.16
    type: gauge
    help: The amount of swap space or virtual memory currently being used by text
      pages on this host - 1.3.6.1.4.1.2021.4.16
  - name: memUsedRealTXT
    oid: 1.3.6.1.4.1.2021.4.17
    type: gauge
    help: The amount of real/physical memory currently being used by text pages on
      this host - 1.3.6.1.4.1.2021.4.17
  - name: memSwapError
    oid: 1.3.6.1.4.1.2021.4.100
    type: gauge
    help: Indicates whether the amount of available swap space (as reported by 'memAvailSwap(4)'),
      is less than the desired minimum (specified by 'memMinimumSwap(12)'). - 1.3.6.1.4.1.2021.4.100
  - name: memSwapErrorMsg
    oid: 1.3.6.1.4.1.2021.4.101
    type: DisplayString
    help: Describes whether the amount of available swap space (as reported by 'memAvailSwap(4)'),
      is less than the desired minimum (specified by 'memMinimumSwap(12)'). - 1.3.6.1.4.1.2021.4.101
  - name: systemStatus
    oid: 1.3.6.1.4.1.6574.1.1
    type: gauge
    help: Synology system status Each meanings of status represented describe below
      - 1.3.6.1.4.1.6574.1.1
  - name: temperature
    oid: 1.3.6.1.4.1.6574.1.2
    type: gauge
    help: Synology system temperature The temperature of Disk Station uses Celsius
      degree. - 1.3.6.1.4.1.6574.1.2
  - name: powerStatus
    oid: 1.3.6.1.4.1.6574.1.3
    type: gauge
    help: Synology power status Each meanings of status represented describe below
      - 1.3.6.1.4.1.6574.1.3
  - name: systemFanStatus
    oid: 1.3.6.1.4.1.6574.1.4.1
    type: gauge
    help: Synology system fan status Each meanings of status represented describe
      below - 1.3.6.1.4.1.6574.1.4.1
  - name: cpuFanStatus
    oid: 1.3.6.1.4.1.6574.1.4.2
    type: gauge
    help: Synology cpu fan status Each meanings of status represented describe below
      - 1.3.6.1.4.1.6574.1.4.2
  - name: modelName
    oid: 1.3.6.1.4.1.6574.1.5.1
    type: OctetString
    help: The Model name of this NAS - 1.3.6.1.4.1.6574.1.5.1
  - name: serialNumber
    oid: 1.3.6.1.4.1.6574.1.5.2
    type: OctetString
    help: The serial number of this NAS - 1.3.6.1.4.1.6574.1.5.2
  - name: version
    oid: 1.3.6.1.4.1.6574.1.5.3
    type: OctetString
    help: The version of this DSM - 1.3.6.1.4.1.6574.1.5.3
  - name: upgradeAvailable
    oid: 1.3.6.1.4.1.6574.1.5.4
    type: gauge
    help: This oid is for checking whether there is a latest DSM can be upgraded -
      1.3.6.1.4.1.6574.1.5.4
  - name: storageIOIndex
    oid: 1.3.6.1.4.1.6574.101.1.1.1
    type: gauge
    help: Reference index for each observed device. - 1.3.6.1.4.1.6574.101.1.1.1
    indexes:
    - labelname: storageIODevice
      type: gauge
    lookups:
    - labels:
      - storageIODevice
      labelname: storageIODevice
      oid: 1.3.6.1.4.1.6574.101.1.1.2
      type: DisplayString
  - name: storageIODevice
    oid: 1.3.6.1.4.1.6574.101.1.1.2
    type: DisplayString
    help: The name of the device we are counting/checking. - 1.3.6.1.4.1.6574.101.1.1.2
    indexes:
    - labelname: storageIODevice
      type: gauge
    lookups:
    - labels:
      - storageIODevice
      labelname: storageIODevice
      oid: 1.3.6.1.4.1.6574.101.1.1.2
      type: DisplayString
  - name: storageIONRead
    oid: 1.3.6.1.4.1.6574.101.1.1.3
    type: counter
    help: The number of bytes read from this device since boot. - 1.3.6.1.4.1.6574.101.1.1.3
    indexes:
    - labelname: storageIODevice
      type: gauge
    lookups:
    - labels:
      - storageIODevice
      labelname: storageIODevice
      oid: 1.3.6.1.4.1.6574.101.1.1.2
      type: DisplayString
  - name: storageIONWritten
    oid: 1.3.6.1.4.1.6574.101.1.1.4
    type: counter
    help: The number of bytes written to this device since boot. - 1.3.6.1.4.1.6574.101.1.1.4
    indexes:
    - labelname: storageIODevice
      type: gauge
    lookups:
    - labels:
      - storageIODevice
      labelname: storageIODevice
      oid: 1.3.6.1.4.1.6574.101.1.1.2
      type: DisplayString
  - name: storageIOReads
    oid: 1.3.6.1.4.1.6574.101.1.1.5
    type: counter
    help: The number of read accesses from this device since boot. - 1.3.6.1.4.1.6574.101.1.1.5
    indexes:
    - labelname: storageIODevice
      type: gauge
    lookups:
    - labels:
      - storageIODevice
      labelname: storageIODevice
      oid: 1.3.6.1.4.1.6574.101.1.1.2
      type: DisplayString
  - name: storageIOWrites
    oid: 1.3.6.1.4.1.6574.101.1.1.6
    type: counter
    help: The number of write accesses to this device since boot. - 1.3.6.1.4.1.6574.101.1.1.6
    indexes:
    - labelname: storageIODevice
      type: gauge
    lookups:
    - labels:
      - storageIODevice
      labelname: storageIODevice
      oid: 1.3.6.1.4.1.6574.101.1.1.2
      type: DisplayString
  - name: storageIOLA
    oid: 1.3.6.1.4.1.6574.101.1.1.8
    type: gauge
    help: The load of disk (%) - 1.3.6.1.4.1.6574.101.1.1.8
    indexes:
    - labelname: storageIODevice
      type: gauge
    lookups:
    - labels:
      - storageIODevice
      labelname: storageIODevice
      oid: 1.3.6.1.4.1.6574.101.1.1.2
      type: DisplayString
  - name: storageIOLA1
    oid: 1.3.6.1.4.1.6574.101.1.1.9
    type: gauge
    help: The 1 minute average load of disk (%) - 1.3.6.1.4.1.6574.101.1.1.9
    indexes:
    - labelname: storageIODevice
      type: gauge
    lookups:
    - labels:
      - storageIODevice
      labelname: storageIODevice
      oid: 1.3.6.1.4.1.6574.101.1.1.2
      type: DisplayString
  - name: storageIOLA5
    oid: 1.3.6.1.4.1.6574.101.1.1.10
    type: gauge
    help: The 5 minute average load of disk (%) - 1.3.6.1.4.1.6574.101.1.1.10
    indexes:
    - labelname: storageIODevice
      type: gauge
    lookups:
    - labels:
      - storageIODevice
      labelname: storageIODevice
      oid: 1.3.6.1.4.1.6574.101.1.1.2
      type: DisplayString
  - name: storageIOLA15
    oid: 1.3.6.1.4.1.6574.101.1.1.11
    type: gauge
    help: The 15 minute average load of disk (%) - 1.3.6.1.4.1.6574.101.1.1.11
    indexes:
    - labelname: storageIODevice
      type: gauge
    lookups:
    - labels:
      - storageIODevice
      labelname: storageIODevice
      oid: 1.3.6.1.4.1.6574.101.1.1.2
      type: DisplayString
  - name: storageIONReadX
    oid: 1.3.6.1.4.1.6574.101.1.1.12
    type: counter
    help: The number of bytes read from this device since boot. - 1.3.6.1.4.1.6574.101.1.1.12
    indexes:
    - labelname: storageIODevice
      type: gauge
    lookups:
    - labels:
      - storageIODevice
      labelname: storageIODevice
      oid: 1.3.6.1.4.1.6574.101.1.1.2
      type: DisplayString
  - name: storageIONWrittenX
    oid: 1.3.6.1.4.1.6574.101.1.1.13
    type: counter
    help: The number of bytes written to this device since boot. - 1.3.6.1.4.1.6574.101.1.1.13
    indexes:
    - labelname: storageIODevice
      type: gauge
    lookups:
    - labels:
      - storageIODevice
      labelname: storageIODevice
      oid: 1.3.6.1.4.1.6574.101.1.1.2
      type: DisplayString
  - name: spaceIOIndex
    oid: 1.3.6.1.4.1.6574.102.1.1.1
    type: gauge
    help: Reference index for each observed device. - 1.3.6.1.4.1.6574.102.1.1.1
    indexes:
    - labelname: spaceIODevice
      type: gauge
    lookups:
    - labels:
      - spaceIODevice
      labelname: spaceIODevice
      oid: 1.3.6.1.4.1.6574.102.1.1.2
      type: DisplayString
  - name: spaceIODevice
    oid: 1.3.6.1.4.1.6574.102.1.1.2
    type: DisplayString
    help: The name of the device we are counting/checking. - 1.3.6.1.4.1.6574.102.1.1.2
    indexes:
    - labelname: spaceIODevice
      type: gauge
    lookups:
    - labels:
      - spaceIODevice
      labelname: spaceIODevice
      oid: 1.3.6.1.4.1.6574.102.1.1.2
      type: DisplayString
  - name: spaceIONRead
    oid: 1.3.6.1.4.1.6574.102.1.1.3
    type: counter
    help: The number of bytes read from this device since boot. - 1.3.6.1.4.1.6574.102.1.1.3
    indexes:
    - labelname: spaceIODevice
      type: gauge
    lookups:
    - labels:
      - spaceIODevice
      labelname: spaceIODevice
      oid: 1.3.6.1.4.1.6574.102.1.1.2
      type: DisplayString
  - name: spaceIONWritten
    oid: 1.3.6.1.4.1.6574.102.1.1.4
    type: counter
    help: The number of bytes written to this device since boot. - 1.3.6.1.4.1.6574.102.1.1.4
    indexes:
    - labelname: spaceIODevice
      type: gauge
    lookups:
    - labels:
      - spaceIODevice
      labelname: spaceIODevice
      oid: 1.3.6.1.4.1.6574.102.1.1.2
      type: DisplayString
  - name: spaceIOReads
    oid: 1.3.6.1.4.1.6574.102.1.1.5
    type: counter
    help: The number of read accesses from this device since boot. - 1.3.6.1.4.1.6574.102.1.1.5
    indexes:
    - labelname: spaceIODevice
      type: gauge
    lookups:
    - labels:
      - spaceIODevice
      labelname: spaceIODevice
      oid: 1.3.6.1.4.1.6574.102.1.1.2
      type: DisplayString
  - name: spaceIOWrites
    oid: 1.3.6.1.4.1.6574.102.1.1.6
    type: counter
    help: The number of write accesses to this device since boot. - 1.3.6.1.4.1.6574.102.1.1.6
    indexes:
    - labelname: spaceIODevice
      type: gauge
    lookups:
    - labels:
      - spaceIODevice
      labelname: spaceIODevice
      oid: 1.3.6.1.4.1.6574.102.1.1.2
      type: DisplayString
  - name: spaceIOLA
    oid: 1.3.6.1.4.1.6574.102.1.1.8
    type: gauge
    help: The load of disk (%) - 1.3.6.1.4.1.6574.102.1.1.8
    indexes:
    - labelname: spaceIODevice
      type: gauge
    lookups:
    - labels:
      - spaceIODevice
      labelname: spaceIODevice
      oid: 1.3.6.1.4.1.6574.102.1.1.2
      type: DisplayString
  - name: spaceIOLA1
    oid: 1.3.6.1.4.1.6574.102.1.1.9
    type: gauge
    help: The 1 minute average load of disk (%) - 1.3.6.1.4.1.6574.102.1.1.9
    indexes:
    - labelname: spaceIODevice
      type: gauge
    lookups:
    - labels:
      - spaceIODevice
      labelname: spaceIODevice
      oid: 1.3.6.1.4.1.6574.102.1.1.2
      type: DisplayString
  - name: spaceIOLA5
    oid: 1.3.6.1.4.1.6574.102.1.1.10
    type: gauge
    help: The 5 minute average load of disk (%) - 1.3.6.1.4.1.6574.102.1.1.10
    indexes:
    - labelname: spaceIODevice
      type: gauge
    lookups:
    - labels:
      - spaceIODevice
      labelname: spaceIODevice
      oid: 1.3.6.1.4.1.6574.102.1.1.2
      type: DisplayString
  - name: spaceIOLA15
    oid: 1.3.6.1.4.1.6574.102.1.1.11
    type: gauge
    help: The 15 minute average load of disk (%) - 1.3.6.1.4.1.6574.102.1.1.11
    indexes:
    - labelname: spaceIODevice
      type: gauge
    lookups:
    - labels:
      - spaceIODevice
      labelname: spaceIODevice
      oid: 1.3.6.1.4.1.6574.102.1.1.2
      type: DisplayString
  - name: spaceIONReadX
    oid: 1.3.6.1.4.1.6574.102.1.1.12
    type: counter
    help: The number of bytes read from this device since boot. - 1.3.6.1.4.1.6574.102.1.1.12
    indexes:
    - labelname: spaceIODevice
      type: gauge
    lookups:
    - labels:
      - spaceIODevice
      labelname: spaceIODevice
      oid: 1.3.6.1.4.1.6574.102.1.1.2
      type: DisplayString
  - name: spaceIONWrittenX
    oid: 1.3.6.1.4.1.6574.102.1.1.13
    type: counter
    help: The number of bytes written to this device since boot. - 1.3.6.1.4.1.6574.102.1.1.13
    indexes:
    - labelname: spaceIODevice
      type: gauge
    lookups:
    - labels:
      - spaceIODevice
      labelname: spaceIODevice
      oid: 1.3.6.1.4.1.6574.102.1.1.2
      type: DisplayString
  - name: iSCSILUNInfoIndex
    oid: 1.3.6.1.4.1.6574.104.1.1.1
    type: gauge
    help: LUN info index - 1.3.6.1.4.1.6574.104.1.1.1
    indexes:
    - labelname: iSCSILUNInfoIndex
      type: gauge
  - name: iSCSILUNUUID
    oid: 1.3.6.1.4.1.6574.104.1.1.2
    type: OctetString
    help: LUN uuid - 1.3.6.1.4.1.6574.104.1.1.2
    indexes:
    - labelname: iSCSILUNInfoIndex
      type: gauge
  - name: iSCSILUNName
    oid: 1.3.6.1.4.1.6574.104.1.1.3
    type: OctetString
    help: LUN name - 1.3.6.1.4.1.6574.104.1.1.3
    indexes:
    - labelname: iSCSILUNInfoIndex
      type: gauge
  - name: iSCSILUNThroughputReadHigh
    oid: 1.3.6.1.4.1.6574.104.1.1.4
    type: gauge
    help: LUN read throughput over 32 bits part - 1.3.6.1.4.1.6574.104.1.1.4
    indexes:
    - labelname: iSCSILUNInfoIndex
      type: gauge
  - name: iSCSILUNThroughputReadLow
    oid: 1.3.6.1.4.1.6574.104.1.1.5
    type: gauge
    help: LUN read throughput in unsigned 32 bit - 1.3.6.1.4.1.6574.104.1.1.5
    indexes:
    - labelname: iSCSILUNInfoIndex
      type: gauge
  - name: iSCSILUNThroughputWriteHigh
    oid: 1.3.6.1.4.1.6574.104.1.1.6
    type: gauge
    help: LUN write throughput over 32 bits part - 1.3.6.1.4.1.6574.104.1.1.6
    indexes:
    - labelname: iSCSILUNInfoIndex
      type: gauge
  - name: iSCSILUNThroughputWriteLow
    oid: 1.3.6.1.4.1.6574.104.1.1.7
    type: gauge
    help: LUN write throughput in unsigned 32 bit - 1.3.6.1.4.1.6574.104.1.1.7
    indexes:
    - labelname: iSCSILUNInfoIndex
      type: gauge
  - name: iSCSILUNIopsRead
    oid: 1.3.6.1.4.1.6574.104.1.1.8
    type: gauge
    help: LUN read iops - 1.3.6.1.4.1.6574.104.1.1.8
    indexes:
    - labelname: iSCSILUNInfoIndex
      type: gauge
  - name: iSCSILUNIopsWrite
    oid: 1.3.6.1.4.1.6574.104.1.1.9
    type: gauge
    help: LUN write iops - 1.3.6.1.4.1.6574.104.1.1.9
    indexes:
    - labelname: iSCSILUNInfoIndex
      type: gauge
  - name: iSCSILUNDiskLatencyRead
    oid: 1.3.6.1.4.1.6574.104.1.1.10
    type: gauge
    help: LUN disk latency when reading - 1.3.6.1.4.1.6574.104.1.1.10
    indexes:
    - labelname: iSCSILUNInfoIndex
      type: gauge
  - name: iSCSILUNDiskLatencyWrite
    oid: 1.3.6.1.4.1.6574.104.1.1.11
    type: gauge
    help: LUN disk latency when writing - 1.3.6.1.4.1.6574.104.1.1.11
    indexes:
    - labelname: iSCSILUNInfoIndex
      type: gauge
  - name: iSCSILUNNetworkLatencyTx
    oid: 1.3.6.1.4.1.6574.104.1.1.12
    type: gauge
    help: LUN transfer data network latency - 1.3.6.1.4.1.6574.104.1.1.12
    indexes:
    - labelname: iSCSILUNInfoIndex
      type: gauge
  - name: iSCSILUNNetworkLatencyRx
    oid: 1.3.6.1.4.1.6574.104.1.1.13
    type: gauge
    help: LUN receive data network latency - 1.3.6.1.4.1.6574.104.1.1.13
    indexes:
    - labelname: iSCSILUNInfoIndex
      type: gauge
  - name: iSCSILUNIoSizeRead
    oid: 1.3.6.1.4.1.6574.104.1.1.14
    type: gauge
    help: LUN average io size when reading - 1.3.6.1.4.1.6574.104.1.1.14
    indexes:
    - labelname: iSCSILUNInfoIndex
      type: gauge
  - name: iSCSILUNIoSizeWrite
    oid: 1.3.6.1.4.1.6574.104.1.1.15
    type: gauge
    help: LUN average io size when writing - 1.3.6.1.4.1.6574.104.1.1.15
    indexes:
    - labelname: iSCSILUNInfoIndex
      type: gauge
  - name: iSCSILUNQueueDepth
    oid: 1.3.6.1.4.1.6574.104.1.1.16
    type: gauge
    help: Num of iSCSI commands in LUN queue - 1.3.6.1.4.1.6574.104.1.1.16
    indexes:
    - labelname: iSCSILUNInfoIndex
      type: gauge
  - name: iSCSILUNType
    oid: 1.3.6.1.4.1.6574.104.1.1.17
    type: OctetString
    help: LUN type - 1.3.6.1.4.1.6574.104.1.1.17
    indexes:
    - labelname: iSCSILUNInfoIndex
      type: gauge
  - name: diskIndex
    oid: 1.3.6.1.4.1.6574.2.1.1.1
    type: gauge
    help: The index of disk table - 1.3.6.1.4.1.6574.2.1.1.1
    indexes:
    - labelname: diskID
      type: gauge
    lookups:
    - labels:
      - diskID
      labelname: diskID
      oid: 1.3.6.1.4.1.6574.2.1.1.2
      type: OctetString
  - name: diskID
    oid: 1.3.6.1.4.1.6574.2.1.1.2
    type: OctetString
    help: Synology disk ID The ID of disk is assigned by disk Station. - 1.3.6.1.4.1.6574.2.1.1.2
    indexes:
    - labelname: diskID
      type: gauge
    lookups:
    - labels:
      - diskID
      labelname: diskID
      oid: 1.3.6.1.4.1.6574.2.1.1.2
      type: OctetString
  - name: diskModel
    oid: 1.3.6.1.4.1.6574.2.1.1.3
    type: OctetString
    help: Synology disk model name The disk model name will be showed here. - 1.3.6.1.4.1.6574.2.1.1.3
    indexes:
    - labelname: diskID
      type: gauge
    lookups:
    - labels:
      - diskID
      labelname: diskID
      oid: 1.3.6.1.4.1.6574.2.1.1.2
      type: OctetString
  - name: diskType
    oid: 1.3.6.1.4.1.6574.2.1.1.4
    type: OctetString
    help: Synology disk type The type of disk will be showed here, including SATA,
      SSD and so on. - 1.3.6.1.4.1.6574.2.1.1.4
    indexes:
    - labelname: diskID
      type: gauge
    lookups:
    - labels:
      - diskID
      labelname: diskID
      oid: 1.3.6.1.4.1.6574.2.1.1.2
      type: OctetString
  - name: diskStatus
    oid: 1.3.6.1.4.1.6574.2.1.1.5
    type: gauge
    help: Synology disk status Each meanings of status represented describe below
      - 1.3.6.1.4.1.6574.2.1.1.5
    indexes:
    - labelname: diskID
      type: gauge
    lookups:
    - labels:
      - diskID
      labelname: diskID
      oid: 1.3.6.1.4.1.6574.2.1.1.2
      type: OctetString
  - name: diskTemperature
    oid: 1.3.6.1.4.1.6574.2.1.1.6
    type: gauge
    help: Synology disk temperature The temperature of each disk uses Celsius degree.
      - 1.3.6.1.4.1.6574.2.1.1.6
    indexes:
    - labelname: diskID
      type: gauge
    lookups:
    - labels:
      - diskID
      labelname: diskID
      oid: 1.3.6.1.4.1.6574.2.1.1.2
      type: OctetString
  - name: raidIndex
    oid: 1.3.6.1.4.1.6574.3.1.1.1
    type: gauge
    help: The index of raid table - 1.3.6.1.4.1.6574.3.1.1.1
    indexes:
    - labelname: raidName
      type: gauge
    lookups:
    - labels:
      - raidName
      labelname: raidName
      oid: 1.3.6.1.4.1.6574.3.1.1.2
      type: OctetString
  - name: raidName
    oid: 1.3.6.1.4.1.6574.3.1.1.2
    type: OctetString
    help: Synology raid name The name of each raid will be showed here. - 1.3.6.1.4.1.6574.3.1.1.2
    indexes:
    - labelname: raidName
      type: gauge
    lookups:
    - labels:
      - raidName
      labelname: raidName
      oid: 1.3.6.1.4.1.6574.3.1.1.2
      type: OctetString
  - name: raidStatus
    oid: 1.3.6.1.4.1.6574.3.1.1.3
    type: gauge
    help: Synology Raid status Each meanings of status represented describe below
      - 1.3.6.1.4.1.6574.3.1.1.3
    indexes:
    - labelname: raidName
      type: gauge
    lookups:
    - labels:
      - raidName
      labelname: raidName
      oid: 1.3.6.1.4.1.6574.3.1.1.2
      type: OctetString
  - name: upsDeviceModel
    oid: 1.3.6.1.4.1.6574.4.1.1
    type: DisplayString
    help: Device model - 1.3.6.1.4.1.6574.4.1.1
  - name: upsDeviceManufacturer
    oid: 1.3.6.1.4.1.6574.4.1.2
    type: DisplayString
    help: Device manufacturer - 1.3.6.1.4.1.6574.4.1.2
  - name: upsDeviceSerial
    oid: 1.3.6.1.4.1.6574.4.1.3
    type: DisplayString
    help: Device serial number. - 1.3.6.1.4.1.6574.4.1.3
  - name: upsDeviceType
    oid: 1.3.6.1.4.1.6574.4.1.4
    type: DisplayString
    help: Device type (ups, pdu, scd, psu) - 1.3.6.1.4.1.6574.4.1.4
  - name: upsDeviceDescription
    oid: 1.3.6.1.4.1.6574.4.1.5
    type: DisplayString
    help: Device description. - 1.3.6.1.4.1.6574.4.1.5
  - name: upsDeviceContact
    oid: 1.3.6.1.4.1.6574.4.1.6
    type: DisplayString
    help: Device administrator name. - 1.3.6.1.4.1.6574.4.1.6
  - name: upsDeviceLocation
    oid: 1.3.6.1.4.1.6574.4.1.7
    type: DisplayString
    help: Device physical location. - 1.3.6.1.4.1.6574.4.1.7
  - name: upsDevicePart
    oid: 1.3.6.1.4.1.6574.4.1.8
    type: DisplayString
    help: Device part number. - 1.3.6.1.4.1.6574.4.1.8
  - name: upsDeviceMACAddr
    oid: 1.3.6.1.4.1.6574.4.1.9
    type: DisplayString
    help: Physical network address of the device. - 1.3.6.1.4.1.6574.4.1.9
  - name: upsInfoStatus
    oid: 1.3.6.1.4.1.6574.4.2.1
    type: DisplayString
    help: UPS status. - 1.3.6.1.4.1.6574.4.2.1
  - name: upsInfoAlarm
    oid: 1.3.6.1.4.1.6574.4.2.2
    type: DisplayString
    help: UPS alarms - 1.3.6.1.4.1.6574.4.2.2
  - name: upsInfoTime
    oid: 1.3.6.1.4.1.6574.4.2.3
    type: DisplayString
    help: Internal UPS clock time - 1.3.6.1.4.1.6574.4.2.3
  - name: upsInfoDate
    oid: 1.3.6.1.4.1.6574.4.2.4
    type: DisplayString
    help: Internal UPS clock date - 1.3.6.1.4.1.6574.4.2.4
  - name: upsInfoModel
    oid: 1.3.6.1.4.1.6574.4.2.5
    type: DisplayString
    help: UPS model - 1.3.6.1.4.1.6574.4.2.5
  - name: upsInfoMfrName
    oid: 1.3.6.1.4.1.6574.4.2.6.1
    type: DisplayString
    help: UPS manufacturer - 1.3.6.1.4.1.6574.4.2.6.1
  - name: upsInfoMfrDate
    oid: 1.3.6.1.4.1.6574.4.2.6.2
    type: DisplayString
    help: UPS manufacturing date - 1.3.6.1.4.1.6574.4.2.6.2
  - name: upsInfoSerial
    oid: 1.3.6.1.4.1.6574.4.2.7
    type: DisplayString
    help: UPS serial number - 1.3.6.1.4.1.6574.4.2.7
  - name: upsInfoVendorID
    oid: 1.3.6.1.4.1.6574.4.2.8
    type: DisplayString
    help: Vendor ID for USB devices - 1.3.6.1.4.1.6574.4.2.8
  - name: upsInfoProductID
    oid: 1.3.6.1.4.1.6574.4.2.9
    type: DisplayString
    help: Product ID for USB devices - 1.3.6.1.4.1.6574.4.2.9
  - name: upsInfoFirmwareName
    oid: 1.3.6.1.4.1.6574.4.2.10.1
    type: DisplayString
    help: UPS firmware - 1.3.6.1.4.1.6574.4.2.10.1
  - name: upsInfoFirmwareAux
    oid: 1.3.6.1.4.1.6574.4.2.10.2
    type: DisplayString
    help: Auxiliary device firmware - 1.3.6.1.4.1.6574.4.2.10.2
  - name: upsInfoID
    oid: 1.3.6.1.4.1.6574.4.2.13
    type: DisplayString
    help: UPS system identifier - 1.3.6.1.4.1.6574.4.2.13
  - name: upsInfoDelayStart
    oid: 1.3.6.1.4.1.6574.4.2.14.1
    type: gauge
    help: Interval to wait before restarting the load (seconds) - 1.3.6.1.4.1.6574.4.2.14.1
  - name: upsInfoDelayReboot
    oid: 1.3.6.1.4.1.6574.4.2.14.2
    type: gauge
    help: Interval to wait before rebooting the UPS (seconds) - 1.3.6.1.4.1.6574.4.2.14.2
  - name: upsInfoDelayShutdown
    oid: 1.3.6.1.4.1.6574.4.2.14.3
    type: gauge
    help: Interval to wait after shutdown with delay command (seconds) - 1.3.6.1.4.1.6574.4.2.14.3
  - name: upsInfoTimerStart
    oid: 1.3.6.1.4.1.6574.4.2.15.1
    type: gauge
    help: Time before the load will be started (seconds) - 1.3.6.1.4.1.6574.4.2.15.1
  - name: upsInfoTimerReboot
    oid: 1.3.6.1.4.1.6574.4.2.15.2
    type: gauge
    help: Time before the load will be rebooted (seconds) - 1.3.6.1.4.1.6574.4.2.15.2
  - name: upsInfoTimerShutdown
    oid: 1.3.6.1.4.1.6574.4.2.15.3
    type: gauge
    help: Time before the load will be shutdown (seconds) - 1.3.6.1.4.1.6574.4.2.15.3
  - name: upsInfoTestInterval
    oid: 1.3.6.1.4.1.6574.4.2.16.1
    type: gauge
    help: Interval between self tests - 1.3.6.1.4.1.6574.4.2.16.1
  - name: upsInfoTestResult
    oid: 1.3.6.1.4.1.6574.4.2.16.2
    type: DisplayString
    help: Results of last self test - 1.3.6.1.4.1.6574.4.2.16.2
  - name: upsInfoDisplayLanguage
    oid: 1.3.6.1.4.1.6574.4.2.17
    type: DisplayString
    help: Language to use on front panel - 1.3.6.1.4.1.6574.4.2.17
  - name: upsInfoContacts
    oid: 1.3.6.1.4.1.6574.4.2.18
    type: DisplayString
    help: UPS external contact sensors - 1.3.6.1.4.1.6574.4.2.18
  - name: upsInfoEffciency
    oid: 1.3.6.1.4.1.6574.4.2.19
    type: gauge
    help: Efficiency of the UPS (ratio of the output current on the input current)
      (percent) - 1.3.6.1.4.1.6574.4.2.19
  - name: upsInfoBeeperStatus
    oid: 1.3.6.1.4.1.6574.4.2.22
    type: DisplayString
    help: UPS beeper status (enabled, disabled or muted) - 1.3.6.1.4.1.6574.4.2.22
  - name: upsInfoType
    oid: 1.3.6.1.4.1.6574.4.2.23
    type: DisplayString
    help: UPS type - 1.3.6.1.4.1.6574.4.2.23
  - name: upsInfoWatchdogStatus
    oid: 1.3.6.1.4.1.6574.4.2.24
    type: DisplayString
    help: UPS watchdog status (enabled or disabled) - 1.3.6.1.4.1.6574.4.2.24
  - name: upsInfoStartAuto
    oid: 1.3.6.1.4.1.6574.4.2.25.1
    type: DisplayString
    help: UPS starts when mains is (re)applied - 1.3.6.1.4.1.6574.4.2.25.1
  - name: upsInfoStartBattery
    oid: 1.3.6.1.4.1.6574.4.2.25.2
    type: DisplayString
    help: Allow to start UPS from battery - 1.3.6.1.4.1.6574.4.2.25.2
  - name: upsInfoStartReboot
    oid: 1.3.6.1.4.1.6574.4.2.25.3
    type: DisplayString
    help: UPS coldstarts from battery (enabled or disabled) - 1.3.6.1.4.1.6574.4.2.25.3
  - name: upsBatteryRuntimeValue
    oid: 1.3.6.1.4.1.6574.4.3.6.1
    type: gauge
    help: Battery runtime (seconds) - 1.3.6.1.4.1.6574.4.3.6.1
  - name: upsBatteryRuntimeLow
    oid: 1.3.6.1.4.1.6574.4.3.6.2
    type: gauge
    help: Remaining battery runtime when UPS switches to LB (seconds) - 1.3.6.1.4.1.6574.4.3.6.2
  - name: upsBatteryRuntimeRestart
    oid: 1.3.6.1.4.1.6574.4.3.6.3
    type: gauge
    help: Minimum battery runtime for UPS restart after power-off (seconds) - 1.3.6.1.4.1.6574.4.3.6.3
  - name: upsBatteryAlarmThreshold
    oid: 1.3.6.1.4.1.6574.4.3.7
    type: DisplayString
    help: Battery alarm threshold - 1.3.6.1.4.1.6574.4.3.7
  - name: upsBatteryDate
    oid: 1.3.6.1.4.1.6574.4.3.8
    type: DisplayString
    help: Battery change date - 1.3.6.1.4.1.6574.4.3.8
  - name: upsBatteryMfrDate
    oid: 1.3.6.1.4.1.6574.4.3.9
    type: DisplayString
    help: Battery manufacturing date - 1.3.6.1.4.1.6574.4.3.9
  - name: upsBatteryPacks
    oid: 1.3.6.1.4.1.6574.4.3.10
    type: gauge
    help: Number of battery packs - 1.3.6.1.4.1.6574.4.3.10
  - name: upsBatteryPacksBad
    oid: 1.3.6.1.4.1.6574.4.3.11
    type: gauge
    help: Number of bad battery packs - 1.3.6.1.4.1.6574.4.3.11
  - name: upsBatteryType
    oid: 1.3.6.1.4.1.6574.4.3.12
    type: DisplayString
    help: Battery chemistry - 1.3.6.1.4.1.6574.4.3.12
  - name: upsBatteryProtection
    oid: 1.3.6.1.4.1.6574.4.3.13
    type: DisplayString
    help: Prevent deep discharge of battery - 1.3.6.1.4.1.6574.4.3.13
  - name: upsBatteryEnergySave
    oid: 1.3.6.1.4.1.6574.4.3.14
    type: DisplayString
    help: Switch off when running on battery and no/low load - 1.3.6.1.4.1.6574.4.3.14
  - name: upsInputVoltageExtend
    oid: 1.3.6.1.4.1.6574.4.4.1.5
    type: DisplayString
    help: Extended input voltage range - 1.3.6.1.4.1.6574.4.4.1.5
  - name: upsInputTransferReason
    oid: 1.3.6.1.4.1.6574.4.4.2.1
    type: DisplayString
    help: Reason for last transfer to battery - 1.3.6.1.4.1.6574.4.4.2.1
  - name: upsInputSensitivity
    oid: 1.3.6.1.4.1.6574.4.4.3
    type: DisplayString
    help: Input power sensitivity - 1.3.6.1.4.1.6574.4.4.3
  - name: upsInputQuality
    oid: 1.3.6.1.4.1.6574.4.4.4
    type: DisplayString
    help: Input power quality - 1.3.6.1.4.1.6574.4.4.4
  - name: upsInputFrequencyExtend
    oid: 1.3.6.1.4.1.6574.4.4.6.5
    type: DisplayString
    help: Extended input frequency range - 1.3.6.1.4.1.6574.4.4.6.5
  - name: upsAmbientTemperatureAlarm
    oid: 1.3.6.1.4.1.6574.4.6.1.2
    type: DisplayString
    help: Temperature alarm (enabled/disabled) - 1.3.6.1.4.1.6574.4.6.1.2
  - name: upsAmbientHumidityAlarm
    oid: 1.3.6.1.4.1.6574.4.6.2.2
    type: DisplayString
    help: Relative humidity alarm (enabled/disabled) - 1.3.6.1.4.1.6574.4.6.2.2
  - name: upsDriverName
    oid: 1.3.6.1.4.1.6574.4.7.1
    type: DisplayString
    help: Driver name - 1.3.6.1.4.1.6574.4.7.1
  - name: upsDriverVersion
    oid: 1.3.6.1.4.1.6574.4.7.2
    type: DisplayString
    help: Driver version (NUT release) - 1.3.6.1.4.1.6574.4.7.2
  - name: upsDriverVersionData
    oid: 1.3.6.1.4.1.6574.4.7.3
    type: DisplayString
    help: Driver version data - 1.3.6.1.4.1.6574.4.7.3
  - name: upsDriverVersionInternal
    oid: 1.3.6.1.4.1.6574.4.7.4
    type: DisplayString
    help: Internal driver version (if tracked separately) - 1.3.6.1.4.1.6574.4.7.4
  - name: upsDriverPollInterval
    oid: 1.3.6.1.4.1.6574.4.7.5
    type: gauge
    help: Poll interval setup in configuration file - 1.3.6.1.4.1.6574.4.7.5
  - name: upsDriverPort
    oid: 1.3.6.1.4.1.6574.4.7.6
    type: DisplayString
    help: Port setup in configuration file - 1.3.6.1.4.1.6574.4.7.6
  - name: upsDriverPollFrequency
    oid: 1.3.6.1.4.1.6574.4.7.7
    type: gauge
    help: Poll frequency - 1.3.6.1.4.1.6574.4.7.7
  - name: upsDriverProductID
    oid: 1.3.6.1.4.1.6574.4.7.8
    type: DisplayString
    help: Product ID - 1.3.6.1.4.1.6574.4.7.8
  - name: upsDriverSnmpVersion
    oid: 1.3.6.1.4.1.6574.4.7.9
    type: DisplayString
    help: Snmp version - 1.3.6.1.4.1.6574.4.7.9
  - name: upsServerInfo
    oid: 1.3.6.1.4.1.6574.4.8.1
    type: DisplayString
    help: Server information - 1.3.6.1.4.1.6574.4.8.1
  - name: upsServerVersion
    oid: 1.3.6.1.4.1.6574.4.8.2
    type: DisplayString
    help: Server version - 1.3.6.1.4.1.6574.4.8.2
  - name: diskSMARTInfoIndex
    oid: 1.3.6.1.4.1.6574.5.1.1.1
    type: gauge
    help: SMART info index - 1.3.6.1.4.1.6574.5.1.1.1
    indexes:
    - labelname: diskSMARTInfoIndex
      type: gauge
  - name: diskSMARTInfoDevName
    oid: 1.3.6.1.4.1.6574.5.1.1.2
    type: OctetString
    help: SMART info device name - 1.3.6.1.4.1.6574.5.1.1.2
    indexes:
    - labelname: diskSMARTInfoIndex
      type: gauge
  - name: diskSMARTAttrName
    oid: 1.3.6.1.4.1.6574.5.1.1.3
    type: OctetString
    help: SMART attribute name - 1.3.6.1.4.1.6574.5.1.1.3
    indexes:
    - labelname: diskSMARTInfoIndex
      type: gauge
  - name: diskSMARTAttrId
    oid: 1.3.6.1.4.1.6574.5.1.1.4
    type: gauge
    help: SMART attribute ID - 1.3.6.1.4.1.6574.5.1.1.4
    indexes:
    - labelname: diskSMARTInfoIndex
      type: gauge
  - name: diskSMARTAttrCurrent
    oid: 1.3.6.1.4.1.6574.5.1.1.5
    type: gauge
    help: SMART attribute current value - 1.3.6.1.4.1.6574.5.1.1.5
    indexes:
    - labelname: diskSMARTInfoIndex
      type: gauge
  - name: diskSMARTAttrWorst
    oid: 1.3.6.1.4.1.6574.5.1.1.6
    type: gauge
    help: SMART attribute worst value - 1.3.6.1.4.1.6574.5.1.1.6
    indexes:
    - labelname: diskSMARTInfoIndex
      type: gauge
  - name: diskSMARTAttrThreshold
    oid: 1.3.6.1.4.1.6574.5.1.1.7
    type: gauge
    help: SMART attribute threshold value - 1.3.6.1.4.1.6574.5.1.1.7
    indexes:
    - labelname: diskSMARTInfoIndex
      type: gauge
  - name: diskSMARTAttrRaw
    oid: 1.3.6.1.4.1.6574.5.1.1.8
    type: gauge
    help: SMART attribute raw value - 1.3.6.1.4.1.6574.5.1.1.8
    indexes:
    - labelname: diskSMARTInfoIndex
      type: gauge
  - name: diskSMARTAttrStatus
    oid: 1.3.6.1.4.1.6574.5.1.1.9
    type: OctetString
    help: SMART attribute status - 1.3.6.1.4.1.6574.5.1.1.9
    indexes:
    - labelname: diskSMARTInfoIndex
      type: gauge
  - name: serviceInfoIndex
    oid: 1.3.6.1.4.1.6574.6.1.1.1
    type: gauge
    help: Service info index - 1.3.6.1.4.1.6574.6.1.1.1
    indexes:
    - labelname: serviceName
      type: gauge
    lookups:
    - labels:
      - serviceName
      labelname: serviceName
      oid: 1.3.6.1.4.1.6574.6.1.1.2
      type: OctetString
  - name: serviceName
    oid: 1.3.6.1.4.1.6574.6.1.1.2
    type: OctetString
    help: Service name - 1.3.6.1.4.1.6574.6.1.1.2
    indexes:
    - labelname: serviceName
      type: gauge
    lookups:
    - labels:
      - serviceName
      labelname: serviceName
      oid: 1.3.6.1.4.1.6574.6.1.1.2
      type: OctetString
  - name: serviceUsers
    oid: 1.3.6.1.4.1.6574.6.1.1.3
    type: gauge
    help: Number of users using this service - 1.3.6.1.4.1.6574.6.1.1.3
    indexes:
    - labelname: serviceName
      type: gauge
    lookups:
    - labels:
      - serviceName
      labelname: serviceName
      oid: 1.3.6.1.4.1.6574.6.1.1.2
      type: OctetString
ciscosw:
  version: 2
  auth:
    community: audit
  walk:
  - 1.3.6.1.2.1.2.2.1
  - 1.3.6.1.2.1.2.2.2
  - 1.3.6.1.2.1.2
  - 1.3.6.1.2.1.31.1
  - 1.3.6.1.2.1.31.1.1.1
  - 1.3.6.1.4.1.9.9.109.1.1.1.1
  metrics:
  - name: ifIndex
    oid: 1.3.6.1.2.1.2.2.1.1
    type: gauge
    help: A unique value, greater than zero, for each interface - 1.3.6.1.2.1.2.2.1.1
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifDescr
    oid: 1.3.6.1.2.1.2.2.1.2
    type: DisplayString
    help: A textual string containing information about the interface - 1.3.6.1.2.1.2.2.1.2
    indexes:
    - labelname: ifIndex
      type: gauge

  - name: sysUpTime
    oid: 1.3.6.1.2.1.1.3
    type: gauge

  - name: BWInOctets
    oid: 1.3.6.1.2.1.31.1.1.1.6
    type: gauge
    indexes:
    - labelname: interface
      type: gauge

    lookups:
       - labels: [interface]
         oid: 1.3.6.1.2.1.2.2.1.2
         labelname: interface
         type: DisplayString
       - labels: [interface]
         oid: 1.3.6.1.2.1.31.1.1.1.18
         labelname: alias
         type: DisplayString

  - name: BWOutOctets
    oid: 1.3.6.1.2.1.31.1.1.1.10
    type: gauge
    indexes:
    - labelname: interface
      type: gauge

    lookups:
       - labels: [interface]
         oid: 1.3.6.1.2.1.2.2.1.2
         labelname: interface
         type: DisplayString
       - labels: [interface]
         oid: 1.3.6.1.2.1.31.1.1.1.18
         labelname: alias
         type: DisplayString
  - name: ifAlias
    oid: 1.3.6.1.2.1.31.1.1.1.18
    type: DisplayString
    help: This object is an 'alias' name for the interface as specified by a network
      manager, and provides a non-volatile 'handle' for the interface - 1.3.6.1.2.1.31.1.1.1.18
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: cpmCPUTotal
    oid: 1.3.6.1.4.1.9.9.109.1.1.1.1.4.1
    type: gauge
    help: The overall CPU busy percentage
junipersw:
  version: 2
  auth:
    community: audit
  max_repetitions: 1  # How many objects to request with GET/GETBULK, defaults to 25.
                         # May need to be reduced for buggy devices.
  retries: 5   # How many times to retry a failed request, defaults to 3.
  timeout: 240s # Timeout for each walk, defaults to 10s.
  walk:
  - 1.3.6.1.2.1.2.2.1.1
  - 1.3.6.1.2.1.2.2.1.2
  - 1.3.6.1.2.1.2.2.1.10
  - 1.3.6.1.2.1.2.2.1.16
  - 1.3.6.1.2.1.31.1.1.1.18
  - 1.3.6.1.4.1.2636.3.1.13.1.8
  metrics:
  - name: ifIndex
    oid: 1.3.6.1.2.1.2.2.1.1
    type: gauge
    help: A unique value, greater than zero, for each interface - 1.3.6.1.2.1.2.2.1.1
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: ifDescr
    oid: 1.3.6.1.2.1.2.2.1.2
    type: DisplayString
    help: A textual string containing information about the interface - 1.3.6.1.2.1.2.2.1.2
    indexes:
    - labelname: ifIndex
      type: gauge

  - name: JBWInOctets
    oid: 1.3.6.1.2.1.2.2.1.10
    type: gauge
    indexes:
    - labelname: interface
      type: gauge

    lookups:
       - labels: [interface]
         oid: 1.3.6.1.2.1.2.2.1.2
         labelname: interface
         type: DisplayString
       - labels: [interface]
         oid: 1.3.6.1.2.1.31.1.1.1.18
         labelname: alias
         type: DisplayString
  - name: JBWOutOctets
    oid: 1.3.6.1.2.1.2.2.1.16
    type: gauge
    indexes:
    - labelname: interface
      type: gauge

    lookups:
       - labels: [interface]
         oid: 1.3.6.1.2.1.2.2.1.2
         labelname: interface
         type: DisplayString
       - labels: [interface]
         oid: 1.3.6.1.2.1.31.1.1.1.18
         labelname: alias
         type: DisplayString
  - name: ifAlias
    oid: 1.3.6.1.2.1.31.1.1.1.18
    type: DisplayString
    help: This object is an 'alias' name for the interface as specified by a network
      manager, and provides a non-volatile 'handle' for the interface - 1.3.6.1.2.1.31.1.1.1.18
    indexes:
    - labelname: ifIndex
      type: gauge
  - name: cpmCPUTotal
    oid: 1.3.6.1.4.1.2636.3.1.13.1.8.9.1.0.0
    type: gauge
    help: The overall CPU busy percentage
```
