CXXFLAGS += -g
CXX ?= g++

PROGS = worker

all: $(PROGS)

clean: distclean

distclean:
	rm -f $(PROGS) *.o

worker2: worker.o
	$(CXX) $(CXXFLAGS) $(CPPFLAGS) $(LDFLAGS) -o worker worker.o

install: all

.PHONY: install all distclean clean
