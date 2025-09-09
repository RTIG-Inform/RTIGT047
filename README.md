![](images/rtig_document_header_logo.png)

# CMS to PID Interface Protocol

# Part 1 - Architecture


## Status of this document

This document is Published.

If there are any comments or feedback arising from the review or use of this document, please contact us at secretariat@rtig.org.uk

# Introduction

## About this document

This document has been produced for the Real Time Information Group (RTIG) in conjunction with Transport for Wales to support the standardisation of the interface between content management systems and displays.

This document is Part 1 covering the architecture and is part of a series of documents covering different aspects of the standard.

## Background

Transport for Wales (TfW) would like to specify a standard interface between the Content Management System (CMS) and real time information Displays, that suppliers would need to comply/work with to enable TfW to procure a single CMS that can interface to multiple displays from a number of vendors.

The standard will specify the minimum capability that is to be expected of all displays supported through the interface (i.e. be able to represent real time vehicle arrival/departure information, text-based messages and hold the scheduled timetable for at least that day's services).

The interface will cater for the following:
* Basic text-based displays
* Graphical displays - in addition to the minimum capability, also be able to provide additional information such as weather, news feeds, advertising, information videos etc.
* Off grid displays - these will not have ready access to power and may not have significant data bandwidth available to show graphical content.

The interface should also cater for fault management data to be passed back to the CMS to enable monitoring and fault rectification.

Initial workshops were held (online) in September 2021 to build a high-level understanding among industry practitioners of what Transport for Wales would like to achieve.

This document is the introduction to the series of documents detailing a standard CMS-to-PID interface.

## Governing Principles

This document has been drafted with the following principles in mind:
* Abstraction - "no need to re-invent the wheel".
    * Lower-level concepts should be abstracted away using existing standards wherever possible; this document focuses solely on the application-level detail.
* Clarity - "grey areas should be minimised".
    * Committee-designed standards often evolve to support multiple different mechanisms of achieving the same result, leading to grey areas in compatibility between products that can all legitimately claim to support the standard - although in many cases, interoperability is limited.
* Simplicity - "less is more".
    * The more complex the interface protocol becomes, the less likelihood there is of industry-wide uptake. Conversely, by keeping the rules to a bare minimum, we hope to encourage wider adoption and compatibility by PID and CMS suppliers.
  
  In summary, it is the intention that this specification should be as lightweight and as high-level as possible. As the project progresses, this document will be updated to reflect the latest status of the interface design.

## Scope

The CMS to PID interface protocol includes:

* Messages to support core public transport timetable and real time information used across all display types.
* Messages to support formatting text and graphical content handling.
* The communications infrastructure (i.e. network technology and protocols) that will be used to support communication between the CMS and the PIDs.
* Security requirements of any parts of the network that rely on the public internet.
* Multi-vendor support - i.e. the use of PIDs from multiple suppliers connected to the CMS by the same communications infrastructure.
* A "discovery" process through which a PID can connect to a management service and receive configuration details. Typically, this will only happen during the initial commissioning phase, or when a PID is replaced or switched due to hardware failure.
* Specification of message structure and content for managing timetabled departures, estimated departures and departure cleardowns.
* "Heartbeat" messaging and general fault reporting.

The following elements are considered to be out of scope of the current project:

* Detailed discussions about network security on private IP-based APNs or VPNs. Where an implementation requires the use of the public internet, it is recommended that the use of TLS is specified as a project requirement by the contracting body, to ensure end-to-end security and prevent "man-in-the-middle" and "denial-of-service" style attacks by malicious actors.
* Integration of legacy or low-power devices - where legacy or low-power PIDs rely on a specific communications infrastructure (e.g. PMR) or a propriety messaging protocol (e.g. binary data packets) it is anticipated that a bidirectional "gateway" device could be used to propagate inbound and outbound messages between the CMS and the PID. It is recommended that where such gateway devices are to be used, the contracting body should specify the maximum acceptable latency that can be introduced by such devices as a project requirement.
* Time synchronisation - it is expected that clock synchronisation between the CMS and PIDs should be managed using NTP or a similar service, and that maximum permitted "drift" between devices should be specified as a project requirement by the contracting body.
* Hardware specification for PIDs - neither the physical characteristics of the customer-visible display nor the technical characteristics of the embedded computer are within scope of this specification. It is anticipated that the shape, size and dot pitch/resolution of the display will be subject to the project requirements of the contracting body, and that the specification of the embedded computer will be left to the discretion of the PID hardware supplier.

## Existing Standards

Whilst SIRI provides a number of relevant message types for CMS-to-PID communication, it does not fully meet the requirements of this project. Where possible, in line with the governing principles, existing message types will be evaluated carefully and reused wherever possible.

## Document Structure

To minimise versioning complexity, the interface protocol is split into multiple parts:

[Part 1](RTIGT047-pt1-CMS-to-PID-Interface.md)

* Communications Infrastructure
* Network Architecture

[Part 2](RTIGT047-pt2-CMS-to-PID-Interface.md)

* Common Data Structures
* Core Content Messages (applicable to all display types)

[Part 3](RTIGT047-pt3-CMS-to-PID-Interface.md)

* Additional Message Types for Graphical Displays
* Additional Message Types for Low Power Displays

[Part 4](RTIGT047-pt4-CMS-to-PID-Interface.md)

* Additional Services (e.g. Fault Management, Reporting, Audio)

Part X

* Additional requirements yet to be defined

## Limitations and The Future

This report reflects the available technology and practices which have been found to be effective at the date of publication. However, technology and its applications are evolving and it is therefore probable that new technologies, new developments of existing technologies, and new ways to adopt and utilse them will evolve along with new business requirements.
Procuring authorities are encouraged to consider new requirements and approaches bearing in mind the standard promoted in this document. Where a new business requirement or technology is developed that is not supported by this standard we encourage you to contact RTIG to discuss how we update the standard to ensure it remains fit for purpose.

## Acknowledgements

RTIG is grateful to Transport for Wales for funding the project that initially developed this standard.
We are grateful to the members of the technical working group who have contributed to its development including Vix Technology, Journeo, r2p Systems UK, Elydium, Transport for London and ITxPT.





