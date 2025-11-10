---
layout: my-post
title: "Building Self PC"
date: 2025-05-29 00:00:00 +0000
categories: hardware pc
page_name: building-self-pc-en
lang: en
---

This is a record of building my first custom PC.  
Since I'm a beginner, there might be some misunderstandings or better ways to choose parts—thank you in advance for your understanding.

## Table of Contents
- [PC Budget](#pc-budget)
- [PC Usage](#pc-usage)
- [Selected Parts](#selected-parts)
- [Actual Costs](#actual-costs)
- [Assembly Tips](#assembly-tips)
- [Benchmark](#benchmark)
- [Final Thoughts](#final-thoughts)

## PC Budget
My target budget was around ¥200,000 (approx. $1,300 USD).

## PC Usage
I aimed to build a PC for the following purposes:
- Comfortable gameplay of modern 3D games  
  (I imagined that I could still play heavier titles by lowering the graphics settings.)
- Development of 3D games using Unity or Unreal Engine

## Selected Parts
I referred to [The Rules of Custom PC Building! 2024](https://bookplus.nikkei.com/atcl/catalog/23/12/07/01160/) and mainly used [kakaku.com](https://kakaku.com/) for price comparisons.  
For the parts I wasn't sure about, I consulted a staff member at a computer specialty store.  
I was worried whether the parts I selected would work together, but the staff kindly gave it a quick check.

### CPU  
#### [Intel Core i5-14600KF](https://www.intel.co.jp/content/www/jp/ja/products/sku/236778/intel-core-i5-processor-14600kf-24m-cache-up-to-5-30-ghz/specifications.html)

I checked the recommended system requirements for [Cyberpunk 2077](https://store.steampowered.com/app/1091500/2077/?l=japanese), which I heard was quite demanding, and chose accordingly (I later learned it's not that heavy anymore).  

I chose Intel Core, which I had researched in advance.  
Though AMD's Ryzen lineup seems future-proof thanks to socket compatibility with the next generation, Intel was my choice for this build.

Cyberpunk's requirements listed the Core i7-12700, but considering price and performance, I opted for the newer Core i5-14600KF.  
I referenced [Dospara's CPU comparison chart](https://www.dospara.co.jp/5info/cts_lp_intel_cpu.html).  
I chose the “F" variant because I planned to use a dedicated GPU and didn't need integrated graphics.

### Motherboard  
#### [msi B760 GAMING PLUS WIFI](https://jp.msi.com/Motherboard/B760-GAMING-PLUS-WIFI)

I chose the motherboard after selecting the GPU, memory, and SSD, making sure it was compatible with all of them.  
Here are the specs I focused on:

| Item | Spec | Notes |
|------|------|-------|
| CPU Socket | LGA1700 | Determined by the CPU |
| Chipset | B760 | Narrowed down by CPU choice. I don't plan on overclocking, so I chose a non-overclockable chipset |
| Memory Type | DIMM DDR5 | Determined by selected memory |
| GPU Interface | PCIe 4.0 ×16 or PCIe 5.0 ×16 | Based on GPU selection |
| M.2 SSD Interface | PCIe 4.0 ×4 | Based on SSD |
| M.2 Size | 2280 | Based on SSD |
| Wi-Fi | Wi-Fi 6 or higher | I just made sure it had Wi-Fi |
| Form Factor | ATX | Chosen as it's the standard |

Although I selected the motherboard after the other parts, it's best to double-check the motherboard's compatibility page.  
In this case, you can check [here](https://jp.msi.com/Motherboard/B760-GAMING-PLUS-WIFI/support#cpu).

### Memory  
#### [Crucial Pro 32GB Kit (16GBx2) DDR5-5600 UDIMM](https://www.crucial.jp/memory/ddr5/cp2k16g56c46u5)

Since the CPU socket is LGA1700, I could choose between DDR4 and DDR5.  
The price difference wasn't large, so I went with the newer DDR5.

To take advantage of dual-channel, I went with 16GB × 2 (32GB total).  
I selected 5600 MT/s as it was the fastest speed available within my budget.

### SSD  
#### [KIOXIA EXCERIA PRO 1TB](https://www.kioxia.com/ja-jp/personal/ssd/exceria-pro.html)

I picked a 1TB M.2 SSD based on price.

Depending on the motherboard, you might need an SSD with a heatsink to avoid thermal throttling.  
The motherboard I selected came with a built-in heatsink.

### Graphics Card  
#### [PNY GeForce RTX 4060 Ti 8GB VERTO Dual Fan](https://www.pny.com/pny-geforce-rtx-4060-ti-8gb-verto-dual-fan)

I chose the GPU based on benchmark results.  
The GeForce RTX 4060 Ti seemed capable of handling Cyberpunk 2077 with ray tracing at around 30fps.

I picked the product based on price among available RTX 4060 Ti models.

### Power Supply Unit  
#### [msi MAG A750GL PCIE5](https://jp.msi.com/Power-Supply/MAG-A750GL-PCIE5)

I chose a 750W power supply.  
Although I couldn't find reliable power consumption info for the GPU online, I visited a PC store where they listed the recommended PSU wattage next to the GPU.  
It recommended 650W, but since 650W models aren't very common (possibly a misconception), I went with 750W.

Also, make sure the connectors match your motherboard and GPU.  
Some motherboards require two EPS12V cables for CPU power.  
Additionally, your PC case may require power for case fans, so be sure to check that too.

I chose a modular PSU, so I can store any unused cables.

The rest of my decision was based on price.  
I didn't pay much attention to power efficiency, but the model I selected is 80 PLUS GOLD certified.

### CPU Cooler  
#### [SCYTHE MUGEN6](https://www.scythe.co.jp/category/product/cpu-cooler/air-cooling/high-end/scmg-6000/)

The CPU cooler was the part I understood the least, so I asked a store staff to choose one for me.  
I requested an air cooler to save on cost.  
If you use a Core i9 (or possibly i7), you may need to go with a liquid cooler instead.

### PC Case  
#### [Antec AX90](https://www.antec.com/product/case/ax90)

Since I hadn't chosen a CPU cooler yet, I also picked the PC case with the help of store staff.

I selected an ATX form factor case that could fit the CPU cooler and graphics card within my budget.

By the way, I didn't check thoroughly at the store, so I assumed the case didn't have LED fans—but it actually did. (They probably won't light up unless connected.)  
Also, I thought it was a non-transparent case, but it turned out to have a half-glass side panel.  
But I think it looks cool, so no complaints.

### Operating System  
#### [Windows 11 Home](https://www.microsoft.com/ja-jp/d/windows-11-home/dg7gmgf0krt0)

I went with Windows 11 instead of Windows 10, as I don't need 32-bit support.

## Actual Costs  
Below are the approximate actual costs:

| Product | Price |
|--------|--------|
| Intel Core i5-14600KF | ¥48,000 |
| msi B760 GAMING PLUS WIFI | ¥19,000 |
| Crucial Pro 32GB Kit (16GBx2) DDR5-5600 UDIMM | ¥16,000 |
| KIOXIA EXCERIA PRO 1TB | ¥13,000 |
| PNY GeForce RTX 4060 Ti 8GB VERTO Dual Fan | ¥58,000 |
| msi MAG A750GL PCIE5 | ¥14,000 |
| SCYTHE MUGEN6 | ¥6,700 |
| Antec AX90 | ¥10,000 |
| Windows 11 Home | ¥18,000 |
| **Total** | **≈ ¥202,700** |

## Assembly Tips  
I referred to [The Rules of Custom PC Building! 2024](https://bookplus.nikkei.com/atcl/catalog/23/12/07/01160/) and a booklet I received from the PC store.  
Generally, following the manuals included with the parts should be enough.

**These tips depend on the parts I selected**, but here are some issues I ran into during assembly:

- **Insert the M.2 SSD at an angle**  
  My motherboard required inserting the SSD at a 30° angle.  
  I initially tried inserting it parallel to the board and struggled for a while.

- **Some case brackets must be physically broken off**  
  I couldn't believe I had to "break" my PC case, but some cases require you to snap off expansion slot covers—just go for it!

- **Don't forget the I/O shield**  
  Before mounting the motherboard, install the I/O shield that comes with it.  
  I forgot and had to remove the board to attach it.  
  Also, the shield's tabs can get caught in the LAN port or break off.  
  It might be easier to choose a motherboard with a pre-installed I/O shield.

- **Install network drivers early**  
  Windows 11 requires a network connection during installation, so it helps to install network drivers first.  
  I forgot and had to use the following workaround to install Windows 11 offline:  
  1. Press `Shift + F10` to open Command Prompt.  
  2. Run `cd oobe`  
  3. Run `BypassNRO.cmd`

## Benchmark  
Using [3DMark](https://store.steampowered.com/app/223850/3DMark/), I got a score of around **2900** on the Steel Nomad test, with frame rates just below 30 FPS.

## Final Thoughts  
This was my first time building a PC, and I found that **choosing the parts** was the most difficult and time-consuming part.  
At first, I considered selecting parts that would allow for future upgrades, but since components degrade over time, I ended up selecting parts based on the current-generation CPU.

The staff at the PC specialty store I visited were incredibly helpful, and I'm really grateful for their advice.

As for the assembly, manuals, books, and videos made the process pretty smooth.  
It took me a full day, but if you're good at working with your hands or used to wiring, you could probably do it much faster.

The happiest moment was when the PC successfully booted up.  
I'm clumsy, so I had several moments of panic thinking I might have broken something.  
I'm already looking forward to upgrading this machine someday!