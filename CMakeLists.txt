cmake_minimum_required(VERSION 2.8)

# Inspired and referenced from https://blog.kitware.com/cmake-and-the-default-build-type
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    message(STATUS "Setting build type to 'Release' as none was specified.")
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release"
        "MinSizeRel" "RelWithDebInfo")
endif()

project(scap-security-guide NONE)

set (CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${PROJECT_SOURCE_DIR}/cmake")
# This is set to silence GNUInstallDirs warning about no language being used with cmake
set(CMAKE_INSTALL_LIBDIR "/nowhere")
include(GNUInstallDirs)
include(FindPythonModule)

if ( "$ENV{PYTHONPATH}" STREQUAL "" )
    set(ENV{PYTHONPATH} "${PROJECT_SOURCE_DIR}")
else()
    set(ENV{PYTHONPATH} "${PROJECT_SOURCE_DIR}:$ENV{PYTHONPATH}")
endif()

set(SSG_CONTENT_INSTALL_DIR "${CMAKE_INSTALL_DATADIR}/xml/scap/ssg/content")
set(SSG_GUIDE_INSTALL_DIR "${CMAKE_INSTALL_DOCDIR}/guides")
set(SSG_TABLE_INSTALL_DIR "${CMAKE_INSTALL_DOCDIR}/tables")
set(SSG_ANSIBLE_ROLE_INSTALL_DIR "${CMAKE_INSTALL_DATADIR}/scap-security-guide/ansible")
set(SSG_BASH_ROLE_INSTALL_DIR "${CMAKE_INSTALL_DATADIR}/scap-security-guide/bash")
set(SSG_KICKSTART_INSTALL_DIR "${CMAKE_INSTALL_DATADIR}/scap-security-guide/kickstart")

set(SSG_MAJOR_VERSION 0)
set(SSG_MINOR_VERSION 1)
set(SSG_PATCH_VERSION 43)
set(SSG_VERSION "${SSG_MAJOR_VERSION}.${SSG_MINOR_VERSION}.${SSG_PATCH_VERSION}")

set(SSG_VENDOR "ssgproject" CACHE STRING "Specify the XCCDF 1.2 vendor string.")

set(SSG_TARGET_OVAL_MAJOR_VERSION "5" CACHE STRING "Which major version of OVAL are we targetting. Only 5 is supported at the moment.")
set(SSG_TARGET_OVAL_MINOR_VERSION "11" CACHE STRING "Which minor version of OVAL are we targetting. Possible choices are 10 or 11.")
set(SSG_TARGET_OVAL_VERSION "${SSG_TARGET_OVAL_MAJOR_VERSION}.${SSG_TARGET_OVAL_MINOR_VERSION}")

option(SSG_OVAL_SCHEMATRON_VALIDATION_ENABLED "If enabled, schematron validation will be performed as part of the ctest tests. Schematron takes a lot of time to complete but can find more issues than just plain XSD validation." TRUE)
option(SSG_SHELLCHECK_BASH_FIXES_VALIDATION_ENABLED "If enabled, shellcheck validation of bash fixes will be performed as part of the ctest tests. Shellcheck tests don't pass right now, this option is discouraged until that's fixed." FALSE)
option(SSG_LINKCHECKER_VALIDATION_ENABLED "If enabled, linkchecker will be used to validate URLs in all the HTML guides and tables." TRUE)
option(SSG_SVG_IN_XCCDF_ENABLED "If enabled, the built XCCDFs will include the SVG SCAP Security Guide logo." TRUE)
option(SSG_SEPARATE_SCAP_FILES_ENABLED "If enabled, separate SCAP files (OVAL, XCCDF, CPE dict, ...) will be installed alongside the source data-streams" TRUE)
option(SSG_JINJA2_CACHE_ENABLED "If enabled, the jinja2 templating files will be cached into bytecode. Also see SSG_JINJA2_CACHE_DIR." TRUE)
set(SSG_JINJA2_CACHE_DIR "${CMAKE_BINARY_DIR}/jinja2_cache" CACHE PATH "Where the jinja2 cached bytecode should be stored. This speeds up builds at the expense of disk space. You can use one location for multiple SSG builds for performance improvements.")

option(SSG_PRODUCT_EXAMPLE "If enabled, the Example SCAP content will be built" FALSE)
option(SSG_PRODUCT_CHROMIUM "If enabled, the Chromium SCAP content will be built" TRUE)
option(SSG_PRODUCT_DEBIAN8 "If enabled, the Debian8 SCAP content will be built" TRUE)
option(SSG_PRODUCT_FEDORA "If enabled, the Fedora SCAP content will be built" TRUE)
option(SSG_PRODUCT_FIREFOX "If enabled, the Firefox SCAP content will be built" TRUE)
option(SSG_PRODUCT_JBOSS_EAP6 "If enabled, the JBoss EAP6 SCAP content will be built" TRUE)
option(SSG_PRODUCT_JBOSS_FUSE6 "If enabled, the JBoss Fuse6 SCAP content will be built" TRUE)
option(SSG_PRODUCT_JRE "If enabled, the JRE SCAP content will be built" TRUE)
option(SSG_PRODUCT_OCP3 "If enabled, the RHOCP3 SCAP content will be built" TRUE)
option(SSG_PRODUCT_OPENSUSE "If enabled, the openSUSE SCAP content will be built" TRUE)
option(SSG_PRODUCT_OSP13 "If enabled, the RHOSP13 SCAP content will be built" TRUE)
option(SSG_PRODUCT_RHEL6 "If enabled, the RHEL6 SCAP content will be built" TRUE)
option(SSG_PRODUCT_RHEL7 "If enabled, the RHEL7 SCAP content will be built" TRUE)
option(SSG_PRODUCT_RHEL8 "If enabled, the RHEL8 SCAP content will be built" TRUE)
option(SSG_PRODUCT_RHV4 "If enabled, the RHV4 SCAP content will be built" TRUE)
option(SSG_PRODUCT_SUSE11 "If enabled, the SLE11 SCAP content will be built" TRUE)
option(SSG_PRODUCT_SUSE12 "If enabled, the SLE12 SCAP content will be built" TRUE)
option(SSG_PRODUCT_UBUNTU14 "If enabled, the Ubuntu14 SCAP content will be built" TRUE)
option(SSG_PRODUCT_UBUNTU16 "If enabled, the Ubuntu16 SCAP content will be built" TRUE)
option(SSG_PRODUCT_UBUNTU18 "If enabled, the Ubuntu18 SCAP content will be built" TRUE)
option(SSG_PRODUCT_WRLINUX "If enabled, the WRLinux SCAP content will be built" TRUE)
option(SSG_PRODUCT_OL7 "If enabled, the Oracle Linux 7 SCAP content will be built" TRUE)
option(SSG_PRODUCT_OL8 "If enabled, the Oracle Linux 8 SCAP content will be built" TRUE)

option(SSG_CENTOS_DERIVATIVES_ENABLED "If enabled, CentOS derivative content will be built from the RHEL content" TRUE)
option(SSG_SCIENTIFIC_LINUX_DERIVATIVES_ENABLED "If enabled, Scientific Linux derivative content will be built from the RHEL content" TRUE)

set(SSG_SHARED "${CMAKE_SOURCE_DIR}/shared")
set(SSG_SHARED_REFS "${SSG_SHARED}/references")
set(SSG_SHARED_TRANSFORMS "${SSG_SHARED}/transforms")
set(SSG_BUILD_SCRIPTS "${CMAKE_SOURCE_DIR}/build-scripts")

message(STATUS "SCAP Security Guide ${SSG_VERSION}")
message(STATUS "(see ${CMAKE_SOURCE_DIR}/BUILD.md for build instructions)")
message(STATUS "")

# Strictly speaking in-source will work but will be very messy, let's
# discourage our users from using them
if ("${CMAKE_SOURCE_DIR}" STREQUAL "${CMAKE_BINARY_DIR}")
    message(FATAL_ERROR "In-source builds are not supported! Please use out of source builds:\n"
       "$ cd scap-security-guide\n"
       "$ rm CMakeCache.txt\n"
       "$ cd build\n"
       "$ cmake ../\n"
       "$ make -j4"
    )
endif()


set(Python_ADDITIONAL_VERSIONS 3 2)
find_package(PythonInterp REQUIRED)

find_python_module(yaml REQUIRED)
find_python_module(jinja2 REQUIRED)
find_python_module(pytest)
find_python_module(pytest_cov)

include(CMakeDependentOption)
cmake_dependent_option(ENABLE_PYTHON_COVERAGE "Enable Python tests with coverage support" ON "PY_PYTEST_COV" OFF)

find_package(OpenSCAP REQUIRED)

if (SSG_TARGET_OVAL_MAJOR_VERSION EQUAL "5" AND SSG_TARGET_OVAL_VERSION_MINOR EQUAL "11" AND NOT "${OSCAP_V_OUTPUT}" MATCHES "OVAL Version: 5.11")
    message(FATAL_ERROR "Your version of OpenSCAP does not support OVAL 5.11, please switch the OVAL target version to 5.10 or lower. $ cmake -DSSG_TARGET_OVAL_MINOR_VERSION=10 ../")
endif()

if (NOT SSG_TARGET_OVAL_VERSION VERSION_EQUAL "5.10" AND NOT SSG_TARGET_OVAL_VERSION VERSION_EQUAL "5.11")
    message(WARNING "You are targetting OVAL version ${SSG_TARGET_OVAL_VERSION}. In SSG we support/test 5.10 and 5.11 only!")
endif()

execute_process(
    COMMAND "${PYTHON_EXECUTABLE}" "${SSG_BUILD_SCRIPTS}/oscap_svg_support.py" "${OPENSCAP_OSCAP_EXECUTABLE}"
    RESULT_VARIABLE OSCAP_SVG_SUPPORT_RESULT
)
# OSCAP_SVG_SUPPORT_RESULT == 0 means SVG is supported
if (SSG_SVG_IN_XCCDF_ENABLED AND NOT OSCAP_SVG_SUPPORT_RESULT EQUAL 0)
    set(SSG_SVG_IN_XCCDF_ENABLED OFF CACHE BOOL "SVG in XCCDF disabled because your version OpenSCAP doesn't support it" FORCE)
    message(WARNING "Your version of OpenSCAP does not support having the SVG logo in the XCCDF, disabling SVG logo.")
endif()

if (SSG_JINJA2_CACHE_ENABLED)
    file(MAKE_DIRECTORY "${SSG_JINJA2_CACHE_DIR}")
    if (NOT EXISTS "${SSG_JINJA2_CACHE_DIR}")
        message(FATAL_ERROR "jinja2 cache dir was set to '${SSG_JINJA2_CACHE_DIR}'. This directory doesn't seem to exist and attempt to create it has failed.")
    endif()
    set(SSG_JINJA2_CACHE_ENABLED_BOOL "true")
else()
    set(SSG_JINJA2_CACHE_ENABLED_BOOL "false")
endif()

find_program(XSLTPROC_EXECUTABLE NAMES xsltproc)
if (NOT XSLTPROC_EXECUTABLE)
    message(SEND_ERROR "xsltproc is required!")
endif()

find_program(XMLLINT_EXECUTABLE NAMES xmllint)
if (NOT XMLLINT_EXECUTABLE)
    message(SEND_ERROR "xmllint is required!")
endif()

find_program(XMLWF_EXECUTABLE NAMES xmlwf)
if (NOT XMLWF_EXECUTABLE)
    message(SEND_ERROR "xmlwf is required!")
endif()

find_program(SED_EXECUTABLE NAMES sed)
if (NOT SED_EXECUTABLE)
    message(SEND_ERROR "sed is required!")
endif()

find_program(SHELLCHECK_EXECUTABLE NAMES shellcheck)
find_program(LINKCHECKER_EXECUTABLE NAMES linkchecker)
find_program(GREP_EXECUTABLE NAMES grep)
find_program(ANSIBLE_PLAYBOOK_EXECUTABLE NAMES ansible-playbook)
find_program(ANSIBLE_LINT_EXECUTABLE NAMES ansible-lint)
find_program(YAMLLINT_EXECUTABLE NAMES yamllint)

configure_file("${CMAKE_SOURCE_DIR}/build_config.yml.in" "${CMAKE_BINARY_DIR}/build_config.yml" @ONLY)

message(STATUS "CMake:")
message(STATUS "build type: ${CMAKE_BUILD_TYPE}")
message(STATUS "generator: ${CMAKE_GENERATOR}")
message(STATUS "source directory: ${CMAKE_SOURCE_DIR}")
message(STATUS "build directory: ${CMAKE_BINARY_DIR}")
message(STATUS " ")

message(STATUS "Tools:")
message(STATUS "python: ${PYTHON_EXECUTABLE} (version: ${PYTHON_VERSION_STRING})")
message(STATUS "python yaml module: ${PY_YAML}")
message(STATUS "python jinja2 module: ${PY_JINJA2}")
message(STATUS "oscap: ${OPENSCAP_OSCAP_EXECUTABLE} (version: ${OSCAP_VERSION})")
message(STATUS "xsltproc: ${XSLTPROC_EXECUTABLE}")
message(STATUS "xmllint: ${XMLLINT_EXECUTABLE}")
message(STATUS "xmlwf: ${XMLWF_EXECUTABLE}")
message(STATUS "sed: ${SED_EXECUTABLE}")
message(STATUS "shellcheck (optional): ${SHELLCHECK_EXECUTABLE}")
message(STATUS "linkchecker (optional): ${LINKCHECKER_EXECUTABLE}")
message(STATUS "grep (optional): ${GREP_EXECUTABLE}")
message(STATUS "python pytest module (optional): ${PY_PYTEST}")
message(STATUS "ansible-playbook module (optional): ${ANSIBLE_PLAYBOOK_EXECUTABLE}")
message(STATUS "ansible-lint module (optional): ${ANSIBLE_LINT_EXECUTABLE}")
message(STATUS "yamllint module (optional): ${YAMLLINT_EXECUTABLE}")
message(STATUS " ")

message(STATUS "Build options:")
message(STATUS "SSG vendor string: ${SSG_VENDOR}")
message(STATUS "Target OVAL version: ${SSG_TARGET_OVAL_VERSION}")
message(STATUS "OVAL schematron validation: ${SSG_OVAL_SCHEMATRON_VALIDATION_ENABLED}")
message(STATUS "shellcheck bash fixes validation: ${SSG_SHELLCHECK_BASH_FIXES_VALIDATION_ENABLED}")
message(STATUS "SVG logo in XCCDFs: ${SSG_SVG_IN_XCCDF_ENABLED}")
message(STATUS "Separate SCAP files: ${SSG_SEPARATE_SCAP_FILES_ENABLED}")
if (SSG_JINJA2_CACHE_ENABLED)
    message(STATUS "jinja2 cache: enabled")
    message(STATUS "jinja2 cache dir: ${SSG_JINJA2_CACHE_DIR}")
else()
    message(STATUS "jinja2 cache: disabled")
endif()
message(STATUS " ")

message(STATUS "Products:")
message(STATUS "Example: ${SSG_PRODUCT_EXAMPLE}")
message(STATUS "Chromium: ${SSG_PRODUCT_CHROMIUM}")
message(STATUS "Debian 8: ${SSG_PRODUCT_DEBIAN8}")
message(STATUS "Fedora: ${SSG_PRODUCT_FEDORA}")
message(STATUS "Firefox: ${SSG_PRODUCT_FIREFOX}")
message(STATUS "JBoss EAP 6: ${SSG_PRODUCT_JBOSS_EAP6}")
message(STATUS "JBoss Fuse 6: ${SSG_PRODUCT_JBOSS_FUSE6}")
message(STATUS "JRE: ${SSG_PRODUCT_JRE}")
message(STATUS "openSUSE: ${SSG_PRODUCT_OPENSUSE}")
message(STATUS "RHOCP3: ${SSG_PRODUCT_OCP3}")
message(STATUS "RHOSP13: ${SSG_PRODUCT_OSP13}")
message(STATUS "RHEL 6: ${SSG_PRODUCT_RHEL6}")
message(STATUS "RHEL 7: ${SSG_PRODUCT_RHEL7}")
message(STATUS "RHEL 8: ${SSG_PRODUCT_RHEL8}")
message(STATUS "RHV 4: ${SSG_PRODUCT_RHV4}")
message(STATUS "SUSE 11: ${SSG_PRODUCT_SUSE11}")
message(STATUS "SUSE 12: ${SSG_PRODUCT_SUSE12}")
message(STATUS "Ubuntu 14: ${SSG_PRODUCT_UBUNTU14}")
message(STATUS "Ubuntu 16: ${SSG_PRODUCT_UBUNTU16}")
message(STATUS "Ubuntu 18: ${SSG_PRODUCT_UBUNTU18}")
message(STATUS "WRLinux: ${SSG_PRODUCT_WRLINUX}")
message(STATUS "Oracle Linux 7: ${SSG_PRODUCT_OL7}")
message(STATUS "Oracle Linux 8: ${SSG_PRODUCT_OL8}")

message(STATUS " ")

# Remove this option when we would like to run ansible-lint and yamllint against our playbooks by default.
# Right now these checks are not performed and need to be enabled by adding -DANSIBLE_CHECKS=ON to cmake
# before running ctest.
option(ANSIBLE_CHECKS "Set to ON to enable ansible-lint and yamllint checks" OFF)
enable_testing()

include(SSGCommon)

add_subdirectory("tests")

# Targets 'stats', 'profile-stats' and 'zipfile' need to be added
# before any product because they will receive dependencies from products added

add_custom_target(stats)
add_custom_target(profile-stats)

ssg_build_bash_remediation_functions()

ssg_build_man_page()

# ZIP only contains source datastreams and kickstarts, people who
# want sources to build from should get the tarball instead.
if(SSG_TARGET_OVAL_VERSION VERSION_EQUAL "5.11")
    ssg_build_zipfile("scap-security-guide-${SSG_VERSION}")
    ssg_build_nist_zipfile("scap-security-guide-${SSG_VERSION}")
else()
    ssg_build_zipfile("scap-security-guide-${SSG_VERSION}-oval-${SSG_TARGET_OVAL_VERSION}")
    ssg_build_nist_zipfile("scap-security-guide-${SSG_VERSION}-${SSG_TARGET_OVAL_VERSION}")
endif()

if (SSG_PRODUCT_EXAMPLE)
    add_subdirectory("example")
endif()
if (SSG_PRODUCT_CHROMIUM)
    add_subdirectory("chromium")
endif()
if (SSG_PRODUCT_DEBIAN8)
    add_subdirectory("debian8")
endif()
if (SSG_PRODUCT_FEDORA)
    add_subdirectory("fedora")
endif()
if (SSG_PRODUCT_FIREFOX)
    add_subdirectory("firefox")
endif()
if (SSG_PRODUCT_JBOSS_EAP6)
    add_subdirectory("eap6")
endif()
if (SSG_PRODUCT_JBOSS_FUSE6)
    add_subdirectory("fuse6")
endif()
if (SSG_PRODUCT_JRE)
    add_subdirectory("jre")
endif()
if (SSG_PRODUCT_OCP3)
    add_subdirectory("ocp3")
endif()
if (SSG_PRODUCT_OPENSUSE)
    add_subdirectory("opensuse")
endif()
if (SSG_PRODUCT_OSP13)
    add_subdirectory("rhosp13")
endif()
if (SSG_PRODUCT_RHEL6)
    add_subdirectory("rhel6")
endif()
if (SSG_PRODUCT_RHEL7)
    add_subdirectory("rhel7")
endif()
if (SSG_PRODUCT_RHEL8)
    add_subdirectory("rhel8")
endif()
if (SSG_PRODUCT_RHV4)
	#    add_subdirectory("rhv4")
endif()
if (SSG_PRODUCT_SUSE11)
    add_subdirectory("sle11")
endif()
if (SSG_PRODUCT_SUSE12)
    add_subdirectory("sle12")
endif()
if (SSG_PRODUCT_UBUNTU14)
    add_subdirectory("ubuntu1404")
endif()
if (SSG_PRODUCT_UBUNTU16)
    add_subdirectory("ubuntu1604")
endif()
if (SSG_PRODUCT_UBUNTU18)
    add_subdirectory("ubuntu1804")
endif()

if (SSG_PRODUCT_WRLINUX)
    add_subdirectory("wrlinux")
endif()

if (SSG_PRODUCT_OL7)
    add_subdirectory("ol7")
endif()

if (SSG_PRODUCT_OL8)
    add_subdirectory("ol8")
endif()


ssg_define_guide_and_table_tests()

install(FILES "${CMAKE_SOURCE_DIR}/LICENSE"
    DESTINATION ${CMAKE_INSTALL_DOCDIR})
install(FILES "${CMAKE_SOURCE_DIR}/README.md"
    DESTINATION ${CMAKE_INSTALL_DOCDIR})
install(FILES "${CMAKE_SOURCE_DIR}/Contributors.md"
    DESTINATION ${CMAKE_INSTALL_DOCDIR})

install(FILES "${CMAKE_BINARY_DIR}/scap-security-guide.8"
    DESTINATION "${CMAKE_INSTALL_MANDIR}/man8")

# We use CPack to generate the tarball with all sources and
# packages for testing

# only CPack should follow
set(CPACK_CMAKE_GENERATOR "Unix Makefiles")
set(CPACK_SOURCE_GENERATOR "TBZ2")
set(CPACK_SOURCE_PACKAGE_FILE_NAME "scap-security-guide-${SSG_VERSION}" CACHE INTERNAL "tarball basename")
set(CPACK_SOURCE_IGNORE_FILES
"\\\\.git.*"

"build/"
"~$"

"\\\\CMakeLists.txt.user"
)
# Common definitions for RPM and DEB packages
set(CPACK_PACKAGE_VERSION ${SSG_VERSION})
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Security guidance and baselines in SCAP formats")
set(CPACK_PACKAGE_VENDOR "scap-security-guide")
# The package contact is needed to build the deb package
set(CPACK_PACKAGE_CONTACT "open-scap-list@redhat.com")
set(CPACK_PACKAGE_RELOCATABLE FALSE)

# This adds "${?dist} to Release field in spec file
set(CPACK_RPM_PACKAGE_RELEASE "1%{?dist}")
set(CPACK_RPM_EXCLUDE_FROM_AUTO_FILELIST_ADDITION "/usr/share/xml;/usr/share/man;/usr/share/man/man8")
set(CPACK_RPM_PACKAGE_GROUP "Applications/System")
set(CPACK_RPM_PACKAGE_LICENSE "BSD-3-Clause")
set(CPACK_RPM_PACKAGE_URL "https://www.open-scap.org/security-policies/scap-security-guide/")
set(CPACK_RPM_PACKAGE_ARCHITECTURE "noarch")
set(CPACK_RPM_PACKAGE_REQUIRES "xml-common, openscap-utils >= 1.0.8")
set(CPACK_RPM_PACKAGE_PROVIDES "openscap-content")
set(CPACK_RPM_PACKAGE_DESCRIPTION "The %{name} project provides a guide for configuration of the
system from the final system's security point of view. The guidance is
specified in the Security Content Automation Protocol (SCAP) format and
constitutes a catalog of practical hardening advice, linked to government
requirements where applicable. The project bridges the gap between generalized
policy requirements and specific implementation guidelines. The system
administrator can use the oscap command-line tool from the openscap-utils
package to verify that the system conforms to provided guidelines.
The %{name} package also contains HTML formatted documents containing
hardening guidances that have been generated from XCCDF benchmarks.
")

# Change the default file name of the RPMs
# %{release} includes release number and dist type
# This only has effect with cmake v3.6 or higher
set(CPACK_RPM_FILE_NAME "%{name}-%{version}-%{release}.rpm")
# For older versions of cmake (e.g. v2.8) file name is defined like below
set(CPACK_PACKAGE_FILE_NAME "scap-security-guide-${SSG_VERSION}")

set(CPACK_GENERATOR "RPM;DEB")
include(CPack)
