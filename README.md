# amd_nri_plugin
AMD burstable POD scheduling options on ccx's


Kubernetes treats all CPU instances on a Worker Node as one big pool. The cgroup CPU controller cpuset for all Burstable PODs is the same, all CPUs. Linux will migrate (schedule) processes for a Cotainer (application) across all CPU's regardless of the Worker Node physical topology; Sockets, NUMA, ccx (CPU groups on Chipletes), and SMT.

By making cpuset that match CCX and placing Containers within one of these cpuset slices, Linux will only schedule and migrate the processes of those Containers (applications) within the range of CPUs for a single CCX. This lowers applicant event processing jitter, increases application performace (less OS migration tax, lower scaling penalty for shared objects and SW syncronization devices). 

This NRI pluging is targeted at AMD CPU's only for now. 

# POD and Container placement options (these are global to the whole Worker Node):

**Packed:** Allocate all Containers for a POD the same cpuset colerilating with a CCX, and evenly distribute the PODs across all CCX's based on spec:resources:cpu:request quanta.

**Spread N:** Build a Container cpuset that is distributed based on NUMA and a maximum of N CCX's.


# Future scheduling options:

**Exclusive:** Guranteed CPU QoS, supported with Packed mode. Guranteed PODs aquired CPU's from whole or partial claimed CCX from one direction (affected Burstable PODs beging relocated) and Burstable PODs packed onto the remaing whole CCX. Support for sidecar Container resource sharing or exclusive co-CCX location can be configured.

**MinPower:** Pack Containers onto the minimum number of CCX. Fill CCX up to 60% before utilizing more CCX's. As Containers are removed and CPU resources freed, rebalance packed CCX's to only use the minimum required. Linux power governor can be applied to un-used CCX, thus saving power while not reducing Container SLA.

# Topology learning Options

**SysFs:** This is the default. On bare metal Linux the sysFs information can be trusted.

**Config Profile:** The plugin uses a config file with YAML format

``
kind: amdNriConfig
version: 1.00.01
description: config file for all AMD public cloud instances (VM's). Guest OS
  topology mapping. NUMA and CCX.
profiles:
  azure6x:
      # NRI POD placement policy; packed (default), spreadN where N is the number CCX to use 
      Policy: packed
      #smt on or off for physical instance, used for cpu number processing
      smt: on
      #reservedCpus  map all kube-system name space containers to these cpus
      reservedcpus: 0,1,64,65
         
      numas:
        numa0: 
          ccxs:
            ccx0:
              # cpus 0,32:1,33:...
              cpus: 0-7,64-71
              # or 0,1:2,3:...
            ccx1: 
              cpus: 8-15,72-79
            ccx2:
              cpus: 16-23,68-95
            ccx3:
              cpus: 24-31,68-95
        numa1:
          ccxs:
            ccx0:
              cpus: 32-39,96,103
            ccx1:
              cpus: 40-47,104-111
            ccx2:
              cpus: 48-55,112-119
            ccx3:
              cpus: 56-63,120-127
              ``