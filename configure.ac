dnl Autoconf script for DomTerm
AC_INIT([DomTerm],[1.3.0],,domterm) dnl Also update package.json and domterm.spec
AC_CONFIG_SRCDIR([doc/DomTerm.texi])
AM_MAINTAINER_MODE([enable])

DOMTERM_VERSION="${PACKAGE_VERSION}"
DOMTERM_YEAR="2019"

AC_CONFIG_AUX_DIR(autotools-aux)
AM_INIT_AUTOMAKE([foreign subdir-objects no-dependencies])
AC_CANONICAL_HOST
AC_EXEEXT
AC_OBJEXT
AC_PROG_CC
AC_PROG_CXX
AC_PROG_SED
AC_PROG_MKDIR_P
AC_PROG_LN_S
AC_CHECK_FUNC([inotify_init], [HAVE_INOTIFY=1], [HAVE_INOTIFY=0])
AC_CHECK_FUNC([getrandom], [HAVE_GETRANDOM=1], [HAVE_GETRANDOM=0])
AC_CHECK_LIB(magic, magic_open, [HAVE_LIBMAGIC=1; LIBMAGIC_LIBS=-lmagic], [HAVE_LIBMAGIC=0])

AC_ARG_WITH(java,
  AS_HELP_STRING(--without-java,Build Java help library))

AC_ARG_WITH(xterm.js,
  AS_HELP_STRING(--with-xterm.js=/path/to/xterm.js/build))

AC_ARG_WITH(java_websocket,
  AS_HELP_STRING(--with-java_websocket@<:@=/path/to/java_websocket.jar@:>@,Build server using Java-WebServer))

AC_ARG_WITH(libwebsockets,
  AS_HELP_STRING(--with-libwebsockets,Build server using libwebsockets))

AC_ARG_WITH(asciidoctor,
  AS_HELP_STRING(--with-asciidoctor,Use AsciiDoctor to build help and man pages))

AC_ARG_WITH(qtwebengine,
  AS_HELP_STRING(--with-qtwebengine,Build QtWebEngine front-end))

AC_ARG_ENABLE(java-pty,
  AS_HELP_STRING(--enable-java-pty,Enable support for PTYs from Java))

AC_ARG_ENABLE(ld-preload,
  AS_HELP_STRING(--enable-ld-preload,Make use of LD_PRELOAD to modify standard functions))

AC_ARG_WITH(javafx,
  AS_HELP_STRING(--with-javafx@<:@=jfxrt.jar@:>@,Enable support for JavaFX))

AC_ARG_WITH(closure_compiler,
  AC_HELP_STRING("--with-closure-compiler[=@<:@=compiler.jar@:>],compress JavaScript with Closure))

AC_ARG_ENABLE(compiled-in-resources)

pathsep="$PATH_SEPARATOR"
case ${host} in
  *-*-mingw*|*-*-cygwin*) CLASSPATH_SEPARATOR=";" ;;
  *) CLASSPATH_SEPARATOR="$PATH_SEPARATOR"
esac
case "${srcdir}" in
  ".")  conf_classpath="";;
  *) conf_classpath="${srcdir}";;
esac

case ${host} in
  *darwin*)
    JAVA_INCLUDE_SUBDIR=darwin
    LIBEXT=.dylib
    ;;
  *-*-mingw*|*-*-cygwin*)
    JAVA_INCLUDE_SUBDIR=win32
    LIBEXT=.dll
    ;;
  *linux*)
    JAVA_INCLUDE_SUBDIR=linux
    LIBEXT=.so
    ;;
  *)
    LIBEXT=.so
    ;;
esac

AC_CHECK_PROGS(QMAKE, qmake-qt5 qmake, qmake)

AS_IF([test -z "$with_asciidoctor"],
    AC_CHECK_PROG(with_asciidoctor, asciidoctor, "yes", "no"))
AM_CONDITIONAL(WITH_ASCIIDOCTOR, test "$with_asciidoctor" = "yes")

AM_CONDITIONAL(WITH_QTWEBENGINE, test "$with_qtwebengine" = "yes")

AM_CONDITIONAL(WITH_LIBWEBSOCKETS, test "$with_libwebsockets" != "no")

AM_CONDITIONAL(WITH_JAVA, test "$with_java" == "yes")

