include ../../make.project

MAINDIR = $(shell pwd)/../..
MISTRALDIR ?= .

OSTYPE          :=      $(shell uname -s | \
                        tr '[:upper:]' '[:lower:]' | \
                        sed \
			-e s/cygwin.*/cygwin/ \
                        -e s/irix../irix/ )

PYVERSION	:=	$(shell python -V 2>&1 | \
                        tr '[:upper:]' '[:lower:]' | \
                        sed \
			-e s/" 2.4."*/2.4/ \
			-e s/" 2.5."*/2.5/ \
			-e s/" 2.6."*/2.6/ )


ifeq ($(OSTYPE),darwin)
BUNDLE = -bundle -undefined suppress 
HACK = -I/usr/lib/gcc/i686-apple-darwin9/4.2.1/include
else
BUNDLE = -shared 
endif


ifeq ($(PYVERSION),python2.6)
PYLIB = `python2.6-config --cflags` $(HACK) 
PYOBJ = `python2.6-config --prefix`
SWIG = swig 
else 
ifeq ($(PYVERSION),python2.5)
PYLIB = `python2.5-config --cflags` 
PYOBJ = `python2.5-config --prefix`
SWIG = swig
else 
ifeq ($(PYVERSION),python2.4)
PYLIB = `/usr/bin/python2.4-config --cflags` $(HACK)  
PYOBJ = `/usr/bin/python2.4-config --prefix`
SWIG = swig -classic
else 
PYLIB = `python-config --cflags` $(HACK) 
PYOBJ = `python-config --prefix`
SWIG = swig
endif
endif
endif


BOOSTDIR=/Users/ehebrard/work/dev/boost_1_44_0



#OPTFLAGS = -O3 #-m32
#OPTFLAGS = -pg 
#OPTFLAGS = -finstrument-functions 
OPTFLAGS = -O3

COMPILFLAGS = -Wall \
-D_UNIX \
-fPIC \
#-D_DEBUGSEARCH \
#-D_DEBUGPROPAG \
#-D_VIRTUAL_TRACE \
#-D_DEBUGNOGOOD 
#-D_DEBUGPROPAGSTACK 
#-D_DEBUGSCHED -D_DEBUGMODEL #

#-Wl,--hash-style=sysv
#-D_DEBUGSEARCH \
#-D_DEBUGPROPAG \
#-D_SAFE \
#-D_DEBUGNOGOOD \
#-D_DEBUGMODEL \
#-D_WEIGHT_STATS 
#-D_DEBUGPROPAGSTACK \
#-D_DEBUGAC \
#-D_DEBUGWRAP 
#-D_WEIGHT_STATS 
#-D_PLOT_GINI \
#-D_DEBUGDECOMP \
#-D_DEBUGPROPAG \
#-D_CPLEX -fPIC -m32 -fexceptions -DNDEBUG -DIL_STD \
#-D_NARRAY \
#-D_XERCES \
#-D_MORESTATS
#-D_COINLP \
#-D_LINUX \
#-D_CONSISTENCYCHECK \

CCC = g++ $(OPTFLAGS) $(COMPILFLAGS)

BIN=$(MISTRALDIR)/bin
SRC=$(MISTRALDIR)/lib/src
MOD=$(MISTRALDIR)/models
OBJ=$(MISTRALDIR)/lib/obj
INC=$(MISTRALDIR)/include

OBJDIR = $(MISTRALDIR)/obj
SRCDIR = $(MISTRALDIR)/src
PYTHON = $(MISTRALDIR)/python


CFLAGS += -I$(INC) -I$(SRCDIR)\
`xml2-config --cflags`

LFLAGS = -L$(OBJ) \
`xml2-config --libs`


#-----------------------------------------------------------------------------
# Main Program
#-----------------------------------------------------------------------------

MAINNAME	=	Mistral
MAINOBJ		=	$(MAINNAME).o
MAINSRC		=	$(addprefix $(SRCDIR)/,$(MAINOBJ:.o=.cpp))
MAINOBJFILES	=	$(addprefix $(OBJDIR)/,$(MAINOBJ))

#-----------------------------------------------------------------------------
# Interface
#-----------------------------------------------------------------------------

