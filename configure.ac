#
# Copyright 2010 Red Hat, Inc.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; see the file COPYING.  If not, write to
# the Free Software Foundation, 675 Mass Ave, Cambridge, MA 02139, USA.
#

# bootstrap / init
m4_define([sheepdog_version], [0.9.50])

m4_define([git_version],
	m4_esyscmd([git describe --tags --dirty 2> /dev/null | sed 's/^v//' \
| tr '-' '_' | tr -d '\n']))

AC_INIT([sheepdog], m4_default(git_version, sheepdog_version),
	[sheepdog@lists.wpkg.org])
AM_INIT_AUTOMAKE([-Wno-portability, subdir-objects])

AC_CONFIG_SRCDIR([dog/dog.c])
AC_CONFIG_HEADER([include/config.h])

AC_CANONICAL_HOST

AC_LANG([C])

m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES(yes)])

if make --help 2>&1 | grep -q no-print-directory; then
	AM_MAKEFLAGS="$AM_MAKEFLAGS --no-print-directory";
fi

if make --help 2>&1 | grep -q quiet; then
	AM_MAKEFLAGS="$AM_MAKEFLAGS --quiet"
fi

if libtool --help 2>&1 | grep -q quiet; then
	AM_LIBTOOLFLAGS="--quiet";
fi

m4_ifndef([PKG_PROG_PKG_CONFIG], AC_MSG_ERROR([pkg-config not found]))

dnl Fix default variables - "prefix" variable if not specified
if test "$prefix" = "NONE"; then
	prefix="/usr"

	dnl Fix "localstatedir" variable if not specified
	if test "$localstatedir" = "\${prefix}/var"; then
		localstatedir="/var"
	fi
	dnl Fix "sysconfdir" variable if not specified
	if test "$sysconfdir" = "\${prefix}/etc"; then
		sysconfdir="/etc"
	fi
	dnl Fix "libdir" variable if not specified
	if test "$libdir" = "\${exec_prefix}/lib"; then
		if test -e /usr/lib64; then
			libdir="/usr/lib64"
		else
			libdir="/usr/lib"
		fi
	fi
fi

