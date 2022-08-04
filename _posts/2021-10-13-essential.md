---
layout: post
title: "[논문 요약] Essential Roles of Exploiting Internal Parallelism of Flash Memory State Drives in High-Speed Data Processing"
subtitle: ""
categories: paper
tags: [SSD, internal-parallelism, pattern]
---

## Conference

IEEE Symposium on High-Performance Computer Architecture (HPCA'11)

## Authors

Feng Chen, Rubao Lee, Xiaodong Zhang

## Idea

SSD performance is no longer highly sensitive to access patterns, but rather to other factors, such as data access interferences and physical data layout.

Need to focus on internal parallelism of SSDs.

### New findings

1. **Write performance is largely independent of access patterns** (regardless of being sequential or random), and can even outperform reads, which is opposite to the long-existing commo understanding about slow writes on SSDs.
2. One performance concern comes from **interference between concurrent reads and writes**, which causes substantial performance degradataion.
3. Parallel I/O performance is sensitive to **physical data-layout mapping**, which is largely not observed without parallelism.
4. Existing application designs **optimized for magnetic disks can be suboptimal for running on SSDs with parallelism**

### A generalized SSD model

*domain*: a set of flash memories that share a specific set of resources (e.g. channels)

*chunk*: a unit of data that is continuously allocated within one domain

*stripe*: a set of chunks across each of *N* domains

Interested in examining three key factors:

- **chunk size**: The size of the largest unit of data that is continuously mapped within an individual domain.
- **Interleaving degree**: The number of domains at the same level. The interleaving degree is essentially determined by the redundancy of the resources.
- **Mapping policy**: The method that detemines the domain to which a chunk of logical data is mapped. This policy determines the physical data layout

Treat an SSD as a 'black box'. Assume the mapping follows some repeatable but unknown patterns.

Observe the 'reactions' of the SSD, measured in several key metrics, e.g. **latency and bandwidth.**
