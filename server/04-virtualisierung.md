# Server Phase 4 – Virtualisierung

**Dauer:** 6–8 Wochen · **Voraussetzung:** Phasen 1–3, Netzwerk Phase 2 (VLANs)

---

## 1. Konzepte

- [ ] **Hypervisor Typ 1** (bare metal: ESXi, Hyper-V, KVM/Proxmox, Nutanix AHV, Xen) vs. **Typ 2** (VirtualBox, VMware Workstation)
- [ ] Vorteile: Konsolidierung, Isolation, schnelle Bereitstellung, HA, Live-Migration, Snapshots
- [ ] Ressourcen: vCPU, vRAM, Overcommitment (CPU-Ratio z. B. 4:1), Ballooning, Memory Compression, Swapping
- [ ] **CPU Ready / Steal Time** – VM wartet auf physische CPU (zu stark überbucht)
- [ ] NUMA-Sizing (VM nicht größer als ein NUMA-Knoten, wenn vermeidbar)
- [ ] Virtuelle Hardware: Paravirtualisierte Treiber (**VMXNET3**, **PVSCSI**, **VirtIO**), Guest Tools (VMware Tools, open-vm-tools, qemu-guest-agent)
- [ ] Disk-Typen: Thin vs. Thick (Lazy/Eager Zeroed), RDM/Passthrough
- [ ] **Snapshots sind kein Backup** – und dürfen nicht wochenlang stehen bleiben (Delta-Dateien wachsen, Performance, Datastore voll)
- [ ] Templates, Klone, Customization (Sysprep, cloud-init)
- [ ] P2V / V2V-Migration

## 2. Virtuelles Netzwerk

- [ ] Virtueller Switch, Portgruppen/VLAN-Tagging (VST), Uplinks (physische NICs)
- [ ] VMware: **Standard vSwitch** vs. **Distributed Switch** (vDS); VMkernel-Ports (Management, vMotion, vSAN, iSCSI, NFS)
- [ ] NIC-Teaming-Policies (Route based on originating port ID, IP hash ↔ LACP), LACP auf vDS
- [ ] Netzwerk-Trennung: Management, vMotion, Storage, VM-Traffic (eigene VLANs, teils eigene NICs)
- [ ] Proxmox/KVM: Linux-Bridge (`vmbr0`), VLAN-aware Bridge, Open vSwitch, SDN
- [ ] Hyper-V: externer/interner/privater vSwitch, SET (Switch Embedded Teaming)
- [ ] Overlay: VMware NSX (Grundzüge)

## 3. VMware vSphere

> Trotz Lizenzänderungen nach der Broadcom-Übernahme in vielen RZ noch Standard – Kenntnisse sind gefragt.

- [ ] **ESXi** (Host), **vCenter Server** (zentrale Verwaltung, VCSA-Appliance)
- [ ] Datacenter → Cluster → Hosts → VMs; Ordner, Resource Pools, Tags
- [ ] Datastores: **VMFS** (Block: FC/iSCSI/lokal), **NFS**, **vSAN**
- [ ] **vMotion** (Live-Migration Compute), **Storage vMotion**, Cross-vCenter
- [ ] **HA** (Neustart von VMs bei Host-Ausfall; Admission Control, Isolation Response, Heartbeat-Datastores)
- [ ] **DRS** (automatische Lastverteilung, Affinity/Anti-Affinity-Regeln – z. B. beide DCs nie auf demselben Host!)
- [ ] Fault Tolerance (Grundzüge), EVC (CPU-Kompatibilität für vMotion)
- [ ] Wartungsmodus, **Lifecycle Manager** (Updates/Images)
- [ ] Rollen & Rechte, SSO, Alarme
- [ ] VM-Dateien: `.vmx` (Konfig), `.vmdk` (Disk-Deskriptor + `-flat.vmdk`), `-delta.vmdk` / `-sesparse.vmdk` (Snapshots), `.nvram`, `vmware.log`
```bash
# ESXi-Shell (SSH nur temporär aktivieren!)
esxcli system version get
esxcli network nic list
esxcli network ip interface ipv4 get
esxcli network vswitch standard list
esxcli storage core device list
esxcli storage vmfs extent list
esxcli storage core path list           # Multipathing
esxcli software vib list
esxcli system maintenanceMode set --enable true
vim-cmd vmsvc/getallvms
vim-cmd vmsvc/power.getstate <vmid>
vmkping -I vmk1 -d -s 8972 10.30.0.12   # Jumbo-Frame-Test über bestimmten VMkernel-Port
esxtop                                  # c=CPU (%RDY!), m=Memory, d=Disk (DAVG/KAVG), n=Netz
```
- [ ] **PowerCLI** (PowerShell für vSphere):
```powershell
Connect-VIServer vcenter.firma.local
Get-VM | Sort-Object MemoryGB -Descending | Select -First 10 Name, NumCpu, MemoryGB, PowerState
Get-VM | Get-Snapshot | Select VM, Name, Created, SizeGB       # alte Snapshots finden!
Get-VMHost | Select Name, ConnectionState, Version
Move-VM -VM app01 -Destination esx02.firma.local
```