# check stolen from gnulib/m4/gnu-make.m4
if ! ${MAKE-make} --version /cannot/make/this >/dev/null 2>&1; then
	AC_MSG_ERROR([you don't seem to have GNU make; it is required])
fi

AC_PROG_CC
AM_PROG_AS
AC_PROG_INSTALL
AC_PROG_LN_S
AC_PROG_MAKE_SET
AC_PROG_RANLIB
AC_CHECK_PROGS([GROFF], [groff])

AM_MISSING_PROG(AUTOM4TE, autom4te, $missing_dir)

# Checks for libraries.
AC_CHECK_LIB([socket], [socket])
AC_CHECK_LIB([rt], [clock_gettime], , AC_MSG_ERROR(librt not found))

# Checks for header files.
AC_FUNC_ALLOCA
AC_HEADER_DIRENT
AC_HEADER_STDC
AC_HEADER_SYS_WAIT
AC_CHECK_HEADERS([arpa/inet.h fcntl.h limits.h netdb.h netinet/in.h stdint.h \
		  stdlib.h string.h sys/ioctl.h sys/param.h sys/socket.h \
		  sys/time.h syslog.h unistd.h sys/types.h getopt.h malloc.h \
		  sys/sockio.h utmpx.h])

AC_CHECK_HEADERS([urcu.h urcu/uatomic.h],,
	AC_MSG_ERROR(liburcu 0.6.0 or later is required))

# Checks for typedefs, structures, and compiler characteristics.
AC_C_CONST
AC_TYPE_UID_T
AC_C_INLINE
AC_TYPE_SIZE_T
AC_HEADER_TIME
AC_C_VOLATILE

AC_CHECK_SIZEOF(short)
AC_CHECK_SIZEOF(int)
AC_CHECK_SIZEOF(long)
AC_CHECK_SIZEOF(long long)
SIZEOF_SHORT=$ac_cv_sizeof_short
SIZEOF_INT=$ac_cv_sizeof_int
SIZEOF_LONG=$ac_cv_sizeof_long
SIZEOF_LONG_LONG=$ac_cv_sizeof_long_long
AC_SUBST(SIZEOF_SHORT)
AC_SUBST(SIZEOF_INT)
AC_SUBST(SIZEOF_LONG)
AC_SUBST(SIZEOF_LONG_LONG)

# Checks for header files.
AC_CHECK_HEADERS([sys/eventfd.h])
AC_CHECK_HEADERS([sys/signalfd.h])
AC_CHECK_HEADERS([sys/timerfd.h])

# Checks for library functions.
AC_FUNC_CLOSEDIR_VOID
AC_FUNC_ERROR_AT_LINE
AC_REPLACE_FNMATCH
AC_FUNC_FORK
AC_PROG_GCC_TRADITIONAL
AC_FUNC_MALLOC
AC_FUNC_MEMCMP
AC_FUNC_REALLOC
AC_FUNC_SELECT_ARGTYPES
AC_TYPE_SIGNAL
AC_FUNC_VPRINTF
AC_CHECK_FUNCS([alarm alphasort atexit bzero dup2 endgrent endpwent fcntl \
		getcwd getpeerucred getpeereid gettimeofday inet_ntoa memmove \
		memset mkdir scandir select socket strcasecmp strchr strdup \
		strerror strrchr strspn strstr fallocate])

AC_CONFIG_FILES([Makefile
		dog/Makefile
		sheep/Makefile
		sheepfs/Makefile
		include/Makefile
		script/Makefile
		lib/Makefile
		man/Makefile
		shepherd/Makefile
		tests/unit/Makefile
		tests/unit/mock/Makefile
		tests/unit/dog/Makefile
		tests/unit/sheep/Makefile
		tools/Makefile])

### Local business

# ===============================================
# Helpers
# ===============================================

## helper for CC stuff
cc_supports_flag() {
	local CFLAGS="$@"
	AC_MSG_CHECKING([whether $CC supports "$@"])
	AC_COMPILE_IFELSE([AC_LANG_SOURCE([int main(){return 0;}])] ,
			  [RC=0; AC_MSG_RESULT([yes])],
			  [RC=1; AC_MSG_RESULT([no])])
	return $RC
}

## cleanup
AC_MSG_NOTICE(Sanitizing prefix: ${prefix})
case $prefix in
  NONE) prefix=/usr/local;;
esac

AC_MSG_NOTICE(Sanitizing exec_prefix: ${exec_prefix})
case $exec_prefix in
  dnl For consistency with Sheepdog, map NONE->$prefix
  NONE)   exec_prefix=$prefix;;
  prefix) exec_prefix=$prefix;;
esac

## local defines
PACKAGE_FEATURES=""

LINT_FLAGS="-weak -unrecog +posixlib +ignoresigns -fcnuse \
	-badflag -D__gnuc_va_list=va_list -D__attribute\(x\)="

AM_CONDITIONAL(BUILD_SHA1_HW, [[[[ $host = *x86_64* ]]]])

AC_ARG_ENABLE([fatal-warnings],
	[  --enable-fatal-warnings : enable fatal warnings. ],
	[ default="no" ])

AC_ARG_ENABLE([debug],
	[  --enable-debug          : enable debug build. ],
	[ default="no" ])

AC_ARG_ENABLE([assert],
	[  --enable-assert          : enable assert. ],,
	[ enable_assert="yes" ],)

AC_ARG_ENABLE([unittest],
	[  --enable-unittest       : enable unittest. ],
	[ default="no" ])

AC_ARG_ENABLE([coverage],
	[  --enable-coverage       : coverage analysis of the codebase. ],
	[ default="no" ])
AM_CONDITIONAL(BUILD_COVERAGE, test x$enable_coverage = xyes)

AC_ARG_ENABLE([corosync],
	[  --enable-corosync       : build corosync cluster driver ],,
	[ enable_corosync="yes" ],)
AM_CONDITIONAL(BUILD_COROSYNC, test x$enable_corosync = xyes)

AC_ARG_ENABLE([zookeeper],
	[  --enable-zookeeper      : build zookeeper cluster driver ],,
	[ enable_zookeeper="no" ],)
AM_CONDITIONAL(BUILD_ZOOKEEPER, test x$enable_zookeeper = xyes)

AC_ARG_ENABLE([shepherd],
	[  --enable-shepherd       : build shepherd cluster driver ],,
	[ enable_shepherd="no" ],)
