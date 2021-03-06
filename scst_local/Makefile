#
# A Makefile for the scst-local ...
#

SHELL=/bin/bash

KMOD 		:= $(shell pwd)/kernel

ifneq ($(PATCHLEVEL),)
ccflags-y	+= -I$(KBUILD_EXTMOD)/../scst/include
ccflags-y	+= $(call cc-option,-Wextra) -Wno-unused-parameter\
	-Wno-missing-field-initializers

#ccflags-y += -DCONFIG_SCST_LOCAL_DIRECT_PROCESSING

obj-m	:= scst_local.o
else
######### BEGIN OUT-OF-TREE RULES #########

ifndef PREFIX
        PREFIX=/usr/local
endif

ifeq ($(KVER),)
  ifeq ($(KDIR),)
    KVER := $(shell uname -r)
    KDIR := /lib/modules/$(KVER)/build
  else
    KVER := $(strip $(shell						\
	cat $(KDIR)/include/config/kernel.release 2>/dev/null ||	\
	make -s -C $(KDIR) kernelversion))
  endif
else
  KDIR := /lib/modules/$(KVER)/build
endif

ifeq ($(INSTALL_MOD_PATH),)
  export INSTALL_MOD_PATH := $(DESTDIR)
endif

INSTALL_DIR := $(INSTALL_MOD_PATH)/lib/modules/$(KVER)/extra

SCST_DIR := $(shell echo "$$PWD")/../scst/src

# The file Modules.symvers has been renamed in the 2.6.18 kernel to
# Module.symvers. Find out which name to use by looking in $(KDIR).
MODULE_SYMVERS:=$(shell if [ -e "$(KDIR)/Modules.symvers" ]; then \
		       echo Modules.symvers; else echo Module.symvers; fi)

all:
	$(MAKE) -C $(KDIR) M=$(shell pwd)				\
		KBUILD_EXTRA_SYMBOLS=$(SCST_DIR)/$(MODULE_SYMVERS)

install: all
	KDIR=$(KDIR) ../scripts/sign-modules
	$(MAKE) -C $(KDIR) M=$(shell pwd)				\
		$$([ -n "$(DEPMOD)" ] && echo "DEPMOD=$(DEPMOD)")	\
		CONFIG_MODULE_SIG_ALL= modules_install

uninstall:
	rm -f $(INSTALL_DIR)/scst_local.ko
	-/sbin/depmod -b $(INSTALL_MOD_PATH)/ -a $(KVER)

########## END OUT-OF-TREE RULES ##########
endif

clean:
	@$(MAKE) -C $(KDIR) M=$(shell pwd) clean
	@$(RM) tags $(MODULE_SYMVERS) Module.markers modules.order

extraclean: clean
	rm -f *.orig *.rej

release-archive:
	../scripts/generate-release-archive scst_local "$$(sed -n 's/^#define[[:blank:]]SCST_LOCAL_VERSION[[:blank:]]*\"\([^\"]*\)\".*/\1/p' scst_local.c)"

.PHONY: all tgt install uninstall clean extraclean

