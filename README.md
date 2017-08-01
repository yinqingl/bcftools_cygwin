This is a modified repository for BCFtools. It contains changes for installation of BCFtools in cygwin environment. 
Yinqing Li, yinqingl@csail.mit.edu, 2017 

Changes:
	problem with dynamic link library, cygwin needs to see all function reference during linking
		provide function reference in linking
		htslib_default_libs = -lz -lm -lbz2 -llzma
		LIBS     = $(htslib_default_libs)
		HTSLIBDLL = $(HTSDIR)/libhts.dll.a
		
		%.cygdll: %.c version.h version.c
			$(CC) $(PLUGIN_FLAGS) $(CFLAGS) $(ALL_CPPFLAGS) $(EXTRA_CPPFLAGS) $(LDFLAGS) -o $@ version.c $< libbcftools.cygdll.a $(HTSLIBDLL) $(LIBS)
		
		Makefile structure:
		
		variable: dependency
			what to do, $@ refers to the variable, $< refers to the dependency
		
		static libraries should be provided in an order, such that functions referenced in earlier ones should be contained in the later ones
		
		libhts.dll.a is a dynamic library used in htslib dynamic linking
		libbcftools.cygdll.a is a dynamic library used in bcftools linking, it contains everything other than the main.c
		
	provide libbcftools.a, libbcftools.cygdll.a
		the two contain the same set of functions, the first one is a static library and the second one is a dynamic library
		
		AR     = ar
		RANLIB = ranlib
		LIBHTS_SOVERSION= 2
		
		all: lib-static lib-shared $(PROGRAMS) $(TEST_PROGRAMS) plugins
			tell the makefile to follow this order
		
		lib-static: libbcftools.a
		lib-shared: cyghts-$(LIBHTS_SOVERSION).dll
		
		libbcftools.a: $(BCF_OBJS)
			@-rm -f $@
			$(AR) -rc $@ $(BCF_OBJS)
			-$(RANLIB) $@
		
		this generates an archive of compiled library (ar), with index (ranlib), as a static library
		
		cyghts-$(LIBHTS_SOVERSION).dll: $(BCF_OBJS) $(HTSLIBDLL)
			$(CC) -shared -Wl,--out-implib=libbcftools.cygdll.a -Wl,--export-all-symbols -Wl,--enable-auto-import $(LDFLAGS) -o $@ -Wl,--whole-archive $(BCF_OBJS) -Wl,--no-whole-archive $(HTSLIBDLL) $(ALL_LIBS) $(GSL_LIBS) -lpthread
		
		this generates a dynamic link library
		
	use the static library to link bcftools main program
		
		OBJS = main.o libbcftools.a
		bcftools: $(OBJS) $(HTSLIB_LIB)
			$(CC) $(DYNAMIC_FLAGS) -pthread $(ALL_LDFLAGS) -o $@ $(OBJS) $(HTSLIB_LIB) -lm $(ALL_LIBS) $(GSL_LIBS)
		
	plugins reference function bcftools_version from the main.c
		it is necessary to separate the function from the main.c and include it in the libbcftools.a
		see bcftools.c
	
	provide makefile rules for bcftools.c
		bcftools.o: bcftools.c version.h $(bcftools_h)
		BCF_OBJS     =  bcftools.o бн

See below for information on the original repository for BCFtools.

This is the official development repository for BCFtools. It contains all the vcf* commands
which previously lived in the htslib repository (such as vcfcheck, vcfmerge, vcfisec, etc.)
and the samtools BCF calling from bcftools subdirectory of samtools. 

For a full documentation, see [bcftools GitHub page](http://samtools.github.io/bcftools/). 

Other useful links:
------------------

File format specifications live on [HTS-spec GitHub page](http://samtools.github.io/hts-specs/)
[htslib](https://github.com/samtools/htslib)
[samtools](https://github.com/samtools/samtools)
[tabix](https://github.com/samtools/tabix)


