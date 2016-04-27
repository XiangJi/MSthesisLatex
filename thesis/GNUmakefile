# Targets:
# - pdf: pdf generated using pdflatex
# - ps: ps generated using latex
# - dvi: dvi generated using latex
# - html: html using latex and tex4ht
# - clean: cleans up all intermediate files
# - figclean: removes any pdf figures from the figures directory that
#   were created for pdflatex
# - flat: process all includes and create a single output .tex file
# - realclean: clean, figclean, and deletion of the output directory
# - dist: creates a distribution of all necessary files

# Directories:
# - output: .pdf, .dvi, .ps, html/ end up here. Created if needed.
# - build: intermediate files end up here. Created if needed.
# - source: all your .tex and .bib files.
# - figures: all your .eps figures. .pdf figures are generated from
#   the .eps figures and stored in this directory when you build using
#   pdflatex
# - styles: .bst, .cls, .sty

# Assumes:
# - GNU make (maybe make, probably gmake on your system)
# - bin directory has latexmk version 2.0 JCC 0.98a or better
# - distill, ps2pdf, or eps2pdf is installed
# - pdflatex is installed
# - wwwis is installed
# - To make your document compatible with both latex and pdflatex, add
#   this to the preamble:
#     % Include EPS figures for latex, and PDF figures for pdflatex
#     \ifx\pdfoutput\undefined
#     \usepackage[final]{graphicx}
#     \else
#     \usepackage[final,pdftex]{graphicx}
#     \fi
#
#     % Graphics are in figures directory
#     \graphicspath{{../figures/}}
#   and do figures in the document like this:
#     \includegraphics{the_file_name_without_extension}
# - tex4ht is installed (for html output)

.SUFFIXES:  .tex .eps .pdf

# ----------------------- BEGIN USER CONFIGURATION -----------------------

DOCNAME = dissertation

# We'll use David Coppit's installation of teTeX because the department's is
# hosed.
PATH = /home/dwc3q/teTeX/bin:$(shell echo $$PATH)

LATEXMK = bin/latexmk
LATEX = latex
PDFLATEX = pdflatex
BIBTEX = bibtex
# The flags to dvips help it to generate postscript which converts to
# PDF better.
DVIPS = dvips -Ppdf -Z -tletter
PSTOPDF = distill
# Distill doesn't do some fonts right, but makes *much smaller* PDF
# files. An alternative is ps2pdf
#PSTOPDF=eps2pdf
#PSTOPDF=ps2pdf
TEX4HT = tex4ht
T4HT = t4ht
WWWIS = wwwis
FLATTEN = bin/flatten
TEMPDIR := $(shell date +tempdir-%H-%M-%S-$$$$-$$RANDOM)

# There is one more thing you might want to change below, and that is
# the $(PDFFILE) rule. The alternative below allows you to produce
# .pdf using latex & distill instead of pdflatex.

# ------------------------ END USER CONFIGURATION ------------------------

VERSION = 0.23

MAINFILE = source/dissertation.tex
MAINWWWFILE = source/dissertation-www.tex

TEXFILES = $(shell find source -name '*.tex')
BIBFILES = $(shell find source -name '*.bib')
DISTFILE = $(shell date +$(DOCNAME)-%Y-%m-%d)
EPSFIGFILES = $(shell find figures -name '*.eps')
FIGFILES = $(shell find figures -maxdepth 1 -type f 2>/dev/null)

PDFFILE = $(DOCNAME:%=output/%.pdf)
PSFILE = $(DOCNAME:%=output/%.ps)
DVIFILE = $(DOCNAME:%=output/%.dvi)
HTMLFILE = $(MAINWWWFILE:source/%.tex=output/html/%.html)
PDFFIGFILES = $(EPSFIGFILES:.eps=.pdf)
BUILDTEXFILES = $(TEXFILES:source/%=build/%)
FLATMAINFILE = $(MAINFILE:source/%.tex=output/%-flat.tex)
TMPLDISTFILE = dissertation-template-$(VERSION)

