## surge

> The format of the profile follows the format of the INI file, with [Section] segments, which are used to divide the different paragraphs and separate the settings.

# .conf file format
Profile content
The format of the profile follows the format of the INI file, with [Section] segments, which are used to divide the different paragraphs and separate the settings.
The configuration lines of each section have their own specific syntax, such as [General], [Replica], and [MITM] sections are simply of the form key = value

[General]
key = value
# [Rule] section
Each rule consists 3 parts: rule type, a matcher (except for the FINAL rule), and a proxy policy:

           TYPE,         VALUE,         POLICY
Example:   DOMAIN-SUFFIX,apple.com,     DIRECT
           IP-CIDR,      192.168.0.0/16,ProxyA
# Ruleset
           TYPE,         VALUE,         
Example:   DOMAIN-SUFFIX,apple.com,     
           IP-CIDR,      192.168.0.0/16

# .list file format
[TYPE],[VALUE]
Example:   DOMAIN-SUFFIX,apple.com,     

只有两段，如果VALUE后面还有内容，则不合法，应该删除

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/broven) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
