Inter-VLAN Routing with SVI

## 📌 Overview
Enterprise switching lab implementing VLANs, VTP domain,
SVI inter-VLAN routing on a Layer 3 core switch, and
trunk/access port configuration across multiple switches.
Goal: Full communication between VLAN 10 and VLAN 40.

---

## 🗺️ Network Topology

    [Internet]
        |
      [CORE] — Layer 3 Switch (VTP Server)
        |
    ----------------------------------------
    |         |         |         |         |
  [SW1]    [SW2]    [SW3]    [SW4]    [SW5]
    |         |         |         |         |
  PC1-4    PC5-8   PC9-12  PC13-16   WIN-SRV

---

## 📋 VLAN Plan

| VLAN ID | Name | Department |
|---------|------|------------|
| VLAN 10 | IT | IT Department |
| VLAN 20 | HR | HR Department |
| VLAN 30 | SEC | Security Department |
| VLAN 40 | ACC | Accounts Department |
| VLAN 50 | CYBER | WIN-SRV (Cyber) |

---

## 📋 PC to VLAN Assignment

### SW1
| PC | VLAN |
|----|------|
| PC1 | VLAN 10 |
| PC2 | VLAN 20 |
| PC3 | VLAN 30 |
| PC4 | VLAN 40 |

### SW2
| PC | VLAN |
|----|------|
| PC5 | VLAN 10 |
| PC6 | VLAN 20 |
| PC7 | VLAN 30 |
| PC8 | VLAN 40 |

### SW3
| PC | VLAN |
|----|------|
| PC9 | VLAN 10 |
| PC10 | VLAN 20 |
| PC11 | VLAN 30 |
| PC12 | VLAN 40 |

### SW4
| PC | VLAN |
|----|------|
| PC13 | VLAN 10 |
| PC14 | VLAN 20 |
| PC15 | VLAN 30 |
| PC16 | VLAN 40 |

### SW5
| Device | VLAN |
|--------|------|
| WIN-SRV | VLAN 50 |

---

## 📌 Why switchport trunk encapsulation dot1q

- Some Cisco switches support both ISL and
  dot1q encapsulation methods
- dot1q (IEEE 802.1Q) is the industry standard
  encapsulation for trunk links
- Without this command the switch does not know
  which encapsulation to use for the trunk
- ISL is old Cisco proprietary method
- dot1q works with all vendors and devices
- Always use dot1q in modern networks
- Encapsulation command must be written BEFORE
  switchport mode trunk otherwise it gives error

| Encapsulation | Type |
|--------------|------|
| dot1q | Industry standard — used everywhere |
| ISL | Old Cisco proprietary — not recommended |

---

## 📌 Why SVI is Used Instead of Router

- CORE is a Layer 3 switch so it can do
  routing internally without a separate router
- SVI (Switched Virtual Interface) is a
  virtual interface created for each VLAN
- It acts as the gateway for all devices
  in that VLAN
- Traffic between VLANs goes through CORE
  switch internally — much faster than
  going through a separate router

| Method | How It Works |
|--------|-------------|
| Router on a Stick | One router handles all VLANs via subinterfaces |
| SVI on Layer 3 Switch | Switch itself routes between VLANs internally |

---

## 📌 Why VTP Server and Client

- Without VTP every switch needs VLANs
  configured manually one by one
- VTP Server (CORE) creates VLANs once
- VTP Client (SW1 SW2 SW3 SW4 SW5) receives
  VLANs automatically
- Saves time and prevents mistakes

| Switch | VTP Mode | Role |
|--------|----------|------|
| CORE | Server | Creates and sends VLANs |
| SW1 | Client | Receives VLANs automatically |
| SW2 | Client | Receives VLANs automatically |
| SW3 | Client | Receives VLANs automatically |
| SW4 | Client | Receives VLANs automatically |
| SW5 | Client | Receives VLANs automatically |

---

## ✅ What I Configured

### 1. VTP Domain on CORE Switch (Server Mode)
- CORE switch set as VTP Server
- All other switches set as VTP Client
- VLANs created only on CORE and automatically
  pushed to all client switches

    vtp mode server
    vtp domain corvit

### 2. VLAN Creation on CORE Switch
- Created all VLANs on CORE switch only
- VTP automatically pushed to all client switches

    vlan 10
    name IT
    vlan 20
    name HR
    vlan 30
    name SEC
    vlan 40
    name ACC
    vlan 50
    name CYBER

### 3. VTP Client Mode on All Other Switches
- Configured on SW1 SW2 SW3 SW4 SW5
- VLANs received automatically from CORE

    vtp mode client
    vtp domain corvit

### 4. Trunk Configuration on CORE Switch
- dot1q encapsulation added before trunk mode
- This is mandatory on Layer 3 switches

    interface ethernet 1/0
    switchport trunk encapsulation dot1q
    switchport mode trunk

    interface ethernet 0/0
    switchport trunk encapsulation dot1q
    switchport mode trunk

    interface ethernet 0/1
    switchport trunk encapsulation dot1q
    switchport mode trunk

    interface ethernet 0/2
    switchport trunk encapsulation dot1q
    switchport mode trunk

    interface ethernet 0/3
    switchport trunk encapsulation dot1q
    switchport mode trunk

### 5. Trunk Configuration on SW1 SW2 SW3 SW4 SW5
- Same trunk command on all access switches
- Uplink port toward CORE set as trunk

    interface ethernet 1/1
    switchport trunk encapsulation dot1q
    switchport mode trunk

