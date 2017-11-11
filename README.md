## Workstation Setup

### CUDA toolkit installation (includes NVIDIA driver, CUDA toolkit and samples)

1. Physically install the NVIDIA GeForce GTX 1080 Ti card on PCI Express 3.0 dual width x16 slot of motherboard and connect 6-pin and 8-pin power adaptors.
2. Verify that 

```
$ update-pciids
$ lspci | grep -i nvidia
```

Reference: 
[GTX 1080 Ti User Guide](workstation-setup/GTX_1080_Ti_User_Guide.pdf)
[CUDA Installation Guide Linux](workstation-setup/CUDA_Installation_Guide_Linux.pdf)




