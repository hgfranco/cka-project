# Henry's CKA Project

## Goal

Prepare for the CKA and build practical Kubernetes understanding by turning a lab copy of **whatishenrylisteningto.com** into a small microservice application covering every exam domain. Use the official CNCF curriculum, KodeKloud lessons/labs, and custom troubleshooting exercises together.

The live site stays on Render. This repository starts with planning only; application changes, AWS provisioning, DNS changes, and migration are future work.

## Seven-step plan

The agreed path from the preparation conversation, organized into seven milestones:

1. **Plan the lab:** review the existing app, define service boundaries, and choose an AWS budget and lab access approach.
2. **Containerize the app:** prepare web, API, and Spotify worker containers for a separate lab copy.
3. **Build Kubernetes:** create one EC2 control-plane node and two workers using kubeadm, containerd, kubelet, kubectl, and a CNI.
4. **Deploy and connect services:** add Deployments, Services, ConfigMaps, Secrets, probes, and Ingress/Gateway routing for the lab.
5. **Practice all domains:** add persistent storage, scheduling, autoscaling, RBAC, Helm/Kustomize, CRDs/operators, upgrades, and etcd backup/restore exercises.
6. **Troubleshoot and prepare for the exam:** deliberately break the disposable lab, repair it, repeat weak areas, and complete timed practice alongside the October 19–22, 2026 course. Target the exam shortly afterward; date TBD.
7. **Compare with EKS:** after CKA preparation, deploy the lab application to EKS and compare managed versus self-managed operations. Any production migration is a separate decision.

## CKA domains

Verified October 3, 2026 against the [official CNCF curriculum](https://github.com/cncf/curriculum/blob/master/cka/README.md). Use the [Linux Foundation CKA page](https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka/) to recheck exam details before booking.

| Domain | Weight | Project practice |
|---|---:|---|
| Cluster Architecture, Installation & Configuration | 25% | kubeadm, RBAC, upgrades, etcd recovery, Helm/Kustomize, CRDs/operators, CNI/CSI/CRI |
| Workloads & Scheduling | 15% | Deployments, configuration, probes, resources, autoscaling, affinity, taints/tolerations |
| Services & Networking | 20% | Services, DNS, Ingress, Gateway API, NetworkPolicy |
| Storage | 10% | StorageClasses, PVs/PVCs, persistence and binding failures |
| Troubleshooting | 30% | Application, networking, node and control-plane failures; logs, events and kubelet diagnostics |

## Proposed architecture

Self-managed learning cluster first; EKS later. Agreed service boundaries: Web displays the site, API serves listening status and metrics, and Worker polls Spotify and collects artist-origin data. PostgreSQL runs inside Kubernetes with persistent storage.

```text
AWS EC2 lab (kubeadm + containerd + CNI)
├── k8s-control: API server, scheduler, controller manager, etcd
├── k8s-worker-01: kubelet + application workloads
└── k8s-worker-02: kubelet + application workloads

Lab traffic → Ingress / Gateway
              ├── frontend Service → web Deployment (UI)
              └── API Service → API Deployment (current track/history)
                                  ↕
                           PostgreSQL (PVC-backed storage)
                                  ↑
                           Spotify worker Deployment ← Spotify API
```

The worker polls Spotify and updates shared data; the API serves it to the UI. Explore `cka.whatishenrylisteningto.com` as a future lab hostname. No DNS or live Render changes are part of this setup.

## Infrastructure and cost control

Use Terraform to provision AWS infrastructure. Kubernetes manifests will describe the application services and PostgreSQL. The lab must support shutdown and restart while preserving database data, Spotify token state, and cluster state.

Recommended daily pause approach: gracefully stop workloads and then stop the EBS-backed EC2 nodes; restart the existing nodes for the next session. EBS storage remains billable while compute is stopped. Keep persistent data on EBS-backed volumes and add backups plus a verified recovery exercise. A full Terraform teardown is a separate operation: it can delete managed storage and must not be the routine pause command. Exact storage, backup, and pause/resume implementation remains pending.

References: [AWS stop/start behavior](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/how-ec2-instance-stop-start-works.html), [Terraform resource destruction](https://developer.hashicorp.com/terraform/language/resources/destroy).

## Progress / status

**Updated:** October 3, 2026  
**Current phase:** Step 1 — planning  
**Completed:** Initial roadmap; app source review; Web/API/Worker split; PostgreSQL in Kubernetes decision; Terraform requirement captured  
**Next:** Set the AWS lab budget, one decision at a time  
**Decisions pending:** Budget, lab access, CNI, storage provisioner, routing controller, pause/resume and backup implementation  
**KodeKloud section:** TBD  
**Exam date:** TBD

| Step | Status | Evidence / notes |
|---|---|---|
| 1. Plan the lab | In progress | App reviewed; service split and database agreed; budget pending |
| 2. Containerize | Not started | |
| 3. Build cluster | Not started | |
| 4. Deploy and connect | Not started | |
| 5. Cover all domains | Not started | |
| 6. Troubleshoot and take CKA | Not started | Course: October 19–22, 2026 |
| 7. Compare with EKS | Not started | After CKA preparation |

After each session, update the phase, next actions, and evidence above. Record weak areas here: container logs (`--previous`, `-c`), PV vs. PVC, scheduling, ServiceAccounts, DaemonSets, probes, and scaling; mark improvements based on completed labs.
