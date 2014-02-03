CC=gcc
PACKAGE=mqtt-sn-tools
VERSION=0.0.3
CFLAGS=-g -Wall -DVERSION=$(VERSION)
LDFLAGS=
TARGETS=mqttSN_pub


all: $(TARGETS)

mqttSN_pub: mqtt-sn.o mqttSN_pub.o
	$(CC) $(LDFLAGS) -o mqttSN_pub $^
  
mqttSN_pub.o: mqttSN_pub.c mqtt-sn.h
	$(CC) $(CFLAGS) -c mqttSN_pub.c

mqtt-sn.o: mqtt-sn.c mqtt-sn.h
	$(CC) $(CFLAGS) -c mqtt-sn.c

clean:
	rm -f *.o $(TARGETS)

dist:
	distdir='$(PACKAGE)-$(VERSION)'; mkdir $$distdir || exit 1; \
	list=`git ls-files`; for file in $$list; do \
		cp -pR $$file $$distdir || exit 1; \
	done; \
	tar -zcf $$distdir.tar.gz $$distdir; \
	rm -fr $$distdir

.PHONY: all clean dist
