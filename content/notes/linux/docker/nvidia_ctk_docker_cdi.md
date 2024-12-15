+++
title = "NVIDIA Container Toolkit with Docker CDI"
date = 2021-12-31
+++
I recently spent an hour (or three) troubleshooting hardware transcoding on my media server.  At some point, an update to the NVIDIA Container Toolkit resulted in CUDA libraries not getting passed through correctly to my containerized Plex installation.

This error appeared in the `Plex Media Server.log` file every time I attempted to play a file that needed transcoding:
```
Dec 15, 2024 03:45:14.374 [140307690093368] ERROR - [Req#105/Transcode] [FFMPEG] - Cannot load libcuda.so.1
Dec 15, 2024 03:45:14.374 [140307690093368] ERROR - [Req#105/Transcode] [FFMPEG] - Could not dynamically load CUDA
```

Running `nvidia-smi` inside the container also presented the following error:
```
NVIDIA-SMI couldn't find libnvidia-ml.so library in your system. Please make sure that the NVIDIA Display Driver is properly installed and present in your system.
Please also try adding directory that contains libnvidia-ml.so to your system PATH.
```

Checking the library directory `/usr/lib/x86_64-linux-gnu/` in the container revealed the missing piece:
```
libcuda.so -> libcuda.so.1 (MISSING)
libcuda.so.535.183.01
libnvcuvid.so.535.183.01
libnvidia-allocator.so.535.183.01
libnvidia-cfg.so.535.183.01
libnvidia-eglcore.so.535.183.01
libnvidia-egl-gbm.so.1.1.0
libnvidia-egl-wayland.so.1.1.10
libnvidia-glcore.so.535.183.01
libnvidia-glsi.so.535.183.01
libnvidia-glvkspirv.so.535.183.01
libnvidia-ml.so.535.183.01
libnvidia-pkcs11-openssl3.so.535.183.01
libnvidia-ptxjitcompiler.so.535.183.01
libnvidia-rtcore.so.535.183.01
libnvidia-tls.so.535.183.01
```

Note the absence of default libraries - in this case, components like `libnvidia-ml.so.565.57.01` which should have a symlink called `libnvidia-ml.so` which makes it the default version.  This also explains the error above for `libcuda.so.1`, which should have been reachable via `libcuda.so` if the link weren't missing.

I saw that the `nvidia-container-toolkit` had undergone quite a few updates since I first set it all up, so I tried updating the NVIDIA drivers on my system from **535.183.01** to **565.57.01** to see if installing newer drivers somehow forced the CTK to relink everything.  That didn't help, and instead resulted in more libraries for the new version that still weren't linking to defaults.

I manually ran the `ldconfig` linker inside the container and observed that it correctly generated the symlinks.  Plex could now see the CUDA library and `nvidia-smi` could report the GPU, but this was only a temporary fix and wouldn't survive a container rebuild.

Research led me to a [GitHub issue on the `nvidia-container-toolkit` project](https://github.com/NVIDIA/nvidia-container-toolkit/issues/128) related to [CDI (container device interface) functionality](https://github.com/NVIDIA/nvidia-container-toolkit/blob/main/cmd/nvidia-ctk/README.md).  CDI support seems to be a relatively new feature of the CTK which manages presenting the correct driver libraries through the container runtime.  I didn't have any CDI configurations present on my system, so I followed the docs to generate one.
```
nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml
nvidia-ctk runtime configure --runtime=docker --cdi.enabled
systemctl restart docker
```

I also had to the add the device `nvidia.com/gpu=all` to the Plex service in my `docker-compose.yaml` file so the NVIDIA container runtime knows to use the CDI configuration.
```
  plex:
    image: ghcr.io/linuxserver/plex
    runtime: nvidia
    environment:
      - NVIDIA_VISIBLE_DEVICES=all
    devices:
      - nvidia.com/gpu=all
```

Once I rebuilt the Plex container with the CDI configuration, all of the NVIDIA libraries in the container were correctly linked with default `.so.` versions and hardware transcoding worked as expected.