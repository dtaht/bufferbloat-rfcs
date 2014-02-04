DRAFT=draft-taht-kelly-hunt-dhpv4-to-slac-naming-00

DRAFTS=$(DRAFT).txt $(DRAFT).html $(DRAFT).xml

all: $(DRAFTS)

$(DRAFT).txt: back.mkd middle.mkd template.xml
	bash pandoc2rfc -t template.xml -x ../transform.xsl -T back.mkd middle.mkd
	   @mv -f draft.txt $(DRAFT).txt

$(DRAFT).html: back.mkd middle.mkd template.xml
	bash pandoc2rfc -t template.xml -x ../transform.xsl -M back.mkd middle.mkd
	   @mv -f draft.html $(DRAFT).html

$(DRAFT).xml: back.mkd middle.mkd template.xml
	bash pandoc2rfc -t template.xml -x ../transform.xsl -X back.mkd middle.mkd
	  @mv -f draft.xml $(DRAFT).xml

.PHONY: clean
clean:
	rm -f draft.txt draft.xml draft.html
