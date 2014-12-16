#DAP Packaging guidelines

> Draft 0.2

DevAssistant provides an easy extendability through Assistants. These
assistants are normally distributed via the [DevAssistant Package
Index](https://dapi.devassistant.org) in the form of DAP (DevAssistant Package)
files. If you want to include a DAP into the main Fedora repositories, you can
do so. This document outlines the rules involved.

**Note:** Packaged DAPs only work with DevAssistant 0.10.0 and newer. They can
not be used with DevAssistant versions 0.9.\* packaged in Fedora 21 and
earlier.

##How to package a DAP

We recommend you use the tool
[dap2rpm](https://github.com/devassistant/dap2rpm), which creates a SPEC file
from your DAP automatically. It is packaged in Fedora Rawhide and 21's
repositories as dap2rpm.

##Naming guidelines

* Every (RPM) package name must start with ``dap-``, which is followed by the
  name of the Assistant itself, all lowercase. For example, the DAP named
  ``Openscad`` will be named ``dap-openscad`` in the SPEC file. In this
  document, the second part of the name (i. e. the substring ``openscad``) will
  be referenced as ``%{shortname}``.
* There is no difference between packages that provide Assistants, and those
  that provide only auxiliary files, snippets, metapackages etc.

##Dependencies

* All packages must Require ``devassistant-ui``. This means that the user has
  at least one user interface for DevAssistant installed, so they can use the
  DAP without futher installing anything.
* Requires: ``devassistant`` is not allowed, as that would install more
  packages for DA than the user may intend or need.
* All packages must BuildRequire ``devassistant-devel``. (for macros,
  installation and lint)
* Each package must Require packages that are specified in the file
  ``/meta.yaml`` in the section ``dependencies``. These names, of course, must
  contain the ``dap-`` prefix. These Requires are generated automatically by
  dap2rpm.

##Architecture

* All daps must be architecture independent, i. e. ``BuildArch: noarch``.

##Macros

* The macro denoting the parent directory where the package files go is
  ``%{assistant_path}``. This currently expands to ``/usr/share/devassistant``.

##Prep section

* DAP must be unpacked with ``setup`` like other packages, it is not
  permissible to install the source DAP directly.
* If needed, you may patch the source files in this section.

##Build section

* You must re-pack the (optionally patched) DAP in the ``%build`` section using
  the macro ``%repack_assistant``. This is needed for installing and checking.

##Install section

* Normally, all files are installed via the ``%install_assistant`` macro
* If installed manually, the package files are installed into
  ``%{buildroot}%{assistant_path}`` in subdirectories conforming to the DAP
  layout, i. e. the ``assistants/crt/foo`` directory is installed into
  ``%{buildroot}%{assistant_path}/assistants/crt/foo``.

##Check section

* You must run the ``%check_assistant`` macro in the ``%check`` section, and
  it must pass.
* If run manually, you must run the command ``da pkg lint -w
  %{shortname}-%{version}.dap`` (or similar, if your package's name differs).
  The lint must produce no errors. Warnings are permissible.

##Files section

* Package's files and directories are installed into ``%{assistant_path}``.
* Do not install everything like ``%{assistant_path}/*``, list each folder
  separately (e. g. ``assistants``, ``snippets``, ``icons``, etc.).
* The ``doc`` directory in the DAP should be installed via the ``%doc`` macro,
  in the same ``%{assistant_path}`` directory.
* The ``meta.yaml`` file must be installed as
  ``%{assistant_path}/meta/%{shortname}.yaml``.

##Sample SPEC

    %global shortname openscad

    Name:           dap-%{shortname}
    Version:        0.0.2
    Release:        1%{?dist}
    Summary:        Create 3D printing projects for OpenSCAD

    BuildArch:      noarch

    License:        GPLv3+ and GPLv2 with exceptions
    URL:            https://github.com/3DprintFIT/dap-openscad
    Source0:        https://dapi.devassistant.org/download/%{shortname}-%{version}.dap

    BuildRequires:  devassistant-devel
    Requires:       devassistant-ui
    Requires:       dap-common_args
    Requires:       dap-git
    Requires:       dap-github

    %description
    This assistants helps you to create new OpenSCAD project for 3D printing.
    We use it in our 3D printing lab to store our 3D printers on Github.

    Projects created with this assistant have a `Makefile` to build the 3D models form OpenSCAD sources.
    To do so, run `make`. You can also generate the images by `make images` or print plates with `make arrange`.
    Observe the generated `Makefile` to see all available options.


    %prep
    %setup -q -n %{shortname}-%{version}

    %build
    %repack_assistant

    %install
    %install_assistant

    %check
    %check_assistant

    %files
    %doc %{assistant_path}/doc/%{shortname}
    %{assistant_path}/assistants/crt/%{shortname}*
    %{assistant_path}/files/crt/%{shortname}*
    %{assistant_path}/icons/crt/%{shortname}*
    %{assistant_path}/meta/%{shortname}.yaml

    %changelog
    * Thu Dec 11 2014 Tomas Radej <tradej@redhat.com> - 0.0.2-1
    Initial package

