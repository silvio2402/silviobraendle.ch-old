---
title: "How To Reproduce OVPN Problem"
categories:
  - Networking
tags:
header:
  overlay_image: /assets/images/how-to-reproduce-ovpn-problem/console-ovpn-disconnected.png
  overlay_filter: 0.5 # same as adding an opacity of 0.5 to a black background

gallery_dpingerconnected:
  - url: /assets/images/how-to-reproduce-ovpn-problem/dpinger-console-window-connected.png
    image_path: /assets/images/how-to-reproduce-ovpn-problem/dpinger-console-window-connected.png
    alt: "dpinger Console Window Connected"
    title: "dpinger Console Window Connected"

gallery_dpingerdisconnected:
  - url: /assets/images/how-to-reproduce-ovpn-problem/dpinger-console-window-disconnected.png
    image_path: /assets/images/how-to-reproduce-ovpn-problem/dpinger-console-window-disconnected.png
    alt: "dpinger Console Window Disconnected"
    title: "dpinger Console Window Disconnected"
---

In this post I will explain the steps required to measure and reproduce the issue with [OVPN Public IPv4](https://www.ovpn.com/en/features/public-ipv4).

## The Issue

The issue I had with OVPN Public IPv4 was that it would dis- and reconnect every 10 minutes.

## Measuring Network Disconnections

I use [dpinger](https://github.com/dennypage/dpinger) to measure network disconnections. I wasn't able to measure it with just [ping](https://linux.die.net/man/8/ping) for some reason.

### Installing dpinger on Windows

dpinger requires linux to run. For this reason, you have to use [Windows Subsystem for Linux (WSL)](https://docs.microsoft.com/en-us/windows/wsl/) to run it. The next steps will guide you to install WSL, which you may skip if you have it installed already.

#### Installing Windows Subsystem for Linux

1. Run the following command **as administrator** to install WSL and an operating system (Ubuntu by default):
   ```bash
   wsl --install
   ```
   From this point on, you can use the `wsl` command to launch the default shell.
2. Launch the Linux shell:
   ```bash
   wsl
   ```

Since you are now inside of Ubuntu, you can now continue the steps to install dpinger on Linux.

### Installing dpinger on Linux

1. Install git, make and clang with apt:
   ```bash
   sudo apt-get install git make clang -y
   ```
2. Clone and open the dpinger repository:
   ```bash
   git clone https://github.com/dennypage/dpinger.git
   cd dpinger
   ```
3. Build dpinger:
   ```bash
   make
   ```

### Using dpinger

The following command will run dpinger against Cloudflare DNS ([1.1.1.1](https://1.1.1.1)) in the foreground while averaging results for 2s:

```bash
sudo ./dpinger -f -t 2s 1.1.1.1
```

#### Understanding results

The output format of dpinger is:<br>
<code>latency_avg</code> <code>latency_stddev</code> <code>loss_pct</code>

As long as the internet is connected, it should look something like this:

{% include gallery id="gallery_dpingerconnected" class="full nomargin" %}

When the internet suddenly disconnects, it will look like this:

{% include gallery id="gallery_dpingerdisconnected" class="full nomargin" %}

I'm not sure why it shows packet loss as <code>0</code>. I've also seen packet loss at <code>100</code> once, but since no packets go through, the latencies can't be calculated and thus it shows the latencies as <code>0</code>.

## Measuring the OVPN Static IPv4 Problem

1. Download and install the OVPN App as instructed [here](https://www.ovpn.com/en/guides/windows).
2. Open the OVPN App.
3. The problem only occurs with the Public IPv4 Addon, so choose "Public IPv4":
   [![Select OVPN Public IPv4 Addon](/assets/images/how-to-reproduce-ovpn-problem/ovpn-public-ipv4.png)](/assets/images/how-to-reproduce-ovpn-problem/ovpn-public-ipv4.png)
4. Click Connect:
   [![Click OVPN Connect](/assets/images/how-to-reproduce-ovpn-problem/ovpn-connect.png)](/assets/images/how-to-reproduce-ovpn-problem/ovpn-connect.png)
5. Run dpinger:
   ```bash
   sudo ./dpinger -f -t 2s 1.1.1.1
   ```
6. Wait at least 10min for the VPN to disconnect and watch the output. When a disconnection occurs, it will look like this:
   [![Click OVPN Connect](/assets/images/how-to-reproduce-ovpn-problem/console-ovpn-disconnected.png)](/assets/images/how-to-reproduce-ovpn-problem/console-ovpn-disconnected.png)
