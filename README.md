# Vultr vs Lambda Labs: which is cheaper per GPU hour, who actually needs InfiniBand, and what the listed rates leave out (with a per-GPU price table)

Lambda lists an 8-GPU H100 SXM node at **$3.99 per GPU-hour**. Vultr's listed rate for the same node is **$2.30**, though Vultr's public plan API quotes **$2.99 on-demand** and treats $2.30 as the preemptible figure. Either way, Hopper is cheaper on Vultr. Blackwell flips: Lambda's B200 is $6.69 per GPU-hour, Vultr's 8-GPU B200 node is $3.20 preemptible but $8.50 on-demand, which puts Lambda ahead at list.

Which number you actually pay depends on the billing type attached to the rate, and that is the part most Vultr vs Lambda Labs write-ups skip. Below are the same-GPU prices with the billing type spelled out, three cost lines that never appear in a per-hour figure (interconnect, egress, block storage), and a rule for choosing based on the shape of your job.

> **Quick pick:** a job that fits on one node — inference, a fine-tune that already fits in 80 GB, rendering, a Windows workstation — starts cheapest on Vultr, where the L40S 48 GB is $1.67 per GPU-hour with no commitment. 👉 [See Vultr's current GPU rates](https://www.vultr.com/pricing/?ref=9738262-9J)
>
> A job that needs 16 or more GPUs on a single fabric, with Slurm or Kubernetes delivered on top, is Lambda's 1-Click Clusters. No Vultr rate competes with that, because Vultr does not sell it.

All prices below are list rates per GPU-hour in USD as of November 2026, taken from provider pricing APIs and independent price trackers. Vultr runs 33 data center regions across six continents and sells both NVIDIA and AMD cards; Lambda sells NVIDIA only.

## Vultr's GPU lineup and what each card is for

Vultr rents GPUs as whole cards in a VM, and on the older A16 and A40 parts, as slices of a card shared between machines. That slice option is why Vultr shows up in conversations about inference endpoints and virtual desktops, where a full H100 is wasted money.

| GPU | VRAM | Node shape | Per GPU-hr | Fits |
| --- | --- | --- | --- | --- |
| NVIDIA L40S | 48 GB | 1 to 8 GPUs | $1.67 | Vision inference, rendering, LoRA fine-tunes |
| NVIDIA A40 | 48 GB | 1 to 4 GPUs | $1.71 | Virtual desktops, transcoding, light inference |
| NVIDIA A100 PCIe | 80 GB | 1 to 8 GPUs | $2.40 | LLM inference, mid-size training |
| NVIDIA A100 SXM | 80 GB | 8-GPU node | $1.49 listed / $2.80 API | Dense training where NVLink matters |
| NVIDIA H100 SXM | 80 GB | 8-GPU node | $2.30 preemptible / $2.99 on-demand | Large-model training, high-throughput serving |
| NVIDIA GH200 | 96 GB | Single GPU | $1.99 | Memory-bound inference on one big HBM pool |
| NVIDIA B200 | 180 GB | 8-GPU node | $3.20 preemptible / $8.50 on-demand | Frontier-scale training |
| AMD MI300X / MI325X / MI355X | 192 to 288 GB | 8-GPU nodes | $1.85 to $2.59 listed | Model fits AMD's software stack; NVIDIA alternative |

Two things to know before reading that table as gospel. First, the H100, B200 and A100 SXM rows carry two prices because Vultr's public plan API and its marketing pages disagree: the API lists the 8-GPU H100 node at $23.92/hour ($2.99 per GPU) with `deploy_ondemand` off and a preemptible rate of $18.40/hour ($2.30 per GPU), and the B200 node at $68/hour ($8.50 per GPU) against a $3.20 preemptible rate. Treat the lower figures as "starting at" prices.

Second, Vultr's Cloud GPU documentation has long listed only the A16, A40, A100 and L40S. The H100, B200, GH200 and AMD cards appear in the plan API and on the pricing page as 8-GPU HGX configurations. What's in stock varies by region, so check the deploy screen rather than the docs page. 👉 [Compare the current per-GPU rates](https://www.vultr.com/pricing/?ref=9738262-9J)

Vultr's own comparison page claims up to 20% savings over Lambda. That number is a vendor claim; the table above is a better guide for the specific card you need.

## Same GPU, side by side

| GPU | Lambda | Vultr | Lambda premium at listed rates |
| --- | --- | --- | --- |
| H100 SXM 80 GB, 8-GPU | $3.99 | $2.30 preemptible / $2.99 on-demand | +73.5% |
| B200 180 GB, 8-GPU | $6.69 | $3.20 preemptible / $8.50 on-demand | +109.1% at the low rate; Vultr is 27% dearer on-demand |
| A100 SXM 80 GB, 8-GPU | $2.79 | $1.49 listed / $2.80 API | +87.2% at the low rate; a wash on-demand |
| GH200 96 GB, single | $2.29 | $1.99 | +15.1% |

The percentages come from the lower Vultr figures. Swap in Vultr's on-demand API rates and the H100 gap narrows to about a third, the A100 gap disappears, and the B200 gap reverses.

Watch the A100 comparisons in particular. Several trackers show Lambda at $1.99 against Vultr at $2.40 and call Lambda 17% cheaper. That compares a 40 GB PCIe A100 on Lambda with an 80 GB PCIe A100 on Vultr. Match the memory before you match the price.

Where Lambda is unambiguously cheaper is the small end: a Tesla V100 at $0.79 per GPU-hour, an RTX A6000 at $1.09 and an A10 at $1.29, none of which Vultr lists. If your workload runs happily on 24 GB, Vultr's cheapest card is the L40S at $1.67 per GPU-hour.

## Three differences that move the bill more than the hourly rate

### Multi-node training

Lambda's 1-Click Clusters run from 16 to more than 2,000 B200 or H100 GPUs on NVIDIA Quantum-2 InfiniBand, delivered with managed Kubernetes or Slurm already configured. Superclusters extend that to longer reserved commitments on single-tenant hardware.

Vultr documents its Cloud GPU line as individual VMs. Eight-GPU HGX nodes give you NVLink inside the box; a cross-node InfiniBand fabric with a scheduler on top is not part of the documented product. If your training run needs tensor or pipeline parallelism across nodes, the per-hour price difference is irrelevant, because one of the two options doesn't offer the thing you need. Ask Vultr sales directly if you're planning a multi-node deployment there; the documentation lags the catalogue.

This is also the widest gap in the third-party scoring. On a shared 11-criterion rubric, Lambda scores 82 for networking and storage against Vultr's 50, and 76 for GPU availability against 58. Vultr scores higher on integrations (76 vs 68) and documentation (78 vs 72).

### Egress and storage

Lambda charges nothing for outbound traffic, with no cap. Vultr charges $10 per TB beyond the allowance included with each instance, which runs from 0.5 TB to 15 TB per month depending on the plan. A single-GPU L40S or A100 plan includes 10 TB, so most single-node workloads never see the meter. A team pushing 20 TB a month out of one instance that includes 10 TB pays roughly $100; the same traffic on Lambda costs nothing.

Storage cuts the other way. The closest comparable block-storage plan is $20/month on Lambda and $10/month on Vultr, and Vultr also sells object storage at $18/month. Vultr's catalogue adds managed PostgreSQL at $60/month, Redis at $15 and a load balancer at $10, none of which Lambda offers around its GPUs. Run both lines against your own numbers: a checkpoint-heavy job with light egress usually favors Vultr, a serving job with heavy egress usually favors Lambda.

### Commitment terms

Lambda sells on-demand instances, 1-year and 3-year reserved capacity, and committed clusters. Vultr bills hourly with no minimum commitment, plus preemptible capacity. Hourly billing without a contract is the reason Vultr's headline rates look low, and it's also why you should read them carefully: a preemptible 8-GPU H100 node can be reclaimed mid-run, which is fine for a batch inference sweep and expensive for a three-week training job.

## Which one to pick

- **Inference or a small fine-tune on one GPU.** Vultr. The L40S at $1.67 per GPU-hour, sliced A40s for lighter work, and 33 regions to put the endpoint near your users.
- **A model that needs more than 80 GB on one card.** Vultr's AMD MI300X, MI325X and MI355X nodes go to 288 GB of VRAM, and Lambda sells NVIDIA only.
- **Sixteen GPUs or more on one fabric.** Lambda's 1-Click Clusters. Nothing in Vultr's documented GPU line covers this.
- **You already run Kubernetes, managed databases or object storage on Vultr.** Stay. GPU instances sit in the same account, network and control panel as the rest of your stack, which removes a migration and a second bill.
- **You want the cheapest possible 24 GB card.** Lambda, at $0.79 to $1.29 per GPU-hour. Vultr has no equivalent.
- **You want capacity held for a year or more.** Lambda's reserved terms. Vultr's strength is the opposite: nothing to sign.

## Before you click deploy

Vultr's B200 listing is the clearest case of a rate that needs confirming. The pricing page and the plan API differ by more than 2×, and the low figure is the preemptible one. Check the billing type in the console before you budget a training run around it.

Also confirm three things that pricing pages don't answer: whether the card you want is in stock in your region, how much egress your specific instance includes, and whether the rate quoted is on-demand or preemptible. Lambda has its own capacity caveat: on-demand instances are first come, first served, so popular cards sell out, and anything larger than a single node goes through a sales conversation.

👉 [Start with Vultr's GPU list and pick the node that matches your memory requirement](https://bit.ly/VuLtr)

## FAQ

**Is Vultr cheaper than Lambda?**

Depends on the card and the billing type. On H100 SXM, Vultr is cheaper at both its $2.30 preemptible and $2.99 on-demand rates. On B200, Vultr is cheaper preemptible ($3.20 vs $6.69) and more expensive on-demand ($8.50 vs $6.69). On the 80 GB A100 SXM, Vultr's $1.49 listed rate beats Lambda's $2.79, while Vultr's own API rate of $2.80 makes it a tie. On small cards, Lambda wins: $0.79 for a V100 against a $1.67 floor on Vultr.

**Does Vultr offer H100 and B200 GPUs?**

Yes, as 8-GPU HGX nodes, priced in Vultr's public plan API and on its pricing page, alongside GH200 and AMD MI300X, MI325X and MI355X. Vultr's Cloud GPU documentation still describes only the A16, A40, A100 and L40S, so treat the docs as out of date and the deploy screen as the source of truth.

**Can Vultr run multi-node training?**

Vultr's documented Cloud GPU product is single VMs, including 8-GPU nodes with NVLink inside the chassis. A cross-node InfiniBand fabric with managed Kubernetes or Slurm isn't part of that documentation, which is what Lambda's 1-Click Clusters provide. For anything spanning multiple nodes, get the answer from Vultr in writing before you commit.

**Does Lambda charge for egress?**

No. Outbound traffic is free and uncapped. Vultr charges $10 per TB beyond the allowance bundled with each instance, which ranges from 0.5 TB to 15 TB per month.

**Which is better for a production inference endpoint?**

Vultr, for most teams. Per-hour billing with no commitment, GPUs in 33 regions, sliced A40s for light loads, and your Kubernetes cluster and managed database in the same account. Lambda's advantage is concentrated in the single large node and the multi-node cluster, not in geographically distributed serving.
