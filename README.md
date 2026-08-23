# eSim 2.5 Installation on Ubuntu 25.04

## FOSSEE eSim — Screening Task 4

This repository contains the work completed for **Task 4: eSim Upgradation
(CSE and Related Fields)** of the FOSSEE eSim screening task.

The objective of this task was to install **eSim 2.5 on Ubuntu 25.04**,
identify the installation and compatibility problems encountered during the
process, modify the installer where required, and verify that the complete
installation could proceed successfully.

The investigation was performed one error at a time. After each correction,
the installer was rerun so that the next genuine installation blocker could
be identified.

---

# Author

**Neeraj Kumar**

B.Tech CSE (CyberSecurity and DigitalForensics), Third Year
VIT Bhopal University
Kothrikalan, Sehore, Madhya Pradesh, India

---

# System Environment

| Component | Details |
|---|---|
| Operating System | Ubuntu 25.04 (Plucky) |
| Architecture | 64-bit |
| Shell | Bash |
| Package Manager | APT |
| eSim | 2.5 |
| KiCad | 8.0.8 |
| Ngspice | 35 |
| GHDL | 4.1.0 |
| LLVM | 20.1.2 |
| Verilator | 4.210 |
| Python | 3.13.3 |
| Git | 2.48.1 |

---

# Repository

GitHub fork:

https://github.com/NeerajKumar011/eSim

Working branch:

`installers`

The installer modifications described in this README were made on the
`installers` branch.

---

# Repository Setup

The eSim repository was cloned from the candidate's GitHub fork.

### Commands used

