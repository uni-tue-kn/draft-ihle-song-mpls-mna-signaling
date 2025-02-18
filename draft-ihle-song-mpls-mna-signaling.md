---
title: "Signaling MNA Capabilities Using IGP"
abbrev: "SIG"
category: info

docname: draft-ihle-song-mpls-mna-signaling-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: WG Working Group
keyword:
 - signaling
 - mpls
 - mna
venue:
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  github: "uni-tue-kn/draft-ihle-song-mpls-mna-signaling"
  latest: "https://uni-tue-kn.github.io/draft-ihle-song-mpls-mna-signaling/draft-ihle-song-mpls-mna-signaling.html"

author:
 -
    fullname: Fabian Ihle
    organization: University of Tuebingen
    city: Tuebingen
    country: Germany
    email: fabian.ihle@uni-tuebingen.de
 -
    fullname: Xueyan Song
    organization: ZTE Corporation
    country: China
    email: song.xueyan2@zte.com.cn
 -
    fullname: Michael Menth
    organization: University of Tuebingen
    city: Tuebingen
    country: Germany
    email: michael.menth@uni-tuebingen.de

normative:

informative:


--- abstract

This document defines capabilities of nodes supporting MPLS Network Actions (MNA) and how to signal them using IS-IS and OSPF.
The capabilities include the Readable Label Depth (RLD), supported network action opcodes, and the maximum sizes of differently scoped NAS.
For IS-IS routers, a sub-TLVs encoding is leveraged.


--- middle

# Introduction

With the MPLS Network Action (MNA) framework, network actions are encoded in the MPLS stack.
{{?I-D.ietf-mpls-mna-hdr}} defines the encoding of such network actions and their data in the MPLS stack.
These network actions are processed by all nodes on a path (hop-by-hop), by only selected nodes, or on an ingress-to-egress basis.
LSRs have different capabilites that result from the available hardware resources.
An ingress LER that pushes network actions to an MPLS stack MUST ensure that all nodes on the path can read and support the network actions.
For that purpose, the MNA capabilities of an LSR need to be signaled to the ingress LER.

This document defines the required parameters of LSRs regarding their MNA capability and proposes a signaling extension using IGP such as IS-IS and OSPF.

## Terminology

{::boilerplate bcp14-tagged}

### Abbreviations
This document makes use of the terms defined in {{?I-D.ietf-mpls-mna-hdr}} and in {{?I-D.ietf-mpls-mna-fwk}}.

# Definition of MNA Capabilities

In this section, the parameters an LSR SHOULD signal to the ingress LER to indicate its MNA capabilities are defined.

## The Readable Label Depth (RLD)

The Readable Label Depth (RLD) is the number of LSEs an LSR can parse without performance impact{{?I-D.ietf-mpls-mna-fwk}}.
An LSR is required to search the MPLS stack for Network-Action Substacks (NAS) that have to be processed by this node.
The ingress LER that pushes the network actions MUST ensure that a hop-by-hop-scoped network action is readable at each LSR on the path, i.e., that it is placed within the RLD of each node.
For this purpose, multiple copies of the hop-by-hop-scoped NAS may be placed in the stack.

### Example

An example for the RLD parameter is given in {{fig-rld_example}}. With an RLD of 5, an LSR is capable of reading labels A, B, C, D, and E but not D.
An RLD of 8 is required in this example to read the full MPLS stack.

