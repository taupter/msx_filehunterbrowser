Source: https://github.com/Konamiman/MSX-UNAPI-specification/blob/master/docs/Introduction%20to%20MSX-UNAPI.md

# Introduction to MSX-UNAPI

This document introduces MSX-UNAPI, a standard procedure for defining, discovering and using new APIs (Application Program Interfaces) for MSX computers. For the detailed specification, please read [the MSX-UNAPI specification document](MSX%20UNAPI%20specification%201.1.md).

## 1. Motivation

In the last years, several MSX hobbyists have developed various kinds of amateur hardware for MSX computers. Usually, each of these pieces of hardware comes with a ROM containing an API (Application Program Interface), which consists of a series of routines that allow developers to interact with the hardware.

As each device has its own API, devices are not interchangeable from the software point of view. For example, InterNestor Lite works only with the ethernet card ObsoNET, and it will not work with any other card developed in the future.

The MSX-UNAPI specification aims to solve this problem, by defining a set of rules for creating interchangeable API implementations.

## 2. Key concepts

The complete MSX-UNAPI specification may seem complicated at first glance, but it relies on just a few key concepts, which are enumerated below.

**Note:** In the following text, the terms "API specification" and "API implementation" refer to API specifications and implementations that obey the rules of the MSX-UNAPI specification.

* An **API specification** is a set of routines for performing a set of concrete tasks. Each specification has a short alphanumeric identifier that serves as a signature to uniquely distinguish it from other specifications.

  For example, an API specification for ethernet cards could have the identifier ETHERNET and be composed of three routines: send packet, receive packet and check network state.

* An **API implementation** is the realization in code of an API specification. There may be several implementation of the same API specification, and since all of them implement the same set of routines, they are interchangeable. Each implementation has a short name that serves to distinguish it from other implementations.

  For example, "ObsoNET BIOS" and "DenyoNet BIOS" could be the names of two implementations of the API whose identifier is ETHERNET. A TCP/IP stack prepared to deal with the ETHERNET API could work with both implementations.

* The MSX-UNAPI specification provides a set of minimal rules that must be followed by all compliant API specifications and implementations. This is done to ease the development of software that make use of API implementations.

  The main rules are: API implementation code must be placed in ROM, in mapped RAM or in page 3 RAM; there must be one single call point for all the API routines (routine number is passed in register A); and there must be one routine that informs about the API name and version. All of this is detailed in the MSX-UNAPI specification document.

* More than one implementation of a given API specification may be installed at the same time. The MSX extended BIOS mechanism is used to discover and locate the available implementations.

  Usually, when more than one implementation is found, it does not matter which one is used to perform the tasks offered by the API specification. However, if necessary, implementations can be distinguished thanks to their names.

## 3. Example

This example provides pseudo-code for an hypothetical TCP/IP stack that relies on the ETHERNET API to send and receive data. In the code, the names A, B, C, HL and DE refer to Z80 registers; other names refer to routines or variables. Semicolons (;) indicate that the rest of the line is a comment.

Please refer to [the MSX-UNAPI specification document](MSX%20UNAPI%20specification%201.1.md) for details about how to call API routines and extended BIOS, as well as Z80 registers usage.

    PRINT "Welcome to this wonderful ETHERNET API aware TCP/IP stack!"
    PRINT "Now I'm going to search for ETHERNET API implementations...

    POKE &HF847,"ETHERNET"+0
    A=0
    B=0
    DE=&H2222
    CALL &HFFCA ; The EXTBIO hook

    IF B=0 THEN
       PRINT "Ooops!"
       PRINT "I haven't found any ETHERNET API implementation!"
       END
    ENDIF

    PRINT "I've found "+B+" implementations of the ETHERNET API!"
    PRINT "I'll use implementation with index 1"

    ; Obtain implementation location (address, slot and/or segment)
    ; and as the first task, obtain its name and version

    POKE &HF847,"ETHERNET"+0 ; Not necessary if memory not modified
    A=1 ; The implementation index
    DE=&H2222
    CALL &HFFCA ; The EXTBIO hook
    ApiSlot=A
    ApiSegment=B
    ApiEntry=HL

    A=0 ; 0 is the index for the API information routine
    CALL EXE_UNAPI
    PRINT "The API name is: "+READ_UNAPI(HL)
    PRINT "The API version is: "+B+"."+C

    ; Now assume that per the ETHERNET API specification,
    ; routine 3 returns A=1 if network is available or 0 otherwise

    A=3
    CALL EXE_UNAPI
    IF A=0 THEN
       PRINT "Ooops!  No network!"
       END
    ENDIF

    PRINT "Network OK!  Let's internetwork!"
    ; etc etc...


    ;--- This routine calls the API routine whose index is passed in A

    EXE_UNAPI:
       IF ApiEntry>=&HC000 THEN
          CALL ApiEntry
       ELSE IF ApiSegment=&HFF THEN
          CALL ApiEntry AT SLOT ApiSlot
       ELSE
          CALL ApiEntry AT SEGMENT ApiSegment AT SLOT ApiSlot
       RETURN


    ;--- This routine reads the API memory whose address
    ;--- is passed as parameter, until a zero is found

    READ_UNAPI(Address):
       HL=Address
       String=""
       LOOP:
       IF Address>=&HC000 THEN
          A=PEEK(HL)
       ELSE IF ApiSegment=&HFF THEN
          A=READ (HL) AT SLOT ApiSlot
       ELSE
          A=READ (HL) AT SEGMENT ApiSegment AT SLOT ApiSlot
       ENDIF
       IF A<>0 THEN
          String=String+A
          HL=HL+1
          GOTO LOOP
       RETURN String

## Appendix A. Acknowledgements

The original text version of this document was produced using xml2rfc v1.34 (of http://xml.resource.org/) from a source in RFC-2629 XML format.

## Appendix B. Document version history

* Version 0.2: Done some minor changes proposed by Tanni, in order to clarify the text.





