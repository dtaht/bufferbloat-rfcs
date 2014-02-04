DRAFT=draft-taht-home-gateway-best-practices-00

DRAFTS=$(DRAFT).txt $(DRAFT).html $(DRAFT).xml

all: $(DRAFTS)

FILES=back.mkd middle.mkd template.xml

$(DRAFT).txt: $(FILES) 
	bash pandoc2rfc -t template.xml -x ../transform.xsl -T back.mkd middle.mkd 
	   @mv -f draft.txt $(DRAFT).txt

$(DRAFT).html: $(FILES)
	bash pandoc2rfc -t template.xml -x ../transform.xsl -H back.mkd middle.mkd 
	  @mv -f draft.html $(DRAFT).html

$(DRAFT).xml: $(FILES)
	bash pandoc2rfc -t template.xml -x ../transform.xsl -X back.mkd middle.mkd 
	  @mv -f draft.xml $(DRAFT).xml

.PHONY: clean
clean:
	rm -f $(DRAFTS)