AM_CONDITIONAL(BUILD_SHEPHERD, test x$enable_shepherd = xyes)

AC_ARG_WITH([initddir],
	[  --with-initddir=DIR     : path to init script directory. ],
	[ INITDDIR="$withval" ],
	[ INITDDIR="$sysconfdir/init.d" ])

AC_ARG_ENABLE([trace],
	[  --enable-trace           : enable trace],,
	[ enable_trace="${enable_debug}" ],)
AM_CONDITIONAL(BUILD_TRACE, test x$enable_trace = xyes)

PKG_CHECK_MODULES([fuse],[fuse], HAVE_FUSE="yes", HAVE_FUSE="no")
AC_ARG_ENABLE([sheepfs],
	[  --enable-sheepfs         : enable sheepfs],,
	[ enable_sheepfs=$HAVE_FUSE ],)
AM_CONDITIONAL(BUILD_SHEEPFS, test x$enable_sheepfs = xyes)

AC_ARG_ENABLE([http],
	[ --enable-http : enable http request service (default no) ],,
	[ enable_http="no" ],)
AM_CONDITIONAL(BUILD_HTTP, test x$enable_http = xyes)

AC_ARG_ENABLE([nfs],
	[ --enable-nfs : enable nfs server service (default no) ],,
	[ enable_nfs="no" ],)
AM_CONDITIONAL(BUILD_NFS, test x$enable_nfs = xyes)

AC_ARG_ENABLE([diskvnodes],
	[ --enable-diskvnodes : enable disk as vnodes (default no) ],,
	[ enable_diskvnodes="no" ],)
AM_CONDITIONAL(BUILD_DISKVNODES, test x$enable_diskvnodes = xyes)

AC_ARG_ENABLE([earthquake],
	[ --enable-earthquake : enable earthquake debugger (default no) ],,
	[ enable_earthquake="no" ],)
AM_CONDITIONAL(BUILD_EARTHQUAKE, test x$enable_earthquake = xyes)

AC_ARG_ENABLE([lttng-ust],
	[ --enable-lttng-ust : enable tracing feature based on LTTng UST (default no) ],,
	[ enable_lttng_ust="no" ],)
AM_CONDITIONAL(BUILD_LTTNG_UST, test x$enable_lttng_ust = xyes)

AC_ARG_ENABLE(systemd, AS_HELP_STRING([--enable-systemd],[enable systemd support]),enable_systemd=$enableval,enable_systemd="no")

dnl systemd detection
if test x$enable_systemd = xno ; then
    have_systemd=no;
else
    PKG_CHECK_MODULES([SYSTEMD],
        [libsystemd >= 209],
        [have_systemd=yes])
fi

if test x$have_systemd = xyes; then
    AC_DEFINE(HAVE_SYSTEMD,1,[Have systemd])
fi

if test x$enable_systemd = xyes -a x$have_systemd != xyes ; then
    AC_MSG_ERROR([Explicitly requested systemd support, but systemd not found])
fi

##### systemd unit files
AC_ARG_WITH([systemdsystemunitdir],
AS_HELP_STRING([--with-systemdsystemunitdir=DIR], [Directory for systemd service files]),
    [],
    [
    PKG_CHECK_EXISTS([systemd],
      [with_systemdsystemunitdir=$($PKG_CONFIG --variable=systemdsystemunitdir systemd)],
      [with_systemdsystemunitdir=no])
    ])
if test "x$with_systemdsystemunitdir" != xno; then
   AC_SUBST([systemdsystemunitdir], [$with_systemdsystemunitdir])
fi
AM_CONDITIONAL(HAVE_SYSTEMD, [test "x$have_systemd" != "xno" -a -n "$with_systemdsystemunitdir" -a "x$with_systemdsystemunitdir" != xno ])


CP=cp
OS_LDL="-ldl"
case "$host_os" in
	*linux*)
		AC_DEFINE_UNQUOTED([SHEEPDOG_LINUX], [1],
				   [Compiling for Linux platform])
		OS_CFLAGS=""
		OS_CPPFLAGS=""
		OS_LDFLAGS=""
		OS_DYFLAGS=""
		DARWIN_OPTS=""
	;;
	*)
		AC_MSG_ERROR([Unsupported OS? hmmmm])
	;;
esac

AC_SUBST(CP)
# *FLAGS handling goes here

