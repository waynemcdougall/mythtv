include config.mak

SRC_PATH = ${SRC_PATH_BARE}/external/FFmpeg
SRC_DIR = $(SRC_PATH)

vpath %.c    $(SRC_DIR)
vpath %.cpp  $(SRC_DIR)
vpath %.h    $(SRC_DIR)
vpath %.S    $(SRC_DIR)
vpath %.asm  $(SRC_DIR)
vpath %.v    $(SRC_DIR)
vpath %.texi $(SRC_DIR)
vpath %/fate_config.sh.template $(SRC_DIR)

PROGS-$(CONFIG_FFMPEG)   += ffmpeg
PROGS-$(CONFIG_FFPLAY)   += ffplay
PROGS-$(CONFIG_FFPROBE)  += ffprobe
PROGS-$(CONFIG_FFSERVER) += ffserver

PROGS      := $(PROGS-yes:%=%$(EXESUF))
INSTPROGS   = $(PROGS-yes:%=%$(PROGSSUF)$(EXESUF))
OBJS        = $(PROGS-yes:%=%.o) cmdutils.o
TESTTOOLS   = audiogen videogen rotozoom tiny_psnr base64
HOSTPROGS  := $(TESTTOOLS:%=tests/%) doc/print_options
TOOLS       = qt-faststart trasher
TOOLS-$(CONFIG_ZLIB) += cws2fws
MYTHPROGS   = $(addprefix myth, ${PROGS})

BASENAMES   = ffmpeg ffplay ffprobe ffserver
ALLPROGS    = $(BASENAMES:%=%$(PROGSSUF)$(EXESUF))
ALLPROGS_G  = $(BASENAMES:%=%$(PROGSSUF)_g$(EXESUF))
ALLMANPAGES = $(BASENAMES:%=%.1)

FFLIBS-$(CONFIG_AVDEVICE) += avdevice
FFLIBS-$(CONFIG_AVFILTER) += avfilter
FFLIBS-$(CONFIG_AVFORMAT) += avformat
FFLIBS-$(CONFIG_AVRESAMPLE) += avresample
FFLIBS-$(CONFIG_AVCODEC)  += avcodec
FFLIBS-$(CONFIG_POSTPROC) += postproc
FFLIBS-$(CONFIG_SWRESAMPLE)+= swresample
FFLIBS-$(CONFIG_SWSCALE)  += swscale

FFLIBS := avutil

