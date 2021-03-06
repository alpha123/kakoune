debug ?= yes
ifeq ($(debug),yes)
    CPPFLAGS += -DKAK_DEBUG
    suffix := .debug
else
    ifeq ($(debug),no)
        CXXFLAGS += -O3
        suffix := .opt
    else
        $(error debug should be either yes or no)
    endif
endif

sources := $(wildcard *.cc)
objects := $(addprefix ., $(sources:.cc=$(suffix).o))
deps := $(addprefix ., $(sources:.cc=$(suffix).d))

PREFIX ?= /usr/local
DESTDIR ?= # root dir

bindir := $(DESTDIR)$(PREFIX)/bin
sharedir := $(DESTDIR)$(PREFIX)/share/kak
docdir := $(DESTDIR)$(PREFIX)/share/doc/kak

CXXFLAGS += -std=gnu++11 -g -Wall -Wno-reorder -Wno-sign-compare -pedantic

os := $(shell uname)

ifeq ($(os),Darwin)
    LIBS += -lncurses -lboost_regex-mt
else ifeq ($(os),FreeBSD)
    LIBS += -ltinfow -lncursesw -lboost_regex
    CPPFLAGS += -I/usr/local/include
    LDFLAGS += -L/usr/local/lib
else ifeq ($(os),Haiku)
    LIBS += -lncursesw -lboost_regex -lnetwork -lbe
else ifneq (,$(findstring CYGWIN,$(os)))
    LIBS += -lncursesw -lboost_regex -ldbghelp
else
    LIBS += -lncursesw -lboost_regex
    CPPFLAGS += -I/usr/include/ncursesw
    LDFLAGS += -rdynamic
endif

kak : $(objects)
	$(CXX) $(LDFLAGS) $(CXXFLAGS) $(objects) $(LIBS) -o $@

-include $(deps)

.%$(suffix).o: %.cc
	$(CXX) $(CPPFLAGS) $(CXXFLAGS) -MD -MP -MF $(addprefix ., $(<:.cc=$(suffix).d)) -c -o $@ $<

test:
	cd ../test && ./run
tags:
	ctags -R

clean:
	rm -f .*.o .*.d kak tags

XDG_CONFIG_HOME ?= $(HOME)/.config

userconfig:
	mkdir -p $(XDG_CONFIG_HOME)/kak/autoload
	ln -s $(CURDIR)/../rc/*.kak $(XDG_CONFIG_HOME)/kak/autoload/

install: kak
	mkdir -p $(bindir)
	install -m 0755 kak $(bindir)
	mkdir -p $(sharedir)/rc
	install -m 0644 ../share/kak/kakrc $(sharedir)
	install -m 0644 ../rc/* $(sharedir)/rc
	[ -e $(sharedir)/autoload ] || ln -s rc $(sharedir)/autoload
	mkdir -p $(sharedir)/colors
	install -m 0644 ../colors/* $(sharedir)/colors
	mkdir -p $(docdir)
	install -m 0644 ../README.asciidoc $(docdir)
	install -m 0644 ../doc/* $(docdir)

.PHONY: tags userconfig install