## 4. Proxmox VE (ideal fürs Heimlabor)

- [ ] KVM-VMs + **LXC-Container**, Web-GUI, Cluster (Corosync), HA, Live-Migration
- [ ] Storage: lokal (LVM-Thin, ZFS), NFS, iSCSI, **Ceph** (integriert)
- [ ] **Proxmox Backup Server** (dedupliziert, inkrementell)
```bash
pveversion -v
qm list ; qm start 100 ; qm shutdown 100 ; qm config 100
qm migrate 100 pve02 --online
pct list ; pct enter 200
pvecm status ; pvecm nodes        # Cluster & Quorum
pvesm status                      # Storage
ha-manager status
```

## 5. Microsoft Hyper-V

- [ ] Generation 1 vs. 2 VMs, dynamischer Speicher, VHDX, Checkpoints (Standard vs. Production)
- [ ] **Failover-Cluster** + Cluster Shared Volumes (CSV), Live-Migration, Storage Spaces Direct (Azure Stack HCI)
- [ ] Hyper-V-Replica
```powershell
Get-VM ; Start-VM app01 ; Stop-VM app01
Get-VMSwitch
Get-VMSnapshot -VMName app01
Move-VM app01 -DestinationHost hv02
Get-ClusterNode ; Get-ClusterGroup ; Get-ClusterSharedVolume
Test-Cluster
```

## 6. KVM / libvirt (Linux nativ)

```bash
virsh list --all
virsh start vm01 ; virsh shutdown vm01 ; virsh destroy vm01   # destroy = hart aus
virsh console vm01
virsh dominfo vm01 ; virsh dumpxml vm01
virsh snapshot-create-as vm01 vor-update
virt-install --name vm02 --memory 4096 --vcpus 2 --disk size=40 --cdrom rocky.iso --os-variant rocky9
qemu-img info disk.qcow2 ; qemu-img convert -O qcow2 disk.vmdk disk.qcow2
```

## 7. Weitere Plattformen (Grundzüge)

- [ ] **Nutanix** (HCI, AHV, Prism)
- [ ] **OpenStack** (Nova, Neutron, Cinder, Glance, Keystone) – Private Cloud
- [ ] XCP-ng / Citrix Hypervisor
- [ ] **Hyperconverged Infrastructure (HCI)** vs. klassische 3-Tier-Architektur (Compute + SAN + Netzwerk)

## 8. Cluster-Grundlagen (gilt überall)

- [ ] **Quorum** (Mehrheit: bei 3 Knoten darf 1 ausfallen; Witness bei geraden Zahlen)
- [ ] **Split-Brain** und wie Fencing/STONITH es verhindert
- [ ] Heartbeat-Netze, Redundanz der Cluster-Kommunikation
- [ ] N+1-Kapazität: Reicht der Cluster, wenn ein Host ausfällt? (Rechnen!)
- [ ] Rolling Updates im Wartungsmodus

## 9. Praxisaufgaben

- [ ] Proxmox-Cluster aus 3 Knoten (zur Not verschachtelt in VMs), Live-Migration einer VM
- [ ] Shared Storage per NFS oder Ceph, HA-Gruppe konfigurieren, einen Knoten „hart ausschalten“ und HA beobachten
- [ ] Template mit cloud-init erstellen, 5 VMs daraus klonen
- [ ] VMware ESXi (Evaluierung) + vCenter im verschachtelten Labor (falls Ressourcen reichen) oder im Betrieb mit Ausbilder: vMotion, Snapshot, Wartungsmodus
- [ ] PowerCLI-Report: alle VMs mit Snapshots älter als 3 Tage
- [ ] Kapazitätsrechnung: Cluster mit 4 Hosts à 512 GB RAM, Auslastung 70 % – übersteht er einen Host-Ausfall?

## Meine Notizen

