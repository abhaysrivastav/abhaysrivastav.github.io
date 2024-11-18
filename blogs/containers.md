# Containers in a Week  

## Baremetal Infrastructure 
## Virtualized Infrastructure
## Containerized Infrastructure 

## What is Virtualization ?

### Hypervisor (Virtual Box, Hyper- V, ESX/ESXi)

### Type-I
- Does not provide the host OS
### Type- II 
- Provides with Host OS

### CRI (Container Runtime Interface)
    - Manages containers life cycle.  
    - Docker , Podman, Containered , CRI-o
    - Runs on HOST OS. 
    - Dont have GUEST OS, means they share underlying HOST OS.
    - All container runs as a processes and control is provided by CRI. 
    - Container need container's images that contains application related binaries/libs. It share the packages from HOST OS as well.


