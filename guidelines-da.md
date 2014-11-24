#DA Packaging guidelines

> Draft 0.1

DevAssistant provides an easy extendability through Assistants. These
assistants are normally distributed via the [DevAssistant Package
Index](https://dapi.devassistant.org) in the form of DAP (DevAssistant Package)
files. If you want to include a DAP into the main Fedora repositories, you can
do so. This document outlines the rules involved.

##How to package a DAP

We recommend you use the tool
[dap2rpm](https://github.com/devassistant/dap2rpm), which creates a SPEC file
from your DAP automatically.

##Naming guidelines

* Every (RPM) package name must start with ``dap-``, which is followed by the
  name of the Assistant itself, all lowercase. For example, the package
  ``Openscad`` will be called ``dap-openscad``.
* There is no difference between packages that provide Assistants, and those
  that provide only auxiliary files, snippets, metapackages etc.

##Dependencies

* All packages must Require ``devassistant``.
* All packages must BuildRequire ``devassistant-devel``. (for macros)
* Each package must Require packages that are specified in the file
  ``/meta.yaml`` in the section ``dependencies``. These names, of course, must
  contain the ``dap-`` prefix.

##Macros

* The macro denoting the parent directory where the package files go is
  ``%{assistant_path}``. This currently expands to ``/usr/share/devassistant``.

##Check section

> Idea

* You must run ``da pkg lint %{shortname}-%{version}.dap`` (or similar, if your
  package's name differs) in the ``%check`` section, and the lint must produce
  no errors.

##Install section

* Normally, all files are installed via the ``%install_assistant`` macro
* If installed manually, the package files are installed into
  ``%{buildroot}%{assistant_path}`` in subdirectories conforming to the DAP
  layout, i. e. the ``assistants/crt/foo`` directory is installed into
  ``%{buildroot}%{assistant_path}/assistants/crt/foo``.

##Files section

* Package's files and directories are installed into ``%{assistant_path}``.
* Do not install everything like ``%{assistant_path}/*``, list each folder
  separately (e. g. ``assistants``, ``snippets``, ``icons``, etc.).
* The ``doc`` directory in the DAP should be installed via the ``%doc`` macro,
  in the same ``%{assistant_path}`` directory.

##Sample SPEC

    %global shortname openscad

    Name:           dap-%{shortname}
    Version:        0.0.2
    Release:        1%{?dist}
    Summary:        Create 3D printing projects for OpenSCAD

    License:        GPLv3+ and GPLv2 with exceptions
    URL:            https://github.com/3DprintFIT/dap-openscad
    Source0:        https://dapi.devassistant.org/download/openscad-0.0.2.dap

    BuildRequires:  devassistant-devel
    Requires:       devassistant
    Requires:       dap-common_args
    Requires:       dap-git
    Requires:       dap-github

    %description
    This assistants helps you to create new OpenSCAD project for 3D printing.
    We use it in our 3D printing lab to store our 3D printers on Github.

    Projects created with this assistant have a `Makefile` to build the 3D models
    form OpenSCAD sources.
    To do so, run `make`. You can also generate the images by `make images` or
    print plates with `make arrange`.
    Observe the generated `Makefile` to see all available options.


    %prep
    %setup -q

    %install
    %install_assistant

    %files
    %doc %{assistant_path}/doc/%{shortname}*
    %{assistant_path}/assistants/crt/%{shortname}*
    %{assistant_path}/icons/%{shortname}*

    %changelog
    Wed Nov 19 2014 tradej <tradej@redhat.com> - 0.0.2dev-1
    - Initial package

