SRC=$(wildcard *.dot)
OBJECTS=$(patsubst %.dot,%.svg,$(SRC))
SRC_NEATO=$(wildcard neato/*.dot)
OBJECTS_NEATO=$(patsubst %.dot,%.svg,$(SRC_NEATO))
OPTIONS=-Nfontname=sans-serif -Nfontsize=10 -Efontname=sans-serif -Efontsize=10

all: $(OBJECTS) $(OBJECTS_NEATO)

%.svg: %.dot
	dot $(OPTIONS) -Tsvg $< > $@

neato/%.svg: neato/%.dot
	dot $(OPTIONS) -Kneato -Tsvg $< > $@

clean:
	rm -f $(OBJECTS) $(OBJECTS_NEATO)

.PHONY: all clean

