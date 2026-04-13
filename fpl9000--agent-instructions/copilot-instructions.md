## agent-instructions

> - This computer is a Windows 11 system with Cygwin Bash installed.

# Command Execution

- This computer is a Windows 11 system with Cygwin Bash installed.

- The `run_shell_command` tool that you use to execute shell commands uses `cmd.exe`, because this is a Windows system, but you prefer to execute shell commands using Cygwin Bash.  The next item explains how you will use Bash instead of `cmd.exe`.

- When you need to execute a shell command, always execute it by running command `bash gemini_tmp.sh`, where `gemini_tmp.sh` is a Bash shell script written by you that contains the Bash command(s) you need to execute.  Additional details:

  - Even though tool `run_shell_command` uses `cmd.exe` to execute commands, the command you provide to the tool explicitly invokes `bash`, so `cmd.exe` is simply used to "trampoline" into Bash.

  - Most Linux commands are available in the Bash shell, such as `cd`, `ls`, `grep`, `find`, `wc`, `cp`, `mv`, `rm`, `sed`, `awk`, `sha1sum`, `sha256sum`, and many others.

  - The last line of script `gemini_tmp.sh` should be `/bin/rm "$0"`, which deletes the script so you don't have to delete it.

  - If you need to execute the `rm` command using Bash, always execute it using its full pathname (`/bin/rm`).  This avoids a confirmation prompt displayed by my personal `rm` wrapper script.


## Pathnames in Bash Commands

- When giving pathnames to commands executed by the Bash shell, always use forward slashes as the directory separator, even though this is a Windows system.

- Absolute pathnames in Bash commands should start with a Windows drive letter and a forward slash. For example: `C:/path/to/file`.


## Executing Python Code

- If you need to execute Python code, Python version 3.13.3 is available by running command `python C:/temp/gemini.py`, where `C:/temp/gemini.py` is a Python script written by you and containing the Pythond command(s) you need to execute.

- As with Bash commands, you do not need to delete `C:/temp/gemini.py` after executing it, but the next time you write Python code into that file, you must overwrite all of its contents with the new commands.

- If the Python code in file `C:/temp/gemini.py` has pathnames embedded in the code, the pathnames should follow the same rules as above for pathnames in Bash commands.


## Executing MinGW/MSYS2 Applications

- If you need to execute MinGW/MSYS2 applications, first modify environment variable `PATH` to have folder `/cygdrive/c/apps/msys64/mingw64/bin` at the front.

  - Do this by prefixing the command with a temporary assignment to `PATH`, as follows: `PATH="/cygdrive/c/apps/msys64/mingw64/bin:$PATH" COMMAND ARG1 ARG2 ...`.  Do not make persistent changes to `PATH`.


# Creating and Editing Files

## Newline Conventions

- New files that you create should use Unix-style newlines (a single line-feed character) instead of Windows-style newlines (a carriage return and line-feed pair).

- But when you modify existing files, you must use the same newline convention as the rest of the file.

- Never convert the entirety of a existing file from one newline convention to the other.


## Writing Source Code

- Keep lines of source code less than 100 columns wide.

- Avoid single-character identfiers.  In loops, use identifiers such as `index`, `counter`, and `loopCount` instead of single-character identifiers.

- When adding imports to Go code, you must add the imports along with the code that references those imports, otherwise the intelligent editor in VSCode will remove the unreferenced imports.

- All scripts that you create must be Bash scripts.  Never write a Windows batch script unless explicitly instructed to do so.

- In Bash scripts, all variable names must be fully uppercase.  For example: `COUNT=0`, `FILENAME="file.txt"`, etc.

- In Bash scripts, local variables in Bash functions must start with a leading underscore to avoid accidentally shadowing global variables.  For example: `local _COUNTER=0`.

- In Bash scripts use the new test command (`[[ ... ]]`) instead of the traditional test command (`[ ... ]`).  Be careful to use the proper argument syntax for the new test command, such as using `&&` instead of `-a` to indicate Boolean AND operations, and `||` instead of `-o` to indicate Boolean OR operations, as well as the other syntax differences required by the test command.


## Comments in Source Code

- Always write well-commented source code.

- Comments should be complete sentences that start with a capital letter and end with a period.

- Sentences in comments should be separated by two spaces for readability.

- Put comments on the line above the code they reference, rather than on the same line.

- Comments can appear on the same line as code only if the comment is very short.  In this case, the comment is exempt from the rule that it must be a complete sentence.

- Comments should explain the purpose and rationale of the code and not simply restate what the code does.

- Do not talk to the user through comments.

- Aim to have nearly the same number of lines of comments as lines of code.  This is a guideline not a hard and fast rule.

- Do not comment trivial code, such as Python and Go `import` statements or the initialization of local variables, unless the comment explains something important for a developer to understand.


# Building Applications

- When running commands to build executables, always make the executable name end with `.exe`, because this is a Windows system.

- When building a graphical Go application that uses the Fyne GUI toolkit, always modify environment variable `PATH` to use MinGW tools as described above in section [Executing MinGW/MSYS2 Applications](#executing-mingwmsys2-applications).

- When building a graphical Go application, always pass switch `-ldflags "-H windowsgui"` so that the application does not create a console window when it is launched.

- When using GCC to build a graphical application, always pass switch `-Wl,--subsystem,windows` so that the application does not create a console window when it is launched.


# Testing Graphical Applications

- You have no way to see the GUI of a graphical application, which makes it difficult for you to test graphical applications.

- Thus, you must test graphical applications as follows:

  1. Tell the user to test the graphical application's GUI before you launch it.

  2. Launch the graphical application, and wait for it to terminate.  If it produces any error messages on standard error or standard output, you will see them just as you see the output of a non-graphical application.

  3. After the app terminates, ask the user for their testing feedback, then wait for the user to enter their feedback.

  4. Take the appropriate actions based on the user's feedback and any output produced by the application.


# Mathematical Caclulations

- When you need to perform mathematical calculations while thinking or planning, always use Python to do the math.  This guarantees mathematical correctness in your reasoning and responses.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fpl9000)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/fpl9000)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