if test "$with_java" = "no" && test -z "${with_java_websocket}"
then
  with_java_websocket="no"
fi
case "${with_java_websocket}" in
  yes | "")
    conf_classpath=java_websocket.jar${CLASSPATH_SEPARATOR}$conf_classpath
    ;;
  no)
    ;;
  *)
    conf_classpath=${with_java_websocket}${CLASSPATH_SEPARATOR}$conf_classpath
    ;;
esac
AM_CONDITIONAL(WITH_JAVA_WEBSOCKET,
  test "$with_java_websocket" != "no" -a "$with_java_websocket" != "")

case ${with_javafx} in
  "yes" | "no" | "")
    ;;
  *)
    if test -f "${with_javafx}" ; then true
    elif test -f "${with_javafx}/rt/lib/jfxrt.jar"
    then with_javafx="${with_javafx}/rt/lib/jfxrt.jar"
    elif test -f "${with_javafx}/lib/jfxrt.jar"
    then with_javafx="${with_javafx}/lib/jfxrt.jar"
    elif test -f "${with_javafx}/jre/lib/jfxrt.jar"
    then with_javafx="${with_javafx}/jre/lib/jfxrt.jar"
    fi
    conf_classpath=${with_javafx}${CLASSPATH_SEPARATOR}$conf_classpath ;;
esac
AM_CONDITIONAL(WITH_JAVAFX, test "$with_javafx" != "" -a "$with_javafx" != "no")

AM_CONDITIONAL(ENABLE_JAVA_PTY, test "$enable_java_pty" = "yes")

if test "$with_java" != "no"
then
  if test '(' -z "${JAVA_HOME}" ')' -o '(' '!' -f "${JAVA_HOME}/bin/javac" ')'
  then
    if test -x /usr/libexec/java_home
    then JAVA_HOME=`/usr/libexec/java_home`
    else
      javac_path=`command -v javac`
      if test -n "${javac_path}"
      then JAVA_HOME=`readlink -f $javac_path | sed -e 's#/bin/javac##'`
      fi
    fi
  fi
fi

AM_CONDITIONAL(ENABLE_LD_PRELOAD, test "$enable_ld_preload" = "yes")

dnl FIXME We might want to figure out datadir_relative by
dnl decl comparing datadir to bindir or prefix.
dnl If datadir and prefix set, remove prefix from datadir.
dnl If bindir is set, use bindir without last component instead of prefix.
dnl Otherwise default to "share"
DOMTERM_DIR_RELATIVE="/share/domterm"

AC_SUBST(DOMTERM_DIR_RELATIVE)

bindir_parent=`echo "$bindir" | sed -e 's|[^/]*/bin||'`

HAVE_OPENSSL=0
PKG_PROG_PKG_CONFIG
if test "$with_libwebsockets" != "no"; then
  PKG_CHECK_MODULES(JSON_C, json-c)
  dnl kludge around broken OpenSSL config in MacOS:
  OPENSSL_CONFIG=/usr/local/opt/openssl/lib/pkgconfig
  test -d "$OPENSSL_CONFIG" && \
      export PKG_CONFIG_PATH="$OPENSSL_CONFIG:$PKG_CONFIG_PATH"
  if pkg-config openssl; then
    OPENSSL_CFLAGS=`pkg-config --cflags openssl`
    OPENSSL_LIBS=`pkg-config --libs openssl`
    HAVE_OPENSSL=1
  else
    AC_MSG_WARN(openssl library not found)
  fi
  PKG_CHECK_MODULES(LIBCAP, libcap, true, true)
  case "${with_libwebsockets}" in
    "yes" | "")
      PKG_CHECK_MODULES(LIBWEBSOCKETS, [libwebsockets >= 2.2.0])
      ;;
    *)
      #LIBWEBSOCKETS_LIBS="-L ${with_libwebsockets}/lib -lwebsockets"
      LIBWEBSOCKETS_LIBS="${with_libwebsockets}/lib/libwebsockets.a"
      LIBWEBSOCKETS_CFLAGS="-I${with_libwebsockets}/include"
      ;;
  esac
  if test -z "$enable_compiled_in_resources"; then
    AC_LANG(C)
    AC_MSG_CHECKING([if LWS_WITH_ZIP_FOPS is enabled in libwebsockets])
    save_CFLAGS="$CFLAGS"
    CFLAGS="$CFLAGS $LIBWEBSOCKETS_CFLAGS"
    AC_COMPILE_IFELSE([AC_LANG_SOURCE([#include <lws_config.h>
    #ifndef LWS_WITH_ZIP_FOPS
    #error missing LWS_WITH_ZIP_FOPS
    #endif
    ])],
    [LWS_WITH_ZIP_FOPS=yes; enable_compiled_in_resources=no],
    [LWS_WITH_ZIP_FOPS=no; enable_compiled_in_resources=yes])
    AC_MSG_RESULT($LWS_WITH_ZIP_FOPS)
    CFLAGS=$save_CFLAGS
  fi
