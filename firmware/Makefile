# check if OS is Windows for Windows users --> ca65 oddities with current windows version/naming
# ENSURE that cc65 is in the C drive and is simply c/cc65 !!!!

# searches for NT in $(shell uname) --> works with MINGW...unsure about others
UNAME_OUT := $(shell uname -a 2>/dev/null)
UNAME_SEARCH := NT

ifeq ($(OS), Windows_NT)
	OSFLAGS += WIN
else ifeq ($(findstring $(UNAME_SEARCH), $(UNAME_OUT)), $(UNAME_SEARCH))
	OSFLAGS += WIN
else
	OSFLAGS += OTHER
endif

$(info OS: $(OSFLAGS))
$(info OS: $(shell uname))

# Instruct that compilation and assembly should be done using cc65/ca65 and
# specify some required flags (https://cc65.github.io/doc/customizing.html#toc9)
AS = ca65
CC = cc65
CFLAGS = --cpu 65C02 -t none -O
ASFLAGS =

ifeq ($(OSFLAGS), WIN)
	ASFLAGS = --cpu W65C02
else
	ASFLAGS = --cpu 65C02
endif

# Where the intermediate-stage Assembly/object files will be saved
BUILD_DIR = build
DUMP_DIR = dump

# Add prefix to generated .o file
BIN_PREFIX =

# Define a list of all C and Assemnbly source files, and the assembled object file locations
CXX_SRC = vram.c Q9_6.c
CXX_BIN = $(addprefix ${BUILD_DIR}/o/${BIN_PREFIX},$(CXX_SRC:.c=.o))

ASM_SRC = crt0.s vectors.s io.s fw_headers.s stop.s mapache64_zero_page.s reset.s verify_firmware.s
ASM_BIN = $(addprefix ${BUILD_DIR}/o/${BIN_PREFIX},$(ASM_SRC:.s=.o))

# List of files in the for_template directory
# add proper powershell commands
ifeq ($(OSFLAGS), WIN)
	FOR_TEMPLATE_FILES_IN = $(shell find for_template -type f)
	FOR_TEMPLATE_FILES_OUT = $(subst for_template/, template/, $(FOR_TEMPLATE_FILES_IN))
else
	FOR_TEMPLATE_FILES_IN = $(shell find for_template -type f)
	FOR_TEMPLATE_FILES_OUT = $(subst for_template/, template/, $(FOR_TEMPLATE_FILES_IN))
endif

$(info for_template done)

# Path to cc65 installation directory (change as needed)
# WIN version assumes cc65 is in their C drive as its OWN independent folder (change)
CC65_INSTALL =
ifeq ($(OSFLAGS), WIN)
	CC65_INSTALL += /c/cc65
else
	CC65_INSTALL += /usr/share/cc65
endif

# Path to ld65 template library
SUPERVISION_LIB = ${CC65_INSTALL}/lib/supervision.lib

# Previx name for all flashable binary files
MACHINE = mapache64

.PHONY: all template clean

# By default (with no build target argument, i.e. `make`), build the template
# and set it up in a new directory
all: clean template

template: template/backend/firmware.bin

template/%: for_template/%
	@mkdir -p $(dir $@)
	cp -r $< $@

template/backend/headers/: $(shell find headers -type f)
	@mkdir -p template
	cp -r headers/ $@

template/backend/lib/: ${CXX_BIN} ${ASM_BIN}
	@mkdir -p template
	cp -r ${BUILD_DIR}/o/ $@

template/backend/${MACHINE}.lib: ${BUILD_DIR}/${MACHINE}.lib
	@mkdir -p template
	cp -r ${BUILD_DIR}/${MACHINE}.lib $@

template/backend/${MACHINE}.cfg: ${MACHINE}.cfg
	@mkdir -p template
	cp -r ${MACHINE}.cfg $@

template/backend/firmware.bin: ${FOR_TEMPLATE_FILES_OUT} template/backend/headers/ template/backend/lib/ template/backend/${MACHINE}.lib template/backend/${MACHINE}.cfg
# TO DO: Fix template/Makefile clock skew
	@sleep 1
	make -C template/ dump &&\
	cp template/${DUMP_DIR}/firmware.bin $@ &&\
	make -C template/ clean

# Create the ${MACHINE} machine libraries
${BUILD_DIR}/${MACHINE}.lib:
	@mkdir -p ${BUILD_DIR}
	cp ${SUPERVISION_LIB} $@

# Compile .c source files
${BUILD_DIR}/o/${BIN_PREFIX}%.o: backend_src/%.c $(shell find headers -type f)
	@mkdir -p ${BUILD_DIR}/s
	$(CC) -Oi -I headers $(CFLAGS) $< -o ${BUILD_DIR}/s/$(notdir $(<:.c=.s))
	@mkdir -p ${BUILD_DIR}/o
	$(AS) $(ASFLAGS) ${BUILD_DIR}/s/$(notdir $(<:.c=.s)) -o $@

# Compile .s source files
${BUILD_DIR}/o/${BIN_PREFIX}%.o: backend_src/%.s
	@mkdir -p ${BUILD_DIR}/o
	$(AS) $(ASFLAGS) $^ -o $@

# Purge all built files
clean:
	rm -rf ${BUILD_DIR} template
