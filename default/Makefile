###############################################################################
# Makefile for the project Toorums' Quest
###############################################################################

## General Flags
PROJECT = TOORUMQU
GAME = TOORUMQU
MCU = atmega644
TARGET = $(GAME).elf
CC = avr-gcc
INFO=../gameinfo.properties
UZEBIN_DIR = ../../../bin


## Escape spaces in mixer path (due to custom sounds.inc)
EMPTY:=
SPACE:= $(EMPTY) $(EMPTY)
SPACE_ESC:= \\$(SPACE)
MIX_PATH:= $(realpath ../data/sounds.inc)
MIX_PATH_ESC:= $(subst $(SPACE),$(SPACE_ESC),$(MIX_PATH))

## Kernel settings
KERNEL_DIR = ../../../kernel
KERNEL_OPTIONS  = -DVIDEO_MODE=3 -DSOUND_MIXER=1 -DTRUE_RANDOM_GEN=0 -DINTRO_LOGO=0 -DRAM_TILES_COUNT=36 -D_FS_USE_WRITE=1 -D_PF_USE_WRITE=1 -DMUSIC_ENGINE=STREAM
KERNEL_OPTIONS += -DSOUND_CHANNEL_5_ENABLE=1 -DMIXER_WAVES=\"$(MIX_PATH_ESC)\"
KERNEL_OPTIONS += -DSCREEN_TILES_V=26 -DVRAM_TILES_V=26 -DFIRST_RENDER_LINE=29

## Options common to compile, link and assembly rules
COMMON = -mmcu=$(MCU)

## Compile options common for all C compilation units.
CFLAGS = $(COMMON)
CFLAGS += -Wall -gdwarf-2 -std=gnu99 -DF_CPU=28636360UL -Os -mcall-prologues -fsigned-char -ffunction-sections -fno-toplevel-reorder
CFLAGS += -fdata-sections -ffreestanding -fomit-frame-pointer -fmerge-all-constants
CFLAGS += -MD -MP -MT $(*F).o -MF dep/$(@F).d 
CFLAGS += $(KERNEL_OPTIONS)


## Assembly specific flags
ASMFLAGS = $(COMMON)
ASMFLAGS += $(CFLAGS)
ASMFLAGS += -x assembler-with-cpp -Wa,-gdwarf2

## Linker flags
LDFLAGS = $(COMMON)
LDFLAGS += -Wl,-Map=$(GAME).map 
LDFLAGS += -Wl,-gc-sections 


## Intel Hex file production flags
HEX_FLASH_FLAGS = -R .eeprom

HEX_EEPROM_FLAGS = -j .eeprom
HEX_EEPROM_FLAGS += --set-section-flags=.eeprom="alloc,load"
HEX_EEPROM_FLAGS += --change-section-lma .eeprom=0 --no-change-warnings


## Objects that must be built in order to link
OBJECTS = uzeboxVideoEngineCore.o uzeboxCore.o uzeboxSoundEngine.o uzeboxSoundEngineCore.o uzeboxVideoEngine.o mmc.o pff.o $(GAME).o 

## Objects explicitly added by the user
LINKONLYOBJECTS = 

## Include Directories
INCLUDES = -I"$(KERNEL_DIR)" 

## Resources
RESOURCES = ../data/music-compressed.inc

## Build
all: $(RESOURCES) $(TARGET) $(GAME).hex $(GAME).eep $(GAME).lss $(GAME).uze size 

## Rebuild graphics resources

## Rebuild music resources
../data/music-compressed.inc: ../data/music-compressed.inc ../data/music-compress.cfg
	$(UZEBIN_DIR)/mconvert ../data/music-compress.cfg

## Compile Kernel files

mmc.o: $(KERNEL_DIR)/petitfatfs/mmc.c
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<

pff.o: $(KERNEL_DIR)/petitfatfs/pff.c
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<

uzeboxVideoEngineCore.o: $(KERNEL_DIR)/uzeboxVideoEngineCore.s
	$(CC) $(INCLUDES) $(ASMFLAGS) -c  $<

uzeboxSoundEngineCore.o: $(KERNEL_DIR)/uzeboxSoundEngineCore.s
	$(CC) $(INCLUDES) $(ASMFLAGS) -c  $<

uzeboxCore.o: $(KERNEL_DIR)/uzeboxCore.c
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<

uzeboxSoundEngine.o: $(KERNEL_DIR)/uzeboxSoundEngine.c
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<

uzeboxVideoEngine.o: $(KERNEL_DIR)/uzeboxVideoEngine.c
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<


## Compile game sources


$(GAME).o: ../$(GAME).c
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<

##Link
$(TARGET): $(OBJECTS)
	 $(CC) $(LDFLAGS) $(OBJECTS) $(LINKONLYOBJECTS) $(LIBDIRS) $(LIBS) -o $(TARGET)

%.hex: $(TARGET)
	avr-objcopy -O ihex $(HEX_FLASH_FLAGS)  $< $@

%.eep: $(TARGET)
	-avr-objcopy $(HEX_EEPROM_FLAGS) -O ihex $< $@ || exit 0

%.lss: $(TARGET)
	avr-objdump -h -S $< > $@

%.uze: $(TARGET)
	-$(UZEBIN_DIR)/packrom $(GAME).hex $@ $(INFO)

UNAME := $(shell sh -c 'uname -s 2>/dev/null || echo not')
#AVRSIZEFLAGS := -A ${TARGET}
#ifneq (,$(findstring MINGW,$(UNAME)))
#AVRSIZEFLAGS := -C --mcu=${MCU} ${TARGET}
#endif

size: ${TARGET}
	@echo
	@avr-size -C --mcu=${MCU} ${TARGET}

## Clean target
.PHONY: clean
clean:
	-rm -rf $(OBJECTS) $(GAME).* dep/* *.uze


## Other dependencies
-include $(shell mkdir dep 2>/dev/null) $(wildcard dep/*)

