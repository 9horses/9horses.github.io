# Makefile to build css files via sass preprocessor

all:
	sass --update sass:css

TARGS=$(wildcard css/*.css.map)
clean:
	-rm $(TARGS) $(patsubst %.css.map,%.css,$(TARGS))
