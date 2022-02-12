#
#
#

V = verilator
VFIX =
COSIM = n
TOP = --top-module top
RTL = ../rtl
V_INC = +incdir+$(RTL) 

V_DEFINE = -j 4 +define+debug=1 +define+SIMULATION=1   -CFLAGS "-I../sim/imgui -I../sim/vinc -I../sim/ -O3" 
#V_DEFINE += --converge-limit 2000 -Wno-WIDTH -Wno-IMPLICIT -Wno-MODDUP -Wno-UNSIGNED -Wno-CASEINCOMPLETE -Wno-CASEX -Wno-SYMRSVDWORD -Wno-COMBDLY -Wno-INITIALDLY -Wno-BLKANDNBLK -Wno-UNOPTFLAT -Wno-SELRANGE -Wno-CMPCONST -Wno-CASEOVERLAP -Wno-PINMISSING -Wno-MULTIDRIVEN
#V_DEFINE += --threads 8  # this slows it way down
V_DEFINE += -Wno-UNOPTFLAT

UNAME_S := $(shell uname -s)
UNAME_M := $(shell uname -m)

ifeq ($(UNAME_S), Darwin) #APPLE
# 	ar breaks on Apple Silicon right now
	ifeq ($(UNAME_M), arm64)
		VFIX = sed -i'' -e 's/__ALL.a/__ALL.o/g' obj_dir/Vtop.mk
	endif
	ECHO_MESSAGE = "Mac OS X"
	LIBS += -framework OpenGL -framework Cocoa -framework IOKit -framework CoreVideo `sdl2-config --libs`
	LIBS += -L/usr/local/lib -L/opt/local/lib

	CXXFLAGS += `sdl2-config --cflags` -Iimgui
	CXXFLAGS += -I/usr/local/include -I/opt/local/include 
	CFLAGS = $(CXXFLAGS) -Iimgui
endif

ifeq ($(UNAME_S), Linux) #LINUX
	ECHO_MESSAGE = "Linux"
	LIBS += -lGL -ldl `sdl2-config --libs`

	CXXFLAGS += `sdl2-config --cflags` -Iimgui
	CFLAGS = $(CXXFLAGS)
endif

ifeq ($(findstring MINGW,$(UNAME_S)),MINGW)
	ECHO_MESSAGE = "MinGW"
	LIBS += -lgdi32 -lopengl32 -limm32 `pkg-config --static --libs sdl2`

	CXXFLAGS += `pkg-config --cflags sdl2`
	CFLAGS = $(CXXFLAGS)
endif

CFLAGS += $(CC_OPT) $(CC_DEFINE) -Iimgui
LDFLAGS = $(LIBS)
EXE = ./obj_dir/Vtop
V_OPT = -O3 --x-assign fast --x-initial fast --noassert 
CC_OPT = -O

V_SRC = \
	centipede_sim.v \
	$(RTL)/centipede.v \
	$(RTL)/p6502.v \
	$(RTL)/pokey.v \
	$(RTL)/ram.v \
	$(RTL)/rom.v \
	$(RTL)/color_ram.v \
	$(RTL)/pf_rom.v \
	$(RTL)/pf_ram_dp.v \
	$(RTL)/vprom.v \
	$(RTL)/hs_ram.v

C_SRC = \
	sim_main.cpp  \
	sim/sim_bus.cpp \
	sim/sim_clock.cpp \
	sim/sim_console.cpp \
	sim/sim_video.cpp \
	sim/sim_input.cpp \
 	sim/imgui/imgui_impl_sdl.cpp \
	sim/imgui/imgui_impl_opengl2.cpp \
	sim/imgui/imgui_draw.cpp \
	sim/imgui/imgui_widgets.cpp \
	sim/imgui/imgui_tables.cpp \
	sim/imgui/imgui.cpp

VOUT = obj_dir/Vtop.cpp

all: $(EXE)

$(VOUT): $(V_SRC)  Makefile
	$V -cc $(V_OPT) -LDFLAGS "$(LDFLAGS) " -exe --trace --savable --Mdir ./obj_dir $(V_DEFINE) $(V_INC) $(TOP) -CFLAGS $(CFLAGS) $(V_SRC) $(C_SRC)
	$(VFIX)

$(EXE): $(VOUT) $(C_SRC)
#	(cd obj_dir; make OPT="-fauto-inc-dec -fdce -fdefer-pop -fdse -ftree-ccp -ftree-ch -ftree-fre -ftree-dce -ftree-dse" -f Vtop.mk)
	(cd obj_dir; make -f Vtop.mk)
	$(EXE)

clean:
	rm -f obj_dir