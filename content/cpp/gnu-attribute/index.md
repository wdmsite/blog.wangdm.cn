---
title: "GNU Attribute"
date: 2024-03-20T18:40:00+08:00
slug: gnu-attribute
categories:
- Cpp
tags:
- GNU
- attribute
draft: false
---

## socket basic api

```c
struct student
{
    char name[7];
    uint32_t id;
    char subject[5];
} __attribute__ ((aligned(4))); 

struct student
{
    char name[7];
    uint32_t id;
    char subject[5];
} __attribute__ ((packed));
```