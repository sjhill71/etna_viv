TOP=..

include $(TOP)/Makefile.inc

COMMON_FLAGS = -I$(TOP)/lib -I$(TOP)/include_$(GCABI) -g -std=c99 -fPIC  -I../resources -I ../include
CFLAGS += $(COMMON_FLAGS) 
CXXFLAGS += $(COMMON_FLAGS) 
LDFLAGS += -lm -lpthread

TARGETS = reset viv_info
COMPANION_OBJS = ../resources/companion_array.o ../resources/companion_mesh.o ../resources/companion_texture.o
ETNA_OBJS = ../lib/viv.o ../lib/etna.o ../lib/etna_rs.o ../lib/etna_fb.o ../lib/etna_mem.o ../lib/etna_bswap.o ../lib/etna_tex.o 

all: $(TARGETS)

clean:
	rm -f *.o ../lib/*.o
	rm -f $(TARGETS)

cube: cube.o ../lib/write_bmp.o ../lib/viv.o
	$(CC) $(CFLAGS) -o $@ $^  $(LDFLAGS)

cube_companion: cube_companion.o ../lib/write_bmp.o ../lib/viv.o $(COMPANION_OBJS)
	$(CC) $(CFLAGS) -o $@ $^  $(LDFLAGS)

cube_etna: cube_etna.o ../lib/write_bmp.o ../lib/viv.o ../lib/esTransform.o
	$(CC) $(CFLAGS) -o $@ $^  $(LDFLAGS)

cube_etna2: cube_etna2.o ../lib/write_bmp.o ../lib/esTransform.o $(ETNA_OBJS)
	$(CC) $(CFLAGS) -o $@ $^  $(LDFLAGS)

ps_sandbox_etna: ps_sandbox_etna.o ../lib/write_bmp.o ../lib/viv.o ../lib/esTransform.o
	$(CC) $(CFLAGS) -o $@ $^  $(LDFLAGS)

etna_test: etna_test.o ../lib/write_bmp.o ../lib/esTransform.o $(ETNA_OBJS)
	$(CC) $(CFLAGS) -o $@ $^  $(LDFLAGS)

reset: reset.o ../lib/viv.o
	$(CC) $(CFLAGS) -o $@ $^  $(LDFLAGS)

viv_info: viv_info.o
	$(CC) $(CFLAGS) -o $@ $^  $(LDFLAGS)