~~~~
{::include ./drawings/rld_example.txt}
~~~~
{: #fig-rld_example title="Example MPLS stack of 8 MPLS LSE illustrating the concept of RLD."}

## Maximum NAS Sizes

A NAS in the MNA header encoding is at least 2 LSEs and at most 17 LSEs large{{?I-D.ietf-mpls-mna-hdr}}.
At an LSR, at least two NAS, a select-scoped and a hop-by-hop-scoped NAS, are possible.
With two maximum-sized NAS, an LSR is required to reserve 34 LSEs in hardware to be able to process network actions.
This consumes hardware resources that may be needed to encode other LSEs, e.g., forwarding labels for SR-MPLS paths.
Many use cases in the MNA framework{{?I-D.ietf-mpls-mna-use-cases}} do not require a maximum-sized NAS of 17 LSEs to encode the network action and their ancillary data.
Therefore, by signaling the maximum-supported NAS size of an MNA implementation from an LSR to an ingress LER, the allocated resources for NAS can be reduced and more resources are available for other purposes.

An LSR SHOULD signal the maximum-supported size of a NAS for each scope, i.e., the parameters maxLSEs_NAS^Sel, maxLSEs_NAS^HBH, and maxLSEs_NAS^I2E.
Those parameters include the Format A, B, C, and D LSEs from {{?I-D.ietf-mpls-mna-hdr}} in a NAS.

- For select-scoped NAS, the ingress LER MUST NOT push NAS to the MPLS stack that exceed the MNA capabilities of the node the select-scoped NAS is destined for.
- For hop-by-hop-scoped NAS, the ingress LER MUST NOT push NAS to the MPLS stack that exceed the MNA capabilities of any node on the path.
- For I2E-scoped NAS, the ingress LER MUST NOT push NAS to the MPLS stack that exceed the MNA capabilities of the egress node.

### Example

{{fig-rld_example}} illustrates the different NAS sizes in an MPLS stack that are signaled to the LSR.
In this example, a select-scoped NAS has a maximum size of 4 LSEs, a hop-by-hop-scoped NAS of 7 LSEs, and an I2E-scoped NAS of 4 LSEs.

~~~~
{::include ./drawings/nas_sizes_example.txt}
~~~~
{: #fig-nas_sizes_example title="Example MPLS stack illustrating the different NAS sizes."}


## Supported Network Action Opcodes
An LSR MUST signal the network action opcodes it supports.
If a network action opcode is not signaled, it is assumed that this opcode is not supported by the node.

# Signaling MNA Capabilites
This section defines a method for IGP routers to advertise the maximum supportable numbers of LSEs in I2E-scoped NAS, Select-scoped NAS, and HBH-scoped NAS.

## Using IS-IS
This section defines the signaling of maximum number of LSEs that can be supported for specific Network-Action Substacks (NAS) using IS-IS node and link advertisement.
{{?rfc7981}} defines the IS-IS Router Capability TLV that supports optional sub-TLVs to signal capabilities.
Further, {{?rfc8491}} introduces the Maximum SID Depth (MSD) that defines a sub-TLV for node- and link-specific advertisement based on {{?rfc7981}}.
The MSD describes the number of LSEs a node is capable of imposing and is not restricted to Segment Routing (SR).
In domains where SR is not enabled, the MSD defines the maximum label depth.
MSD values may be signaled by the forwarding plane or may be provisioned by the control plane.

### RLD Advertisment
For the Readable Label Depth advertisement, a sub-TLV is requested in {{?I-D.draft-ietf-mpls-mna-fwk}}.

### MSD-NAS Advertisement
To signal the maximum numbers of LSEs for NAS with different scopes, this document introduces new sub-TLVs based on {{?rfc8491}}.
The MSD-NAS Sub-TLV is defined node- or link-specific as below:

~~~~
{::include ./drawings/is-is_signaling.txt}
~~~~
{: #fig-is-is_signaling title="Node MSD-NAS Sub-TLV."}

- Type: 15 (link-specifc) or 23 (node-specific) {{?rfc8491}}.
- Length: variable (multiple of 2 octets); represents the total length of the Value field
- Value: field consists of one or more pairs of a 1-octet MSD-Type and 1-octet MSD-Value
- MSD-Type: value defined in the "IGP MSD-Types" registry created by the IANA Considerations section of this document (I2E, HBH, or Select).
- MSD-Value: number in the range of 2-17.

This sub-TLV is optional.
The scope of the advertisement is specific to the deployment.

### Supported Network Action Opcodes
tbd

## Using OSPF
tbd

### RLD Advertisment
tbd

### MSD-NAS Advertisement
tbd

### Supported Network Action Opcodes
tbd

# Security Considerations

The security issues discussed in {{?I-D.ietf-mpls-mna-hdr}} and {{?rfc8491}} apply to this document.



# IANA Considerations

This docuement requests the allocation of following codepoints in the "IGP MSD-Types.

| Value |  Name  | Data Plane                      |  Reference
| ---------- |  -------------------------------- |  -------------------
|    3    |  Readable Label Depth | MPLS |  {{?I-D.draft-ietf-mpls-mna-fwk}}
|    4    |  MSD of select-scoped NAS | MPLS |  This document
|    5    |  MSD of I2E-scoped NAS | MPLS |   This document
|    6    |  MSD of HBH-scoped NAS | MPLS |  This document
{: #table_iana title="IGP Signaling Sub-TLV allocation."}

--- back
