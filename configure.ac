# Initialization
AC_INIT([LibRaw], m4_esyscmd([./version.sh]), [info@libraw.org], [], [http://www.libraw.org])
AM_INIT_AUTOMAKE([foreign no-define])
#AM_SILENT_RULES([yes])
AC_CONFIG_MACRO_DIR([m4])
AC_CANONICAL_HOST

# Tools to use
AC_PROG_CXX
AC_PROG_CC
AC_PROG_LIBTOOL
AC_ENABLE_SHARED
AC_ENABLE_STATIC
AC_LIBTOOL_WIN32_DLL
AC_LIBTOOL_SETUP
AC_SUBST(LIBTOOL_DEPS)

# Config files to generate

AC_CONFIG_FILES([
	Makefile
	libraw.pc
	libraw_r.pc
])

# check if we want OpenMP support
AC_ARG_ENABLE([openmp],
	[  --enable-openmp    Enable OpenMP support],
	[case "${enableval}" in
		yes) 	openmp=true ;;
		no)		openmp=false ;;
		*)		AC_MSG_ERROR([bad value ${enableval} for --enable-openmp]) ;;
	esac],[openmp=true])

if test x$openmp = xtrue ; then
	AX_OPENMP([
		CXXFLAGS="$CXXFLAGS $OPENMP_CFLAGS"
		CFLAGS="$CFLAGS $OPENMP_CFLAGS"
		AC_SUBST([PC_OPENMP],[" $OPENMP_CFLAGS"])		
	],[
		AC_MSG_WARN([OpenMP support cannot be enabled because your system doesn't support it.])		
	])

	SAVE_LDFLAGS="$LDFLAGS"
	LDFLAGS="-nostdlib"

	AC_MSG_CHECKING([whether OpenMP works without explicit -l])

	AC_LINK_IFELSE(
		[AC_LANG_PROGRAM([[#include <omp.h>]], [[omp_get_num_threads();]])],
		[OPENMP_LIBS=""; AC_MSG_RESULT([yes])],	# -l not required if success
		[
			AC_MSG_RESULT([no])
			AC_MSG_CHECKING([for OpenMP library])

			# try -lomp (clang), -lgomp (gcc), -liomp5 (intel)
			LDFLAGS="-nostdlib -lomp $SAVE_LDFLAGS"
			AC_LINK_IFELSE(
				[AC_LANG_PROGRAM([[#include <omp.h>]], [[omp_get_num_threads();]])],
				[OPENMP_LIBS="-lomp"; AC_MSG_RESULT([found -lomp])],
				[
					LDFLAGS="-nostdlib -lgomp $SAVE_LDFLAGS"
					AC_LINK_IFELSE(
						[AC_LANG_PROGRAM([[#include <omp.h>]], [[omp_get_num_threads();]])],
						[OPENMP_LIBS="-lgomp"; AC_MSG_RESULT([found -lgomp])],
						[
							LDFLAGS="-nostdlib -liomp5 $SAVE_LDFLAGS"
							AC_LINK_IFELSE(
								[AC_LANG_PROGRAM([[#include <omp.h>]], [[omp_get_num_threads();]])],
								[OPENMP_LIBS="-liomp5"; AC_MSG_RESULT([found -liomp5])],
								[AC_MSG_ERROR([No suitable OpenMP library found])]
							)
						]
					)
				]
			)
		]
	)

	LDFLAGS="$SAVE_LDFLAGS"

	AC_SUBST([OPENMP_LIBS])

fi

# check for libjpeg v8
AC_ARG_ENABLE([jpeg],
	[  --enable-jpeg    Enable JPEG support for Lossy compressed DNG files],
	[case "${enableval}" in
		yes)	jpeg=true ;;
		no)	jpeg=false ;;
		*)	AC_MSG_ERROR([bad value ${enableval} for --enable-jpeg]) ;;
	esac],[jpeg=true])

if test x$jpeg = xtrue; then
        AC_CHECK_LIB([jpeg], [jpeg_mem_src],
                [
                        AC_CHECK_HEADERS([jpeglib.h], [
                                CPPFLAGS="$CPPFLAGS -DUSE_JPEG -DUSE_JPEG8"
                                LIBS="$LIBS -ljpeg"
                                AC_SUBST([PACKAGE_LIBS_PRIVATE],"-ljpeg $PACKAGE_LIBS_PRIVATE")
                        ], AC_MSG_WARN([no jpeg headers found]))
                ],
                AC_MSG_WARN([libjpeg not found])
        )
fi

# check if we want zlib support
AC_ARG_ENABLE([zlib],
	[  --enable-zlib    Enable zlib support for deflate compressed DNG files],
	[case "${enableval}" in
		yes)	zlib=true ;;
		no)		zlib=false ;;
		*)		AC_MSG_ERROR([bad value ${enableval} for --enable-zlib]) ;;
	esac],[zlib=true])

if test x$zlib = xtrue; then
	PKG_CHECK_MODULES([ZLIB],[zlib],[
		CPPFLAGS="$CPPFLAGS $ZLIB_CFLAGS -DUSE_ZLIB"
		LIBS="$LIBS $ZLIB_LIBS"
		AC_SUBST([PACKAGE_REQUIRES],[zlib])
	],
		AC_MSG_WARN([zlib support cannot be enabled])
	)
fi

# check if we want LCMS support
AC_ARG_ENABLE([lcms],
	[  --enable-lcms    Enable LCMS support],
	[case "${enableval}" in
		yes)	lcms=true ;;
		no)		lcms=false ;;
		*)		AC_MSG_ERROR([bad value ${enableval} for --enable-lcms]) ;;
	esac],[lcms=true])

if test x$lcms = xtrue; then
	PKG_CHECK_MODULES([LCMS2],[lcms2],[
		CPPFLAGS="$CPPFLAGS $LCMS2_CFLAGS -DUSE_LCMS2"
		LIBS="$LIBS $LCMS2_LIBS"
	        AC_SUBST([PACKAGE_REQUIRES],[lcms2])
	],[
	PKG_CHECK_MODULES([LCMS],[lcms],[
		CPPFLAGS="$CPPFLAGS $LCMS_CFLAGS -DUSE_LCMS"
		LIBS="$LIBS $LCMS_LIBS"
	        AC_SUBST([PACKAGE_REQUIRES],[lcms])
	],[
		AC_MSG_WARN([LCMS support cannot be enabled])
	])
	])
fi

# check if we want build examples
AC_ARG_ENABLE([examples],
	[  --enable-examples    Enable building of examples],
	[case "${enableval}" in
		yes) 	examples=true ;;
		no)		examples=false ;;
		*)		AC_MSG_ERROR([bad value ${enableval} for --enable-examples]) ;;
	esac],[examples=true])
AM_CONDITIONAL([EXAMPLES], [test x$examples = xtrue])

LIBS="$LIBS -lm"

case "${host_os}" in
	*mingw32*) LIBS="$LIBS -lws2_32" ;;
esac

AC_SUBST([LIBRAW_SHLIB_VERSION],m4_esyscmd([./shlib-version.sh]))
AC_SUBST([LIBRAW_RELEASE_VERSION],m4_esyscmd([./version.sh]))

AC_OUTPUT

