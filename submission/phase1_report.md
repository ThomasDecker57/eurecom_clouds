# Phase 1 — Comparison Report

Copy this file to `submission/phase1_report.md`, fill in every answer slot, and commit it:

```bash
cp phase-1-echo-bot/report_template.md submission/phase1_report.md
```

**How to fill it in.** Each answer sits between a pair of `<!--answer:...-->` markers. Replace
the `TODO` line with your answer — **leave the markers themselves untouched**, they are how the
grader finds your answers. Anything you write outside the markers is ignored, so add extra
prose, tables or images freely.

Numbers must **match `submission/phase1_report.json`** exactly — that file is produced by
`measure.py` and is what your figures are checked against. Copy them across; do not retype from
memory.

---

## 1. Setup facts

**Your image size**, in MB, from `gcloud artifacts docker images list` (number only, e.g. `142`):

<!--answer:image_size_mb--> 
50
<!--/answer-->

**The custom VPC and subnet you created** (names only):

<!--answer:vpc_name-->
echo-net
<!--/answer-->

<!--answer:subnet_name-->
echo-subnet
<!--/answer-->

---

## 2. Measurements

Copy these from `submission/phase1_report.json`. Cold is `cold_ms`; warm median is
`stats.median`. Numbers only, in milliseconds (e.g. `812.4`).

| Platform | Cold (ms) | Warm median (ms) |
|---|---|---|
| IaaS VM | <!--answer:vm_cold_ms-->236.9<!--/answer--> | <!--answer:vm_warm_median_ms-->209.4<!--/answer--> |
| Cloud Run | <!--answer:cloudrun_cold_ms-->2793.6<!--/answer--> | <!--answer:cloudrun_warm_median_ms-->634.2<!--/answer--> |
| Cloud Function | <!--answer:function_cold_ms-->1523.0<!--/answer--> | <!--answer:function_warm_median_ms-->637.9<!--/answer--> |

Include your histogram or plot of the warm distribution below (an image, or the ASCII chart
`measure.py` prints). This is read by a human, not the grader:

<!--answer:latency_plot-->
Phase 1 — deployment measurements
------------------------------------------------------------
[OK  ] vm        cold=  236.9ms  warm_median=  209.4ms
            #############
[OK  ] cloudrun  cold= 2793.6ms  warm_median=  634.2ms
            ########################################
[OK  ] function  cold= 1523.0ms  warm_median=  637.9ms
            ########################################
------------------------------------------------------------
<!--/answer-->

---

## 3. Per-platform analysis

**Scaling behaviour.** Imagine requests arriving one at a time, then ten at once, then a
hundred. For each of the three platforms, answer concretely:

- Does anything get created automatically to absorb the load — and if so, *what* exactly?
  (One platform here creates nothing at all.)
- What is the hard limit, and where does it come from — a flag you passed yourself, or the
  size of the machine you picked?

(~80 words. Your `--machine-type=e2-micro` and `--max-instances=3` are both concrete answers
to the second question.)

<!--answer:scaling_behaviour-->
For the VM (IaaS), nothing scales automatically: it is locked to 1 instance because it is a single, manually managed e2-micro machine. On the contrary, Cloud Run and Cloud Functions scale automatically to absorb the load (from 0 to N instances). The hard limit for these two services is a maximum of 3 instances, which comes directly from the explicit --max-instances=3 parameter passed in the command line during the deployment.
<!--/answer-->

**Deployment effort.** What did you have to build, configure and maintain to get each of the
three running? (~100 words)

<!--answer:deployment_effort-->
The VM required maximum effort: it involved configuring the network infrastructure (custom VPC, subnet), creating firewall rules, connecting via SSH, and running Docker manually. Cloud Run is much simpler, requiring only a Dockerfile, building the image (Artifact Registry), and a single deployment command. Cloud Functions requires the least effort: no image management is needed; you only provide the source code, and the platform automatically containerizes it via Cloud Build during deployment.
<!--/answer-->

---

## 4. Why do two of your three numbers look alike?

With Cloud Run and your Function both deployed, run `gcloud run services list` and paste the
**complete output** here, unedited:

<!--answer:run_services_list-->
✔
SERVICE: echo
REGION: us-central1
URL: https://echo-363619878839.us-central1.run.app
LAST DEPLOYED BY: service-363619878839@gcf-admin-robot.iam.gserviceaccount.com
LAST DEPLOYED AT: 2026-10-26T21:06:38.269257Z

✔
SERVICE: echo-bot
REGION: us-central1
URL: https://echo-bot-363619878839.us-central1.run.app
LAST DEPLOYED BY: thomas.decker@eurecom.fr
LAST DEPLOYED AT: 2026-10-26T20:57:51.349252Z
<!--/answer-->

Now explain what that output tells you about how your Function is actually being run, why its
cold start resembles Cloud Run's so closely, and what — if anything — genuinely differs between
the two deployment paths. (~120 words)

<!--answer:convergence_explanation-->
The command output reveals that Cloud Functions (2nd generation) actually deploys a Cloud Run service, clearly shown by the echo service deployed by the automated gcf-admin-robot sitting right alongside the echo-bot service deployed by my user account. This explains why their cold start and warm request timings closely resemble each other: the underlying execution infrastructure is completely identical. The only genuine difference between the two deployment paths is the developer workflow. For Cloud Run, we must write the Dockerfile, build the image, and push it to Artifact Registry ourselves. For Cloud Functions, we only provide the source code, and Google handles the containerization in the background using Buildpacks before automatically deploying it to Cloud Run.
<!--/answer-->