ENV_CFLAGS="$CFLAGS"
ENV_CPPFLAGS="$CPPFLAGS"
ENV_LDFLAGS="$LDFLAGS"

# debug build stuff
if test "x${enable_debug}" = xyes; then
	AC_DEFINE_UNQUOTED([DEBUG], [1], [Compiling Debugging code])
	OPT_CFLAGS="-O0"
	PACKAGE_FEATURES="$PACKAGE_FEATURES debug"
else
	OPT_CFLAGS="-DNDEBUG"
fi

if test "x${enable_assert}" = xyes; then
	AC_DEFINE_UNQUOTED([ASSERT], [1], [Enabling assert])
	PACKAGE_FEATURES="$PACKAGE_FEATURES assert"
else
	OPT_CFLAGS="-DNASSERT"
fi

# gdb flags
if test "x${GCC}" = xyes; then
	GDB_FLAGS="-ggdb3"
else
	GDB_FLAGS="-g"
fi

if test "x${enable_corosync}" = xyes; then
	PKG_CHECK_MODULES([corosync],[corosync])
	PKG_CHECK_MODULES([libcpg],[libcpg])
	PKG_CHECK_MODULES([libcfg],[libcfg])
	AC_DEFINE_UNQUOTED([HAVE_COROSYNC], 1, [have corosync])
	PACKAGE_FEATURES="$PACKAGE_FEATURES corosync"
fi

if test "x${enable_zookeeper}" = xyes; then
	AC_CHECK_LIB([zookeeper_mt], [zookeeper_init],,
		AC_MSG_ERROR(libzookeeper not found))
	AC_CHECK_HEADERS([zookeeper/zookeeper.h],,
		AC_MSG_ERROR(zookeeper.h header missing))
	AC_DEFINE_UNQUOTED([HAVE_ZOOKEEPER], 1, [have zookeeper])
	PACKAGE_FEATURES="$PACKAGE_FEATURES zookeeper"
fi

if test "x${enable_shepherd}" = xyes; then
	AC_DEFINE_UNQUOTED([HAVE_SHEPHERD], 1, [have shepherd])
	PACKAGE_FEATURES="$PACKAGE_FEATURES shepherd"
fi

if test "x${enable_trace}" = xyes; then
	if test "x${enable_coverage}" = xyes; then
		AC_MSG_ERROR(tracer cannot be used with coverage options)
	fi
	if [[[ $host != *x86_64* ]]]; then
		AC_MSG_ERROR(tracer can be used on x86_64 architectures)
	fi
	AC_CHECK_LIB([bfd], [bfd_openr],,
		AC_MSG_ERROR(requires binutils-dev))
	AC_CHECK_HEADERS([bfd.h],,
		AC_MSG_ERROR(requires binutils-dev))
	AC_DEFINE_UNQUOTED([HAVE_TRACE], 1, [have trace])
	PACKAGE_FEATURES="$PACKAGE_FEATURES trace"
fi

