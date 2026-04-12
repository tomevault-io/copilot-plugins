## buoy

> Ansible role and playbook conventions


# Ansible Conventions

## Role Order (playbook.yml)

1. common
2. docker
3. local_dns
4. wifi_ap
5. ros2_docker
6. command_center
7. captive_portal
8. first_boot

## Image Build vs Runtime

- **docker_image_build=true**: During SD image build (chroot). Skips "Start ROS 2" tasks (no dockerd in chroot)
- **docker_image_prebuilt=true**: ROS image was built on host; skip "Build ROS 2 image" task
- **offline_first_boot=true**: Field deployment; playbook does not install packages or pull images. Set false during initial build.

## Service Ordering

Services that depend on network interfaces must start after interface config:

- **nodogsplash**: `After=wifi-ap-setup.service hostapd.service` (see captive_portal systemd override)
- **buoy-ros**: `After=docker.service`

## Variables (group_vars/all.yml)

- `wifi_interface`: wlan0
- `buoy_hostname`, `buoy_domain`: for .buoy DNS
- `captive_portal_enable`: false by default; set true for Nodogsplash splash page

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wilfriedE) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
