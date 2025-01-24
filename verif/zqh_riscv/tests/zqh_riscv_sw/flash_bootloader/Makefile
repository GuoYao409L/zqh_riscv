
tgt_fn = bootloader.hex.fix

#actual mode should be flash. but simulation is very slow
#use mode=sram to put bootloader code into onchip sram to speedup simulation
mode=sram
ifeq ($(mode),flash)
    lk_fn = linker_flash.ld
    text_seg_base = 0x20000000
    text_seg_max = 0x30000000
    text_addr_base = -0x20000000
else
    lk_fn = linker_sram.ld
    text_seg_base = 0x40000000
    text_seg_max = 0x50000000
    text_addr_base = -0x40000000
endif
data_seg_base = 0x40000000
data_seg_max = 0x50000000

has_ddr=0
ifeq ($(has_ddr),1)
	def_has_ddr=-DHAS_DDR=1
else
	def_has_ddr=
endif
ddr_phy_xn=16
ifeq ($(ddr_phy_xn),8)
    def_has_ddr +=-DDDR_PHY_X8=1
endif
ifeq ($(ddr_phy_xn),16)
    def_has_ddr +=-DDDR_PHY_X16=1
endif
ddr_speed =800
def_has_ddr +=-DDDR_SPEED=$(ddr_speed)

#0x80000000 is uaually ddr space
#0x40000000 is onchip memory(sram) space
#if chip don't have ddr, this space should be set to onchip memory(sram) space
#when mode=sram need. 0x40000000-0x40000x2000 is bootloader space
#when mode=flash, main_mem_base is not used
main_mem_base = 0x40002000

has_eth=1
ifeq ($(has_eth),1)
	def_has_eth=-DHAS_ETH=1
else
	def_has_eth=
endif

#print use uart is very slow in simulation, should use mmio print monitor
print_use_uart=0
def_print_use_uart=
ifeq ($(print_use_uart),1)
    def_print_use_uart=-DPRINT_USE_UART=1
endif

#impliment mode: sim/fpga
imp_mode=sim
def_imp_mode=
ifeq ($(imp_mode),sim)
    def_imp_mode=-DIMP_MODE_SIM=1
endif

XLEN = 64

GCC=riscv$(XLEN)-unknown-elf-gcc
RISCV_GCC_OPTS = -DPREALLOCATE=1 $(def_has_ddr) $(def_has_eth) $(def_imp_mode) -DMAIN_MEM_BASE=$(main_mem_base) $(def_print_use_uart) -mcmodel=medany -static -std=gnu99 -O1 -ffast-math -fno-common -fno-builtin-printf -march=rv$(XLEN)g -g
RISCV_LINK_OPTS = -static -nostdlib -nostartfiles -lm -lgcc -T $(src_dir)/$(lk_fn)
OBJCOPY=riscv$(XLEN)-unknown-elf-objcopy --only-section=.text* --only-section=.rodata*
OBJDUMP=riscv$(XLEN)-unknown-elf-objdump -S --disassemble-all --disassemble-zeroes
HEX_FIX=../../../../scripts/zqh_sw_hex_fix.py

src_dir = .
incs  += -I$(src_dir)/../env -I$(src_dir)/../benchmarks/common -I$(src_dir)/../benchmarks/zqh_common


all: $(tgt_fn)

clean:
	rm -rf *.elf *.hex *.fix *.dump *.fix.data

%.hex.fix: %.hex
	$(HEX_FIX) --hex_in=$< --addr_base=$(text_addr_base) --addr_min=$(text_seg_base) --addr_max=$(text_seg_max) --hex_out=$<".fix"
	$(HEX_FIX) --hex_in=$< --addr_base=0x00000000 --addr_min=$(data_seg_base) --addr_max=$(data_seg_max) --hex_out=$<".fix.data"

%.hex: %.elf
	$(OBJCOPY) -O verilog $< $@

%.elf: %.S init.c $(lk_fn) $(wildcard $(src_dir)/../benchmarks/common/*) $(wildcard $(src_dir)/../benchmarks/zqh_test0/*)
	$(GCC) $(incs) $(RISCV_GCC_OPTS) init.c $< $(RISCV_LINK_OPTS) -static -Wl,--no-gc-sections -o $@
	$(OBJDUMP) -D $@ | tee $@.dump
