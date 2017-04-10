ops-config-yaml
===============

What is ops-config-yaml?
------------------------
ops-config-yaml is a library that is used by applications to parse and access YAML hardware description files for subsystems in a switch. The ops-config-yaml library also includes a closely tied function to execute i2c commands to devices identified in the devices.yaml file for a subsystem.

What is the structure of the repository?
----------------------------------------
* tests/ contains all of the unit tests of ops-config-yaml based on ops-test-framework
* src/ contains the source files for ops-config-yaml
* include/ contains the header files that applications should include when using ops-config-yaml

What is the license?
--------------------
The ops-config-yaml library may be used under the terms of the Apache 2.0 license. For more details refer to [COPYING](https://git.openswitch.net/cgit/openswitch/ops-config-yaml/tree/COPYING).

What other documents are available?
===================================
For a high level design of ops-config-yaml, refer to [DESIGN](https://www.openswitch.net/documents/dev/ops-config-yaml/DESIGN).
For answers to common questions, read [FAQ](https://git.openswitch.net/cgit/openswitch/ops-config-yaml/tree/FAQ.md).
For the current list of contributors and maintainers, refer to [AUTHORS](https://git.openswitch.net/cgit/openswitch/ops-config-yaml/tree/AUTHORS.md).

For general information about OpenSwitch project refer to [http://www.openswitch.net](http://www.openswitch.net).
