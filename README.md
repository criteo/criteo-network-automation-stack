# Criteo Network Automation Stack

## Context

At Criteo, we have decided to fully opensource our automation stack. It is based on Saltstack, OpenConfig and supports Juniper JunOS, Arista EOS and SONiC.

This repository aims to help to document and navigate our stack, as it consists of different repositories.
It will include our documentation, our FAQ, and potentially other things.

At the moment, not everything is opensource as we are releasing it progressively (some cleaning is needed on our side).

## Global design

Our approach to automation is opinionated. There are tons of ways of doing network configuration, and choices have to be made at some points.

            ┌──────────────┐  ┌────────────────────┐
            │ Network CMDB │  │ Other data sources │
            └──────────┬───┘  └──┬─────────────────┘
                       │         │
                       │         │
                       │         │
                       │         │
                    ┌──▼─────────▼──┐
                    │  OpenConfig   │
                    │ converter API │
                    └───────┬───────┘
                            │
                            │
                            │
                      ┌─────▼─────┐
                      │ Saltstack │
                      └─────┬─────┘
                            │
                            │
                            │
                            │
                   ┌────────▼────────┐
                   │ Network devices │
                   └─────────────────┘

### Network CMDB

The Network CMDB contains data relative to the business and is completely agnostic to the network OS. The models are designed to describe the objects themselves rather than the configuration from devices perspective. The main principle is also to avoid any data duplication which could lead to mismatch configuration.

For instance, we represent the BGP session itself with two joined table describing peers: DeviceBGPSession <==> BGPSession <==> DeviceBGPSession.
* DeviceBGPSession contains the local-as but not the peer-as, avoiding data duplication. The peer-as being the local-as of the other neighbor.
* BGPSession contains all information peers have in common, like status (in production, maintenance etc...) or MD5 password

### OpenConfig converter API

This API aggregate data from their source of truths (Network CMDB or possibly any other data sources).

Then, it computes these data to provide OpenConfig JSON for each device as an output.

We use OpenConfigValidator to validate data against the OpenConfig YANG models.

### Saltstack

Salt will take OpenConfig data and convert them as Network configuration. At the moment, we are using templates to do that.

The end goal is to simply forward this OpenConfig data to the Network OS so it can apply it. At best, OpenConfig is partially implemented at all in Network Operating Systems.

## Repositories

* [SONiC Salt Deployer](https://github.com/criteo/sonic-salt-deployer): script to deploy salt-minion on SONiC devices and configure it
* [SONiC Saltstack](https://github.com/criteo/sonic-saltstack): all states/execution modules we have for SONiC
* [SONiC utilities](https://github.com/criteo/criteo-sonic-utilities): set of SONiC scripts to get information in JSON format (used by some SONiC saltstack modules)
* [OpenConfigValidator](https://github.com/criteo/OpenConfig-validator): library which permits to validate data over OpenConfig models via Ygot

Coming later:
* Network CMDB
* OpenConfig converter API
* OpenConfig Salt modules
