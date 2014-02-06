bufferbloat-rfcs
================

Beginning of drafts of bufferbloat related RFCs

We are trying to generate several RFC's for fq_codel, codel and best practices for edge networks. 

These are being written via pandoc2rfc which requires some setup.

* On a linux system, you can do the following:

git clone https://github.com/dtaht/bufferbloat-rfcs.git

You need some dependencies for xml2rfc, which I get from

sudo apt-get install xml2rfc; sudo apt-get remove  xml2rfc # ancient version

You need pandoc2rfc, so get the python version of xml2rfc from:

wget https://pypi.python.org/packages/source/x/xml2rfc/xml2rfc-2.4.5.tar.gz

and pandoc, which I get for linux from

apt-get install pandoc

and the git tree for pandoc2rfc

git clone https://github.com/miekg/pandoc2rfc/

cd pandoc2rfc; sudo python setup.py install; cd ..
tar xvzf xml2rfc-2.4.5.tar.gz
cd xml2rfc-2.5.5; sudo python setup.py install; cd ..

cd bufferbloat-rfcs
cd some_directory
make
firefox *.html
-

* On a mac

these are available from macports, but it's been fiddly to get the
python version