if test "x${enable_sheepfs}" = xyes; then
	AC_CHECK_HEADERS([fuse.h],,
		AC_MSG_ERROR(fuse.h header missing),
		[#define _FILE_OFFSET_BITS 64])
	if test "x${enable_http}" = xyes; then
		AC_CHECK_LIB([curl], [curl_easy_init],,
			     AC_MSG_ERROR(libcurl not found))
		AC_CHECK_HEADERS([curl/curl.h],,
		AC_MSG_ERROR(curl.h header missing))
	fi
	AC_DEFINE_UNQUOTED([HAVE_SHEEPFS], 1, [have sheepfs])
	PACKAGE_FEATURES="$PACKAGE_FEATURES sheepfs"
	PKG_CHECK_EXISTS(fuse >= 2.8.0,
		[AC_DEFINE_UNQUOTED([FUSE_SUPPORT_BIGWRITES],
			1, [Support -obig_writes for fuse])]);
fi

if test "x${enable_http}" = xyes; then
	AC_CHECK_HEADERS([fcgiapp.h],,
		AC_MSG_ERROR(fcgiapp.h header not found))
	AC_CHECK_LIB([fcgi], [FCGX_Accept],,
		AC_MSG_ERROR(libfcgi not found))
	AC_DEFINE_UNQUOTED(HAVE_HTTP, 1, [have http])
	PACKAGE_FEATURES="$PACKAGE_FEATURES http"
fi

if test "x${enable_nfs}" = xyes; then
	AC_CHECK_HEADERS([rpc/rpc.h],,
		AC_MSG_ERROR(rpc.h header not found))
	AC_DEFINE_UNQUOTED(HAVE_NFS, 1, [have nfs])
	PACKAGE_FEATURES="$PACKAGE_FEATURES nfs"
fi

if test "x${enable_diskvnodes}" = xyes; then
	AC_DEFINE_UNQUOTED(HAVE_DISKVNODES, 1, [have diskvnodes])
fi

if test "x${enable_earthquake}" = xyes; then
	AC_DEFINE_UNQUOTED(HAVE_EARTHQUAKE, 1, [have earthquake])
fi

if test "x${enable_lttng_ust}" = xyes; then
	AC_CHECK_HEADERS([lttng/tracepoint.h lttng/tracepoint-event.h],,
		AC_MSG_ERROR(headers of LTTng not found))
	AC_DEFINE_UNQUOTED(HAVE_LTTNG_UST, 1, [have LTTng ust])
	PACKAGE_FEATURES="$PACKAGE_FEATURES LTTng-ust"
fi

# extra warnings
EXTRA_WARNINGS=""

WARNLIST="
	all
	shadow
	missing-prototypes
	missing-declarations
	strict-prototypes
	pointer-arith
	write-strings
	bad-function-cast
	missing-format-attribute
	format=2
	format-security
	format-nonliteral
	no-long-long
	unsigned-char
	gnu89-inline
	no-strict-aliasing
	"

case "${host}" in
	arm*)
	;;
	*)
	WARNLIST="${WARNLIST} cast-align"
	;;
esac

for j in $WARNLIST; do
	if cc_supports_flag -W$j; then
		EXTRA_WARNINGS="$EXTRA_WARNINGS -W$j";
	fi
done

if test "x${enable_coverage}" = xyes && \
		cc_supports_flag -ftest-coverage && \
		cc_supports_flag -fprofile-arcs ; then
	AC_MSG_NOTICE([Enabling Coverage (enable -O0 by default)])
	OPT_CFLAGS="-O0"
	COVERAGE_CFLAGS="-ftest-coverage -fprofile-arcs"
	COVERAGE_LDFLAGS="-ftest-coverage -fprofile-arcs"
	PACKAGE_FEATURES="$PACKAGE_FEATURES coverage"
	enable_unittest="yes"
else
	COVERAGE_CFLAGS=""
	COVERAGE_LDFLAGS=""
fi

if test "x${enable_unittest}" = xyes; then
	PKG_CHECK_MODULES([CHECK], [check >= 0.9.4])
fi
AM_CONDITIONAL(BUILD_UNITTEST, test x$enable_unittest = xyes)


if test "x${enable_fatal_warnings}" = xyes && \
		cc_supports_flag -Werror ; then
	AC_MSG_NOTICE([Enabling Fatal Warnings (-Werror)])
	WERROR_CFLAGS="-Werror"
	PACKAGE_FEATURES="$PACKAGE_FEATURES fatal-warnings"
else
	WERROR_CFLAGS=""
fi

if test "x${enable_trace}" = xyes && \
		cc_supports_flag -pg ; then
	AC_MSG_NOTICE([Enabling trace (-pg)])
	TRACE_CFLAGS="-pg"
else
	TRACE_CFLAGS=""
fi

if test "x${enable_lttng_ust}" = xyes ; then
	AC_MSG_NOTICE([Enabling LTTng ust (-llttng-ust -lurcu-bp)])
	LIBS+="-llttng-ust -lurcu-bp -ldl"
	LTTNG_UST_CFLAGS="-DENABLE_LTTNG_UST"
else
	LTTNG_UST_CFLAGS=""
fi

# final build of *FLAGS
CFLAGS="$ENV_CFLAGS $OPT_CFLAGS $GDB_FLAGS $OS_CFLAGS \
	$TRACE_CFLAGS $COVERAGE_CFLAGS $EXTRA_WARNINGS $WERROR_CFLAGS \
	$LTTNG_UST_CFLAGS -D_GNU_SOURCE -D_LGPL_SOURCE -std=gnu99"
