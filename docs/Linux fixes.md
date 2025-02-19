---
layout: default
title: Linux fixes
nav_order: 2
---

# Preamble
In the two years that I've been using Linux, I have encountered a lot of bugs; most of them being self-inflicted. The ones that aren't and that I believe might be relevant for others will be linked here. The scheme for the issues below is as follows:

```md
# Issue
Description
## Solution
Steps to solve the issue
## Other suggestions
Alternative suggestions compiled from the internet
```

For further context, I use EndeavourOS, which is where the issues originated in unless specified otherwise.

# Proton audio crackling

When launching games, you might be able to occasionally hear a popping sound, usually when something bassy happens in-game. It's seemingly a very common bug, with a lot of conflicting solutions.

## Solution
The solution that worked for me is creating the file `/etc/pipewire/pipewireconf.d/fix-crackle.conf` containing the following:

```
context.properties = {
    default.clock.min-quantum = 2048
}
```

This fixed the issue permanently on all games. An alternative solution is to add `PULSE_LATENCY_MSEC=60 %command%` as a launch option for the affected app(s). This adds a delay of N milliseconds, which (assumedly) gives pipewire more time to process the audio. 50 should work in most cases, but more might be needed depending on if crackling is still present.

## Other suggestions
- Installing the `faudio` package from your distribution's repository
- Setting a niceness limit

    Linux manages resource allocations using a 'niceness' level, this can allegedly cause issues with Wine. To fix it, add `<YOUR-USER-NAME> - nice - 20` to `/etc/security/limits.conf`.


# Apex legends stuttering

Running Apex legends without any special config has been a miss in my experience, with it always stuttering out of the box on both Kubuntu and EndeavourOS. Whether I am just unlucky or whether it is an actually widespread issue I have no idea, but I bashed my head against the wall enough times to get it to run smoothly.

## Solution

- Disable shader Pre-caching 

    This is a major hassle even prior to starting the game, when enabled you will see a "Compiling Vulkan shaders" screen which can take as long as several hours to complete! With new drivers, Pre-caching is no longer necessary.

> !!! Below only applies to NVIDIA GPUs, I have not verified whether it works with AMD GPUs !!!

- Set `__NV_PRIME_RENDER_OFFLOAD=1 __VK_LAYER_NV_optimus=NVIDIA_only __GLX_VENDOR_LIBRARY_NAME=nvidia DXVK_ASYNC=1 prime-run %command%` as the launch options

    A prerequisite for this is installing the `nvidia-prime` package from your distribution's repository.

- Use [Proton-GE](https://github.com/GloriousEggroll/proton-ge-custom) instead of the stock Proton versions

## Other suggestions

None



