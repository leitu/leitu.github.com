# Testing with New debian on public Cloud

## Google Cloud Platform

Suprised that google ship Debian 9 very quickly, and update the existing Debian 8 as well

```bash
debian-8-jessie-v20170619                         debian-cloud       debian-8                              READY
debian-9-stretch-v20170619                        debian-cloud       debian-9                              READY
```

## AWS

AWS release this 2 days later afer Google Cloud Platform.

```bash
debian-stretch-hvm-x86_64-gp2-2017-06-21-52542 - ami-27072e31
```

## Azure

umm.. The main is still debian 8

```bash
  {
    "offer": "Debian",
    "publisher": "credativ",
    "sku": "8",
    "urn": "credativ:Debian:8:latest",
    "urnAlias": "Debian",
    "version": "latest"
  },
```

but they shipped, but you are not able to see it from command

```bash
  {
    "offer": "Debian",
    "publisher": "credativ",
    "sku": "9-DAILY",
    "urn": "credativ:Debian:9-DAILY:9.0.201706270",
    "version": "9.0.201706270"
  }
```