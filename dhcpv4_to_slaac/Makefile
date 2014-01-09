all:	draft.txt draft.html draft.xml

draft.txt: back.mkd middle.mkd template.xml
	bash pandoc2rfc -t template.xml -x ../transform.xsl -T back.mkd middle.mkd

draft.html: back.mkd middle.mkd template.xml
	bash pandoc2rfc -t template.xml -x ../transform.xsl -M back.mkd middle.mkd

draft.xml: back.mkd middle.mkd template.xml
	bash pandoc2rfc -t template.xml -x ../transform.xsl -X back.mkd middle.mkd

.PHONY: clean
clean:
	rm -f draft.txt draft.xml draft.html
