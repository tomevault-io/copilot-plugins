## go-mib-parser

> Project Outline


# Project goal

This project is a library written in GO that implements a minimal `SMIv2` parser specifically designed for parsing SNMP MIB files.

## Sources

The following sources are to be used as a basis for your understanding of the projects specific domain.

1. <https://en.wikipedia.org/wiki/Management_information_base>
2. <https://www.rfc-editor.org/rfc/rfc2578>
3. <https://www.rfc-editor.org/rfc/rfc2579>

## Project Terms

* ASN.1 = Abstract Syntax Notation One. This is the standard syntax for defining serializable data structures that MIB files are written in.
* SMIv1 = Defines highly structured tables that are used to group the instances of a tabular object (that is, an object that contains multiple variables). Tables are composed of zero or more rows, which are indexed in a way that allows an SNMP manager to retrieve or alter an entire row with a single Get, GetNext, or Set command.
* SMIv2 = The second version of the SMI is described in RFC 2578 and RFC 2579. It enhances and adds to the SMIv1-specific data types, such as including bit strings, network addresses, and counters. Bit strings are defined only in SMIv2 and comprise zero or more named bits that specify a value. Network addresses represent an address from a particular protocol family. Counters are non-negative integers that increase until they reach a maximum value and then return to zero. In SMIv1, a 32-bit counter size is specified. In SMIv2, 32-bit and 64-bit counters are defined. SMIv2 also specifies information modules, which specify a group of related definitions. Three types of SMI information modules exist: MIB modules, compliance statements, and capability statements. MIB modules contain definitions of interrelated managed objects. Compliance statements provide a systematic way to describe a group of managed objects that must be implemented for conformance to a standard. Capability statements are used to indicate the precise level of support that an agent claims with respect to a MIB group. An NMS can adjust its behavior toward agents according to the capabilities statements associated with each agent.
* MIB = Management information base. These are collections of OIDs along with metadata regarding these OIDs, such as name, data type, description, possible values, as well as other OIDs which are children to the current OID.
* OID = Object identifier. These are unique strings of numbers separated by `.` characters. Each unique string represents a single unique OID in a one-to-one relationship. Each number in the `.` separated sequence represents a parent OID in the chain of OIDs leading to the one identified by the full sequence of numbers. For example, the OID `1.3.6.1.4.1.343` translates to the following:

```text
1 ISO
1.3 identified-organization (ISO/IEC 6523),
1.3.6 DoD,
1.3.6.1 internet,
1.3.6.1.4 private,
1.3.6.1.4.1 IANA enterprise numbers,
1.3.6.1.4.1.343 Intel Corporation
```

## Project structure

* All code should be written in go.
* Any source code files that are to be part of the top level API of the library should be put in the root of the project and should use the package name `mib_parser`
* Any source code files that are implementation details should be grouped in directories based on their area of focus. (For example `tokenizer`, `lexer`, `linker`, etc). These grouped files should use the package name equal to the directory name.
* Any files used for testing the source code should be put in `tests/`.
* The directory `mibs/` is pre populated by correctly MIB files and should not be changed. You are supposed to used these files in your tests in order to verify that the library is working correctly.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Olian04)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Olian04)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
