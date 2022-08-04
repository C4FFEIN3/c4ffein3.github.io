---
layout: post
title: "[논문 요약] Automatic I/O stream management based on file characteristics"
subtitle: ""
categories: paper
tags: [SSD, clustering, hot-cold-seperation]
---

## Conference

Hotstorage'21

## Authors

Yuqi Zhang, Ni Xue, Yangxu Zhou

## Idea

### File types
    - Append-only
        - Only be written sequentially
        - Invalid only after the file is deleted
        - Lifetimes of wirte-ahead logs are obviusly shorter than the lifetimes of data files
    - Inplace update
        - Random writes
        - The data files of relational databases (MySQL, PostgreSQL)
        - The files exists for a long time
        - The data files are created or deleted when the tables are created or deleted
        - The update frequencies of data in the same table are similar, while those in different table files are different
### Scheme
    - File monitor
        - Information of all opened files (except read-only) by the tool (e.g. inotify)
        - Stores them in the file attribute table
        - The information:
        inode number, name, size, the number of modifications, open time, close time, delete time
        - The stream ID corresponding to each file is recorded in the file attribute table
        and submitted to the virtual file system (VFS)
    - Mapper
        - Assign a stream ID to the file when it is opened
        - Minimizing the mixing of data between different files
            - Consider files with the same parent path and extension as the same type of files
        - Minimizing the lifetime difference between files written to the same stream
    - Remapper
        - If the files remain open for *T* seconds, the remapper is responsible for reassigning new stream IDs to the files with the extracted access characteristics
        - The updated stream IDs only affect future writes
        - K-means++
    - Divider
        - Dynamically allocates different stream IDs for the mapper and remapper
        - IDs used by the mapper and the remapper are 1 to *d*, and (*d*+1) to *N*.
        The divider adjusts *d* every *T* seconds.
