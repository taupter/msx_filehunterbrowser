Source: https://github.com/Konamiman/MSX-UNAPI-specification/blob/master/docs/MSX%20UNAPI%20specification%201.1.md

# MSX-UNAPI: Unified procedure for API definition and discovery on MSX computers

This document describes MSX-UNAPI, a standard procedure for defining, discovering and using new APIs (Application Program Interfaces) for MSX computers. The goal is to provide a unified way to design and use BIOS-like software for the various kinds of new hardware that is being developed for these computers, altough it can be used for software-only solutions as well.

## Index

[1. Introduction](#1-introduction)

[1.1. Motivation](#11-motivation)

[1.2. Goals](#12-goals)

[1.3. Non-goals](#13-non-goals)

[1.4. Glossary](#14-glossary)

[1.5. Sample scenario](#15-sample-scenario)

[2. API specification rules](#2-api-specification-rules)

[2.1. Rule 1: The API identifier and version number](#21-rule-1-the-api-identifier-and-version-number)

[2.2. Rule 2: One single entry point on Z80 page 1 or 3](#22-rule-2-one-single-entry-point-on-z80-page-1-or-3)

[2.3. Rule 3: Z80 registers usage](#23-rule-3-z80-registers-usage)

[2.4. Rule 4: Routine number ranges](#24-rule-4-routine-number-ranges)

[2.5. Rule 5: The API information routine + the implementation name and version](#25-rule-5-the-api-information-routine--the-implementation-name-and-version)

[2.6. Rule 6: Implement code for the API discovery procedure](#26-rule-6-implement-code-for-the-api-discovery-procedure)

[2.7. Rule 7: Install the RAM helper](#27-rule-7-install-the-ram-helper)

[2.8. Rule 8: Avoid segment number FFh](#28-rule-8-avoid-segment-number-ffh)

[2.9. Optional rule 9: Give your routines a meaningful name](#29-optional-rule-9-give-your-routines-a-meaningful-name)

[3. API implementations discovery procedure](#3-api-implementations-discovery-procedure)

[3.1. The MSX extended BIOS](#31-the-msx-extended-bios)

[3.2. Steps of the discovery procedure](#32-steps-of-the-discovery-procedure)

[3.3. How to implement support for the discovery procedure](#33-how-to-implement-support-for-the-discovery-procedure)

[4. The RAM helper](#4-the-ram-helper)

[4.1. Notes on the routine for calling a routine with inline routine identification](#41-notes-on-the-routine-for-calling-a-routine-with-inline-routine-identification)

[5. Specificationless UNAPI applications](#5-specificationless-unapi-applications)

[Appendix A. The standard mapper support routines](#appendix-a-the-standard-mapper-support-routines)

[Appendix B. Acknowledgements](#appendix-b-acknowledgements)

[Appendix C. Document version history](#appendix-c-document-version-history)

## 1. Introduction

### 1.1. Motivation

MSX was presented in 1983 as a standard for home computers, in the form of a set of specifications to be followed by manufacturers. Any brand was able to make its own model of computer; as long as the machine met the MSX standard, it was considered an MSX computer. This allowed for flawless interoperation between computers of different makers, something not as common at the time as it is nowadays.

An important part of the MSX specification was the BIOS, a set of routines that -amongst other things- provided a standarized way to access the harwdare resources of the machine. This provided certain degree of freedom to manufacturers, which could -to some extent- freely choose the hardware parts that would build up their given model of MSX computer.

Time passed and the MSX standard was discontinued by all manufacturers long time ago. However, many people still using their old MSX computers, and some of them even develop new hardware for these machines. Hard disk interfaces, sound cards and network cards are examples of hardware that has been developed for MSX computers past the commercial live of the standard.

Altough the appearance of new hardware is of course a good thing, this causes a problem for software developers. Not having any authority dictating specifications for the MSX standard anymore, each hardware developer freely choses the way their hardware will have to be accessed from software; in other words, there are no specifications to follow at the time of designing the API (Application Program Interface) for the new harwdare. As a result, hardware extensions performing the same functions but being developed by different persons are software-incompatible (unless developers privately agree on using compatible APIs, which is not always possible).

The intent of this document is to propose a solution for this problem: a standard, unified procedure for defining, discovering and using new APIs for MSX computers, aimed especially (but not exclusively) to the hardware development field.

### 1.2. Goals

The main goals of this specification are:

* To provide a minimum set of rules to be followed by APIs that adhere to this specification. These rules are to be taken in account when designing (and later implementing) the various APIs. They are very simple, impose only a few restrictions on compliant code, and in practice allow any kind of API software to be written.

* To provide a standard procedure for discovering and using APIs that adhere to this standard and are installed on the MSX computer running the UNAPI aware software. This procedure is based on the MSX extended BIOS mechanism, and is very easy to implement and even easier to use.

### 1.3. Non-goals

It is NOT the intent of this specification, and/or is outside the scope of this document:

* To impose the described specification to developers or users. Altough the authors hope that the MSX-UNAPI specification will be useful for the MSX community, this document is merely a proposal, and adhering to it is of course optional.

* To arbiter who should desing the APIs. Common sense suggests that APIs should be designed by the first person or team that develops a given type of hardware (or software), but the decision of such thing is outside the scope of this document.

* To describe any API other than for illustration purposes. This documents just tells a procedure for designing APIs; these must be described in sepparate documents.

### 1.4. Glossary

Before proceeding any further, a glossary of terms used along this document will be introduced. It will be useful to understand the rules and procedures described later.

* **MSX-UNAPI**: Short for _"MSX unified API definition and discovery standard"_. A set of rules to be followed when designing and implementing APIs, aimed mainly (but not exclusively) to hardware developers; and a procedure for discovering any such APIs available to application software.

* **API specification**: API stands for _"Application Program Interface"_. The description of a set of routines that perform some action (which usually, but not necessarily, involve accessing some kind of hardware), including the precise description of input and output parameters, as well as the side effects caused by the execution of each routine.

* **UNAPI compliant API specification**: An API specification that conforms to the rules for API specifications described in this document.

* **API specification identifier**: An alphanumeric, case-insensitive string made of up to 15 characters, which uniquely identifies an UNAPI compliant API specification. For example the identifier of an API specification for Ethernet cards could be "ETHERNET". See [Section 2.1](#21-rule-1-the-api-identifier-and-version-number) for more details.

* **The XXXXX UNAPI specification**: The specification for an UNAPI compliant API whose specification identifier is XXXXX. For example, "the Ethernet UNAPI specification".

* **XXXXX UNAPI compliant API implementation**: Real code that builds up a set of routines which conforms to an UNAPI compliant API specification (and more precisely, to the XXXXX UNAPI specification). There may be several implementations of one single specification, each made by a different person or team; but if they strictly conform to the API specification, they are indistinguishable with respect to usage and behavior.

* **API implementation name**: An alphanumeric string made of up to 63 characters, which uniquely identifies an API implementation. For example the name of an implementation of the Ethernet UNAPI could be "ObsoNET", while the name of other implementation could be "UltraNET Supercard". See [Section 2.5](#25-rule-5-the-api-information-routine--the-implementation-name-and-version) for more details.

* **XXXXX UNAPI client software**: Application software that invokes the routines of an XXXXX UNAPI compliant API implementation. For example, a TCP/IP stack could act as client software of the Ethernet UNAPI. The client software must use the discovery procedure (see [Section 3](#3-api-implementations-discovery-procedure)) to locate the available API implementations, and usually it does not matter which one is used if more than one is found.

* **Specificationless UNAPI application**: Application software (typically, a resident application) that does not conform to any concrete UNAPI specification, but that make use of the remaining of the UNAPI infrastructure, namely, the implementation and discovery rules. See [Section 5](#5-specificationless-unapi-applications) for more details.

* **FFh**: Hexadecimal numbers are denoted with the "h" prefix in this document, so for example FFh is 255.

Through this document, and unless otherwise stated, the term "API specification" will actually have the meaning of "UNAPI compliant API specification". The same applies to references to API implementations.

### 1.5. Sample scenario

This section provides an fictional scenary that briefly shows the steps that are needed in order to design and implement UNAPI compliant APIs. The goal is to provide the reader a basic understanding about how all the whole thing works, so that it is easier to read the detailed rules and procedures that are provided in the remaining of the document.

Suppose that a user named H.G. Wells builds a time machine for MSX computers. He wants to make the software to control it to be UNAPI compliant. To achieve this, since it is the first time machine ever developed for MSX, he first must design the API specification. These are the steps he follows for it:

1. Chooses a short name for the API specification. He chooses "TIME_MACHINE" (see [Section 2.1](#21-rule-1-the-api-identifier-and-version-number)).

2. Designs the routines that will compose the API. He designs three routines: travel backwards, travel forwards, and return to original time. These routines will have routine numbers 1 through 3. (See [Section 2.2](#22-rule-2-one-single-entry-point-on-z80-page-1-or-3), [Section 2.3](#23-rule-3-z80-registers-usage) and [Section 2.4](#24-rule-4-routine-number-ranges))

Now he must write a real implementation of the API, to be included in ROM together with the time-travel hardware. He does then the following:

1. Chooses a name for the API implementation: "Well's Time Machine BIOS" (see [Section 2.5](#25-rule-5-the-api-information-routine--the-implementation-name-and-version))

2. Implements the actual code for the three API routines (again, see [Section 2.2](#22-rule-2-one-single-entry-point-on-z80-page-1-or-3), [Section 2.3](#23-rule-3-z80-registers-usage) and [Section 2.4](#24-rule-4-routine-number-ranges)).

3. Implements the API discovery procedure, so that when the time machine is present in a MSX machine, it can be discovered by using extended BIOS (see [Section 3](#3-api-implementations-discovery-procedure)).

The device turns out to sell quite well in MSX users meetings, and some users develop software that uses it, from multi-epoch games to virtual history books. All of this software uses the UNAPI discovery procedure (see [Section 3](#3-api-implementations-discovery-procedure)) in order to find out at run time whether the time machine is present or not.

Soon afterwards, another user with hardware design knowledge, Dr. Brown, develops another kind of time machine for MSX. Since he wants the software that already exists for Well's hardware to be compatible with his own device, he develops the software so that it is compatible with the TIME_MACHINE API specification, by following the same steps that Wells did. There are, however, some important differences with the Well's machine:

1. The API implementation name is now "Brown's flux-capacited time machine".

2. The device is connected to MSX via serial port, so it has no ROM and the API implementation is therefore installed in mapped RAM. When this API is installed, it installs the RAM helper. (see [Section 4](#4-the-ram-helper))

3. There is an extra routine which is needed to calibrate the flux capacitor. This routine is implemented as an implementation- specific routine (see [Section 2.4](#24-rule-4-routine-number-ranges)), and has the routine number 128.

Now that there are two different time travel devices around, an user named M. McFly wants to develop a windowed graphic interface to control the time machines. It develops the software to discover, and interact with, TIME_MACHINE APIs; therefore it works with both kinds of devices. Moreover, as an extra feature, it checks the implementation name of the API implementation it uses (see [Section 2.1](#21-rule-1-the-api-identifier-and-version-number)), and when it is the Brown's implementation, the application offers extra functionality to calibrate the flux capacitor.

## 2. API specification rules

This section describes the rules that must follow any UNAPI compliant API specification, as well as their implementations.

### 2.1. Rule 1: The API identifier and version number

An API specification must have an alphanumeric identifier composed of up to 15 characters. Allowed characters are letters (having an ASCII code below 128), digits, and the following ones: `- _ / . ( )`

The API identifier is case-insensitive, so for example, "ETHERNET" is the same as "Ethernet".

An API specification must have a version number composed of main version number in the range 1-255, and secondary version number in the range 0-255. Specifications with higher version numbers must be backwards compatible with older versions. The first released specification document should have version 1.0. Versions 0.x are allowed as pre-releases, with no guarantee of backwards compatibility between them.

Digits may be used in the identifier when they are meaningful to describe the API (for example "WIFI_IEEE802.3"), but they must NOT be used to indicate the specification version.

### 2.2. Rule 2: One single entry point on Z80 page 1 or 3

There must be one single entry point for the API routines. This entry point must be accessible in any address on one of these places:

* A ROM or non-mapped RAM slot, on Z80 page 1 (address between 4000h and 7FFFh).

* A mapped RAM segment, provided that it will be connected to Z80 page 1 (address between 4000h and 7FFFh) when called.

* System RAM on page 3 (address above C000h).

It is outside the scope of this specification to define how RAM memory (mapped RAM segments and space in system RAM on page 3) is allocated.

If necessary, API routines may return a pointer to hard-coded information to be read by the client software (as the mandatory API information routine does with the API implementation name, see [Section 2.5](#25-rule-5-the-api-information-routine--the-implementation-name-and-version)). In this case, the information must be in the same place as the API implementation itself. That is, in the same ROM slot (in Z80 page 1), in the same RAM segment (if this is the case, a page 1 address is returned), or in system RAM on page 3. This way it is easy to obtain the name by using inter-slot or inter-segment (see [Section 4](#4-the-ram-helper)) reads.

As an exception to the above rule, API implementations residing in page 1 (ROM or a mapped RAM segment) may return an address in system RAM on page 3 (this is useful when ROM implementations need to return dynamic data, for example). Therefore the client software must be always prepared to either perform inter-slot or inter-segment reads, or direct memory reads, in order to retrieve the data.

If necessary, API routines may require the client software to specify a buffer where dynamic data will be read from, or written to (for example, packets to be sent to or retrieved from the network in the case of an Ethernet API). In this case, the API specification may impose a restriction to the client software, so that it must NOT specify any page 1 address as the buffer address.

### 2.3. Rule 3: Z80 registers usage

Z80 registers are used by the API routines in the following way:

* Register A is used at input to specify the routine to be executed. Each routine has associated a number in the range 0-254; how these numbers are allocated is described in [Section 2.4](#24-rule-4-routine-number-ranges). It can be used also as an output parameter, otherwise it is corrupted after the routine is executed.

* Registers F, BC, DE and HL may be used freely as input and/our output parameters. Registers not used to hold output parameters are corrupted after the routine is executed, unless otherwise stated in the routine definition.

* Registers IX, IY must not be used for input parameters, to allow for inter-slot and inter-segment calls. They can be used as output parameters, otherwise they are corrupted after the routine is executed.

* Alternate registers (AF', BC', DE' and HL') are corrupted after the routine is executed.

"The registers are corrupted after the routine is executed" means that the routine code may freely make internal use of the registers, and it does not need to restore their original values before finishing.

### 2.4. Rule 4: Routine number ranges

As stated before, each API routine has associated a number which must be passed in register A when calling the API entry point. The numeric range 0-255 is divided in four ranges of routines:

* 0: This value corresponds to the API information routine. It is a mandatory routine that must be present on all implementations of any UNAPI compliant API. The parameters and behavior of this routine are described in [Section 2.5](#25-rule-5-the-api-information-routine--the-implementation-name-and-version).

* 1-127: Specification routines. The behavior of these routines is defined in the appropriate API specification document.

* 128-254: Implementation-specific routines. API implementation developers may freely use this range of routines to offer additional capabilities not present in the original API specification; if this is done, these routines must be described in the documentation of the implementation. For example, one implementation of the Ethernet UNAPI could use this range of routines to offer WiFi connectivity, while other could offer basic internetworking capability. Note that different implementations may use the same numbers for different routines, always inside this range.

* 255: This value is reserved for a possible future extension mechanism.

Numbers must be assigned to routines in increasing order, starting with 1 for specification routines and starting with 128 for implementation-specific routines; it is not allowed to leave holes in the function number ranges. If client software invokes a non- existing routine (that is, calls the API implementation entry point passing in A a number not assigned to any routine), nothing must happen and the code must return with AF, BC, DE and HL unmodified.

### 2.5. Rule 5: The API information routine + the implementation name and version

Every API implementation, no matter which API specification is implementing, must mandatorily implement the API information routine. This routine has always the routine number 0 and has the following input and output parameters:

* Input: A = 0
* Output:
  * HL = Address of the implementation name string
  * DE = API specification version supported. D=primary, E=secondary.
  * BC = API implementation version. B=primary, C=secondary.

The implementation name string must be composed of up to 63 printable characters and must have a zero byte as termination. The name may be any descriptive text, but must NOT contain any version information, and is case-sensitive.

The implementation name must be placed in the same place as the API implementation itself (see [Section 2.2](#22-rule-2-one-single-entry-point-on-z80-page-1-or-3)).

Usually, client software will use the first suitable API implementation it founds, and will use the implementation name merely to show information to the user. However, if the client software is able to use the implementation-specific features of any given implementation, it must check the implementation name to know wether the desired implementation-specific routines are available or not.

Client software should check the API specification version supported by the implementation, since newer implementation versions may have routines not available in older versions.

Rules for the implementation version are similar to rules for the specification version (see [Section 2.1](#21-rule-1-the-api-identifier-and-version-number)): first release version should be 1.0, and 0.x versions are allowed for pre-releases. However, the backwards compatibility rule now applies to implementation-specific routines only, since the compatibility with the specification routines is indicated by the "API specification version supported" parameter.

It is not allowed for an API implementation to go backwards in the API specification version supported when the API implementation version is increased. For example, if implementation version 2.0 supports specification version 1.5, it is illegal for implementation version 2.1 to claim compliance with the specification version 1.4; it must support specification version 1.5 or higher. Otherwise, backwards compatibility between API implementations could not be assured.

### 2.6. Rule 6: Implement code for the API discovery procedure

Every API implementation, no matter which API specification is implementing, must mandatorily support the API discovery procedure. See [Section 3](#3-api-implementations-discovery-procedure).

### 2.7. Rule 7: Install the RAM helper

Every API implementation that is installed on a mapped RAM segment must, at installation time, check if the RAM helper is installed. If not, it must either install one, or refuse to install. See [Section 4](#4-the-ram-helper).

### 2.8. Rule 8: Avoid segment number FFh

An API implementation that installs on a mapped RAM segment must NOT be installed on a segment whose number is FFh (this is the last existing segment number on 4MByte RAM slots). This is necessary because the discovery procedure will use FFh as a fictitious segment number when the API implementation resides in ROM (see [Section 3.2](#32-steps-of-the-discovery-procedure)).

When using [the standard mapper support routines](#appendix-a-the-standard-mapper-support-routines), this is not an issue, since these routines will never allocate segment FFh even when it is available. However, when that's not the case and the segment for installation must be manually selected, care must be taken to not use segment FFh.

### 2.9. Optional rule 9: Give your routines a meaningful name

In your API specification, you can refer to your ruotines simply by their numbers. However, to make the client software developers life easier, it is advisable to give your routines meaningful names. Tipically, these names will be directly used as constants in source code; therefore, the following guidelines should be followed when choosing the names:

* Use characters that are legal for all assemblers and compilers. Ideally, you should use only letters and the underscore symbol ("_"). Use uppercase letters to improve readability.

* Limit the length of the names to a reasonable maximum, for example 16 characters.

* Do not use names already in use for standard MSX BIOS routines, system work area variables or MSX-DOS function calls.

* It may be useful to prepend the routine names with the first characters of the implementation identifier.

* And of course, choose names that make sense for the routines being named.

For example, an Ethernet UNAPI specification could have the following routine names: ETH_RESET (reset hardware), ETH_GET_FRAME (retrieve incoming frame from the network), or ETH_GET_NETSTAT (check the network status).

Note that this rule is optional, and so are the guidelines listed; you can break them if necessary, simply be wise and think on the other developers.

## 3. API implementations discovery procedure

[Section 2](#2-api-specification-rules) described the rules for designing UNAPI compliant APIs. These API specifications are used by developers to make API implementations, which are composed of real code that once installed on an MSX computer (whatever the installation method is), becomes available to client software.

This section describes the API implementations discovery procedure, that is, the steps that client software must follow to find out how many implementations of a given API specification are available at run time, and to gather information about each implementation (in which slot/segment are placed, where the entry point is, and the name and version information), as well as how API implementations can include support for this procedure.

### 3.1. The MSX extended BIOS

The discovery procedure is based on the MSX extended BIOS, which is a mechanism available on MSX computers that allows standard BIOS to be extended with new routines. This section explains the general rules of the extended BIOS mechanism, and later sections explain how this is used for the UNAPI discovery protocol.

Extended BIOS provides a five byte hook named EXTBIO at address FFCAh. To check whether this hook has been initialized with any value or is uninitialized, look at bit 0 of the byte at address HOKVLD (FB20h). A value of one for this bit means that the hook contains a valid jump instruction, previously set by system code or user code.

The EXTBIO hook contents may be replaced with a jump instruction to custom code, provided that the old hook contents are saved somewhere. If the hook is not initialized, it should first be filled with five RET instructions, and bit 0 of HOKVLD should be set; from this point, consider it as being initialized.

The EXTBIO hook is called with a code named "device identifier" in register D, a "function identifier" in register E, and input parameters in AF, BC and HL. The code called in this way must look at the device identifier to check if the request is for itself. If not, it must jump to the old hook (the one saved at installation time) with AF, BC, DE and HL unmodified.

When the device identifier matches the one expected by the code, it performs the action requested (as per the function identifier) and either returns parameters in, or corrupts, AF, BC and HL; DE is always preserved.

IX, IY and the alternate registers are always corrupted, no matter whether any suitable code for the specified device identifier is executed or not.

This procedure allows to chain together multiple BIOS extensions, each having its own device identifier. These identifiers were once assigned by MSX manufacurers, and later on, users have developed software that patch extended BIOS, freely choosing their own identifiers.

### 3.2. Steps of the discovery procedure

Client software willing to discover how many implementations of a certain API are available at run time must perform these steps:

1. Copy the API specification identifier to address F847h. Append a zero byte as string terminator.

2. Set Z80 registers as follows: A=0, B=0, DE=2222h.

3. Call the EXTBIO hook.

4. When EXTBIO returns, B contains the number of installed implementations of the specified API. DE is preserved and AF, C, HL are corrupted.

Address F847h is a 16 byte buffer named ARG, used normally by Math-Pack (the mathematic routines package of MSX BIOS).

After the previous steps have been performed, and provided that at least one implementation is installed (B>0), do the following to gather information about a given implementation:

1. Make sure that the API specification identifier is still at ARG.

2. Set Z80 registers as follows: A=implementation index (from 1 to the number of available implementations), DE=2222h.

3. Call the EXTBIO hook.

4. When EXTBIO returns, Z80 registers contain the following information about the specified implementation:

  * A = Slot where the implementation code is placed
  * B = RAM segment where the implementation code is placed (FFh if not in mapped RAM)
  * HL = Routines entry point address (if a page 3 address, A and B are meaningless)

    DE is preserved. F and C are corrupted.

With this information, now you know how to call the implementation entry point in order to invoke the API routines:

* If the entry point address is a page 3 address, ignore A and B and perform direct calls to that address.

* Otherwise, if B=FFh, use inter-slot calls (for example by using the BIOS routine CALSLT) to invoke the routines.

* Otherwise, use inter-segment calls (you can use the RAM helper for this, see [Section 4](#4-the-ram-helper)) to invoke the routines.

Usually, the first thing to do at this point is to call the API implementation information routine (see [Section 2.5](#25-rule-5-the-api-information-routine--the-implementation-name-and-version)) to obtain the implementation name and version (this is not mandatory, though). From here, client software can invoke the API routines as needed.

### 3.3. How to implement support for the discovery procedure

[Section 3.2](#32-steps-of-the-discovery-procedure) provided a functional description of the API implementations discovery procedure. For this procedure to work, all implementations of an UNAPI compliant API must include code to support this procedure. This section explains how this code should be implemented.

First, when the API implementation is installed (or at system boot, for ROM based code), the existing EXTBIO hook must be backed up (if EXTBIO hook is not initialized, initialize it first as explained in [Section 3.1](#31-the-msx-extended-bios)). Then, put a jump instruction or an inter-slot or inter-segment call pointing to your EXTBIO manager code.

Second, include in the API implementation code to manage EXTBIO calls (that's where the new EXTBIO hook points to). This code must perform the following steps:

1. Check that register pair DE holds the value 2222h. If not, jump to the old EXTBIO hook with AF, BC, DE, HL unmodified.

2. If A=FFh, jump to the old EXTBIO hook with AF, BC, DE, HL unmodified (this is to allow the RAM helper to be installed, see [Section 4](#4-the-ram-helper)).

3. Check that the string placed in ARG (remember that it is zero-terminated there) matches the identifier of the API specification that your code implements. If not, jump to the old EXTBIO hook with AF, BC, DE, HL unmodified. Remember that the strings comparison must be done in a case-insensitive way.

4. If A=0, increase the value of register B by one, and jump to the old EXTBIO hook with A and DE unmodified.

5. Otherwise, if A=1, put the appropriate information about the implementation in registers A, B, and HL, as described in [Section 3.2](#32-steps-of-the-discovery-procedure), and return (do NOT call the old EXTBIO hook) with DE unmodified.

6. Otherwise, decrease A by one, and jump to the old EXTBIO hook with DE unmodified.

Note that given the way the discovery procedure is designed, the implementations installed in the first place have assigned the highest implementation index numbers. Usually this has no impact on the design on client software but it is a good thing to know it.

## 4. The RAM helper

Once an API implementation has been discovered, it is easy for client software to invoke the API routines if the API entry point is located at page 3 (using direct calls) or in a ROM slot (using inter-slot calls, for example via the BIOS function CALSLT). However, for API implementations installed in a mapped RAM segment, it is not so easy to invoke the API entry point. MSX-DOS 2 provides system routines to execute code placed on an arbitrary segment, but the client code must still ensure that the appropriate RAM slot is switched on page 1. For MSX-DOS 1 it is even worse, since no mapper support routines are provided.

To solve this issue, and in order to ease the development of client software, this specification includes the concept of the RAM helper. The RAM helper consists of a set of routines and an optional mappers table that are placed at system RAM on page 3, each having an entry point in the same area. Two of the routines allow to easily perform inter-segment calls, while the other allows to perform inter-segment data reads.

To check for the presence of the RAM helper, and to obtain the address of its routines, EXTBIO must be called with DE=2222h, HL=0, and A=FFh. If the RAM helper is not installed, then HL=0 at output; otherwise the following register values will be returned:

* HL = Address of a jump table in page 3
* BC = Address of the reduced mappers table in page 3 (zero if not provided)
* A = Number of entries in the jump table

The reduced mappers table **must** be provided when [the standard mapper support routines](#appendix-a-the-standard-mapper-support-routines) are **not** present, but is optional otherwise, since the standard mapper support routines already provide a suitable mappers table. When the RAM helper does not provide a mappers table, the value returned in BC by the EXTBIO call must be zero. Client applications should rely on a mappers table being provided by a RAM helper only when mapper support routines are **not** present.

The value returned in A is always 3 as per this specification. This value is provided because future specification versions may define additional routines, and so client applications may detect which specification version the installed RAM helper conforms to.

The jump table routines are as follow:

#### +0: CALL_MAP: Call a routine in a mapped RAM segment

* Input:
  * IYh = Slot number
  * IYl = Segment number
  * IX = Target routine address (must be a page 1 address)
  * AF, BC, DE, HL = Parameters for the target routine
* Output:
  * AF, BC, DE, HL, IX, IY = Parameters returned from the target routine

#### +3: RD_MAP: Read a byte from a RAM segment

* Input:
  * A = Slot number
  * B = Segment number
  * HL = Address to be read from (higher two bits will be ignored)
* Output:
  * A = Data readed from the specified address
  * F, BC, DE, HL, IX, IY preserved

#### +6: CALL_MAPI: Call a routine in a mapped RAM segment, with inline routine identification

* Input:
  * AF, BC, DE, HL = Parameters for the target routine
* Output:
  * AF, BC, DE, HL, IX, IY = Parameters returned from the target routine

The routine is to be called as follows:

    CALL CALLSEG

    CALLSEG:
      CALL <address of CALL_MAPI>
      DB &Bmmeeeeee
      DB <segment number>
      ;no RET is needed here

 where

* `mm` is the mapper slot, as an index (0 to 3) in either the mappers table provided by [the standard mapper support routines](#appendix-a-the-standard-mapper-support-routines) or the reduced mappers table provided by the RAM helper, whichever is available.

* `eeeeee` is the routine to be called, as an index (0 to 63) of a jump table that starts at address 4000h of the segment. That is, 0 means 4000h, 1 means 4003h, 2 means 4006h, etc.

The reduced mappers table is as follows:

* +0: Slot number of the primary mapper
* +1: Maximum segment number of the primary mapper
* +2: Slot number of second mapper
* +3: Maximum segment number of the secondary mapper
* ...

The table contains at least one and at most four entries, one for each of the memory mappers present in the system. Each entry contains the slot number and the maximum available segment number (this will be FEh for 4MB mappers, since segment FFh can't be used, see [Section 2.8](#28-rule-8-avoid-segment-number-ffh)). There's a zero byte after the last entry.

The reduced mappers table is needed when [the standard mapper support routines](#appendix-a-the-standard-mapper-support-routines) are not present and has two uses:

1. It allows to translate mapper indexes to mapper slot numbers when using CALL_MAPI.

2. It informs implementation installers about how many mappers are present in the system and how many segments each one has, without the need to manually perform memory tests. Normally these installers should provide the user the option to decide which segment to use, and default to use the maximum segment number available of one of the available mappers if the user does not provide any segment number. (Note that this if the standard mapper support routines are available, ALL_SEG should be used to properly allocate a segment instead)

The installation code for every API implementation that is to be installed on a mapped RAM segment must check if the RAM helper is installed in the system (by using the procedure based on a EXTBIO call described above). If not, the implementation installer has two options: either install a RAM helper by itslef, appropriately patching the EXTBIO hook so that the RAM helper can be discovered by using the explained procedure; or refuse to install, thus requiring the user to previously install a RAM helper in order to install that implementation. This way, client applications can always safely assume that the RAM helper is installed if implementations that reside in mapped RAM are installed.

How page 3 RAM is allocated in order to install the RAM helper is outside the scope of this specification.

Having a RAM helper installed is mandatory when installing API implementations which reside in mapped RAM, but using it is optional; client software may choose to use its own code for calling the API routines and reading the API data. However, it is recommended to use the RAM helper since it significantly reduces the complexity of client software.

### 4.1. Notes on the routine for calling a routine with inline routine identification

The routine for calling a routine with inline routine identification is intended for helping in the process of patching hooks (mainly the EXTBIO hook) to implementations that do not otherwise need to allocate or modify RAM on page 3. If the implementation has the code for the discovery procedure available at an entry that belongs to a jump table that starts at address 4000h (the start of the segment where it resides), and saves the previous state of the hook in the segment itself, then it is enough to set up the contents of the hook with the five bytes of code needed to call the routine (one CALL and two DB). No further action is required on page 3 at install time.

Note that a double indirection is needed here: a CALL is made to a memory location which in turn contains a CALL to the real routine. The implementation of this routine must increase the stack pointer by two, so that execution resumes at the address placed after the first call, ignoring whatever is placed after the two DB directives. It is necessary to implement it this way, in order to be able to place a call to this routine in the EXTBIO hook, which is just five bytes long.

## 5. Specificationless UNAPI applications

The UNAPI specification is primarily intended for the development of software that conforms to a given UNAPI compliant API specification. However, the infrastructure offered by UNAPI for the implementation and discovery of API implementations may be useful as well to develop normal applications that do not follow any standard, but still need to offer services to and be discoverable by the user (typically, TSRs, Terminate and Stay Resident programs).

For this reason, the concept of specificationless UNAPI applications is introduced. Such applications are identical to regular UNAPI implementations, except for the following:

1. The API identifier is an empty string. That is, when performing the discovery procedure (see [Section 3](#3-api-implementations-discovery-procedure)), a single zero byte must be placed by client software in ARG, and this is what the specificationless application discovery code must search for. The application is then identified exclusively by the implementation name, which becomes the application name.

2. There is no concept of Specification routines vs Implementation-specific routines as defined in [Section 2.4](#24-rule-4-routine-number-ranges). Instead, the whole range of function numbers 1-254 are implementation-specific routines, that is, the specificationless application may define these routines as desired (or define no routines at all). Note however that the API information routine (see [Section 2.5](#25-rule-5-the-api-information-routine--the-implementation-name-and-version)) is still mandatory, and that function number 255 is still reserved.

3. Since specificationless application do not conform to any API specification, the API specification version number provided by the API information routine (see [Section 2.5](#25-rule-5-the-api-information-routine--the-implementation-name-and-version)) does not make sense. This routine should return this value as Version 0.0 (that is, DE=0).

Apart from these differences, the whole set of rules enumerated in this document still apply, including the discovery procedure and the rules that refer to routine registers usage.

Specificationless applications will tipically take the form of TSRs that install on RAM, but this is not mandatory and ROM specificationless applications are allowed as well.

## Appendix A. The standard mapper support routines

Section ["The RAM helper"](#4-the-ram-helper) mentions the _standard mapper support routines_. These are a set of routines (plus a mappers table) that allow a system-wide management of memory mapper segments, so that multiple applications can allocate and use mapped memory without conflicts.

Standard mapper support routines are an integral part of MSX-DOS 2, and as such, they are explained in the _MSX-DOS 2 Program Interface Specification_ document. This section provides a "cheat sheet" of these routines for conveniece.

### The mappers table

To get the mappers table you must call EXTBIO (`0FFCAh`) with the following input registers: `A=0`, `D=4`, `E=1`. If the mapper support routines are available, you'll get the following output:

* A = Slot number of primary mapper - if zero, mapper support routines are not available
* DE = Preserved
* HL = Start address of mapper variable table

The mappers table has one entry for the primary mapper, plus additional entries for other mappers present in the system. Each entry has the following format:

* +0: Slot number of the mapper
* +1: Total number of 16k RAM segments
* +2: Number of free 16k RAM segments
* +3: Number of 16k RAM segments allocated to the system
* +4: Number of 16k RAM segments allocated to the user
* +5...+7: Unused, always zero

A zero byte follows the last entry in the table.

### The mapper support routines

To get the starting address of the mapper support routines you must call EXTBIO (`0FFCAh`) with the following input registers: `A=0`, `D=4`, `E=2`. If the mapper support routines are available, you'll get the following output:

* A = Total number of memory mapper segments - if zero, mapper support routines are not available
* B = Slot number of primary mapper
* C = Number of free segments of primary mapper
* DE = Preserved
* HL = Start address of jump table

The jump table is as follows:

| Offset | Routine | Purpose |
|---|---|---|
|+0h | ALL_SEG | Allocate a 16k segment |
|+3h | FRE_SEG | Free a 16k segment |
|+6h | RD_SEG |  Read byte from segment |
|+9h | WR_SEG |  Write byte to a segment |
|+Ch | CAL_SEG | Inter-segment call, address in registers |
|+Fh | CALLS |   Inter-segment call, address in line after the call instruction |
|+12h |PUT_PH |  Put segment into the page that contains address HL |
|+15h |GET_PH |  Get current segment for the page that contains address HL |
|+18h |PUT_P0 |  Put segment into page 0 |
|+1Bh |GET_P0 |  Get current segment for page 0 |
|+1Eh |PUT_P1 |  Put segment into page 1 |
|+21h |GET_P1 |  Get current segment for page 1 |
|+24h |PUT_P2 |  Put segment into page 2 |
|+27h |GET_P2 |  Get current segment for page 2 |
|+2Ah |PUT_P3 |  Not supported since page 3 must never be changed, does nothing |
|+2Dh |GET_P3 |  Get current segment for page 3 |

Notes:

* Except for ALL_SEG and FRE_SEG, the routines don't check that the supplied segment number actually exist.
* None of the routines do any slot switching, so the required mapper slot must be already visible in the appropriate page. For RD_SEG and WR_SEG this is always page 2.
* The segment in page 3 is never changed by these routines.


#### ALL_SEG: Allocate a 16k segment

* Input:
  * A = 0: allocate user segment
  * A = 1: allocate system segment
  * B = 0: allocate primary mapper
  * B != 0: allocate FxxxSSPP slot address (primary mapper, if 0):
    * xxx = 000: allocate specified slot only
    * xxx = 001: allocate other slots than specified
    * xxx = 010: try to allocate specified slot and, if it failed, try another slot (if any)
    * xxx = 011 try to allocate other slots than specified and, if it failed, try specified slot

* Output:
  * Carry set: no free segments
  * Carry clear: segment allocated
  * A: new segment number
  * B: slot address of mapper slot (0 if called as B=0)

In MSX-DOS 2 user segments are automatically freed when the program allocating them terminates, while system segments must be freed explicitly.


#### FRE_SEG: Free a 16k segment

* Input:
  * A: segment number to free
  * B = 0: primary mapper
  * B != 0: mapper other than primary

* Output:
  * Carry set: error
  * Carry clear: segment freed OK


#### RD_SEG: Read byte from segment

* Input:
  * A: segment number to read from
  * HL: address within this segment

* Output:
  * A: value of byte at that address
  * All other registers preserved


#### WR_SEG: Write a byte to a segment

* Input:
  * A: segment number to write to
  * HL: address within this segment
  * E: value to write
  
* Output: none, corrupts A


#### CAL_SEG: Inter-segment call, address in registers

* Input:
  * IYh: segment number to be called
  * IX = address to call
  * AF, BC, DE, HL passed to called routine

* Output:  
  * AF, BC, DE, HL, IX and IY returned from called routine, all others corrupted


#### CALLS: Inter-segment call, address in line after the call instruction

* Input:
  * AF, BC, DE, HL passed to called routine, other registers corrupted

* Output:  
  * AF, BC, DE, HL, IX and IY returned from called routine, all others corrupted

This routine must be used as follows:

```
CALL  CALLS
DB    SEGMENT
DW    ADDRESS
```

#### PUT_PH: Put segment into the page that contains address HL

* Input:
  * HL = any address within the page that will be changed
  * A = segment number

* Output: none, all registers preserved


#### GET_PH: Get current segment for the page that contains address HL

* Input:
  * HL = any address within the page to be checked

* Output:
  * A = segment number
  * All other registers preserved


#### PUT_P0/1/2/3: Put segment into page 0/1/2/3

* Input:
  * A = segment number

* Output: none, all registers preserved


#### GET_P0/1/2/3: Get current segment for page 0/1/2/3

* Input: none

* Output:
  * A = segment number
  * All others preserved


## Appendix B. Acknowledgements

The original text version of this document was produced using xml2rfc v1.34 (of http://xml.resource.org/) from a source in RFC-2629 XML format.

## Appendix C. Document version history

* Version 0.2

  * Added the "The big picture" section ([Section 1.5](#15-sample-scenario))

* Version 0.3 

  * "The big picture" section ([Section 1.5](#15-sample-scenario)) has been renamed to "Sample scenario"
  * Added the routine to call address 4010h in the "The RAM helper" section ([Section 4](#4-the-ram-helper))

* Version 0.4

  * The maximum length for implementation names has been changed from 64 to 63 characters ([Section 1.4](#14-glossary) and [Section 2.5](#25-rule-5-the-api-information-routine--the-implementation-name-and-version)), so a full length name will be 64 bytes long including the zero termination byte.

  * Slightly modified the explanation of the discovery procedure to make more clear (sort of) the fact that the API identifier must be zero-terminated only when it is copied to ARG (that is, the zero byte is NOT part of the identifier itself). See [Section 3.2](#32-steps-of-the-discovery-procedure) and [Section 3.3](#33-how-to-implement-support-for-the-discovery-procedure).
   
  * Added information about the alternate registers usage in [Section 2.3](#23-rule-3-z80-registers-usage).
   
  * Added the "avoid segment FFh" rule (see [Section 2.8](#28-rule-8-avoid-segment-number-ffh)).
   
  * The usage of "#" as hexadecimal prefix was added in the glossary (see [Section 1.4](#14-glossary)) in the original text document. In this document the "h" suffix is used instead.

* Version 1.0 

  * Added the rule about routine names (see [Section 2.9](#29-optional-rule-9-give-your-routines-a-meaningful-name)).

* Version 1.1

  * Added the concept of specificationless applications (see [Section 5](#5-specificationless-unapi-applications)).
   
  * Modified the RAM helper specification (see [Section 4](#4-the-ram-helper)) so that:
   
    * Installing the RAM helper when it is not installed is now optional (RAM based implementation installers may refuse to install instead).
    
    * When invoking the EXTBIO hook with A=FFh, additional information is returned in registers A and DE.
     
    * The routine for invoking a routine in the segment address 4010h has been replaced by a routine for invoking a routine in a jump table of up to 64 entries.
     
    * A mappers table is now generated in addition to the jump table.

* Version 1.2

    * The mappers table provided by the RAM helper is now mandatory when the standard mapper support routines are **not** present, optional otherwise.

    * Added [Appendix A. The standard mapper support routines](#appendix-a-the-standard-mapper-support-routines)
