# Kubernetes Home Lab

[![GitHub stars](https://img.shields.io/github/stars/kaninda/kubernetes-home-lab?style=social)](https://github.com/kaninda/kubernetes-home-lab)
[![GitHub forks](https://img.shields.io/github/forks/kaninda/kubernetes-home-lab?style=social)](https://github.com/kaninda/kubernetes-home-lab)
[![GitHub last commit](https://img.shields.io/github/last-commit/kaninda/kubernetes-home-lab)](https://github.com/kaninda/kubernetes-home-lab)

Bare-metal Kubernetes cluster built with **kubeadm** on physical hardware.

This documentation describes the full lifecycle of the cluster:
from hardware design to installation and operations.

---

## 🎯 Project Goals

- Understand Kubernetes internals
- Build a production-like bare-metal cluster
- Practice CKA/CKAD concepts
- Maintain a fully reproducible setup

---

## 🏗 Architecture Summary

- Single control plane
- Two worker nodes
- Ubuntu Server 22.04 LTS
- containerd runtime
- Calico CNI
- Dedicated Pod CIDR (10.244.0.0/16)

---

## 📚 Documentation Structure

- **Overview** → Design and topology
- **Installation** → Step-by-step cluster setup
- **Operations** → Validation and reset procedures

---

![overview](diagrams/k8s_overview.svg){ width="40%" }



