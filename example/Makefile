TARGET = VmuExample-DeCrayonised
IPBIN = ./IP.BIN

# directories
#------------------------------------------------------------------------------

# CD ROM contents will be placed here for packaging into a cdi
CDFS_DIR = cdfs

# each subdirectory <part> will produce <part>.img in CDFS_DIR
ROMDISKS_DIR = romdisks

# each subdirectory contains assets for conversion. outputs are placed in
# ROMDISKS_DIR/<part>, where <part> is the name of the subdirectory
ASSETS_DIR = assets

# list of directories to be automatically created
SUBDIRS = $(CDFS_DIR) $(ROMDISKS_DIR)

# files
#------------------------------------------------------------------------------

OFILES = $(patsubst %.c,%.o,$(wildcard *.c))
SCRAMBLED = $(CDFS_DIR)/1st_read.bin
ROMDISK_IMGS = $(patsubst %,$(CDFS_DIR)/%.img,$(notdir $(wildcard $(ROMDISKS_DIR)/* $(ASSETS_DIR)/*)))
DTEX_FILES = $(patsubst $(ASSETS_DIR)/%.png,$(ROMDISKS_DIR)/%.dtex,$(wildcard $(ASSETS_DIR)/*/*.png))

# list of files to be removed by make clean
INTERMEDIATES = $(OFILES) $(SCRAMBLED) $(ROMDISK_IMGS)
INTERMEDIATES += $(DTEX_FILES) $(addsuffix .pal,$(DTEX_FILES))

# flags
#------------------------------------------------------------------------------

CFLAGS = $(KOS_CFLAGS) -I../include
# CFLAGS += -I$(KOS_BASE)/addons/zlib

LIBS = -lz -lm -lcrayon_vmu $(KOS_LIBS)
KOS_LDFLAGS += -L../lib

# main targets
#------------------------------------------------------------------------------

all: cdi

cdi: rm-cdi $(TARGET).cdi

elf: rm-elf $(TARGET).elf

clean: rm-elf rm-cdi rm-bin rm-iso
	-rm -f $(INTERMEDIATES)

rm-%:
	-rm -f $(TARGET).$*

echo-%:
	@echo $($*)

# build rules
#------------------------------------------------------------------------------

#include $(KOS_BASE)/Makefile.rules

# create cdi image from CDFS_DIR
$(TARGET).cdi: $(TARGET).elf $(ROMDISK_IMGS)
	mkdcdisc -D $(CDFS_DIR) -e $(TARGET).elf -N -o $(TARGET).cdi
#$(TARGET).cdi: $(SCRAMBLED) $(ROMDISK_IMGS)
#	mkisofs -G $(IPBIN) -C 0,11702 -J -l -r -o $(TARGET).iso $(CDFS_DIR)
#	cdi4dc $(TARGET).iso $(TARGET).cdi

# create SCRAMBLED executable from elf
$(SCRAMBLED): $(CDFS_DIR) $(TARGET).elf
	sh-elf-objcopy -R .stack -O binary $(TARGET).elf $(TARGET).bin
	$(KOS_BASE)/utils/scramble/scramble $(TARGET).bin $@

# link objects into elf
$(TARGET).elf: $(OFILES)
	$(KOS_CC) $(KOS_LDFLAGS) -o $@ $(KOS_START) $^ $(LIBS)

# create ROMDISK_IMGS in CDFS_DIR from subdirectories of ASSETS_DIR
$(CDFS_DIR)/%.img: $(CDFS_DIR) $(DTEX_FILES)
	$(KOS_GENROMFS) -f $@ -d $(ASSETS_DIR)/$* -v
# $(KOS_GENROMFS) = /opt/toolchains/dc/kos/utils/genromfs/genromfs
# -x param might be able to remove the pack dir entirely...dunno yet

# create output directories if not present
$(SUBDIRS):
	mkdir -p $@
