# Project name = executable name ?
# ============================================================
## The name of your project (without the .c)
# TARGET = blink
## Or name it automatically after the enclosing directory
TARGET = $(lastword $(subst /, ,$(CURDIR)))
# Object files: will find all .c/.h files in current directory
#  and in LIBDIR.  If you have any other (sub-)directories with code,
#  you can add them in to SOURCES below in the wildcard statement.
SOURCES=$(wildcard *.c $(LIBDIR)/*.c)
OBJECTS=$(SOURCES:.c=.o)
HEADERS=$(SOURCES:.c=.h)

# Microcontroller / Serial port
# ============================================================
ARCH        = attiny85
F_CPU       = 1000000
PORT        = /dev/ttyUSB0
PROGRAMMER  = stk500v1
BAUDRATE    = 19200

# Paths
# ============================================================
AVR_PATH       = /opt/arduino-1.8.5/hardware/tools/avr
AVRDUDE_CONFIG = $(AVR_PATH)/etc/avrdude.conf

# Programs
# ============================================================
CC         = $(AVR_PATH)/bin/avr-gcc
LD         = $(AVR_PATH)/bin/avr-gcc
OBJCOPY    = $(AVR_PATH)/bin/avr-objcopy
OBJDUMP    = $(AVR_PATH)/bin/avr-objdump
AVRDUDE    = $(AVR_PATH)/bin/avrdude
AVRSIZE    = $(AVR_PATH)/bin/avr-size
STTY       = stty

# Compile Switches
# ============================================================
CFLAGS   ?= -Os

# Flags for scanf/printf
# -------------------------
##LDFLAGS  += -Wl,-u,vfprintf -Wl,-u,vfscanf 
##LDLIBS   += -lprintf_flt -lscanf_flt -lm

# Flags for microcontroller
# ----------------------------
CPPFLAGS  += -mmcu=$(ARCH) -DF_CPU=$(F_CPU)UL -DBAUD=$(BAUDRATE)UL
LDFLAGS   += -mmcu=$(ARCH)

# Flags for C language version
# --------------------------
CFLAGS   += -std=gnu99

# Flags for warnings
# ---------------------
CFLAGS  += -Wall

# Spezial warnings
# ---------------------
CFLAGS  += -Wstrict-prototypes -Wmissing-prototypes \
           -Wmissing-declarations -Wredundant-decls \
           -Wnested-externs -Wbad-function-cast \
           -Wshadow -Wpointer-arith \
           -Wsign-compare -Wfloat-equal \
           -Wunreachable-code  \
           -Wwrite-strings -Wconversion \
           -Waggregate-return  -Winline -Wcast-align

# Flags for reducing size of executable
# --------------------------------------
# Pack structures
CFLAGS  += -fpack-struct

# Remove unused code
# ----------------------
CFLAGS  += -ffunction-sections -fdata-sections 
LDFLAGS += -Wl,--gc-sections

# Explicit rules
# ============================================================
.PHONY: all flash clean debug test disasm disassemble avrdude_terminal size

all: $(TARGET).hex
	@echo "done"

$(TARGET).elf: $(OBJECTS)
	$(LD) $(LDFLAGS) -o $@ $^ $(LDLIBS)

flash:
	$(STTY) -F $(PORT) hupcl
	$(AVRDUDE) -vvv -C $(AVRDUDE_CONFIG) -p $(ARCH) -c $(PROGRAMMER) -P $(PORT) -b $(BAUDRATE) -U flash:w:$(TARGET).hex

clean:
	rm -f $(OBJECTS) $(TARGET).elf $(TARGET).hex $(TARGET).lst

debug:
	@echo
	@echo "Source files:"   $(SOURCES)
	@echo "MCU:   " $(ARCH)
	@echo "F_CPU: " $(F_CPU)  
	@echo "BAUD:  " $(BAUDRATE)
	@echo "PROGR: " $(PROGRAMMER)
	@echo "PORT:  " $(PORT)
	
test:
	$(AVRDUDE) -v -C $(AVRDUDE_CONFIG) -p $(ARCH) -c $(PROGRAMMER) -b $(BAUDRATE) -P $(PORT)
	
disassemble: $(TARGET).lst

disasm: disassemble

avrdude_terminal:
	$(AVRDUDE) -v -C $(AVRDUDE_CONFIG) -p $(ARCH) -c $(PROGRAMMER) -b $(BAUDRATE) -P $(PORT) -nt
	
size:  $(TARGET).elf
	$(AVRSIZE) -C --mcu=$(ARCH) $(TARGET).elf

# Template rules
# ============================================================
%.hex: %.elf
	$(OBJCOPY) -O ihex -R .eeprom $< $@

%.lst: %.elf
	$(OBJDUMP) -h -S -t -C $< > $@
# disasambler output
%.lst: %.o
	$(OBJDUMP) -h -S -t -C $< > $@
# preaprocessor output: > make target.E
%.E:%.c
	$(CC) $(CPPFLAGS) -E -o $@ $<
# assembler output: > make target.s
%.s:%.c
	$(CC) $(CPPFLAGS) -S -o $@ $<
