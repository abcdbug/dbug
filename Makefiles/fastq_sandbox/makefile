.SECONDARY:
.DELETE_ON_ERROR:
.SECONDEXPANSION:

BWA = bwa
SAMTOOLS = samtools
REF = /usr/local/ref/e_coli.fa
BWA_ALN = /Users/pz/Dropbox/make/fastq_sandbox/bwa_aln.sh

FASTQFILES = $(wildcard *_R1_*.fastq.gz)

default: $(FASTQFILES:.fastq.gz=.bam)

%.sai: %.fastq.gz $(BWA_ALN)
	$(BWA_ALN) $(REF) $< $@

%.sam: %.sai $$(subst R1,R2, %.sai) %.fastq.gz $$(subst R1,R2, %.fastq.gz)
	 $(BWA) sampe $(REF) $^ > $@

%.sam: %.sai %.fastq.gz
	$(BWA) samse $(REF) $^ > $@

%.bam: %.sam
	$(SAMTOOLS) view -bS $< > $@

