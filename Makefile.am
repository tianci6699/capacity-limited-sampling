# The goal of -Wno-syntax here is just to suppress the Automake warning
# about overriding distdir, below.
AUTOMAKE_OPTIONS = foreign -Wno-syntax subdir-objects
ACLOCAL_AMFLAGS = -I m4
#SUBDIRS = datapath
SUBDIRS =

if HAVE_DPKG_BUILDPACKAGE
distcheck-hook:
	cd $(srcdir) && dpkg-buildpackage -rfakeroot -us -uc
	cd $(srcdir) && fakeroot ./debian/rules clean
else
distcheck-hook:
endif

AM_CPPFLAGS = $(SSL_CFLAGS) -g
AM_CPPFLAGS += -I $(top_srcdir)/include
AM_CPPFLAGS += -I $(top_srcdir)/lib

AM_CFLAGS = -Wstrict-prototypes    #######-Werror

if NDEBUG
AM_CPPFLAGS += -DNDEBUG
AM_CFLAGS += -fomit-frame-pointer
else
AM_LDFLAGS = -export-dynamic
endif

CLEANFILES =
DISTCLEANFILES =
EXTRA_DIST =
TESTS =
TESTS_ENVIRONMENT =
bin_PROGRAMS =
bin_SCRIPTS =
#dist_commands_DATA =
dist_man_MANS =
dist_pkgdata_SCRIPTS =
dist_sbin_SCRIPTS =
man_MANS =
noinst_HEADERS =
noinst_LIBRARIES =
noinst_PROGRAMS =
noinst_SCRIPTS =

EXTRA_DIST += README.hwtables soexpand.pl regress

ro_c = echo '/* -*- mode: c; buffer-read-only: t -*- */'

SUFFIXES = .in
.in:
	$(PERL) $(srcdir)/soexpand.pl -I$(srcdir) < $< | \
	    sed -e 's,[@]LOGDIR[@],$(LOGDIR),g' \
		-e 's,[@]PKIDIR[@],$(PKIDIR),g' \
                -e 's,[@]RUNDIR[@],$(RUNDIR),g' \
                -e 's,[@]pkgdatadir[@],$(pkgdatadir),g' \
                -e 's,[@]PERL[@],$(PERL),g' > $@

include lib/automake.mk
include nbee_link/automake.mk
include oflib/automake.mk
include oflib-exp/automake.mk
include secchan/automake.mk
include utilities/automake.mk
include udatapath/automake.mk
include include/automake.mk
include debian/automake.mk

netpdldir = $(datadir)/openflow
netpdl_DATA = customnetpdl.xml
AM_CPPFLAGS += -DNETPDLDIR='"$(netpdldir)"'
