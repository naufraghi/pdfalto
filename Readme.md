# pdfalto

[![Build Status](https://travis-ci.org/kermitt2/pdfalto.svg?branch=master)](https://travis-ci.org/kermitt2/pdfalto)

**pdfalto** is a command line executable for parsing PDF files and producing structured XML representations of the PDF content in [ALTO](https://github.com/kermitt2/pdfalto/blob/master/schema/alto.xsd) format. 

**pdfalto** is a fork of [pdf2xml](http://sourceforge.net/projects/pdf2xml), developed at XRCE, with modifications for robustness, addition of features and output enhanced format in [ALTO](https://github.com/altoxml/documentation/wiki) (including in particular space information, useful for instance for further machine learning processing). It is based on the Xpdf library.  

The latest stable version is *0.2*. 

# Requirements

* compilers : clang 3.6 or gcc 4.9
* makefile generator : cmake 3.12.0
* fetching dependencies : wget

# Usage

General usage is as follow: 

```
 pdfalto [options] <PDF-file> [<xml-file>]
  -f <int>               : first page to convert
  -l <int>               : last page to convert
  -verbose               : display pdf attributes
  -noText                : do not extract textual objects
  -noImage               : do not extract Images (Bitmap and Vectorial)
  -noImageInline         : do not include images inline in the stream
  -outline               : create an outline file xml (i.e. a table of content) as additional file
  -annotation            : create an annotations file xml as additional file
  -blocks                : add blocks informations whithin the structure
  -readingOrder          : blocks follow the reading order
  -fullFontName          : fonts names are not normalized
  -nsURI <string>        : add the specified namespace URI
  -opw <string>          : owner password (for encrypted files)
  -upw <string>          : user password (for encrypted files)
  -filesLimit <int>      : limit of asset files be extracted to the value specified
  -q                     : don't print any messages or errors
  -v                     : print version info
  -h                     : print usage information
  -help                  : print usage information
  --help                 : print usage information
  -?                     : print usage information
  --saveconf <string>    : save all command line parameters in the specified XML <file>
  -ocr                   : recognises all characters that are missing from unicode, or replace with placeholder to flag it.
```

In addition to the [ALTO](https://github.com/altoxml/documentation/wiki) file describing the PDF content, the following files are generated:

* `_metadata.xml` file containing a pdf file metadata (generate metadata information in a separate XML file as ALTO schema does not support that).

* `_annot.xml` file containing a description of the annotations in the PDF (e.g. GOTO, external http links, ...) obtained with `-annotation` option

* `_outline.xml` file containing a possible PDF-embedded table of content (aka outline) obtained with `-outline` option

* `.xml_data/` subdirectory containing the vectorial (.vec) and bitmap images (.png) embedded in the PDF, this is generated by default when the option `-noImage` is not present

# Dependencies
All dependencies are provided as static libraries corresponding to each operating system.

Dependencies can be recompiled by running this [script](https://github.com/kermitt2/pdfalto/blob/master/install_deps.sh)

See [compiling dependencies procedures](Dependencies_INSTALL.md) for further details.
##### Known issues ([issue 41](https://github.com/kermitt2/pdfalto/issues/41)) might occur whille building, in this case you'll need to compile the dependencies before building pdflato.

# Build

* NOTE for windows : it's recommended to use Cygwin and install standard libraries (either for cland or gcc)
> git clone https://github.com/kermitt2/pdfalto.git && cd pdfalto

* Xpdf 4.00 is shipped as git submodule, to download it: 

> git submodule update --init --recursive

* Build pdfalto:

> cmake ./

> make

The executable `pdfalto` is generated in the root directory. Additionally, this will create a static library for xpdf-4.00 at the following path `xpdf-4.00/build/xpdf/lib/libxpdf.a` and all the libraries and their respective subdirectory. 

# Future work

- Text like containing block element characters (https://unicode.org/charts/PDF/U2B00.pdf) are used as placeholders for unknown character unicodes, instead of what would be expected when visually inspecting the text. The reason for these unsolved character unicode values is that the actual characters are glyphs that are embedded in the PDF document which use free unicode range for embedded fonts, not the right unicode. The only way to extract the valid text for those special characters is to use OCR at glyph level . This is our targeted main future enhancement, relying on a custom Deep Learning approach.

- map special characters in secondary fonts to their expected unicode 

- try to optimize speed and memory

- see the issue tracker for further tasks

# Changes

- support unicode composition of characters

- generalize reading order to all blocks (it was limited to the blocks of the first page)

- use subscript/superscript text font style attribut

- use SVG as a format for vectorial images

- propagate unsolved character unicode value (free unicode range for embedded fonts) as encoded special character in ALTO (so-called "placeholder" approach)

- generate metadata information in a separate XML file (as ALTO schema does not support that)

- use the latest version of xpdf, version 4.00

- add cmake

- [ALTO](https://github.com/altoxml/documentation/wiki) output is replacing custom Xerox XML format

- encode URI (using `xmlURIEscape` from libxml2) for the @href attribute content to avoid blocking XML wellformedness issues. From our experiments, this problem happens in average for 2-3 scholar PDF out of one thousand.

- output coordinates attributes for the BLOCK elements when the `-block` option is selected,

- add a parameter `-readingOrder` which re-order the blocks following the reading order when the -block option is selected. By default in pdf2xml, the elements followed the PDF content stream (the so-called _raw order_). In xpdf, several text flow orders are available including the raw order and the reading order. Note that, with this modification and this new option, only the blocks are re-ordered.

  From our experiments, the raw order can diverge quite significantly from the order of elements according to the visual/reading layout in 2-4% of scholar PDF (e.g. title element is introduced at the end of the page element, while visually present at the top of the page), and minor changes can be present in up to 100% of PDF for some scientific publishers (e.g. headnote introduced at the end of the page content). This additional mode can be thus quite useful for information/structure extraction applications exploiting pdf2xml output. 

- use the latest version of xpdf, version 3.04.


# Contributors

xpdf is developed by Glyph & Cog, LLC (1996-2017) and distributed under GPL2 or GPL3 license. 

pdf2xml is orignally written by Hervé Déjean, Sophie Andrieu, Jean-Yves Vion-Dury and  Emmanuel Giguet (XRCE) under GPL2 license. 

pdfalto has been modified and forked by Patrice Lopez (patrice.lopez@science-miner.com) and Achraf Azhar (achraf.azhar@inria.fr).

The windows version has been built originally by [@pboumenot](https://github.com/boumenot) and ported on windows 7 for 64 bit, then for windows (native and cygwin) by [@lfoppiano](https://github.com/lfoppiano) and [@flydutch](https://github.com/flydutch).  

# License

As the original pdf2xml and main dependency xpdf, pdfalto is distributed under GPL2 license. 

# Useful links

Some tools for converting alto to other formats:

- https://github.com/filak/hOCR-to-ALTO
- https://github.com/UB-Mannheim/ocr-fileformat