### 6. SVI Configuration on CORE Switch
- Created SVI for each VLAN
- Each SVI acts as default gateway for its VLAN

    interface vlan 10
    ip address 192.168.10.1 255.255.255.0
    no shutdown

    interface vlan 20
    ip address 192.168.20.1 255.255.255.0
    no shutdown

    interface vlan 30
    ip address 192.168.30.1 255.255.255.0
    no shutdown

    interface vlan 40
    ip address 192.168.40.1 255.255.255.0
    no shutdown

    interface vlan 50
    ip address 192.168.50.1 255.255.255.0
    no shutdown

### 7. Enable IP Routing on CORE Switch
- CORE is Layer 3 so IP routing must be enabled
- Without this inter-VLAN routing will not work

    ip routing

### 8. Access Port Configuration on SW1 SW2 SW3 SW4
- Each PC port set to access mode
- Each port assigned to correct VLAN

    interface ethernet 0/0
    switchport mode access
    switchport access vlan 10

    interface ethernet 0/1
    switchport mode access
    switchport access vlan 20

    interface ethernet 0/2
    switchport mode access
    switchport access vlan 30

    interface ethernet 0/3
    switchport mode access
    switchport access vlan 40

### 9. Access Port for WIN-SRV on SW5

    interface ethernet 0/1
    switchport mode access
    switchport access vlan 50

### 10. Gateway on Each PC
- Each PC given IP address in its VLAN subnet
- Gateway set to SVI IP of CORE switch

| VLAN | PC IP Range | Gateway |
|------|-------------|---------|
| VLAN 10 | 192.168.10.x | 192.168.10.1 |
| VLAN 20 | 192.168.20.x | 192.168.20.1 |
| VLAN 30 | 192.168.30.x | 192.168.30.1 |
| VLAN 40 | 192.168.40.x | 192.168.40.1 |
| VLAN 50 | 192.168.50.x | 192.168.50.1 |

### 11. Verification Commands

    show vlan brief
    show vtp status
    show interfaces trunk
    show ip route
    ping 192.168.40.x

---

## 🔄 Traffic Flow — VLAN 10 to VLAN 40

    [PC1 - VLAN 10]
          |
        [SW1] — trunk — [CORE]
                            |
                          SVI routes VLAN10 to VLAN40
                            |
                        [SW1/SW2/SW3/SW4]
                            |
                    [PC4/PC8/PC12/PC16 - VLAN 40]

---

## ❌ Errors I Faced & How I Fixed Them

### Error 1 — VLANs Not Appearing on Client Switches
**Problem:**
VLANs created on CORE not showing on SW1 SW2 SW3 SW4

**Cause:**
VTP domain name was different on client switches

**Fix:**
- Set same domain name corvit on all switches
- VLANs appeared automatically ✅

---

### Error 2 — Ping Failing Between VLANs
**Problem:**
PC in VLAN 10 could not ping PC in VLAN 40

**Cause:**
IP routing was not enabled on CORE switch

**Fix:**

    ip routing

Inter-VLAN ping worked after this ✅

---

### Error 3 — Trunk Command Giving Error
**Problem:**
switchport mode trunk command giving error
on CORE switch

**Cause:**
Encapsulation was not set before trunk mode
on Layer 3 switch

**Fix:**

    interface ethernet 1/0
    switchport trunk encapsulation dot1q
    switchport mode trunk

Trunk configured successfully ✅

---

### Error 4 — SVI Not Coming Up
**Problem:**
VLAN interface showing down even after configuration

**Cause:**
VLAN was not active or no port was assigned to it

**Fix:**
- Verified VLAN exists using show vlan brief
- Assigned at least one access port to the VLAN
- SVI came up automatically ✅

---

### Error 5 — Trunk Not Passing All VLANs
**Problem:**
Some VLANs not passing through trunk links

**Cause:**
Port was set to access mode instead of trunk mode

**Fix:**

    interface ethernet 1/0
    switchport trunk encapsulation dot1q
    switchport mode trunk

All VLANs passed through trunk correctly ✅

---

## 🔧 Tools Used
- EVE-NG — used to build and run the full
  switching topology
- SecureCRT — used to connect to each switch
  via terminal and enter all commands
- Cisco Layer 3 Switch (CORE)
- Cisco Layer 2 Switches (SW1 SW2 SW3 SW4 SW5)
- Windows Server (WIN-SRV) on VLAN 50

---

## 📚 What I Learned
- How VTP Server and Client mode works
- How SVI enables inter-VLAN routing on
  a Layer 3 switch without a router
- Why ip routing command is needed on
  Layer 3 switch
- Why dot1q encapsulation must be set before
  trunk mode on Layer 3 switches
- How trunk ports carry multiple VLANs
- How access ports assign PCs to VLANs
- How VTP domain pushes VLANs automatically
  to all client switches
- Difference between Layer 2 and Layer 3 switch
- Difference between dot1q and ISL encapsulation

---

## 🎯 Tasks Completed
- [x] Configure VTP domain corvit on all switches
- [x] Set CORE as VTP Server
- [x] Set SW1 SW2 SW3 SW4 SW5 as VTP Client
- [x] Create VLANs 10 20 30 40 50 on CORE
- [x] Configure dot1q encapsulation on all trunks
- [x] Configure trunk on CORE and all switches
- [x] Configure SVI for each VLAN on CORE
- [x] Enable ip routing on CORE
- [x] Configure access ports on SW1 SW2 SW3 SW4
- [x] Assign WIN-SRV to VLAN 50 on SW5
- [x] Set gateway on all PCs
- [x] Ping successfully from VLAN 10 to VLAN 40
