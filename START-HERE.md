# Shadow-RPC-Operator

Shadow Operator resources for running a high performance &amp; high stability Shadow Node to earn rewards

# Overview

Please make sure to review this comprehensive overview of the [Shadow Protocol](https://shdw.genesysgo.com/the-shadow-protocol)

This guide is currently intended only for setting up servers from the [Solana Server Program](https://solana.foundation/server-program).

**IMPORTANT:** We are currently in alpha testing of Shadow Node Operators. Please reach out to a team member of GenesysGo to express interest in being an alpha tester. During this short phase of testing, there is no active $SHDW token reward emission (as the IDO has not yet occurred). Additionally, we may often make the request that machines be turned off or removed for adjustments. This is a volunteer alpha phase and we appreciate anyone willing to help contribute their time and resources.

**UPDATE** 03/10 - Alpha testing pool is closed for a short period while we build out both the protocol that oversees the shadow nodes as well as the shadow drive integrations.

**HARDWARE CONSIDERATIONS**
A Solana Shadow Node has similar hardware requirements to the Solana RPC with the addition of additional storage capacity. This will change over time as the Shadow Protocol evolves. These hardware details below are for **early alpha Shadow Nodes and the following beta Shadow Nodes**

If you are utilizing the Solana Server Program for your hardware, please make sure you have two larger drives as detailed below:

Minimums from Solana Server Program:
*  CPU - 24 core, 2.8GHZ, AVX2 support (Prefer 3.0GHZ+ native), must boost above 3ghz
*  RAM - 256GB
*  HD  x 3
       - Boot - 200GB SSD
       - Ledger - 3.8TB NVME
       - Storage - 3.8TB NVME or SSD
*  NIC - 10gig
*  Connection - 1GB/s symmetrical

Custom Builders see above minimums or the below general guidance. These are generalizations and subject to change.
# Operating a Shadow Node

This resource has a step by step guide covering:
1) [Configuring a Shadow Node](https://github.com/Shadowy-Super-Coder-DAO/Shadow-RPC-Operator/blob/main/1-How-to-configure-RPC-node.md)
2) [Connecting](https://github.com/Shadowy-Super-Coder-DAO/Shadow-RPC-Operator/blob/main/2-How-to-connect-to-network.md) the Shadow Node to the `SSC-DAO Shadow Network`
3) [Maintaining](https://github.com/Shadowy-Super-Coder-DAO/Shadow-RPC-Operator/blob/main/3-how-to-maintain-rpc-node.md) your Shadow Node for maximum rewards payouts

If you are just getting started, walking through the `1-Configuring a Shadow Node` will be immensely helpful in learning the design of the machines and basic linux commands. The guide is designed to support a Linux novice. 

Once you have completed Steps 1-3 you should notify the GenesysGo team and receive the Shadow Operator role in Discord. This grants you access to the Shadow Ops and Shadow health monitoring channels.
# Rewards for Operating a Shadow Node

Please see this [resource](https://shdw.genesysgo.com/shadow-infrastructure-overview/shadow-operators) for how rewards will work:

# Other Resources & Learning

Official Solana RPC resources to reference are located in the [Solana Documentation](https://docs.solana.com/running-validator).

For developers interested in a more automated control we are releasing an advanced Shadow Node operator manual in due time.

For more advanced developers and those interested in deploying their node and scaling their Shadow Node(s) with the ansible automation engine please see [this resource by Triton](https://github.com/rpcpool).

# Credits

Thank you to our alpha test Shadow Operators and advisors for the thoughts and technical contributions.

Thanks to the great group at rpcpool / Triton One for their support in the Solana RPC Discord channel and for providing the broader community their resources.

We also learned very early from agjell (andrebo in Discord) who more focuses on validators but offered a great guide that helped many of the validators you see running today get online. If you are also interested in getting your very own validator up and running check out [his repo](https://github.com/agjell/sol-tutorials).

Finally, thanks to everyone that actively offers help and advice in the Solana Discord and the SSC DAO discord.

Don't hesitate to ask questions or submit feedback to this DAO repository!