#
# configure.ac for libnids.
#
# Dug Song <dugsong@monkey.org>
# ... mantained by Nergal
# Modified by Ali Abdulkadir <autostart.ini@gmail.com>

AC_INIT(
    [libnids],
    [1.24],
    [https://github.com/sgeto/libnids/issues],
    [libdnet],
    [https://github.com/sgeto/libnids])

AC_CONFIG_AUX_DIR([build-aux])
AC_CONFIG_SRCDIR([src/libnids.c])

AC_CANONICAL_TARGET

AC_CONFIG_HEADER(src/config.h)
AC_CONFIG_MACRO_DIR([m4])

# Guess the root install directory (if none is provided)
if test "${prefix}" = "NONE"; then
    if test -w /usr/local; then
        prefix=/usr/local
    else
        AC_PREFIX_PROGRAM($CC)
    fi
fi

if test "${exec_prefix}" = "NONE"; then
    exec_prefix=$prefix
fi

# To speed up the configure process on Windows, we will define a WIN32
# variable this early and use it to exclude those checks that are not​
# needed or will always fail on Windows.
case "$target_os" in
*mingw*)
    WIN32=yes;;
*)
    WIN32=no;;
esac
AC_SUBST(WIN32)

# Checks for programs.
AC_PROG_CC
AC_PROG_INSTALL
AC_PROG_RANLIB

# Checks for header files.
AC_HEADER_STDC
AC_CHECK_HEADERS(sys/time.h syslog.h unistd.h)

# Checks for typedefs, structures, and compiler characteristics.
AC_C_CONST
AC_C_INLINE
AC_HEADER_TIME
AC_C_BIGENDIAN

# XXX - Linux sux.
CFLAGS="$CFLAGS -D_BSD_SOURCE"
# Checks for library functions.
AC_CHECK_FUNCS(gettimeofday)
# XXX - Solaris sux.
AC_CHECK_LIB(socket, socket)
AC_CHECK_LIB(nsl, gethostbyname)

case "$target_cpu" in
	alpha*|arm*|hp*|mips*|sparc*)
		ac_cv_lbl_unaligned_fail=yes
		;;
	*)
		ac_cv_lbl_unaligned_fail=no
		;;
esac
if test $ac_cv_lbl_unaligned_fail = yes ; then
            AC_DEFINE(LBL_ALIGN,1,[if unaligned access fails])
fi

# Checks for libpcap
AC_MSG_CHECKING(for libpcap)
AC_ARG_WITH(libpcap,
[  --with-libpcap=DIR	use libpcap build directory],
[ case "$withval" in
  yes|no)
     AC_MSG_RESULT(no)
     ;;
  *)
     AC_MSG_RESULT($withval)
     if test -f $withval/pcap.h -a -f $withval/libpcap.a; then
        owd=`pwd`
        if cd $withval; then withval=`pwd`; cd $owd; fi
	PCAP_CFLAGS="-I$withval -I$withval/bpf"
	PCAPLIB="-L$withval -lpcap"
     else
        AC_MSG_ERROR(pcap.h or libpcap.a not found in $withval)
     fi
     ;;
  esac ],
[ if test -f ${prefix}/include/pcap.h; then
     PCAP_CFLAGS="-I${prefix}/include"
     PCAPLIB="-L${exec_prefix}/lib -lpcap"
  elif test -f /usr/include/pcap/pcap.h; then
     PCAP_CFLAGS="-I/usr/include/pcap"
     PCAPLIB="-lpcap"
  else
	TMP=$LIBS
	LIBS="-lpcap $LIBS"
	AC_LINK_IFELSE([AC_LANG_PROGRAM([[#include <pcap.h>]], [[pcap_open_offline("","")]])],[LIBPCAP_FOUND=1],[LIBPCAP_FOUND=0])
	LIBS=$TMP
	if test $LIBPCAP_FOUND = 1 ; then
		PCAPLIB="-lpcap"
	else
		AC_MSG_ERROR(libpcap not found)
	fi
  fi
  AC_MSG_RESULT(yes) ]
)
AC_SUBST(PCAP_CFLAGS)
AC_SUBST(PCAPLIB)

# Checks for libglib2.8
AC_ARG_ENABLE(libglib,
[  --disable-libglib	use glib2 for multiprocessing support],
[
    AC_MSG_RESULT(skipping glib2 support)
],
[
    PKG_PROG_PKG_CONFIG
    PKG_CHECK_MODULES(GLIB, glib-2.0 >= 2.2.0,[
	PKG_CHECK_MODULES(GTHREAD, gthread-2.0 >= 2.2.0,,)
	AC_CHECK_LIB(gthread-2.0,g_thread_init,,,$GTHREAD_LIBS)
    ],)
]
)

# Checks for libnet
AC_MSG_CHECKING(for libnet)
AC_ARG_ENABLE(libnet, 
[  --disable-libnet	whether to include code requiring libnet],
[ case "$enableval" in
  yes)
     AC_MSG_RESULT(yes)
     ;;
  no)
     AC_MSG_RESULT(no)
     LIBNET_VER=-1
     ;;
  *)
        AC_MSG_ERROR(no arguments expected for --disable-libnet)
     ;;
  esac ]
)

AC_ARG_WITH(libnet,
[  --with-libnet=DIR	use libnet build directory],
[ case "$withval" in
  yes)
     ;;
  no)
     LIBNET_VER=-1
     AC_MSG_RESULT(no)
     ;;
  *)
     AC_MSG_RESULT($withval)
     if test -f $withval/include/libnet.h -a -f $withval/lib/libnet.a -a -f $withval/libnet-config ; then
        owd=`pwd`
        if cd $withval; then withval=`pwd`; cd $owd; fi
	LNET_CFLAGS="-I$withval/include `$withval/libnet-config --defines`"
	LNETLIB="-L$withval/lib -lnet"
     elif test -f $withval/include/libnet.h -a -f $withval/src/libnet.a; then
        owd=`pwd`
        if cd $withval; then withval=`pwd`; cd $owd; fi
	LNET_CFLAGS="-I$withval/include"
	LNETLIB="-L$withval/src -lnet"
     else
	echo "A working combination of libnet.h, libnet.a and libnet-config not found in $withval; get libnet from www.packetfactory.net/projects/libnet and reinstall"
        AC_MSG_ERROR(libnet)
     fi
     ;;
  esac ],
