.DEFAULT_GOAL := check

SHELL := /bin/bash

CDDL_DIR := ./
IMPORTS_DIR := $(CDDL_DIR)imports/
EXPORTS_DIR := $(CDDL_DIR)exports/

include tools.mk
include funcs.mk
include $(IMPORTS_DIR)import-frags.mk

%.cbor: %.diag
	$(diag2cbor) $< > $@

IRIM := irim
ICORIM := icorim
ICE := ice
ISPDM := ispdm
INTEL_PROFILE := intel-profile

check:: check-$(IRIM) check-$(IRIM)-examples
check:: check-$(ICORIM) check-$(ICORIM)-examples
check:: check-$(ICE) check-$(ICE)-examples
check:: check-$(ISPDM) check-$(ISPDM)-examples
check:: export-all

# Get Intel Profile frags - no dependencies
include $(CDDL_DIR)profile-frags.mk
PROFILE_DEPS := $(addprefix $(CDDL_DIR), $(PROFILE_FRAGS))

# Get imports frags - no dependencies
IMPORT_DEPS := $(addprefix $(IMPORTS_DIR), $(IMPORT_FRAGS))

# Make Intel RIM (irim) - CoMID only
IRIM_START := concise-mid-tag
IRIM_FRAGS += $(IMPORT_DEPS)
IRIM_FRAGS += $(PROFILE_DEPS)

IRIM_EXAMPLES := $(wildcard examples/$(IRIM)-*.diag)

$(eval $(call cddl_check_template,$(IRIM),$(IRIM_FRAGS),$(IRIM_EXAMPLES),$(IRIM_START)))

# Make Intel CoRIM (icorim)
ICORIM_START := corim
ICORIM_FRAGS += $(IMPORT_DEPS)
ICORIM_FRAGS += $(PROFILE_DEPS)

ICORIM_EXAMPLES := $(wildcard examples/$(ICORIM)-*.diag)

$(eval $(call cddl_check_template,$(ICORIM),$(ICORIM_FRAGS),$(ICORIM_EXAMPLES),$(ICORIM_START)))

# Make Intel Concise Evidence (ice)
ICE_START := concise-evidence
ICE_FRAGS += $(IMPORT_DEPS)
ICE_FRAGS += $(PROFILE_DEPS)

ICE_EXAMPLES := $(wildcard examples/$(ICE)-*.diag)

$(eval $(call cddl_check_template,$(ICE),$(ICE_FRAGS),$(ICE_EXAMPLES),$(ICE_START)))

# Make Intel SPDM Evidence (ispdm)
ISPDM_START := spdm-toc
ISPDM_FRAGS += $(IMPORT_DEPS)
ISPDM_FRAGS += $(PROFILE_DEPS)

ISPDM_EXAMPLES := $(wildcard examples/$(ISPDM)-*.diag)

$(eval $(call cddl_check_template,$(ISPDM),$(ISPDM_FRAGS),$(ISPDM_EXAMPLES),$(ISPDM_START)))

$(IMPORT_DEPS): check-imports

check-imports:
	$(MAKE) -C $(IMPORTS_DIR)

# Make exports
$(eval $(call cddl_exp_template,irim,$(PROFILE_DEPS),$(EXPORTS_DIR),$(IMPORT_FRAGS)))
$(eval $(call cddl_exp_template,ice,$(PROFILE_DEPS),$(EXPORTS_DIR),$(IMPORT_FRAGS)))
$(eval $(call cddl_exp_template,ispdm,$(PROFILE_DEPS),$(EXPORTS_DIR),$(IMPORT_FRAGS)))
$(eval $(call cddl_exp_template,intel-profile,$(PROFILE_DEPS),$(EXPORTS_DIR),$(IMPORT_FRAGS)))

AUTOGEN_FRAGS := $(addprefix $(CDDL_DIR), $(ICORIM)-autogen.cddl)
AUTOGEN_FRAGS += $(addprefix $(CDDL_DIR), $(IRIM)-autogen.cddl)
AUTOGEN_FRAGS += $(addprefix $(CDDL_DIR), $(ICE)-autogen.cddl)
AUTOGEN_FRAGS += $(addprefix $(CDDL_DIR), $(ISPDM)-autogen.cddl)

AUTOGEN_EXPORTS := $(addprefix $(EXPORTS_DIR), $(ICORIM)-autogen.cddl)
AUTOGEN_EXPORTS += $(addprefix $(EXPORTS_DIR), $(IRIM)-autogen.cddl)
AUTOGEN_EXPORTS += $(addprefix $(EXPORTS_DIR), $(ICE)-autogen.cddl)
AUTOGEN_EXPORTS += $(addprefix $(EXPORTS_DIR), $(ISPDM)-autogen.cddl)

# Individual export rules
$(EXPORTS_DIR)/%.cddl: $(CDDL_DIR)/%-autogen.cddl
	mkdir -p $(EXPORTS_DIR)
	cp $< $@

# Aggregate export target
export-all: check-$(ICORIM) check-$(IRIM) check-$(ISPDM) check-$(ICE) exp-$(INTEL_PROFILE)
	cp $(AUTOGEN_FRAGS) $(EXPORTS_DIR)
CLEANFILES += $(AUTOGEN_EXPORTS)

clean: ; rm -f $(CLEANFILES) ; $(MAKE) -C $(IMPORTS_DIR) clean

# Extract the CBOR tags defined by CoRIM/CoMID (i.e., those in the 5xx space)
cbor-tags.txt: $(wildcard *.cddl) ; grep -h '#6\.5' *cddl | sort -u -t'=' -k2 > $@

debug: ; echo irim ; echo $(IRIM_FRAGS)

code-pts: $(wildcard *.cddl) ; grep -v '^;' *.cddl | grep -E -h -o ': \-[0-9]{1,3}|= \-[0-9]{1,3}' | uniq | sort -u -t':' -t'='