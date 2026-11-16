# H100 rental: what a GPU-hour actually costs, which Vultr plan fits, and how many cards your model needs (full price table)

H100 rental prices currently run from $1.30 to $12.29 per GPU-hour, with a median around $3.90. Public listings at both ends of that range are selling the same silicon, so the spread tells you more about the seller than about the chip.

Vultr sits near the bottom of the range. Its 8-GPU H100 bare metal plan lists at **$23.92 per hour for the whole machine**, which comes out to **$2.99 per GPU-hour**, and that figure is the same in every Vultr location. Below is the full Vultr GPU price table, an explanation of why $2.99 and $1.30 are not the same product, and a rule for sizing your rental to the model you actually want to run.

> **Quick pick:** For multi-GPU training, fine-tuning runs that need NVLink inside a node, or anything you plan to keep up for days, Vultr's `vbm-112c-2048gb-8-h100-gpu` gives you eight H100 SXM cards on bare metal at **$23.92/hr ($2.99/GPU-hr)**. It is sold as a whole machine, so this is not the plan for a one-card experiment. 👉 [See Vultr's current H100 bare metal pricing](https://www.vultr.com/pricing/?ref=9738262-9J#bare-metal)

## Every GPU plan Vultr lists right now

Vultr publishes nine GPU bare metal plans. Prices below are on-demand, per whole machine, and identical across regions.

| Plan | GPU | Cards | Total $/hr | Per GPU $/hr | CPU / RAM | Order |
| --- | --- | --- | --- | --- | --- | --- |
| `vbm-64c-2048gb-8-l40-gpu` | NVIDIA L40S | 8 | $13.37 | $1.67 | 64 cores / 2048 GB | [View plan](https://www.vultr.com/pricing/?ref=9738262-9J#bare-metal) |
| `vbm-72c-480gb-gh200-gpu` | NVIDIA GH200 | 1 | $1.99 | $1.99 | 72 cores / 480 GB | [View plan](https://www.vultr.com/pricing/?ref=9738262-9J#bare-metal) |
| `vbm-48c-1024gb-4-a100-gpu` | NVIDIA A100 | 4 | $9.59 | $2.40 | 48 cores / 1024 GB | [View plan](https://www.vultr.com/pricing/?ref=9738262-9J#bare-metal) |
| `vbm-256c-3072gb-8-mi355x-gpu` | AMD MI355X | 8 | $20.72 | $2.59 | 256 cores / 3072 GB | [View plan](https://www.vultr.com/pricing/?ref=9738262-9J#bare-metal) |
| `vbm-112c-2048gb-8-a100-gpu` | NVIDIA A100 SXM | 8 | $22.40 | $2.80 | 112 cores / 2048 GB | [View plan](https://www.vultr.com/pricing/?ref=9738262-9J#bare-metal) |
| `vbm-112c-2048gb-8-h100-gpu` | NVIDIA H100 SXM | 8 | $23.92 | $2.99 | 112 cores / 2048 GB | [View plan](https://www.vultr.com/pricing/?ref=9738262-9J#bare-metal) |
| `vbm-256c-2048gb-8-mi300x-gpu` | AMD MI300X | 8 | $31.92 | $3.99 | 256 cores / 2048 GB | [View plan](https://www.vultr.com/pricing/?ref=9738262-9J#bare-metal) |
| `vbm-256c-3072gb-8-mi325x-gpu` | AMD MI325X | 8 | $36.92 | $4.62 | 256 cores / 3072 GB | [View plan](https://www.vultr.com/pricing/?ref=9738262-9J#bare-metal) |
| `vbm-256c-3072gb-8-b200-gpu` | NVIDIA B200 | 8 | $68.00 | $8.50 | 256 cores / 3072 GB | [View plan](https://www.vultr.com/pricing/?ref=9738262-9J#bare-metal) |

Two things the table makes obvious. The H100 is priced at almost exactly the same point as the 8-card A100 SXM ($22.40 vs $23.92 per hour), so if your workload can use FP8 and the Transformer Engine, the newer card costs 7% more per hour for roughly double the low-precision throughput. And the per-GPU figure falls as you buy bigger: the 4-card A100 is $2.40/GPU-hr while the 8-card A100 SXM is $2.80. The four-card box is genuinely the cheaper per-card deal, which is unusual and worth knowing if a 320GB pool of VRAM is enough for you.

## What you actually get for $2.99 per GPU-hour

Vultr's H100 listing is the SXM part, not the PCIe one. That matters more than any other hardware detail on this page.

The H100 SXM carries 80GB of HBM3 at 3.35 TB/s and connects to its neighbours over NVLink at 900 GB/s inside the node. The H100 PCIe carries 80GB of HBM2e at roughly 2 TB/s, draws 350W instead of 700W, and links cards in pairs through a 600 GB/s bridge. For single-GPU inference the two are close. For tensor parallelism, pipeline parallelism, or anything that moves activations between cards on every step, SXM is the one you want, and a lot of the $1.30–$1.99 listings floating around the market are PCIe.

Specs you can hold Vultr to, since they are the reference numbers for the SXM card: 16,896 CUDA cores, 528 fourth-generation Tensor Cores, 1,979 TFLOPS of dense FP8 (3,958 with sparsity), 989 TFLOPS dense FP16/BF16, 67 TFLOPS FP32, 34 TFLOPS FP64, up to 700W configurable TDP, and MIG partitioning into as many as seven 10GB instances.

Vultr quotes 34 data center locations across six continents for its H100 clusters, and the platform holds ISO and SOC 2+ certifications. Tenancy is single-tenant bare metal, and the multi-node path runs through Vultr Clusters over NVIDIA InfiniBand, orchestrated with Slurm or Kubernetes. 👉 [Check the full 8-GPU H100 configuration](https://www.vultr.com/products/cloud-gpu/nvidia-h100?ref=9738262-9J)

The contractual detail worth flagging: Vultr states that clusters need **no reservations or special requests** and can be deployed at will. That is the opposite of how most providers handle multi-node H100 capacity, where anything beyond 90 days goes to a sales call.

## Three pricing traps that make H100 quotes incomparable

### Dense versus sparse TFLOPS

NVIDIA publishes two numbers for the same operation. The H100's FP8 Tensor Core figure is 1,979 TFLOPS dense and 3,958 sparse. Some provider pages quote the sparse number as if it were the headline spec. Both are correct, but a comparison that puts a sparse 3,958 next to a dense 1,979 makes one card look twice as fast as an identical card. Normalise to dense before you compare anything.

### SXM, PCIe, and NVL are three different products

All three carry about 80GB and all three get called "H100." The NVL variant actually has 94GB at 3.9 TB/s. The PCIe variant runs at 1,513 TFLOPS FP8 versus the SXM card's 1,979. A $1.40/GPU-hr PCIe listing and a $2.99/GPU-hr SXM listing are not competing on price for the same job, so don't treat the gap as a discount.

### Whole machine versus per-GPU

This is where Vultr's number needs reading carefully. The $2.99 is arithmetic: $23.92 divided by eight cards. You cannot rent one of those cards for $2.99. The minimum purchase is the eight-GPU box. If you were planning a two-hour experiment on a single H100, the entry ticket here is $23.92/hr for capacity you will mostly leave idle, and a per-second marketplace pod at $1.30–$2.00 would cost you less than a dollar for the same two hours.

## How many H100s does your model actually need?

One 80GB card holds less than the 80GB suggests, because weights share the memory with activations and the KV cache. Budget roughly 20% headroom:

| Precision | Largest model that fits on one 80GB H100 |
| --- | --- |
| FP16 / BF16 | ~13B parameters |
| 8-bit | ~34B parameters |
| 4-bit | ~70B parameters |

A 70B model at FP16 needs somewhere around five to six cards once you account for activations during training, which puts you into multi-GPU territory and makes NVLink relevant. A 7B or 13B fine-tune fits on one card, but Vultr's smallest H100 unit is eight. That mismatch is the single biggest reason to look elsewhere for small jobs.

For scale context: Llama 3.1 405B was trained on more than 16,000 H100s. Nobody rents that on a credit card, but it explains why 8-card nodes and InfiniBand fabric are the unit of sale rather than single GPUs.

## The monthly bill, and where renting stops making sense

Multiply the hourly rate by 730 hours and the numbers get uncomfortable in a useful way.

- Vultr H100, one card's share: **$2.99 × 730 ≈ $2,183/month**
- Vultr H100, full 8-card node: **$23.92 × 730 ≈ $17,462/month**
- Buying one H100 outright: **$25,000–$30,000 per card**

At $2.99 per GPU-hour, $25,000 of hardware equals about 8,361 GPU-hours, or roughly 348 days of continuous running. Buy at $30,000 and the crossover moves to about 418 days. That comparison ignores power, cooling, rack space, replacement parts, and the fact that a card you own depreciates whether or not it is busy. It also ignores that you probably will not run at 100% utilisation, which pushes the real crossover further out.

So the decision rule is roughly: anything under a year of intermittent use is a rental, and even a year-long continuous run is close enough that the operational burden of owning should decide it.

## How Vultr compares with other H100 options

Numbers below are verified public list prices from each provider, per GPU-hour, on-demand unless noted. They are here to help you place Vultr, not to send you anywhere.

| Provider | H100 variant | On-demand $/GPU-hr | Billing / notes |
| --- | --- | --- | --- |
| Vultr | H100 SXM, 8-card bare metal | $2.99 | Hourly, whole machine, same price in every region |
| Akash | H100 80GB | $2.73 | Per GPU-hour, no egress fees, reserved clusters quoted |
| Together AI | H100 SXM | $3.99 | Reserved 7–30 days $3.69, 31–90 days $3.45, 91+ days by quote |
| Runpod | H100 PCIe | $1.99 Community / $2.39 Secure | Per-second billing |
| Edgevana | H100 PCIe 80GB | $2.56 | $1,870/month, bare metal |
| Lium | H100 80GB HBM3 | $1.30–$2.90 | Per-second, no minimum, marketplace pricing |

The $1.30 at the bottom of that table is a live marketplace ask from an individual provider, and marketplace asks move with supply and demand. It is not a rate you can book on demand next month, and it is not an apples-to-apples substitute for an 8-card SXM node with InfiniBand attached. Vultr's $2.99, by contrast, is a fixed list price that does not change by region and does not require a reservation.

One more Vultr mechanism: the bare metal GPU plans support preemptible deployment at a separate, lower hourly rate published through Vultr's plan API. Vultr's public pages do not state when a preemptible server gets reclaimed, so treat that discount as suitable for checkpointed, restartable work only.

## Who should not rent H100s on Vultr

- **You want one card for an afternoon.** Eight-card minimum, $23.92/hr. Use a per-second marketplace instead.
- **You want the cheapest possible H100 and PCIe is fine.** Vultr's H100 is SXM. If your workload is single-GPU inference, the PCIe card at $1.40–$2.39 elsewhere does the same job for less.
- **You need a reserved multi-month contract with an SLA and a signature.** Vultr sells capacity at will, without reservations, which is great for flexibility and less useful if procurement needs a 12-month committed document.
- **You need a card bigger than 80GB for long-context serving.** The H200 at 141GB or the B200 at 180GB is the right shape, and Vultr lists B200 at $8.50/GPU-hr if the budget allows.

## Before you place the order

Confirm four things, because each one changes the bill:

1. **Card count.** The H100 SKU is eight GPUs. Check whether your framework's parallelism strategy actually uses all eight, or whether you are paying for idle silicon.
2. **Region.** Vultr prices the plan the same everywhere, but latency and data residency do not follow the price. With 34 locations, pick the one nearest your data, not the one nearest your team.
3. **Storage and transfer.** The hourly GPU rate does not include block storage, object storage, or data transfer. If your dataset is measured in terabytes, price that separately before you commit.
4. **Preemptible versus on-demand.** If you deploy preemptible for the lower rate, make sure your job checkpoints often enough to survive a reclaim, since Vultr does not publish the reclaim policy.

Final rule of thumb. Eight H100 SXM cards on bare metal with InfiniBand and no reservation paperwork, for a training or fine-tuning run that lasts days rather than minutes: Vultr at $2.99/GPU-hr is the straightforward pick, and the price does not move when you switch region. One card, a few hours, a notebook and a 13B model: rent a single PCIe card by the second somewhere else and come back when the job grows into a node.

👉 [Compare all Vultr GPU plans and current hourly rates](https://www.vultr.com/pricing/?ref=9738262-9J#bare-metal)
