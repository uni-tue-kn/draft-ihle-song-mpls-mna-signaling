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

TODO Abstract


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

The readable label depth (RLD) is the number of LSEs an LSR can parse without performance impact{{?I-D.ietf-mpls-mna-fwk}}.
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

## NAS Sizes

A NAS in the MNA header encoding is at least 2 LSEs and at most 17 LSEs large{{?I-D.ietf-mpls-mna-hdr}}.
At an LSR, at least two NAS, a select-scoped and a hop-by-hop-scoped NAS, are possible.
With two maximum-sized NAS, an LSR is required to reserve 34 LSEs in hardware to be able to process network actions.
This consumes hardware resources that may be needed to encode other LSEs, e.g., forwarding labels for SR-MPLS paths.
Many use cases in the MNA framework{{?I-D.ietf-mpls-mna-use-cases}} do not require a maximum-sized NAS of 17 LSEs to encode the network action and their ancillary data.
Therefore, by signaling the maximum-supported NAS size of an MNA implementation from an LSR to an ingress LER, the allocated resources for NAS can be reduced and more resources are available for other purposes.
An LSR SHOULD signal the maximum-supported size of a NAS for each scope, i.e., the parameters maxLSEs_NAS^Sel, maxLSEs_NAS^HBH, and maxLSEs_NAS^I2E.
Those parameters include the Format A, B, C, and D LSEs from {{?I-D.ietf-mpls-mna-hdr}} in a NAS.
For select-scoped NAS, the ingress LER MUST NOT push NAS to the MPLS stack that exceed the MNA capabilities of the node the select-scoped NAS is destined for.
For hop-by-hop-scoped NAS, the ingress LER MUST NOT push NAS to the MPLS stack that exceed the MNA capabilities of any node on the path.
For I2E-scoped NAS, the ingress LER MUST NOT push NAS to the MPLS stack that exceed the MNA capabilities of the egress node.

### Example

{{fig-rld_example}} illustrates the different NAS sizes in an MPLS stack that are signaled to the LSR.
In this example, a select-scoped NAS has a maximum size of 4 LSEs, a hop-by-hop-scoped NAS of 7 LSEs, and an I2E-scoped NAS of 4 LSEs.

~~~~
{::include ./drawings/nas_sizes_example.txt}
~~~~
{: #fig-nas_sizes_example title="Example MPLS stack illustrating the different NAS sizes."}


## Supported Network Action Opcodes

# Signaling MNA Capabilites

## Using IS-IS

## Using OSPF

# Security Considerations

The security issues discussed in {{?I-D.ietf-mpls-mna-hdr}} apply to this document.


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