[ if test "x"$LIBNET_VER = "x"-1 ; then
	AC_MSG_RESULT(skipping libnet)
  else
	if test -f ${prefix}/include/libnet.h -a ${exec_prefix}/lib/libnet.a ; then
	     LNET_CFLAGS="-I${prefix}/include `${exec_prefix}/bin/libnet-config --defines 2>/dev/null`"
	     LNETLIB="-L${exec_prefix}/lib -lnet"
	else
	     LNET_CFLAGS="`libnet-config --defines 2>/dev/null`"
	     LNETLIB="-lnet"
	fi
  fi ]
)

if test "x"$LIBNET_VER != "x"-1 ; then
	TMPC="$CFLAGS"
	TMPL="$LIBS"
	CFLAGS="$CFLAGS $LNET_CFLAGS"
	LIBS="$LNETLIB $LIBS"
	AC_LINK_IFELSE([AC_LANG_PROGRAM([[#include <libnet.h>]], [[libnet_get_prand(0)]])],[LIBNET_FOUND=1],[LIBNET_FOUND=0])
	CFLAGS="$TMPC"
	LIBS="$TMPL"
	if test $LIBNET_FOUND = 1 ; then
		LNETLIB="-lnet"
		AC_MSG_RESULT(yes)
	else
		echo "Working libnet not found; get it from www.packetfactory.net/projects/libnet and reinstall"
		AC_MSG_ERROR(libnet)
	fi
fi

AC_SUBST(LNET_CFLAGS)
AC_SUBST(LNETLIB)

BUILD_SHARED=
AC_MSG_CHECKING(whether to build shared library)
AC_ARG_ENABLE(shared,
[  --enable-shared	enable building shared libraries],
[ case "$enableval" in
  yes)
     AC_MSG_RESULT(yes)
     BUILD_SHARED=shared
     ;;
  no)
     AC_MSG_RESULT(no)
     ;;
  *)
        AC_MSG_ERROR(no arguments expected for --enable-shared)
     ;;
  esac ],
  [AC_MSG_RESULT(no)]
)
AC_SUBST(BUILD_SHARED)

AC_MSG_CHECKING(the name of struct icmp)
AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[#include <sys/types.h>
#include <netinet/in.h>
#include <netinet/ip.h>
#include <netinet/ip_icmp.h>]], [[struct icmphdr h;int c=h.type]])],[ICMPHEADER=1],[ICMPHEADER=0])
AC_SUBST(ICMPHEADER)
if test $ICMPHEADER = 1 ; then 
	AC_MSG_RESULT(struct icmphdr) ; else AC_MSG_RESULT(struct icmp) 
fi
AC_MSG_CHECKING(if tcp states are defined)
AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[#include <sys/types.h>
#include <netinet/in.h>
#include <netinet/tcp.h>]], [[int c=TCP_ESTABLISHED]])],[TCPSTATES=1],[TCPSTATES=0])
AC_SUBST(TCPSTATES)
if test $TCPSTATES = 1 ; then 
	AC_MSG_RESULT(yes) ; else AC_MSG_RESULT(no) 
fi

AC_MSG_CHECKING(for bsd-ish struct udphdr)
AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[#include <sys/types.h>
#include <netinet/in.h>
#include <netinet/udp.h>]], [[struct udphdr h;int c=h.uh_ulen]])],[HAVE_BSD_UDPHDR=1],[HAVE_BSD_UDPHDR=0])
AC_SUBST(HAVE_BSD_UDPHDR)
if test $HAVE_BSD_UDPHDR = 1 ; then 
	AC_MSG_RESULT(yes) ; else AC_MSG_RESULT(no) 
fi
if test "x"$LIBNET_VER != "x"-1 ; then 
	TMP=$CFLAGS
	CFLAGS="$CFLAGS $LNET_CFLAGS"
	AC_MSG_CHECKING(libnet version)
	AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[#include <libnet.h>]], [[int c=LIBNET_PTAG_INITIALIZER]])],[LIBNET_VER=1],[LIBNET_VER=0])
	CFLAGS=$TMP
	if test $LIBNET_VER = 1 ; then 
		AC_MSG_RESULT(looks new) ; else AC_MSG_RESULT(looks old) 
	fi
fi
AC_SUBST(LIBNET_VER)
AC_CONFIG_FILES([Makefile src/Makefile samples/Makefile])
AC_OUTPUT