```bash
$ git clone git@github.com:NeerajKumar011/eSim.git

$ cd eSim

$ git checkout installers

Verify the repository and branch:

$ git remote -v

Expected:

origin  git@github.com:NeerajKumar011/eSim.git (fetch)
origin  git@github.com:NeerajKumar011/eSim.git (push)
$ git branch --show-current

Expected:

installers
eSim 2.5 Installation

The official eSim 2.5 release package was used for the installation test.

The native Ubuntu installation procedure is:

$ unzip eSim-2.5.zip

$ cd eSim-2.5

$ chmod +x install-eSim.sh

$ ./install-eSim.sh --install

The installer was also modified and tested from the cloned repository:

$ cd ~/eSim/Ubuntu

$ chmod +x install-eSim.sh

$ ./install-eSim.sh --install

The corrected installer was subsequently tested against the released
eSim-2.5 package in:

~/Downloads/eSim-2.5
Installation Process

The installation was handled using the following procedure:

Start eSim installation
        |
        v
Run installer
        |
        v
Observe first error
        |
        v
Identify source of error
        |
        v
Apply smallest required correction
        |
        v
Check the modified script
        |
        v
Rerun installer
        |
        v
Confirm previous error is gone
        |
        v
Continue to next installation stage

This process was repeated until the installer completed successfully.

Issues Identified During Installation

Six major installation/compatibility errors were identified:

Missing fi in install-eSim.sh
/: Is a directory caused by misplaced configuration writes
Installer exits when KiCad 8.0 is already installed
NGHDL rejects Ubuntu 25.04
libcanberra-gtk-module has no installation candidate
GHDL 4.1.0 rejects LLVM 20.1.2
Error 1 — Missing fi in install-eSim.sh
Observed Error
./install-eSim.sh: line 70: syntax error near unexpected token `}'
./install-eSim.sh: line 70: `}'
Cause

The top-level installer contained an if [[ -f "$SCRIPT" ]] block but did
not close the block with fi.

Bash therefore reached the closing } of the function while the if
statement was still incomplete.

Correction

The missing fi was added immediately after:

bash "$SCRIPT" "$ARGUMENT"

The resulting structure was:

if [[ -f "$SCRIPT" ]]; then
    echo "Running script: $SCRIPT $ARGUMENT"
    bash "$SCRIPT" "$ARGUMENT"
fi
Verification
$ bash -n install-eSim.sh

The syntax check completed successfully without errors.

Error 2 — /: Is a directory
Observed Error
./install-eSim.sh: line 62: /: Is a directory
./install-eSim.sh: line 63: /: Is a directory
./install-eSim.sh: line 64: /: Is a directory
./install-eSim.sh: line 65: /: Is a directory
./install-eSim.sh: line 66: /: Is a directory
./install-eSim.sh: line 67: /: Is a directory
./install-eSim.sh: line 68: /: Is a directory
Cause

Seven configuration-writing commands were present in the top-level
install-eSim.sh even though they belonged to the Ubuntu-specific installer
where the required configuration variables were defined.

The affected commands wrote values such as:

[eSim]
eSim_HOME
LICENSE
KicadLib
IMAGES
VERSION
MODELICA_MAP_JSON

Because the required destination variables were not correctly initialized in
that context, the shell attempted to redirect output to /.

Correction

The seven misplaced configuration-writing lines were removed from:

Ubuntu/install-eSim.sh

The lines removed were:

echo "[eSim]" >> $config_dir/$config_file
echo "eSim_HOME = $eSim_Home" >> $config_dir/$config_file
echo "LICENSE = %(eSim_HOME)s/LICENSE" >> $config_dir/$config_file
echo "KicadLib = %(eSim_HOME)s/library/kicadLibrary.tar.xz" >> $config_dir/$config_file
echo "IMAGES = %(eSim_HOME)s/images" >> $config_dir/$config_file
echo "VERSION = %(eSim_HOME)s/VERSION" >> $config_dir/$config_file
echo "MODELICA_MAP_JSON = %(eSim_HOME)s/library/ngspicetoModelica/Mapping.json" >> $config_dir/$config_file
Verification
$ bash -n install-eSim.sh

$ git diff --check

The corrected installer was then rerun using the official eSim 2.5 package.

The previous /: Is a directory messages no longer appeared and the installer
progressed to the next stage.

Error 3 — Installer Stops When KiCad 8.0 Is Already Installed
Observed Behaviour

The installer reached:

Installing KiCad...........................
Ubuntu 25.04 detected.
KiCad 8.0 is already installed.

and then terminated instead of continuing with the remaining installation
steps.

Cause

The installKicad() function contained:

exit 0

exit 0 terminates the complete shell process. It does not simply leave the
installKicad() function.

Therefore the following installation stages were not reached.

Correction

The command was changed from:

exit 0

to:

return 0

in:

Ubuntu/install-eSim-scripts/install-eSim-25.04.sh
Verification
$ nl -ba install-eSim-scripts/install-eSim-25.04.sh | sed -n '130,140p'

$ bash -n install-eSim-scripts/install-eSim-25.04.sh

The installer was rerun.

Instead of stopping after:

KiCad 8.0 is already installed.

it continued to the KiCad library step and then to:

Installing NGHDL

This confirmed that the installer was no longer terminating prematurely.

Error 4 — NGHDL Does Not Support Ubuntu 25.04
Observed Error
Detected Ubuntu Version:
Unsupported Ubuntu version: 25.04 ()
Cause

The NGHDL installer supplied with eSim 2.5 did not have a supported Ubuntu
25.04 installation path.

The main eSim installer successfully reached NGHDL, but the NGHDL dispatcher
rejected Ubuntu 25.04.

Investigation

The NGHDL scripts supplied with the eSim 2.5 package were inspected to identify
the Ubuntu-version check.

$ cd ~/Downloads/eSim-2.5

$ grep -RniE 'Unsupported Ubuntu version|Detected Ubuntu Version|VERSION_ID|lsb_release' nghdl/install-nghdl.sh nghdl/install-nghdl-scripts/

$ ls -lah nghdl/install-nghdl-scripts

$ nl -ba nghdl/install-nghdl.sh | sed -n '1,140p'
Correction

The Ubuntu 25.04 eSim installer was changed to directly use the bundled
Ubuntu 24.04 NGHDL installer:

$ bash install-nghdl-scripts/install-nghdl-24.04.sh --install

instead of the unsupported-version dispatcher.

Verification
$ bash -n install-eSim-scripts/install-eSim-25.04.sh

The corrected installer was run again.

The previous:

Unsupported Ubuntu version: 25.04

error disappeared.

The installer entered the NGHDL dependency installation stage and proceeded
through:

Installing Make
Installing GNAT
Installing LLVM
Installing Clang
Installing Zlib1g-dev
Installing Gtk Canberra modules

The next distinct error was the libcanberra-gtk-module dependency issue.

Error 5 — libcanberra-gtk-module Has No Installation Candidate
Observed Error
Installing Gtk Canberra modules...........................

Error: Package 'libcanberra-gtk-module' has no installation candidate

Error! Kindly resolve above error(s) and try again.
Aborting Installation...
Cause

The bundled NGHDL Ubuntu 24.04 installer requested:

sudo apt install -y libcanberra-gtk-module libcanberra-gtk3-module

Ubuntu 25.04 did not provide a package candidate for the legacy
libcanberra-gtk-module package.

The related GTK3 package was available.

Dependency Investigation

The package availability was checked directly:

$ apt-cache policy libcanberra-gtk-module

$ apt-cache search libcanberra | grep -E 'gtk|canberra'

$ apt-cache policy libcanberra-gtk3-module

$ grep -n -A4 -B2 'canberra' install-nghdl-scripts/install-nghdl-24.04.sh

$ apt policy

The results established that:

libcanberra-gtk-module
Candidate: (none)

while:

libcanberra-gtk3-module

was available.

Additional Investigation

The separate FOSSEE NGHDL repository was cloned to inspect the NGHDL installer
source related to this dependency:

$ git clone https://github.com/FOSSEE/nghdl.git

$ cd ~/nghdl

$ git checkout installers

This repository was used for investigation. It was not used as a replacement
for the official eSim 2.5 installation package.

Correction

The unavailable GTK2 package reference was removed from the bundled NGHDL
Ubuntu 24.04 installer:

$ cd ~/eSim/Ubuntu

$ sed -i "s/libcanberra-gtk-module //" install-nghdl-scripts/install-nghdl-24.04.sh

The available GTK3 dependency remains unchanged.

Verification
$ bash -n install-eSim-scripts/install-eSim-25.04.sh

$ git diff --check

The installer was rerun.

The libcanberra-gtk-module error no longer occurred and the installation
progressed to the GHDL build stage.

The next distinct error was:

Unhandled version llvm 20.1.2
Error 6 — GHDL 4.1.0 Rejects LLVM 20.1.2
Observed Error

During the GHDL configuration stage:

Use full IEEE library
Build machine is: x86_64-linux-gnu
Unhandled version llvm 20.1.2

Error! Kindly resolve above error(s) and try again.
Aborting Installation...
Cause

GHDL 4.1.0's configure script only recognized older LLVM versions and did not
include the LLVM 20.x versions supplied by Ubuntu 25.04.

The original version checks stopped at:

check_version 18.1 $llvm_version ||
Investigation

The relevant GHDL configure section was inspected:

$ grep -n -A5 -B2 'check_version 18.1' nghdl/ghdl-4.1.0/configure

The original configure script did not contain checks for LLVM 20.0 or 20.1.

Correction

The installer was modified so that, immediately before the GHDL configure
step, the GHDL configure script also accepts LLVM 20.0 and LLVM 20.1.

The resulting checks are:

check_version 18.1 $llvm_version ||
check_version 20.0 $llvm_version ||
check_version 20.1 $llvm_version ||

The modification was made in:

Ubuntu/install-eSim-scripts/install-eSim-25.04.sh
Verification

Check the installer syntax:

$ bash -n install-eSim-scripts/install-eSim-25.04.sh

$ git diff --check

After the installation completed:

$ which ghdl
/usr/local/bin/ghdl
$ ghdl --version

The installed version reported:

GHDL 4.1.0 (tarball) [Dunoon edition]
Compiled with GNAT Version: 14.2.0
llvm 20.1.2 code generator

A basic VHDL analysis and elaboration test was also performed:

$ ghdl -a /tmp/test.vhd

$ ghdl -e test

$ echo $?
0

This confirmed that GHDL 4.1.0 was not only installed but functional with
LLVM 20.1.2.

Final Installation Result

After applying the six fixes, the installation completed successfully.

The installer reached:

NGHDL installed successfully
...
-------- eSim Installed Successfully --------
Type "esim" in Terminal to launch it

eSim was then launched successfully.

$ which esim
/usr/bin/esim
Final Verification

The installed tools were checked individually.

eSim
$ which esim
/usr/bin/esim

The eSim GUI launched successfully.

Ngspice
$ which ngspice
/usr/bin/ngspice
$ ngspice -v

Reported:

ngspice-35

A real resistor-divider circuit was simulated and produced:

v(in)  = 5.000000e+00
v(out) = 2.500000e+00
Ngspice exit=0
GHDL
$ which ghdl
/usr/local/bin/ghdl

GHDL 4.1.0 with LLVM 20.1.2 was confirmed and a basic VHDL design was
successfully analyzed and elaborated.

Verilator
$ which verilator
/usr/local/bin/verilator

A minimal Verilog design was checked using:

$ verilator --lint-only test.v

and returned exit code 0.

KiCad
$ which kicad-cli
/usr/bin/kicad-cli

$ kicad-cli --version
8.0.8

KiCad was also launched successfully from the graphical environment.

eSim GUI Verification

After installation, eSim was launched and the following workflow was tested:

eSim started successfully.
The eSim workspace was created.
A test project named test25 was created.
KiCad was opened from the eSim workflow.
KiCad's schematic editor was successfully launched.

The initial "project does not contain any KiCad netlist" message encountered
while testing an empty project was treated as expected behavior because the
project did not yet contain a schematic. It was not considered an additional
installation error.

Modified Files

The principal source changes for Ubuntu 25.04 are located in:

Ubuntu/install-eSim.sh
Ubuntu/install-eSim-scripts/install-eSim-25.04.sh

The changes address:

Bash syntax
Incorrect configuration handling
KiCad installation flow
NGHDL Ubuntu-version handling
GTK Canberra dependency compatibility
GHDL/LLVM 20 compatibility
Commit History

The fixes were committed during the debugging process.

f4f39a58  fix: add missing fi in Ubuntu installer
454326ae  fix: remove misplaced config writes from Ubuntu installer
ba3b6596  fix: continue installation when KiCad 8 is already installed
e25f0510  fix: support NGHDL installation on Ubuntu 25.04
78b38210  fix: update NGHDL GTK dependency for Ubuntu 25.04
ab0ea396  Fix eSim installation on Ubuntu 25.04

The final Error 6 compatibility change is contained in commit:

ab0ea396
What Was Learned

The main compatibility problems were not caused by one missing package alone.
They came from assumptions in the existing installer that were no longer valid
on Ubuntu 25.04.

The most important findings were:

The top-level Bash installer contained a syntax error.
Configuration writes were placed in the wrong script context.
exit 0 was incorrectly used inside the KiCad installation function.
NGHDL did not recognize Ubuntu 25.04.
An obsolete GTK2 Canberra dependency was still requested.
GHDL 4.1.0 did not recognize the LLVM 20.1.2 version supplied by Ubuntu 25.04.

These issues were fixed individually rather than bypassing the installation
process or replacing the complete toolchain.

Result

eSim 2.5 was successfully installed and launched on Ubuntu 25.04.

The integrated components were also verified:

eSim       → working
KiCad      → 8.0.8
Ngspice    → 35
GHDL       → 4.1.0 + LLVM 20.1.2
Verilator  → 4.210

The final installation was validated both at the command line and through the
eSim graphical interface.

Documentation

A detailed report containing the complete error-by-error investigation,
correction steps, verification results, and screenshots is provided separately
for the Task 4 submission.

Repository Link

GitHub Fork:

https://github.com/NeerajKumar011/eSim

Installer Branch:

https://github.com/NeerajKumar011/eSim/tree/installers

Conclusion

This work demonstrates the process of adapting an existing Linux installer to
a newer Ubuntu release by reproducing failures, identifying their actual
causes, making targeted source changes, and verifying each correction before
continuing with the installation.

The final result is a working eSim 2.5 installation on Ubuntu 25.04 with the
major integrated simulation and design tools verified successfully.


### Why this version is stronger

It improves on the reference README in several important ways:

- It documents the **six actual errors**, not generic or unrelated problems.
- It separates **error investigation** from the **actual correction**.
- It shows the **before/after logic** where that matters.
- It records the **exact files changed**.
- It documents **verification**, not just “installation successful.”
- It distinguishes the **official eSim 2.5 package** from the separate NGHDL repository used for Error 5 investigation.
- It demonstrates that the final environment was tested with **real GHDL, Ngspice, Verilator, KiCad, and eSim operations**, rather than only checking whether binaries existed.

The recovered history confirms the six-error progression and the corresponding verification sequence. :contentReference[oaicite:1]{index=1} :contentReference[oaicite:2]{index=2}

Before putting this on GitHub, I would make one final pass against your actual working tree so the **file names and commit hashes in the README exactly match what is currently pushed**.