DATA_FILES := $(wildcard $(SRC_PATH)/presets/*.ffpreset) $(SRC_PATH)/doc/ffprobe.xsd
EXAMPLES_FILES := $(wildcard $(SRC_PATH)/doc/examples/*.c) $(SRC_PATH)/doc/examples/Makefile

SKIPHEADERS = cmdutils_common_opts.h

include common.mak

FF_EXTRALIBS := $(FFEXTRALIBS)
FF_DEP_LIBS  := $(DEP_LIBS)

all: $(PROGS)

$(PROGS): %$(EXESUF): %$(PROGSSUF)_g$(EXESUF)
	$(CP) $< $@$(PROGSSUF)
	$(STRIP) $@$(PROGSSUF)

$(TOOLS): %$(EXESUF): %.o
	$(LD) $(LDFLAGS) -o $@ $< $(ELIBS)

tools/cws2fws$(EXESUF): ELIBS = -lz

${MYTHPROGS}: myth%: %
	$(CP) $< $@

config.h: .config
.config: $(wildcard $(FFLIBS:%=$(SRC_PATH)/lib%/all*.c))
	@-tput bold 2>/dev/null
	@-printf '\nWARNING: $(?F) newer than config.h, rerun configure\n\n'
	@-tput sgr0 2>/dev/null

SUBDIR_VARS := CLEANFILES EXAMPLES FFLIBS HOSTPROGS TESTPROGS TOOLS      \
               ARCH_HEADERS BUILT_HEADERS SKIPHEADERS                    \
               ALTIVEC-OBJS ARMV5TE-OBJS ARMV6-OBJS ARMVFP-OBJS MMI-OBJS \
               MMX-OBJS NEON-OBJS VIS-OBJS YASM-OBJS                     \
               OBJS TESTOBJS

define RESET
$(1) :=
$(1)-yes :=
endef

define DOSUBDIR
$(foreach V,$(SUBDIR_VARS),$(eval $(call RESET,$(V))))
SUBDIR := $(1)/
include $(SRC_PATH_BARE)/external/FFmpeg/$(1)/Makefile
-include $(SRC_PATH_BARE)/external/FFmpeg/$(1)/$(ARCH)/Makefile
include $(SRC_PATH_BARE)/external/FFmpeg/library.mak
endef

$(foreach D,$(FFLIBS),$(eval $(call DOSUBDIR,lib$(D))))

ffplay.o: CFLAGS += $(SDL_CFLAGS)
ffplay_g$(EXESUF): FF_EXTRALIBS += $(SDL_LIBS)
ffserver_g$(EXESUF): LDFLAGS += $(FFSERVERLDFLAGS)

%$(PROGSSUF)_g$(EXESUF): %.o cmdutils.o $(FF_DEP_LIBS)
	$(LD) $(LDFLAGS) -o $@ $< cmdutils.o $(FF_EXTRALIBS)

OBJDIRS += tools

-include $(wildcard tools/*.d)

VERSION_SH  = $(SRC_PATH_BARE)/external/FFmpeg/version.sh
GIT_LOG     = $(SRC_PATH_BARE)/.git/logs/HEAD

.version: $(wildcard $(GIT_LOG)) $(VERSION_SH) config.mak
.version: M=@

version.h .version:
	$(M)$(VERSION_SH) $(SRC_PATH) version.h $(EXTRA_VERSION)
	$(Q)touch .version

# force version.sh to run whenever version might have changed
-include .version

ifdef PROGS
install: install-mythprogs
endif

install: install-libs install-headers

install-libs: install-libs-yes

install-progs-yes:
install-progs-$(CONFIG_SHARED): install-libs

install-progs: install-progs-yes $(PROGS)
	$(Q)mkdir -p "$(BINDIR)"
	$(INSTALL) -c -m 755 $(INSTPROGS) "$(BINDIR)"

install-mythprogs: install-progs-yes $(MYTHPROGS)
	$(Q)mkdir -p "$(BINDIR)"
	$(INSTALL) -c -m 755 $(MYTHPROGS) "$(BINDIR)"

install-data: $(DATA_FILES) $(EXAMPLES_FILES)
	$(Q)mkdir -p "$(DATADIR)/examples"
	$(INSTALL) -m 644 $(DATA_FILES) "$(DATADIR)"
	$(INSTALL) -m 644 $(EXAMPLES_FILES) "$(DATADIR)/examples"

uninstall: uninstall-libs uninstall-headers uninstall-progs uninstall-data

uninstall-progs:
	$(RM) $(addprefix "$(BINDIR)/", $(ALLPROGS))

uninstall-data:
	$(RM) -r "$(DATADIR)"

clean::
	$(RM) $(ALLPROGS) $(ALLPROGS_G)
	$(RM) $(CLEANSUFFIXES)
	$(RM) $(TOOLS)
	$(RM) $(CLEANSUFFIXES:%=tools/%)
	$(RM) coverage.info
	$(RM) -r coverage-html

distclean::
	$(RM) $(DISTCLEANSUFFIXES)
	$(RM) config.* .version version.h libavutil/avconfig.h

config:
	$(SRC_PATH)/configure $(value FFMPEG_CONFIGURATION)

# Without the sed genthml thinks "libavutil" and "./libavutil" are two different things
coverage.info: $(wildcard *.gcda *.gcno */*.gcda */*.gcno */*/*.gcda */*/*.gcno)
	$(Q)lcov -c -d . -b . | sed -e 's#/./#/#g' > $@

coverage-html: coverage.info
	$(Q)mkdir -p $@
	$(Q)genhtml -o $@ $<
	$(Q)touch $@

#include $(SRC_PATH)/doc/Makefile
#include $(SRC_PATH)/tests/Makefile

$(sort $(OBJDIRS)):
	$(Q)mkdir -p $@

# Dummy rule to stop make trying to rebuild removed or renamed headers
%.h:
	@:

# Disable suffix rules.  Most of the builtin rules are suffix rules,
# so this saves some time on slow systems.
.SUFFIXES:

.PHONY: all all-yes alltools check *clean config install*
.PHONY: testprogs uninstall*