CPPFLAGS="$ENV_CPPFLAGS $ANSI_CPPFLAGS $OS_CPPFLAGS"
LDFLAGS="$ENV_LDFLAGS $COVERAGE_LDFLAGS $OS_LDFLAGS"

# substitute what we need:
AC_SUBST([OS_DYFLAGS])

AM_CONDITIONAL(BUILD_HTML_DOCS, test -n "${GROFF}")

AC_SUBST([INITDDIR])
AC_SUBST([LINT_FLAGS])

AC_DEFINE_UNQUOTED([LOCALSTATEDIR], "$(eval echo ${localstatedir})", [localstate directory])

COROSYSCONFDIR=${sysconfdir}/sheepdog
AC_SUBST([COROSYSCONFDIR])
AC_DEFINE_UNQUOTED([COROSYSCONFDIR], "$(eval echo ${COROSYSCONFDIR})", [sheepdog config directory])

AC_DEFINE_UNQUOTED([PACKAGE_FEATURES], "${PACKAGE_FEATURES}", [sheepdog built-in features])

AC_OUTPUT

AC_MSG_RESULT([])
AC_MSG_RESULT([$PACKAGE configuration:])
AC_MSG_RESULT([  Version                  = ${VERSION}])
AC_MSG_RESULT([  Prefix                   = ${prefix}])
AC_MSG_RESULT([  Executables              = ${sbindir}])
AC_MSG_RESULT([  Man pages                = ${mandir}])
AC_MSG_RESULT([  Doc dir                  = ${docdir}])
AC_MSG_RESULT([  Libraries                = ${libdir}])
AC_MSG_RESULT([  Header files             = ${includedir}])
AC_MSG_RESULT([  Arch-independent files   = ${datadir}])
AC_MSG_RESULT([  State information        = ${localstatedir}])
AC_MSG_RESULT([  System configuration     = ${sysconfdir}])
AC_MSG_RESULT([  System init.d directory  = ${INITDDIR}])
AC_MSG_RESULT([  sheepdog config dir      = ${COROSYSCONFDIR}])
AC_MSG_RESULT([  Features                 =${PACKAGE_FEATURES}])
AC_MSG_RESULT([])
AC_MSG_RESULT([$PACKAGE build info:])
AC_MSG_RESULT([  Library SONAME           = ${SONAME}])
AC_MSG_RESULT(m4_shift(local_soname_list))
AC_MSG_RESULT([  Default optimization     = ${OPT_CFLAGS}])
AC_MSG_RESULT([  Default debug options    = ${GDB_CFLAGS}])
AC_MSG_RESULT([  Extra compiler warnings  = ${EXTRA_WARNING}])
AC_MSG_RESULT([  Env. defined CFLAG       = ${ENV_CFLAGS}])
AC_MSG_RESULT([  Env. defined CPPFLAGS    = ${ENV_CPPFLAGS}])
AC_MSG_RESULT([  Env. defined LDFLAGS     = ${ENV_LDFLAGS}])
AC_MSG_RESULT([  OS   defined CFLAGS      = ${OS_CFLAGS}])
AC_MSG_RESULT([  OS   defined CPPFLAGS    = ${OS_CPPFLAGS}])
AC_MSG_RESULT([  OS   defined LDFLAGS     = ${OS_LDFLAGS}])
AC_MSG_RESULT([  OS   defined LDL         = ${OS_LDL}])
AC_MSG_RESULT([  OS   defined DYFLAGS     = ${OS_DYFLAGS}])
AC_MSG_RESULT([  ANSI defined CPPFLAGS    = ${ANSI_CPPFLAGS}])
AC_MSG_RESULT([  Coverage     CFLAGS      = ${COVERAGE_CFLAGS}])
AC_MSG_RESULT([  Coverage     LDFLAGS     = ${COVERAGE_LDFLAGS}])
AC_MSG_RESULT([  Fatal War.   CFLAGS      = ${WERROR_CFLAGS}])
AC_MSG_RESULT([  Trace        CFLAGS      = ${TRACE_CFLAGS}])
AC_MSG_RESULT([  Final        CFLAGS      = ${CFLAGS}])
AC_MSG_RESULT([  Final        CPPFLAGS    = ${CPPFLAGS}])
AC_MSG_RESULT([  Final        LDFLAGS     = ${LDFLAGS}])
