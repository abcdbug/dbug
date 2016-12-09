.SECONDEXPANSION:

.DELETE_ON_ERROR:

LN = ln

BWA = bwa
SAMTOOLS = samtools

REFERENCE = /usr/local/ref/e_coli.fa

FASTQFILES := $(wildcard *_R1_*.fastq.gz)

# Here we are extracting the sample name from each fastq file
# by exploting FASTQ file naming conventions
SAMPLES := $(sort $(foreach a,$(FASTQFILES),$(firstword $(subst _, ,$a))))
#SAMPLES := $(shell arr=$$(ls *R1*.fastq.gz) ; for l in $${arr} ; do echo $$l | cut -d"_" -f1 ; done | sort | uniq | tr '\n' ' ')

default: all

test:
	echo $(SAMPLES)
	
# Adding merged bam to list of targets
all: $(FASTQFILES:.fastq.gz=.bam) $(addsuffix .merged.bam, $(SAMPLES))

%.sai: %.fastq.gz
	$(BWA) aln $(REFERENCE) $< > $@

%.sam: %.sai $$(subst R1,R2,%.sai) %.fastq.gz $$(subst R1,R2,%.fastq.gz)
	$(BWA) sampe $(REFERENCE) $^ > $@

%.sam: %.sai %.fastq.gz
	$(BWA) samse $(REFERENCE) $^ > $@

%.bam: %.sam
	$(SAMTOOLS) view -bS $< > $@

# Before we can merge bam files, we have to sort them
%.sorted.bam: %.bam
	$(SAMTOOLS) sort -O bam -T $<.tmp $< > $@

# Filters list of all bam files to return list of files that contain the variable '$1_%'
find-bam-files = $(sort $(filter $1_% , $(FASTQFILES:.fastq.gz=.sorted.bam)))

# Make allows us to define variables using the define directive
# In the below case, we are defining an abstract rule to create merged bam files
define merge-bam-files
$1.merged.bam: $(call find-bam-files,$1)
	$$(if $$(findstring 1, $$(words $$^)),$$(LN) -fs $$^ $$@,$(SAMTOOLS) merge -f $$@ $$^)
endef

# Two things to note here.
# The call function is unique in that it expands variables with different values
# The syntax of the call function is $(call variable,param,param, ...)
# When make expands the call function, it assigns each param to temporary variables $1, $2, etc. 
# Secondly, the eval function is very special in that it allows you to define new makefile constructs
# The argument to the eval function is expanded, then the results of that expansion are parsed as makefile syntax
$(foreach s,$(SAMPLES),$(eval $(call merge-bam-files,$s)))