IFILE		=	$(PYTHON)/$(MAINNAME).i
PYFILE		=	$(PYTHON)/$(MAINNAME).py
WXXFILE		= 	$(PYTHON)/$(MAINNAME)_wrap.cxx
WOBFILE		= 	$(PYTHON)/$(MAINNAME)_wrap.o
SOFILE		= 	$(PYTHON)/_$(MAINNAME).so

#------------------------------------------------------------
#  make all      : to compile the examples.
#------------------------------------------------------------

PMODELS = $(wildcard $(MOD)/src/*.cpp)
MODAUX = $(PMODELS:.cpp=)
MODELS = $(patsubst $(MOD)/src/%, %, $(MODAUX))
BINS = $(patsubst %, $(BIN)/%, $(MODELS))
EXE = $(BINS:=;)

PINCSRC = $(wildcard $(INC)/*.h)
PLIBSRC = $(wildcard $(SRC)/*.cpp)
PLIBAUX = $(PLIBSRC:.cpp=.o)
PLIBOBJ = $(patsubst $(SRC)/%, $(OBJ)/%, $(PLIBAUX))


wrapper: $(OBJDIR) $(PYTHON) $(OBJ) $(IFILE) $(WXXFILE) $(SOFILE) 

$(OBJDIR):
	@mkdir -p $(OBJDIR)

$(PYTHON):
	@mkdir -p $(PYTHON)

$(OBJ):
	@mkdir -p $(OBJ)

# ------------------------------------------------------------
clean_swig: 
	rm -rf $(OBJDIR)/* $(PYTHON)/* *~ $(SRCDIR)/*~

clean : 
	rm -rf $(OBJ)/*.o $(OBJ)/*.a $(SRC)/*~ $(MOD)/obj/*.o $(MOD)/src/*~ $(MOD)/src/*/*~ $(INC)/*~ *~ $(BIN)/* 

install_python: wrapper
	cd $(PYTHON); python $(MAINDIR)/tools/setup.py install	

$(OBJDIR)/%.o:	$(SRCDIR)/%.cpp
		@echo "-> compiling $@"
		$(CCC) $(CFLAGS) -c -o $@ $< 

$(PYTHON)/%_wrap.o: $(PYTHON)/%_wrap.cxx
		@echo "-> compiling $@"
		$(CCC) $(CFLAGS) -I$(SRCDIR) $(PYLIB)  -c $< -o $@

$(PYTHON)/%_wrap.cxx: $(PYTHON)/%.i
	@echo "-> compiling $@"	
	$(SWIG) -c++ -python $<
	python $(MAINDIR)/tools/finalize.py $(<:.i=.py)

$(PYTHON)/%.i: $(SRCDIR)/%.hpp
	python $(MAINDIR)/tools/mk_interface.py $<

$(PYTHON)/_%.so: $(OBJDIR)/%.o $(PYTHON)/%_wrap.o $(PLIBOBJ)
	@echo "-> linking $@"
	$(CCC) $(BUNDLE) -Wno-long-double $(CFLAGS) $(AUXOBJFILES) $(MAINOBJFILES) $(PLIBOBJ) python/Mistral_wrap.o -o $@ $(LFLAGS)


# The library
lib: $(PLIBOBJ)

$(BIN): 
	mkdir $(BIN)

$(MOD)/obj: 
	mkdir $(MOD)/obj

$(OBJ)/%.o: $(SRC)/%.cpp $(INC)/%.h 
	$(CCC) $(CFLAGS) -I$(BOOSTDIR) -c   $< -o $@ 

# The examples
$(BIN)/%: $(MOD)/obj/%.o $(PLIBOBJ) $(BIN) 
	$(CCC) $(CFLAGS) $< $(PLIBOBJ) -lm -o $@ $(LFLAGS)

%: $(MOD)/obj/%.o $(PLIBOBJ) $(BIN)
	$(CCC) $(CFLAGS) $< $(PLIBOBJ) -lm -o $(BIN)/$@ $(LFLAGS)

$(MOD)/obj/%.o: $(MOD)/src/%.cpp $(MOD)/obj
	$(CCC) -c $(CFLAGS) -I$(BOOSTDIR) -o $@ $<