#+----------------------+
#|    general rules     |
#+----------------------+

pdf: $(PDFFILE)

ps: $(PSFILE)

dvi: $(DVIFILE)

html: $(HTMLFILE)

flat: $(FLATTEN) $(FLATMAINFILE)

dist: distdir
	@echo --- Making $(DISTFILE).tar.gz ---;
	@mv $(TEMPDIR) $(DISTFILE)
	@tar cf $(DISTFILE).tar $(DISTFILE)/*
	@rm -f $(DISTFILE).tar.gz
	@gzip -9 $(DISTFILE).tar
	@rm -rf $(DISTFILE)

# Removes intermediate files
clean:  
	@rm -f core \#* *~ *.log *.ps *.tmp *.pdf *.bak
	@rm -rf build

# Removes PDF figures generated from EPS figures
figclean:  
	@rm -f figures/*.pdf

# Removes intermediate files, PDF figures, and output files
realclean: clean figclean
	@rm -rf output

# Only for template distribution
templatedist: distdir
	@echo --- Making $(TMPLDISTFILE).tar.gz ---;
	@mv $(TEMPDIR) $(TMPLDISTFILE)
	@cp CHANGES README $(TMPLDISTFILE)

	@tar cf $(TMPLDISTFILE).tar $(TMPLDISTFILE)/*
	@rm -f $(TMPLDISTFILE).tar.gz
	@gzip -9 $(TMPLDISTFILE).tar
	@rm -rf $(TMPLDISTFILE)

#+----------------------+
#|   dependent rules    |
#+----------------------+

build/latexmkrc.latex:
	@echo --- Writing latexmkrc for latex ---;
	@if test -d build; then true; else mkdir build; fi
	@echo "\$$bibtex = '$(BIBTEX)';" > $@;
	@echo "\$$latex = '$(LATEX)';" >> $@;
	@echo "\$$dvips = '$(DVIPS)';" >> $@;

build/latexmkrc.pdflatex:
	@echo --- Writing latexmkrc for pdflatex ---;
	@if test -d build; then true; else mkdir build; fi
	@echo "\$$bibtex = '$(BIBTEX)';" > $@;
	@echo "\$$pdflatex = '$(PDFLATEX)';" >> $@;

build/latexmkrc.tex4ht:
	@echo --- Writing latexmkrc for tex4ht ---;
	@if test -d build; then true; else mkdir build; fi
	@echo "\$$bibtex = '$(BIBTEX)';" > $@;
	@echo "\$$latex  = '$(LATEX)';" >> $@;
	@echo "\$$dvips  = '$(DVIPS)';" >> $@;

$(PDFFILE): build/latexmkrc.pdflatex $(PDFFIGFILES) $(TEXFILES) $(BIBFILES)
	@echo --- Making $@ ---
	@cd build; \
	  TEXINPUTS=../source:../styles:$$TEXINPUTS; export TEXINPUTS; \
	  BIBINPUTS=../source:$$BIBINPUTS;           export BIBINPUTS; \
	  BSTINPUTS=../styles:$$BSTINPUTS;           export BSTINPUTS; \
	  ../$(LATEXMK) -r latexmkrc.pdflatex -pdf ../$(MAINFILE)
	@if test -d output; then true; else mkdir output; fi
	@cp $(MAINFILE:source/%.tex=build/%.pdf) $(PDFFILE)

# Comment out the rule above and uncomment the rule below to create PDF from
# the TEX->DVI->PS->PDF chain of commands

#$(PDFFILE): $(PSFILE:output/%=build/%)
#	@echo --- Making $@ ---
#	@cd build; \
#	  $(PSTOPDF) $(PSFILE:output/%=%)
#	@if test -d output; then true; else mkdir output; fi
#	@cp $(MAINFILE:source/%.tex=build/%.pdf) $(PDFFILE)

$(PSFILE): build/latexmkrc.latex $(EPSFIGFILES) $(TEXFILES) $(BIBFILES)
	@echo --- Making $@ ---
	@cd build; \
	  TEXINPUTS=../source:../styles:$$TEXINPUTS; export TEXINPUTS; \
	  BIBINPUTS=../source:$$BIBINPUTS;           export BIBINPUTS; \
	  BSTINPUTS=../styles:$$BSTINPUTS;           export BSTINPUTS; \
	  ../$(LATEXMK) -r latexmkrc.latex -ps ../$(MAINFILE)
	@if test -d output; then true; else mkdir output; fi
	@cp $(MAINFILE:source/%.tex=build/%.ps) $(PSFILE)

$(DVIFILE): build/latexmkrc.latex $(EPSFIGFILES) $(TEXFILES) $(BIBFILES)
	@echo --- Making $@ ---
	@cd build; \
	  TEXINPUTS=../source:../styles:$$TEXINPUTS; export TEXINPUTS; \
	  BIBINPUTS=../source:$$BIBINPUTS;           export BIBINPUTS; \
	  BSTINPUTS=../styles:$$BSTINPUTS;           export BSTINPUTS; \
	  ../$(LATEXMK) -r latexmkrc.latex ../$(MAINFILE)
	@if test -d output; then true; else mkdir output; fi
	@cp $(MAINFILE:source/%.tex=build/%.dvi) $(DVIFILE)

# This is adapted from the htlatex script that comes with tex4ht
$(HTMLFILE): build/latexmkrc.tex4ht $(EPSFIGFILES) $(TEXFILES) $(BIBFILES)
	@echo --- Making $@ ---
	@rm -rf build/html
	@mkdir build/html
	@mkdir build/html/images
	@cd build; \
	  TEXMFCNF=../misc::;                        export TEXMFCNF; \
	  TEXINPUTS=../source:../styles:$$TEXINPUTS; export TEXINPUTS; \
	  BIBINPUTS=../source:$$BIBINPUTS;           export BIBINPUTS; \
	  BSTINPUTS=../styles:$$BSTINPUTS;           export BSTINPUTS; \
	  ../$(LATEXMK) -r latexmkrc.tex4ht ../$(MAINWWWFILE); \
	  $(TEX4HT) -e../misc/tex4ht.env -f/$(MAINWWWFILE)  -i/usr/cs/contrib/share/tex4ht/tex4ht.dir/ht-fonts; \
	  $(T4HT) -e../misc/tex4ht.env -f/$(MAINWWWFILE) -dhtml/ -m644
	@$(WWWIS) build/html/*.html
	@if test -d output; then true; else mkdir output; fi
	@cp -r build/html output

$(FLATTEN):
	@echo --- Making $@ ---
	@cd build; rm -rf flatten; zcat ../misc/flatten.tar.gz | tar xf -; cd flatten; $(MAKE)
	@mv build/flatten/flatten $@

$(FLATMAINFILE): $(TEXFILES)
	@echo --- Making $@ ---
	@if test -d output; then true; else mkdir output; fi
	@cd build; \
	  FLATINPUTS=`pwd`/../source:`pwd`/../styles:$$TEXINPUTS; export FLATINPUTS; \
	  ../$(FLATTEN) ../$(MAINFILE) ../$@

distdir:
	@rm -rf $(TEMPDIR)
	@mkdir $(TEMPDIR)
	@cp GNUmakefile CHANGES README $(TEMPDIR)
	@cp -r source styles bin misc $(TEMPDIR)
	@mkdir $(TEMPDIR)/figures
	@if test '$(FIGFILES)'; then cp $(FIGFILES) $(TEMPDIR)/figures; fi

#+----------------------+
#|     suffix rules     |
#+----------------------+

.eps.pdf:
	@echo --- Making $*.pdf figure from $< ---
	@$(PSTOPDF) $<
