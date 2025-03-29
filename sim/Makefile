default: help

CC := /usr/bin/gcc
RM := /bin/rm
PWD := $(shell pwd)
BENCHMARKER := perl benchmarker

COMMON_FLAGS = -f ../input.vc

GCC_PLIFLAGS = -fPIC -shared -o benchmark_pli.so \
			   ../bench/veriuser.c \
			   ../bench/benchmark_pli.c

VCOMP_FLAGS = $(COMMON_FLAGS) \
	-o simvc \
	$(wildcard ../bench/*.v) \
	../rtl/k68_adder.v \
	../rtl/k68_arb.v \
	../rtl/k68_asx.v \
	../rtl/k68_b2d.v \
	../rtl/k68_buni.v \
	../rtl/k68_calc.v \
	../rtl/k68_ccc.v \
	../rtl/k68_clkgen.v \
	../rtl/k68_cpu.v \
	../rtl/k68_d2b.v \
	../rtl/k68_decode.v \
	../rtl/k68_defines.v \
	../rtl/k68_dpmem.v \
	../rtl/k68_execute.v \
	../rtl/k68_fetch.v \
	../rtl/k68_load.v \
	../rtl/k68_lsx.v \
	../rtl/k68_par_mul.v \
	../rtl/k68_regbank.v \
	../rtl/k68_rox.v \
	../rtl/k68_roxx.v \
	../rtl/k68_sasc.v \
	../rtl/k68_soc.v \
	../rtl/sasc_brg.v \
	../rtl/sasc_fifo4.v \
	../rtl/sasc_top.v \

VBS_FLAGS = -o simi -y ../bench -y ../rtl \
	-I../bench -I../rtl \
	../bench/bench.v

M32 = -m64
CPPFLAGS += $(M32)
LDFLAGS += $(M32)
# +~5% with -O3
OPT_FAST = $(M32) -O3

#OPT_FAST += -ggdb -fverbose-asm -S
#OPT_FAST += -pg
OPT_SLOW = $(OPT_FAST)
#LDFLAGS += -pg
export OPT_FAST
export OPT_SLOW
export LDFLAGS

######################################################################
machine:
	@uname -a
	@cat /etc/*release
	@gcc --version | head
	@grep -i 'model name' /proc/cpuinfo
	@lshw

######################################################################

VERILATOR_EXE = verilator
VERILATOR_VERSION = $(shell $(VERILATOR_EXE) --version)
VERILATOR_FLAGS  = $(COMMON_FLAGS) \
	--cc --timing -Wno-LATCH ../bench/k68_soc_test.v --exe $(PWD)/../bench/bench.cpp  -prefix m68 -Wno-WIDTH \
	-O3 -x-assign 0

ALL_SIMS += vlt-32
vlt-32:
	$(VERILATOR_EXE) $(VERILATOR_FLAGS)
	( cd obj_dir ; $(RM) *.o ; make -j -f m68.mk LDFLAGS=-m32 OPT_FAST="-m32 -O3" OPT_SLOW="-m32 -O3" )
	$(BENCHMARKER) "$@ $(VERILATOR_VERSION)" "obj_dir/m68 %c"

ALL_SIMS += vlt-64
vlt-64:
	$(VERILATOR_EXE) $(VERILATOR_FLAGS)
	( cd obj_dir ; $(RM) *.o ; make -j -f m68.mk LDFLAGS=-m64 OPT_FAST="-m64 -O3" OPT_SLOW="-m64 -O3" )
	$(BENCHMARKER) "$@ $(VERILATOR_VERSION)" "obj_dir/m68 %c"

ALL_SIMS += vlt-32-mt
vlt-32-mt:
	$(VERILATOR_EXE) --threads `nproc` $(VERILATOR_FLAGS)
	( cd obj_dir ; $(RM) *.o ; make -j -f m68.mk LDFLAGS=-m32 OPT_FAST="-m32 -O3" OPT_SLOW="-m32 -O3" )
	$(BENCHMARKER) "$@ $(VERILATOR_VERSION)" "obj_dir/m68 %c"

ALL_SIMS += vlt-64-mt
vlt-64-mt:
	$(VERILATOR_EXE) --threads `nproc` $(VERILATOR_FLAGS)
	( cd obj_dir ; $(RM) *.o ; make -j -f m68.mk LDFLAGS=-m64 OPT_FAST="-m64 -O3" OPT_SLOW="-m64 -O3" )
	$(BENCHMARKER) "$@ $(VERILATOR_VERSION)" "obj_dir/m68 %c"

m68.gprof:  vlt
	gprof obj_dir/m68 > $@

vlt-waves:
	$(VERILATOR_EXE) --trace $(VLT_FLAGS)
	( cd obj_dir ; make -f m68.mk )
	time obj_dir/m68 -l sim.log

######################################################################

VCS_EXE ?= $(value VERILATOR_VCS)
VCS_EXE ?= $(value VCS)
VCS_EXE = vcs
VCS_VERSION = $(shell $(VCS_EXE) -ID 2>&1 | grep 'version =')
VCS_FLAGS = $(COMMON_FLAGS) \
	+define+vcs+1 ../bench/bench.v
VCS_GCC = $(CC)
VCS_GCC_PLIFLAGS = -DVCS -P ../bench/vcs_pli.tab \
				   ../bench/benchmark_pli.c
				   
ALL_SIMS += vcs-32-2state
vcs-32-2state:
	-$(RM) -rf csrc simv.daidir
	$(VCS_EXE) $(VCS_FLAGS)
	$(BENCHMARKER) "$@ $(VCS_VERSION)" "./simv +CYCLES=%c +2state"

ALL_SIMS += vcs-64-2state
vcs-64-2state:
	-$(RM) -rf csrc simv.daidir
	$(VCS_EXE) $(VCS_FLAGS) -full64
	$(BENCHMARKER) "$@ $(VCS_VERSION)" "./simv +CYCLES=%c +2state"

ALL_SIMS += vcs-32-4state
vcs-32-4state:
	-$(RM) -rf csrc simv.daidir
	$(VCS_EXE) $(VCS_FLAGS)
	$(BENCHMARKER) "$@ $(VCS_VERSION)" "./simv +CYCLES=%c"

ALL_SIMS += vcs-64-4state
vcs-64-4state:
	-$(RM) -rf csrc simv.daidir
	$(VCS_EXE) $(VCS_FLAGS) -full64
	$(BENCHMARKER) "$@ $(VCS_VERSION)" "./simv +CYCLES=%c"

vcs-64-4state_pli:
	-$(RM) -rf csrc simv.daidir
	$(VCS_EXE) $(VCS_FLAGS) -full64 +define+BENCHMARK_PLI -cc $(VCS_GCC) $(VCS_GCC_PLIFLAGS) 
	$(BENCHMARKER) "$@ $(VCS_VERSION)" "./simv +CYCLES=%c"

vcs-waves:
	$(VCS_EXE) +cli -I $(VCS_FLAGS)
	time ./simv -l sim.log

######################################################################

ICARUS_EXE ?= $(value VERILATOR_IVERILOG)
ICARUS_EXE ?= $(value IVERILOG)
ICARUS_EXE = iverilog
ICARUS_VERSION = $(shell $(ICARUS_EXE) -V 2>&1 | head -1)
ICARUS_FLAGS = $(COMMON_FLAGS) \
	-o simi \
	../bench/bench.v
ICARUS_GCC = $(CC)
ICARUS_GCC_PLIFLAGS = $(GCC_PLIFLAGS) \
					  -DNC -I../bench/include/nc

ICARUS_PLI_FLAGS = -cadpli=benchmark_pli.so:bootstrap

ALL_SIMS += icarus
icarus:
	time $(ICARUS_EXE) $(ICARUS_FLAGS)
	$(BENCHMARKER) "icarus $(ICARUS_VERSION)" "./simi +CYCLES=%c"

######################################################################

EPICSIM_EXE ?= $(value VERILATOR_EPICSIM)
EPICSIM_EXE ?= $(value EPICSIM)
EPICSIM_EXE = epicsim-driver
EPICSIM_VERSION = $(shell $(EPICSIM_EXE) -V 2>&1 | head -1)
EPICSIM_FLAGS = $(COMMON_FLAGS) \
	-o simi \
	../bench/bench.v
EPICSIM_GCC = $(CC)
EPICSIM_GCC_PLIFLAGS = $(GCC_PLIFLAGS) \
					  -DNC -I../bench/include/nc

EPICSIM_PLI_FLAGS = -cadpli=benchmark_pli.so:bootstrap

ALL_SIMS += icarus
epicsim:
	time $(EPICSIM_EXE) $(EPICSIM_FLAGS)
	$(BENCHMARKER) "epicsim $(EPICSIM_VERSION)" "./simi +CYCLES=%c"

######################################################################

ALL_SIMS += vcomp
vcomp:
	time ~/src/verilator/vcomp-my/src/vcomp  $(VCOMP_FLAGS)
	./simvc

######################################################################

help:
	@echo "Please make one of the below targets (not in parallel):"
	@echo "   machine     Show machine information"
	@echo "   clean       Remove temp files"
	@echo $(ALL_SIMS)
	false

maintainer-copy::
clean mostlyclean distclean maintainer-clean::
	-$(RM) -rf obj_dir *.log *.dmp *.vpd simv* vcs.key csrc INCA_libs simi .vcsmx_rebuild ucli.key