fi
ldomterm_misc_includes=
if (which brew>&/dev/null)
then
  ldomterm_misc_includes="-I$(brew --prefix)/include"
fi
AC_SUBST(ldomterm_misc_includes)

case "$with_closure_compiler" in
  "yes")
    CLOSURE_COMPILER='closure-compiler' ;;
  *".jar")
    CLOSURE_COMPILER='$(JAVA_WITH_PATH)'" -jar ${with_closure_compiler}" ;;
  "no" | "")
    CLOSURE_COMPILER="NONE" ;;
  *)
    CLOSURE_COMPILER="${with_closure_compiler}" ;;
esac
AM_CONDITIONAL(WITH_CLOSURE_COMPILER, test "$CLOSURE_COMPILER" != "NONE")
AC_SUBST(CLOSURE_COMPILER)

AM_CONDITIONAL(ENABLE_COMPILED_IN_RESOURCES,
    test "$enable_compiled_in_resources" = "yes")

AM_CONDITIONAL(WITH_XTERMJS,
    test "$with_xterm_js" != "" -a "$with_xterm_js" != "no")
XTERMJS_PATH="$with_xterm_js"
AC_SUBST(XTERMJS_PATH)

HLIB_FIXED_FILES="\
  hlib/browserkeymap.js \
  hlib/commands.js \
  hlib/domterm.js \
  hlib/domterm-client.js \
  hlib/domterm-core.css \
  hlib/domterm-default.css \
  hlib/domterm-layout.css \
  hlib/domterm-layout.js \
  hlib/domterm-menus.js \
  hlib/domterm-standard.css \
  hlib/FileSaver.js \
  hlib/goldenlayout.js \
  hlib/goldenlayout-base.css \
  hlib/jquery.min.js \
  hlib/jsMenus.css \
  hlib/jsMenus.js \
  hlib/node-sixel.js \
  hlib/qwebchannel.js \
  hlib/ResizeSensor.js \
  hlib/screenfull.min.js \
  hlib/terminal.js \
  hlib/wcwidth.js"
case "$with_xterm_js" in
  ""|"no")
    HLIB_FIXED_FILES="${HLIB_FIXED_FILES} hlib/domterm-parser.js"
    WITH_XTERMJS=0;;
  *) WITH_XTERMJS=1;;
esac
AC_SUBST(WITH_XTERMJS)
AC_SUBST(HLIB_FIXED_FILES)

AC_SUBST(CC)
AC_SUBST(CFLAGS)
AC_SUBST(LIBEXT)
AC_SUBST(SED)
AC_SUBST(CLASSPATH_SEPARATOR)
AC_SUBST(conf_classpath)
AC_SUBST(JAVA_HOME)
AC_SUBST(JAVA_INCLUDE_SUBDIR)
AC_SUBST(LIBWEBSOCKETS_LIBS)
AC_SUBST(LIBWEBSOCKETS_CFLAGS)
AC_SUBST(OPENSSL_LIBS)
AC_SUBST(OPENSSL_CFLAGS)
AC_SUBST(DOMTERM_VERSION)
AC_SUBST(DOMTERM_YEAR)
AC_SUBST(HAVE_GETRANDOM)
AC_SUBST(HAVE_INOTIFY)
AC_SUBST(HAVE_LIBMAGIC)
AC_SUBST(HAVE_OPENSSL)
AC_SUBST(LIBMAGIC_LIBS)

AC_CONFIG_FILES([Makefile hlib/domterm-version.js
                 lws-term/Makefile qtdomterm/dt_version.h lws-term/version.h]
                 share/domterm/electron/package.json:electron/package.json)
AC_OUTPUT
