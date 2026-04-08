## overlook-diagnostics

> This project contains a diagnostic script that is used to

This project contains a diagnostic script that is used to
gather information from various services on the host where
it is run.

Key things to remember about this project:

 - The script is contained in the `overlook-diagnostics.sh` file.

 - The script is written for compatibility with GNU Bash.

 - Any changes to the script should pass the following tests:
    * `shellcheck overlook-diagnostics.sh`

 - The script will run, often as `root`, on one of the following
   Linux flavors:
     * Debian or Ubuntu
     * RedHat Enterprise Linux, or a compatible clone

 - Due to the elevated privileges involved, logic added to
   the script should follow these guidelines:
     * Prioritize safety above all else
     * Follow bash best practices documented at:
         - https://google.github.io/styleguide/shellguide.html
         - https://mywiki.wooledge.org/BashGuide
     * The script will often gather diagnostics from an
       unhealthy system and commands should fail gracefully
     * Side-effects should be minimized. Ideally, the only
       side-effect produced by the script is the creation
       of an output file.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/overlookinfra)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/overlookinfra)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
