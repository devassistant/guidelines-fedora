#DA Packaging guidelines

> Draft 0.1

DevAssistant provides an easy extendability through Assistants. These
assistants are normally distributed via the [DevAssistant Package
Index](https://dapi.devassistant.org). If you, however, want to include an
Assistant into the main Fedora repositories, you can do so. This document
outlines the rules involved.

##How to package an Assistant

We recommend you use the tool
[dap2rpm](https://github.com/devassistant/dap2rpm), which creates a SPEC file
from your package automatically.

##Naming guidelines

* Every package name must start with ``dap-``, which is followed by the name of
  the Assistant itself, all lowercase. For example, the package ``Openscad``
  will be called ``dap-openscad``.

* There is no difference between packages that provide Assistants, and those
  that provide only auxiliary files, snippets, metapackages etc.

##Dependencies

* All packages must depend on ``devassistant``.
* Each package must have an RPM dependency on packages that are specified in
  the file ``/meta.yaml`` in the section ``dependencies``.
