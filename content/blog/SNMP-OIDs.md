---
title: Notes on SNMP MIBs OIDs and Grey Whiskers
banner: /img/banners/banner-29.jpg
date: 2015-09-08
layout: post
tags:
 - monitoring
 - networking
 - zabbix
---

With as many grey whiskers as I have, you would think I could grok SNMP in my sleep by now. Unfortunately, everytime I have to deal with it I get frustrated, wonder where the hell my notes from last time are, and start cursing. From now on, here are my notes!

I'm typically using [Zabbix](http://www.zabbix.com/) to poll SNMP OIDs and place the MIBs on the Zabbix server or the Zabbix proxy responsible for SNMP in `/usr/share/snmp/mibs`.

The MIBs will end in `.txt` but drop that when you reference them. An example might be `/usr/share/snmp/mibs/CMC-TC-MIB.txt`.

# Explore a MIB with `snmptranslate` #

The [snmptranslate](http://www.net-snmp.org/wiki/index.php/TUT:snmptranslate) command can be used to translate the OIDs in a MIB to various representations.

```text
-T TRANSOPTS
       Provides control over the translation of the OID values.  The following TRANSOPTS are available:

       -Td   Print full details of the specified OID.
       -Tp   Print a graphical tree, rooted at the specified OID.
       -Ta   Dump the loaded MIB in a trivial form.
       -Tl   Dump a labeled form of all objects.
       -To   Dump a numeric form of all objects.
       -Ts   Dump a symbolic form of all objects.
       -Tt   Dump a tree form of the loaded MIBs (mostly useful for debugging).
       -Tz   Dump a numeric and labeled form of all objects (compatible with MIB2SCHEMA format).
```

There are other [output options](http://www.net-snmp.org/wiki/index.php/TUT:Customized_Output_Formats) to be found in the `snmpcmd` man page. Use these with `snmpwalk` or `snmpget`.

```text
  -O OUTOPTS            Toggle various defaults controlling output display:
      0:  print leading 0 for single-digit hex characters
      a:  print all strings in ascii format
      b:  do not break OID indexes down
      e:  print enums numerically
      E:  escape quotes in string indices
      f:  print full OIDs on output
      n:  print OIDs numerically
      q:  quick print for easier parsing
      Q:  quick print with equal-signs
      s:  print only last symbolic element of OID
      S:  print MIB module-id plus last element
      t:  print timeticks unparsed as numeric integers
      T:  print human-readable text along with hex strings
      u:  print OIDs using UCD-style prefix suppression
      U:  don't print units
      v:  print values only (not OID = value)
      x:  print all strings in hex format
      X:  extended index format
```

## Examples ##

- `-Ts` **Get a list of all OID names in a MIB**

```bash
snmptranslate -Ts -M /usr/share/snmp/mibs -m RITTAL-CMC-TC-MIB > /tmp/rittal-cmc-tc-oid-list.txt
snmptranslate -Ts -M /usr/share/snmp/mibs -m RITTAL-CMC-TC-MIB | head
.iso.org
.iso.org.dod
.iso.org.dod.internet
.iso.org.dod.internet.directory
.iso.org.dod.internet.mgmt
.iso.org.dod.internet.mgmt.mib-2
.iso.org.dod.internet.mgmt.mib-2.system
.iso.org.dod.internet.mgmt.mib-2.system.sysDescr
.iso.org.dod.internet.mgmt.mib-2.system.sysObjectID
.iso.org.dod.internet.mgmt.mib-2.system.sysUpTime
```

- `-Tl` **Get a labeled list of OID names and numbers**
  This is good for comparisons and figuring out numeric OIDs.

```bash
snmptranslate -Tl -M /usr/share/snmp/mibs -m RITTAL-CMC-TC-MIB > /tmp/rittal-cmc-tc-oid-labeled.txt
snmptranslate -Tl -M /usr/share/snmp/mibs -m RITTAL-CMC-TC-MIB | head
.iso(1).org(3)
.iso(1).org(3).dod(6)
.iso(1).org(3).dod(6).internet(1)
.iso(1).org(3).dod(6).internet(1).directory(1)
.iso(1).org(3).dod(6).internet(1).mgmt(2)
.iso(1).org(3).dod(6).internet(1).mgmt(2).mib-2(1)
.iso(1).org(3).dod(6).internet(1).mgmt(2).mib-2(1).system(1)
.iso(1).org(3).dod(6).internet(1).mgmt(2).mib-2(1).system(1).sysDescr(1)
.iso(1).org(3).dod(6).internet(1).mgmt(2).mib-2(1).system(1).sysObjectID(2)
.iso(1).org(3).dod(6).internet(1).mgmt(2).mib-2(1).system(1).sysUpTime(3)
```

- `-Tp` **Get a nice tree output**

```bash
snmptranslate -Tp -M /usr/share/snmp/mibs -m RITTAL-CMC-TC-MIB > /tmp/rittal-cmc-tc-oid-tree.txt
snmptranslate -Tp -M /usr/share/snmp/mibs -m RITTAL-CMC-TC-MIB | head
+--iso(1)
   |
   +--org(3)
      |
      +--dod(6)
         |
         +--internet(1)
            |
            +--directory(1)
            |
```

- `-TB` **To _grep_ for a OID**

```bash
snmptranslate -TB -Of -M /usr/share/snmp/mibs -m RITTAL-CMC-TC-MIB .\*sensor.\* | head
.iso.org.dod.internet.private.enterprises.rittal.cmcTc.cmcTc#.alarmSensorUnit4
.iso.org.dod.internet.private.enterprises.rittal.cmcTc.cmcTc#.alarmSensorUnit3
.iso.org.dod.internet.private.enterprises.rittal.cmcTc.cmcTc#.alarmSensorUnit2
.iso.org.dod.internet.private.enterprises.rittal.cmcTc.cmcTc#.alarmSensorUnit1
.iso.org.dod.internet.private.enterprises.rittal.cmcTc.cmcTcStatus.cmcTcStatusSensorUnit4
.iso.org.dod.internet.private.enterprises.rittal.cmcTc.cmcTcStatus.cmcTcStatusSensorUnit4.cmcTcStatusUnit4Sensors
.iso.org.dod.internet.private.enterprises.rittal.cmcTc.cmcTcStatus.cmcTcStatusSensorUnit4.cmcTcStatusUnit4Sensors.cmcTcUnit4SensorTable
.iso.org.dod.internet.private.enterprises.rittal.cmcTc.cmcTcStatus.cmcTcStatusSensorUnit4.cmcTcStatusUnit4Sensors.cmcTcUnit4SensorTable.cmcTcUnit4SensorEntry
.iso.org.dod.internet.private.enterprises.rittal.cmcTc.cmcTcStatus.cmcTcStatusSensorUnit4.cmcTcStatusUnit4Sensors.cmcTcUnit4SensorTable.cmcTcUnit4SensorEntry.unit4SensorInfo
.iso.org.dod.internet.private.enterprises.rittal.cmcTc.cmcTcStatus.cmcTcStatusSensorUnit4.cmcTcStatusUnit4Sensors.cmcTcUnit4SensorTable.cmcTcUnit4SensorEntry.unit4SensorUnit
```


# Find the name for a known numberic OID #

I know it is in the RITTAL-CMC-TC-MIB but how can I most easily find the name for `1.3.6.1.4.1.2606.4.2.4.5.2.1.5.5`.

```bash
snmptranslate -Of -M /usr/share/snmp/mibs -m RITTAL-CMC-TC-MIB 1.3.6.1.4.1.2606.4.2.4.5.2.1.5.5
.iso.org.dod.internet.private.enterprises.rittal.cmcTc.cmcTcStatus.cmcTcStatusSensorUnit2.cmcTcStatusUnit2Sensors.cmcTcUnit2SensorTable.cmcTcUnit2SensorEntry.unit2SensorValue.5
```

Add `-Td` to get the description of the OID from the MIB.

```bash
snmptranslate -Td -M /usr/share/snmp/mibs -m RITTAL-CMC-TC-MIB 1.3.6.1.4.1.2606.4.2.4.5.2.1.5.5
RITTAL-CMC-TC-MIB::unit2SensorValue.5
unit2SensorValue OBJECT-TYPE
  -- FROM       RITTAL-CMC-TC-MIB
  SYNTAX        INTEGER
  MAX-ACCESS    read-only
  STATUS        mandatory
  DESCRIPTION   "Value of sensor"
{% raw %}
::= { iso(1) org(3) dod(6) internet(1) private(4) enterprises(1) rittal(2606) cmcTc(4) cmcTcStatus(2) cmcTcStatusSensorUnit2(4) cmcTcStatusUnit2Sensors(5) cmcTcUnit2SensorTable(2) cmcTcUnit2SensorEntry(1) unit2SensorValue(5) 5 }
```
```

# SNMP Tables #

Ah, so it looks like `.5` is an entry the `unit2SensorValue` table. What is in it?

```bash
snmpwalk -c community -v 1 -m RITTAL-CMC-TC-MIB 192.168.0.1 1.3.6.1.4.1.2606.4.2.4.5.2.1.5.5
RITTAL-CMC-TC-MIB::unit2SensorValue.5 = INTEGER: 680
```

# Debugging MIBs #

## Missing MIB ##

Cmd:

```text
{% raw %}
snmptranslate -TBd -Of -M /usr/share/snmp/mibs -m EATON-PXG-MIB '.*xups.*'
Cannot find module (ENTITY-MIB): At line 10 in /usr/share/snmp/mibs/EATON-PXG-MIB.txt
Did not find 'entPhysicalName' in module #-1 (/usr/share/snmp/mibs/EATON-PXG-MIB.txt)
.iso.org.dod.internet.private.enterprises.eaton.xupsIoMIB
xupsIoMIB OBJECT-TYPE
  -- FROM       EATON-OIDS
::= { iso(1) org(3) dod(6) internet(1) private(4) enterprises(1) eaton(534) 3 }
.iso.org.dod.internet.private.enterprises.eaton.xupsObjectId
xupsObjectId OBJECT-TYPE
  -- FROM       EATON-OIDS
::= { iso(1) org(3) dod(6) internet(1) private(4) enterprises(1) eaton(534) 2 }
.iso.org.dod.internet.private.enterprises.eaton.xupsMIB
xupsMIB OBJECT-TYPE
  -- FROM       EATON-OIDS
::= { iso(1) org(3) dod(6) internet(1) private(4) enterprises(1) eaton(534) 1 }
.iso.org.dod.internet.private.enterprises.eaton.xupsMIB.xupsEnvironment
xupsEnvironment OBJECT-TYPE
  -- FROM       EATON-OIDS
::= { iso(1) org(3) dod(6) internet(1) private(4) enterprises(1) eaton(534) xupsMIB(1) 6 }
```
```

Error:

```text
Cannot find module (ENTITY-MIB): At line 10 in /usr/share/snmp/mibs/EATON-PXG-MIB.txt
Did not find 'entPhysicalName' in module #-1 (/usr/share/snmp/mibs/EATON-PXG-MIB.txt)
```

MIB:

{% highlight text lineno %}
EATON-PXG-MIB DEFINITIONS ::= BEGIN

IMPORTS
      MODULE-IDENTITY, OBJECT-TYPE, NOTIFICATION-TYPE, Gauge32,
        Integer32
           FROM SNMPv2-SMI
      TimeStamp
           FROM SNMPv2-TC
      entPhysicalName
           FROM ENTITY-MIB
```

Google says [go here](http://www.oidview.com/mibs/0/ENTITY-MIB.html) to download it to `/usr/share/snmp/mibs/ENTITY-MIB.mib` or `.txt`.

# Eaton #

All the mibs. Get all the mibs.

- http://powerquality.eaton.com/Support/Software-Drivers/Downloads/connectivity-firmware.asp
- http://www.oidview.com/mibs/534/md-534-1.html

# Monitoring Eaton Power Expert Gateway #

What the heck do we have? Well, 

```text
{% raw %}
snmptranslate -Td -Of -M /usr/share/snmp/mibs -m all .iso.org.dod.internet.private.enterprises.powerware.powerChain.pcdMIB
.iso.org.dod.internet.private.enterprises.powerware.powerChain.pcdMIB
pcdMIB MODULE-IDENTITY
  -- FROM       EATON-PCD-MIB
  DESCRIPTION   "The MIB module for objects for common measures that most Eaton Power
         Chain Devices can support.  Some portion of this MIB would be
         supported by most Eaton equipment, including Meters.

         This MIB covers three areas:
                 - Voltage, Current, Frequency, and %Load measures.
                 - Digital (binary) Inputs and Outputs.
                 - Generic (non-specific) sensor readings.

                 All objects are indexed by a device number, since some agents
                 will present data from more than one Power Chain Device.
                 This powerChainDeviceIndex is normally the same as entPhysicalIndex.

        Copyright (C) Eaton Corporation (2007-)."
::= { iso(1) org(3) dod(6) internet(1) private(4) enterprises(1) powerware(534) powerChain(8) 2 }
```
```

```text
snmpwalk -v1 -c public -Of -M /usr/share/snmp/mibs -m all 192.168.0.1 .iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalDescr
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalDescr.1 = STRING: Power Xpert Gateway
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalDescr.10 = STRING: PXG COM 1 Port
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalDescr.11 = STRING: Meter 1
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalDescr.12 = STRING: Meter 1
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalDescr.13 = STRING: Meter 3
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalDescr.14 = STRING: Meter 4
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalDescr.50 = STRING: PXG COM 2 Port
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalDescr.90 = STRING: PXG INCOM Port
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalDescr.91 = STRING: FP5000 2
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalDescr.92 = STRING: FP5000 1
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalDescr.93 = STRING: MS-1A Meter
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalDescr.94 = STRING: MS-1B Meter
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalDescr.95 = STRING: Transfer Switch
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalDescr.96 = STRING: MS-1A Trip
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalDescr.97 = STRING: Main M2BC
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalDescr.98 = STRING: MBP-AX
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalDescr.99 = STRING: DPH-1A
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalDescr.100 = STRING: ATS-1
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalDescr.101 = STRING: TG-1G
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalDescr.102 = STRING: DPH-2A
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalDescr.103 = STRING: DPH-1B
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalDescr.104 = STRING: TG-DC-1
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalDescr.105 = STRING: Elevator
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalDescr.106 = STRING: DPH-1C
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalDescr.107 = STRING: DSH-1B
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalDescr.160 = STRING: PXG COM 3 Port
```

OK, so let's see what we can learn about MS-1B Meter.

```text
snmpwalk -v1 -c public -Of -M /usr/share/snmp/mibs -m all 192.168.0.1 .iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry | grep 94
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalDescr.94 = STRING: MS-1B Meter
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalVendorType.94 = OID: .ccitt.zeroDotZero
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalContainedIn.94 = INTEGER: 90
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalClass.94 = INTEGER: module(9)
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalParentRelPos.94 = INTEGER: 4
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalName.94 = STRING: MS-1B Meter
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalHardwareRev.94 = STRING:
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalFirmwareRev.94 = STRING:
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalSoftwareRev.94 = STRING:
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalSerialNum.94 = STRING:
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalMfgName.94 = STRING: Eaton Corporation
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalModelName.94 = STRING: IQ DP-4000
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalAlias.94 = STRING: /incom/4
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalAssetID.94 = STRING: Customer AssetIdTag
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.entPhysicalIsFRU.94 = INTEGER: true(1)
.iso.org.dod.internet.mgmt.mib-2.entityMIB.entityMIBObjects.entityPhysical.entPhysicalTable.entPhysicalEntry.18.94 = ""
```


```text
snmpwalk -v1 -c public -Of -M /usr/share/snmp/mibs -m all 192.168.0.1 .iso.org.dod.internet.private.enterprises.powerware.powerChain.pcdMIB | grep 94
.iso.org.dod.internet.private.enterprises.powerware.powerChain.pcdMIB.pcdMIBObjects.pcdMeasures.pcdMeasuresTable.pcdMeasuresEntry.pcdMeasuresFrequency.94 = INTEGER: 0 0.01 Hertz
.iso.org.dod.internet.private.enterprises.powerware.powerChain.pcdMIB.pcdMIBObjects.pcdMeasures.pcdPhaseMeasuresTable.pcdPhaseMeasuresEntry.pcdPhaseMeasuresVoltageLL.94.1 = INTEGER: 495 Volts
.iso.org.dod.internet.private.enterprises.powerware.powerChain.pcdMIB.pcdMIBObjects.pcdMeasures.pcdPhaseMeasuresTable.pcdPhaseMeasuresEntry.pcdPhaseMeasuresVoltageLL.94.2 = INTEGER: 492 Volts
.iso.org.dod.internet.private.enterprises.powerware.powerChain.pcdMIB.pcdMIBObjects.pcdMeasures.pcdPhaseMeasuresTable.pcdPhaseMeasuresEntry.pcdPhaseMeasuresVoltageLL.94.3 = INTEGER: 497 Volts
.iso.org.dod.internet.private.enterprises.powerware.powerChain.pcdMIB.pcdMIBObjects.pcdMeasures.pcdPhaseMeasuresTable.pcdPhaseMeasuresEntry.pcdPhaseMeasuresVoltageLN.94.1 = INTEGER: 0 Volts
.iso.org.dod.internet.private.enterprises.powerware.powerChain.pcdMIB.pcdMIBObjects.pcdMeasures.pcdPhaseMeasuresTable.pcdPhaseMeasuresEntry.pcdPhaseMeasuresVoltageLN.94.2 = INTEGER: 285 Volts
.iso.org.dod.internet.private.enterprises.powerware.powerChain.pcdMIB.pcdMIBObjects.pcdMeasures.pcdPhaseMeasuresTable.pcdPhaseMeasuresEntry.pcdPhaseMeasuresVoltageLN.94.3 = INTEGER: 286 Volts
.iso.org.dod.internet.private.enterprises.powerware.powerChain.pcdMIB.pcdMIBObjects.pcdMeasures.pcdPhaseMeasuresTable.pcdPhaseMeasuresEntry.pcdPhaseMeasuresCurrent.91.1 = INTEGER: 94 0.1 Amps RMS
.iso.org.dod.internet.private.enterprises.powerware.powerChain.pcdMIB.pcdMIBObjects.pcdMeasures.pcdPhaseMeasuresTable.pcdPhaseMeasuresEntry.pcdPhaseMeasuresCurrent.94.1 = INTEGER: 7664 0.1 Amps RMS
.iso.org.dod.internet.private.enterprises.powerware.powerChain.pcdMIB.pcdMIBObjects.pcdMeasures.pcdPhaseMeasuresTable.pcdPhaseMeasuresEntry.pcdPhaseMeasuresCurrent.94.2 = INTEGER: 7219 0.1 Amps RMS
.iso.org.dod.internet.private.enterprises.powerware.powerChain.pcdMIB.pcdMIBObjects.pcdMeasures.pcdPhaseMeasuresTable.pcdPhaseMeasuresEntry.pcdPhaseMeasuresCurrent.94.3 = INTEGER: 7641 0.1 Amps RMS
```
