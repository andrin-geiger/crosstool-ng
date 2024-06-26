## Process this file with automake to produce Makefile.in
## vim: set noet :

include verbatim-data.mk

SUBDIRS		= kconfig

bin_SCRIPTS		= ct-ng
CLEANFILES		= ct-ng bash-completion/ct-ng docs/ct-ng.1
EXTRA_DIST		= bootstrap ct-ng.in bash-completion/ct-ng.in \
				  docs/ct-ng.1.in maintainer testing

if INSTALL_BASH_COMPLETION
compdir				= @BASH_COMPLETION_DIR@
comp_SCRIPTS		= bash-completion/ct-ng
endif

man1_MANS	= docs/ct-ng.1

# paths.sh and configure.in are generated by configure
nobase_dist_pkgdata_DATA = $(verbatim_data)
nobase_nodist_pkgdata_DATA = paths.sh config/configure.in

# ct-ng, as renamed by the installation
ctng_progname		= `echo ct-ng | sed '$(transform)'`

USER_MANUAL_FILES = \
	docs/manual/1_Introduction.md \
	docs/manual/2_Installation.md \
	docs/manual/3_Configuration.md \
	docs/manual/4_Building.md \
	docs/manual/5_Toolchain_Usage.md \
	docs/manual/6_Toolchain_Types.md \
	docs/manual/7_Contributing.md \
	docs/manual/8_Internals.md \
	docs/manual/9_Toolchain_Construction.md \
	docs/manual/A_Credits.md \
	docs/manual/B_Known_issues.md \
	docs/manual/C_Setup_OS.md \
	docs/manual/D_Tutorials_features.md

if INSTALL_USER_MANUAL
doc_DATA		= $(USER_MANUAL_FILES)
else
doc_DATA		= docs/MANUAL_ONLINE
endif

do_subst		= ( @SED@ \
			-e 's,[@]docdir[@],$(docdir),g' \
			-e 's,[@]pkgdatadir[@],$(pkgdatadir),g' \
			-e 's,[@]pkglibexecdir[@],$(pkglibexecdir),g' \
			-e 's,[@]progname[@],'$(ctng_progname)',g' \
		| $(SHELL) config.status --file=- )

ct-ng: ct-ng.in Makefile
	$(AM_V_GEN)$(do_subst) < $< >$@-t && chmod a-w,a+x $@-t && mv -f $@-t $@

docs/ct-ng.1: docs/ct-ng.1.in Makefile
	$(AM_V_GEN)$(MKDIR_P) docs && $(do_subst) < $< >$@-t && mv -f $@-t $@

bash-completion/ct-ng: bash-completion/ct-ng.in Makefile
	$(AM_V_GEN)$(MKDIR_P) bash-completion && $(do_subst) < $< >$@-t && mv -f $@-t $@

# automake doesn't offer an option to compress manpages, do it after installation
# Also, lintian is picky about execute-bit on the scripts
install-data-hook:
	rm -f $(DESTDIR)$(man1dir)/$(ctng_progname).1.gz
	gzip -n -9 $(DESTDIR)$(man1dir)/$(ctng_progname).1
	chmod a+x $(DESTDIR)$(pkgdatadir)/scripts/config.guess
	chmod a+x $(DESTDIR)$(pkgdatadir)/scripts/config.sub
	rm -f $(DESTDIR)$(pkgdatadir)/LICENSE
	ln -s COPYING $(DESTDIR)$(pkgdatadir)/LICENSE
if INSTALL_BASH_COMPLETION
	chmod a-x $(DESTDIR)$(compdir)/$(ctng_progname)
endif

uninstall-hook:
	rm -f $(DESTDIR)$(man1dir)/$(ctng_progname).1.gz

dist-hook: fetch-docs create-symlinks gen-tarball-version

.PHONY: fetch-docs
fetch-docs: maintainer/download-docs.sh
	$< $(top_distdir) $(USER_MANUAL_FILES)

.PHONY: create-symlinks
create-symlinks:
	ln -s COPYING $(top_distdir)/LICENSE

.PHONY: gen-tarball-version
gen-tarball-version:
	echo $(VERSION) > $(distdir)/.tarball-version

DIGESTS		?= md5 sha1 sha512
USE_GPG		?= yes

dist-create-release: distcheck
	for a in $(DIST_ARCHIVES); do \
		for d in $(DIGESTS); do \
			$${d}sum $${a} > $${a}.$${d}; \
		done; \
		if [ "$(USE_GPG)" = "yes" ]; then \
			gpg --detach-sign $${a}; \
		fi; \
	done
