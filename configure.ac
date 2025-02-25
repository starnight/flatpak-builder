AC_PREREQ([2.63])

m4_define([flatpak_builder_major_version], [1])
m4_define([flatpak_builder_minor_version], [3])
m4_define([flatpak_builder_micro_version], [1])
m4_define([flatpak_builder_version],
          [flatpak_builder_major_version.flatpak_builder_minor_version.flatpak_builder_micro_version])

AC_INIT([flatpak-builder],
        [flatpak_builder_version],
        [https://github.com/flatpak/flatpak-builder/issues],
        [flatpak-builder],
        [http://flatpak.org/])

GLIB_REQS=2.44
OSTREE_REQS=2017.14
FLATPAK_REQS=0.99.1
SYSTEM_DEBUGEDIT_REQS=5.0
LIBDW_REQS=0.172

AC_USE_SYSTEM_EXTENSIONS
AC_SYS_LARGEFILE
LIBGLNX_CONFIGURE

AC_PROG_CC
AM_PROG_CC_C_O
AC_PROG_CC_STDC
AC_DISABLE_STATIC

AC_CHECK_PROG([FLATPAK], [flatpak], [flatpak], [false])

if test "x$FLATPAK" = xfalse; then
   AC_MSG_ERROR([You need flatpak installed])
else
   FLATPAK_VERSION=`$FLATPAK --version | sed 's,.*\ \([0-9]*\.[0-9]*\.[0-9]*\)$,\1,'`
   AX_COMPARE_VERSION([$FLATPAK_REQS],[gt],[$FLATPAK_VERSION],
                      [AC_MSG_ERROR([You need at least version $FLATPAK_REQS of flatpak, your version is $FLATPAK_VERSION])])
fi


LT_PREREQ([2.2.6])
LT_INIT([disable-static])

AC_CONFIG_SRCDIR([src/builder-main.c])
AC_CONFIG_HEADERS([config.h])
AC_CONFIG_MACRO_DIR([m4])
AM_INIT_AUTOMAKE([1.13.4 no-define no-dist-gzip dist-xz tar-ustar foreign subdir-objects])
AC_PROG_SED

AM_GNU_GETTEXT([external])
AM_GNU_GETTEXT_VERSION([0.18.2])
AC_DEFINE_UNQUOTED(GETTEXT_PACKAGE, "$PACKAGE", [gettext domain])

# Enable silent rules is available
AM_SILENT_RULES([yes])
AM_MAINTAINER_MODE([enable])

dnl This list is shared with https://github.com/ostreedev/ostree
CC_CHECK_FLAGS_APPEND([WARN_CFLAGS], [CFLAGS], [\
-pipe \
-Wall \
-Werror=empty-body \
-Werror=strict-prototypes \
-Werror=missing-prototypes \
-Werror=implicit-function-declaration \
"-Werror=format=2 -Werror=format-security -Werror=format-nonliteral" \
-Werror=pointer-arith -Werror=init-self \
-Werror=missing-declarations \
-Werror=return-type \
-Werror=overflow \
-Werror=int-conversion \
-Werror=parenthesis \
-Werror=incompatible-pointer-types \
-Werror=misleading-indentation \
-Werror=missing-include-dirs \
])
AC_SUBST(WARN_CFLAGS)

AX_VALGRIND_CHECK

PKG_PROG_PKG_CONFIG([0.24])

# For libglnx
AC_CHECK_HEADER([sys/xattr.h], [], [AC_MSG_ERROR([You must have sys/xattr.h from glibc])])

PKG_CHECK_MODULES(BASE, [glib-2.0 >= $GLIB_REQS gio-2.0 gio-unix-2.0 libsoup-2.4 ostree-1 >= $OSTREE_REQS json-glib-1.0 libxml-2.0 >= 2.4 libcurl])

dnl ************************
dnl *** check for libelf ***
dnl ************************
PKG_CHECK_MODULES([LIBELF], [libelf >= 0.8.12], [have_libelf=yes], [have_libelf=maybe])
AS_IF([ test $have_libelf = maybe ], [
  save_LIBS=$LIBS
  AC_CHECK_LIB([elf], [elf_begin], [:], [have_libelf=no])
  AC_CHECK_LIB([elf], [elf_getshdrstrndx], [:], [have_libelf=no])
  AC_CHECK_LIB([elf], [elf_getshdrnum], [:], [have_libelf=no])
  AC_CHECK_HEADER([libelf.h], [:], [have_libelf=no])
  LIBS=$save_LIBS

  if test $have_libelf != no; then
    LIBELF_LIBS=-lelf
    have_libelf=yes
  fi
])

if test x$have_libelf != xyes; then
  AC_MSG_ERROR([libelf not found])
fi

AC_ARG_VAR([DEBUGEDIT], [debugedit executable])
AC_ARG_WITH([system-debugedit],
            [AS_HELP_STRING([--with-system-debugedit], [Use system debugedit [default=check $DEBUGEDIT]])],
            [DEBUGEDIT="$withval"],
            [DEBUGEDIT="${DEBUGEDIT:-false}"])
AS_CASE([$DEBUGEDIT],
        [yes],
            [DEBUGEDIT=debugedit],
        [no],
            [DEBUGEDIT=false],
        [auto],
            [AC_CHECK_PROG([DEBUGEDIT], [debugedit], [debugedit], [false])])
if test "x$DEBUGEDIT" != xfalse; then
    DEBUGEDIT_VERSION=`$DEBUGEDIT -V | sed 's,.*\ \([0-9]*\.[0-9]*\.[0-9]*\)$,\1,'`
    AX_COMPARE_VERSION([$SYSTEM_DEBUGEDIT_REQS],[gt],[$DEBUGEDIT_VERSION],
                      [AC_MSG_ERROR([You need at least version $SYSTEM_DEBUGEDIT_REQS of debugedit to use the system installed version])])
    AM_CONDITIONAL([WITH_SYSTEM_DEBUGEDIT], [true])
else
    PKG_CHECK_MODULES(LIBDW, [libdw >= $LIBDW_REQS], [have_libdw=yes], [have_libdw=no])
    if test "x$have_libdw" == xno; then
        AC_MSG_ERROR([libdw >= $LIBDW_REQS needed by debugedit but not found])
    fi
    AM_CONDITIONAL([WITH_SYSTEM_DEBUGEDIT], [false])
fi

AC_ARG_WITH([yaml],
  [AS_HELP_STRING([--without-yaml],
                  [Disable YAML support [default=auto]])])
AS_IF([test "x$with_yaml" != "xno"],[
  PKG_CHECK_MODULES(YAML, [yaml-0.1], [have_yaml=yes], [have_yaml=no])
], [have_yaml=no])
AS_IF([test "x$have_yaml" = "xno"],[
  AS_IF([test "x$with_yaml" = "xyes"],
        [AC_MSG_ERROR([yaml-0.1 was not found, which is needed for --with-yaml])])
], [AC_DEFINE([FLATPAK_BUILDER_ENABLE_YAML],[1],[Define if yaml supported])])

AC_ARG_ENABLE(documentation,
              AC_HELP_STRING([--enable-documentation], [Build documentation]),,
              enable_documentation=yes)
if test x$enable_documentation = xyes; then
   AC_PATH_PROG([XSLTPROC], [xsltproc])
   if test x$XSLTPROC = x; then
      AC_MSG_ERROR([xsltproc is required to build documentation])
   fi

  dnl check for DocBook DTD in the local catalog
  JH_CHECK_XML_CATALOG([-//OASIS//DTD DocBook XML V4.3//EN],
     [DocBook XML DTD V4.3], [have_docbook_dtd=yes], [have_docbook_dtd=no])
  if test "$have_docbook_dtd" != yes; then
    AC_MSG_ERROR([DocBook DTD is required for --enable-documentation])
  fi

  dnl check for DocBook XSL stylesheets in the local catalog
  JH_CHECK_XML_CATALOG([http://docbook.sourceforge.net/release/xsl/current/manpages/docbook.xsl],
     [DocBook XSL Stylesheets], [have_docbook_style=yes],[have_docbook_style=no])
  if test "$have_docbook_style" != yes; then
    AC_MSG_ERROR([DocBook XSL Stylesheets are required for --enable-documentation])
  fi
fi
AM_CONDITIONAL(BUILD_DOCUMENTATION, test x$enable_documentation = xyes)

AC_ARG_ENABLE(docbook-docs,
        [AS_HELP_STRING([--enable-docbook-docs],[build documentation (requires xmlto)])],
        enable_docbook_docs=$enableval, enable_docbook_docs=auto)
AC_PATH_PROG(XMLTO, xmlto, no)
AC_MSG_CHECKING([whether to build DocBook documentation])
if test x$XMLTO = xno ; then
        have_docbook=no
else
        have_docbook=yes
fi
if test x$enable_docbook_docs = xauto ; then
        if test x$have_docbook = xno ; then
                enable_docbook_docs=no
        else
                enable_docbook_docs=yes
        fi
fi
if test x$enable_docbook_docs = xyes; then
        if test x$have_docbook = xno; then
                AC_MSG_ERROR([Building DocBook docs explicitly required, but DocBook not found])
        fi
        AC_MSG_RESULT(yes)
else
        AC_MSG_RESULT(no)
fi
AM_CONDITIONAL(DOCBOOK_DOCS_ENABLED, test x$enable_docbook_docs = xyes)
AC_ARG_VAR([XMLTO],[Define/override the 'xmlto' location.])
AC_ARG_VAR([XMLTO_FLAGS],[Define/override 'xmlto' options, like '--skip-validation'.])

AC_ARG_WITH([fuse],
            [AS_HELP_STRING([--with-fuse={2,3}],
                            [Only support this major version of FUSE])],
            [fuse_major="$withval"],
            [fuse_major=maximize-compatibility])
AS_IF([test $fuse_major = 2],
      [AC_DEFINE([ASSUME_FUSE_2], [],
                 [Define to speed up FUSE 2 at the cost of FUSE 3 compatibility])])

GLIB_TESTS

FLATPAK_BUILDER_VERSION=flatpak_builder_version
AC_SUBST(FLATPAK_BUILDER_VERSION)

AC_CONFIG_FILES([
Makefile
data/Makefile
doc/Makefile
doc/flatpak-builder-docs.xml
])
AC_OUTPUT

echo ""
echo "          flatpak-builder $FLATPAK_BUILDER_VERSION"
echo "          ============================="
echo ""
