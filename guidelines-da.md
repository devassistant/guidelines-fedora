#DAP Packaging guidelines

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

* Every (RPM) package name must start with ``devassistant-dap-``, which is
  followed by the name of the Assistant itself, all lowercase. For example, the
  DAP named ``Openscad`` will be named ``devassistant-dap-openscad`` in the
  SPEC file. In this document, the second part of the name (i. e. the substring
  ``openscad``) will be referenced as ``%{shortname}``.
* There is no difference between packages that provide Assistants, and those
  that provide only auxiliary files, snippets, metapackages etc.

##Dependencies

* All packages must Require ``devassistant-ui``. This means that the user has
  at least one user interface for DevAssistant installed, so they can use the
  DAP without futher installing anything. No version specification is necessary
  as DevAssistant 0.10.1 is the first version ever to provide
  ``devassistant-ui``.
* ``Requires: devassistant`` is not allowed, as that would install more
  packages for DA than the user may intend or need.
* All packages must have ``BuildRequires: devassistant-devel``. (for macros,
  installation and checking)
* Each package must Require packages that are specified in the file
  ``/meta.yaml`` in the section ``dependencies``. These names, of course, must
  contain the ``devassistant-dap-`` prefix. These Requires are generated
  automatically by dap2rpm.

##Architecture

* All daps must be architecture independent, i. e. ``BuildArch: noarch``.

##Macros

* The macro denoting the parent directory where the package files go is
  ``%{assistant_path}``. This currently expands to ``/usr/share/devassistant``.
* The macro definitions of ``%install_assistant``, ``%repack_assistant``, and
  ``%check_assistant`` are provided by the package ``devassistant-devel``.

##Prep section

* DAP must be unpacked with ``setup`` like other packages, it is not
  permissible to install the source DAP directly.
* If needed, you may patch the source files in this section.

##Build section

* You must re-pack the (optionally patched) DAP in the ``%build`` section using
  the macro ``%repack_assistant``. This is needed for installing and checking.

##Install section

* Files are installed via the ``%install_assistant`` macro
* Using the above macro automatically creates a list of files stored in the
  file ``dap-list``, which can then be used in the ``%files`` section.
* If installed manually, the package files are installed into
  ``%{buildroot}%{assistant_path}`` in subdirectories conforming to the DAP
  layout, i. e. the ``assistants/crt/foo`` directory is installed into
  ``%{buildroot}%{assistant_path}/assistants/crt/foo``.

##Check section

* You must run the ``%check_assistant`` macro in the ``%check`` section, and
  it must pass.
* If run manually, you must run the command ``da pkg lint -w
  %{shortname}-%{version}.dap`` (or similar, if your package's name differs).
  The run must produce no errors. Warnings are permissible.

##Files section

* If the macro ``%install_assistant`` was used in the ``%install`` section, it
  is sufficient to provide the list of files to the ``%files`` section like
  this: ``%files -f dap-files``. This is the default situation when using
  dap2rpm.
* If done manually:
    * Package's files and directories are installed into ``%{assistant_path}``.
    * Do not install everything like ``%{assistant_path}/*``, list each folder
      separately. That includes subdirectories in ``%{assistant_path}/assistants``,
      as shown in these examples:
        * Bad: ``%{assistant_path}/assistants/*``
        * Good: ``%{assistant_path}/assistants/crt/%{shortname}*``
    * The ``doc`` directory in the DAP should be installed via the ``%doc`` macro,
      in the same ``%{assistant_path}`` directory.
    * The ``meta.yaml`` file must be installed as
      ``%{assistant_path}/meta/%{shortname}.yaml``.

## Directory ownership

* The package ``devassistant-core`` owns the directory ``%{assistant_path}``
  itself and the following subdirectories:
    1. ``%{assistant_path}/{assistants,doc,files,icons,meta,snippets}``
    2. ``%{assistant_path}/{assistants}/{crt,twk,prep,extra}``
    3. ``%{assistant_path}/files/{crt,twk,prep,extra,snippets}``
    4. ``%{assistant_path}/icons/{crt,twk,prep,extra}``
* The RPM-packaged DAP must own the following files/directories if and only
  if they are present in the upstream DAP:
    * directories named ``%{shortname}`` in directories listed in point 2
      and 3 of the previous bullet.
        * Example 1: ``%{assistant_path}/assistant/crt/%{shortname}``
        * Example 2: ``%{assistant_path}/files/snippets/%{shortname}``
    * files named ``%{shortname}.yaml`` in the directories
      listed in point 2.
    * a directory named ``%{shortname}`` in the directory
      ``%{assistant_path}/doc/``
    * a file named ``%{shortname}.yaml`` in the directory
      ``%{assistant_path}/meta/``
    * a file named ``%{shortname}.$SUFFIX``, where ``$SUFFIX`` is an image
      file suffix (preferably PNG or SVG), in the directories listed in point
      4.

##Sample SPEC

    %global shortname openscad

    Name:           devassistant-dap-%{shortname}
    Version:        0.0.2
    Release:        1%{?dist}
    Summary:        Create 3D printing projects for OpenSCAD

    BuildArch:      noarch

    License:        GPLv3+ and GPLv2 with exceptions
    URL:            https://github.com/3DprintFIT/dap-openscad
    Source0:        https://dapi.devassistant.org/download/%{shortname}-%{version}.dap

    BuildRequires:  devassistant-devel
    Requires:       devassistant-ui
    Requires:       devassistant-dap-common_args
    Requires:       devassistant-dap-git
    Requires:       devassistant-dap-github

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

    %files -f dap-files

    %changelog
    * Thu Dec 11 2014 Tomas Radej <tradej@redhat.com> - 0.0.2-1
    Initial package

##Sample %files section when specified manually

    %files
    %doc %{assistant_path}/doc/%{shortname}
    %{assistant_path}/assistants/crt/%{shortname}*
    %{assistant_path}/files/crt/%{shortname}*
    %{assistant_path}/icons/crt/%{shortname}*
    %{assistant_path}/meta/%{shortname}.yaml

