
OUT =		libxslt
TEXT =		text
ZIP =		gzip
NGINX_ORG =	/data/www/nginx.org
SHELL =		./umasked.sh

CP =		/data/sites/java/xsls/\*:$(HOME)/java/xsls/\*
RSYNC =		rsync -v -rpc --exclude=.svn
CHMOD =		/bin/chmod -R g=u


define	XSLScript
	java -cp $(CP)							\
		com.pault.StyleSheet					\
		-x com.pault.XX -y com.pault.XX				\
		$(1) xsls/dump.xsls					\
	| sed 's/ *$$//;/^ *$$/N;/\n *$$/D' > $(2)

	if [ ! -s $(2) ]; then rm $(2); fi; test -s $(2)
endef

define	XSLT
	xmllint --noout --valid $2
	xsltproc -o $3							\
		$(shell f=`echo $2 | sed 's,^xml/,,;s,[^/]*/,en/,'`;	\
		[ -f xml/$$f ] && echo --stringparam ORIGIN "$$f")	\
		$(shell p="$4"; [ -n "$$p" ] &&				\
		echo --stringparam $${p%%=*} $${p#*=})			\
		$1 $2
endef

define 	JPEGNORM
	jpegtopnm $1							\
		| pamscale -width=150					\
		| pnmtojpeg -quality=95 -optimize -dct=float		\
		> $2
endef


COMMON_DEPS =								\
		xml/menu.xml						\
		xml/versions.xml					\
		xml/i18n.xml						\
		dtd/content.dtd						\
		xslt/dirname.xslt					\
		xslt/link.xslt						\
		xslt/style.xslt						\
		xslt/body.xslt						\
		xslt/menu.xslt						\
		xslt/ga.xslt						\
		xslt/content.xslt					\

ARTICLE_DEPS =								\
		$(COMMON_DEPS)						\
		dtd/article.dtd						\
		dtd/module.dtd						\
		xslt/article.xslt					\
		xslt/donate.xslt					\
		xslt/directive.xslt					\
		xslt/versions.xslt					\

NEWS_DEPS =								\
		$(COMMON_DEPS)						\
		dtd/news.dtd						\
		xslt/news.xslt						\

DOWNLOAD_DEPS =								\
		$(COMMON_DEPS)						\
		dtd/article.dtd						\
		xslt/download.xslt					\

SECURITY_DEPS =								\
		$(COMMON_DEPS)						\
		dtd/article.dtd						\
		xslt/security.xslt					\

BOOK_DEPS =								\
		$(COMMON_DEPS)						\
		dtd/article.dtd						\
		xslt/books.xslt						\

LANGS =		en ru cn he ja tr

all:		news arx 404 $(LANGS)

news:		$(OUT)/index.html $(OUT)/index.rss
arx:		$(OUT)/2011.html $(OUT)/2010.html $(OUT)/2009.html
404:		$(OUT)/404.html


include 	$(foreach lang, $(LANGS), xml/$(lang)/GNUmakefile)


$(OUT)/index.html:							\
		xml/index.xml						\
		$(NEWS_DEPS)
	$(call XSLT, xslt/news.xslt, $<, $@)

$(OUT)/index.rss:							\
		xml/index.xml						\
		$(NEWS_DEPS)						\
		xslt/rss.xslt
	$(call XSLT, xslt/rss.xslt, $<, $@)


$(OUT)/2011.html							\
$(OUT)/2010.html							\
$(OUT)/2009.html:							\
		xml/index.xml						\
		$(NEWS_DEPS)
	$(call XSLT, xslt/news.xslt, $<, $@, YEAR=$(basename $(notdir $@)))

$(OUT)/404.html:							\
		xml/404.xml						\
		xml/menu.xml						\
		dtd/article.dtd						\
		dtd/content.dtd						\
		xslt/ga.xslt						\
		xslt/error.xslt
	$(call XSLT, xslt/error.xslt, $<, $@)

$(OUT)/%.html:	xml/%.xml						\
		$(ARTICLE_DEPS)
	$(call XSLT, xslt/article.xslt, $<, $@)


# Prevent intermediate .xslt files from being removed.
$(patsubst xsls/%.xsls,xslt/%.xslt,$(wildcard xsls/*.xsls)):

xslt/%.xslt:	xsls/%.xsls						\
		xsls/dump.xsls
	mkdir -p $(dir $@)
	$(call XSLScript, $<, $@)

images:									\
		binary/books/nginx_http_server_jp.jpg			\
		binary/books/nginx_1_web_server.jpg			\
		binary/books/nginx_http_server.jpg			\
		binary/books/nginx_in_practice.jpg

binary/books/nginx_http_server_jp.jpg:	sources/1106030720.jpg
	mkdir -p $(dir $@)
	$(call JPEGNORM, $<, $@)

binary/books/nginx_1_web_server.jpg:					\
		sources/Nginx\ 1\ Web\ Server\ Implementation\ Cookbook.jpg
	mkdir -p $(dir $@)
	$(call JPEGNORM, "$<", $@)

binary/books/nginx_http_server.jpg:	sources/0868OS_MockupCover.jpg
	mkdir -p $(dir $@)
	$(call JPEGNORM, $<, $@)

binary/books/nginx_in_practice.jpg:	sources/20807089-1_o.jpg
	mkdir -p $(dir $@)
	$(call JPEGNORM, $<, $@)


.PHONY:	gzip
gzip:	rsync_gzip
	$(MAKE) do_gzip

rsync_gzip:
	$(CHMOD) $(OUT) $(TEXT)
	$(RSYNC) --delete --exclude='*.gz' $(OUT)/ $(TEXT)/ $(ZIP)/

do_gzip:	$(addsuffix .gz, $(wildcard $(ZIP)/*.html))		\
		$(addsuffix .gz,					\
			$(foreach lang, $(LANGS),			\
			$(foreach dir, . docs docs/faq docs/http,	\
			$(wildcard $(ZIP)/$(lang)/$(dir)/*.html))))	\
		$(ZIP)/index.rss.gz					\
		$(ZIP)/LICENSE.gz					\
		$(ZIP)/en/CHANGES.gz					\
		$(addsuffix .gz, $(wildcard $(ZIP)/en/CHANGES-?.?))	\
		$(ZIP)/ru/CHANGES.ru.gz					\
		$(addsuffix .gz, $(wildcard $(ZIP)/ru/CHANGES.ru-?.?))	\
		$(addsuffix .gz, $(wildcard $(ZIP)/keys/*.key))		\

	find $(ZIP) -type f ! -name '*.gz' -exec test \! -e {}.gz \; -print

	find $(ZIP) -type f -name '*.gz' | \
		while read f ; do test -e "$${f%.gz}" || rm -fv "$$f" ; done

$(ZIP)/%.gz:		$(ZIP)/%
		rm -f $<.gz
		gzip -9cn $< > $<.gz
		touch -r $< $<.gz

draft:	all
	$(CHMOD) $(OUT)
	$(RSYNC) --delete $(OUT)/ $(NGINX_ORG)/$(OUT)/

.PHONY:	binary
binary:
	$(CHMOD) binary
	$(RSYNC) binary/ $(NGINX_ORG)/

copy:
	$(CHMOD) $(ZIP) binary
	$(RSYNC) $(ZIP)/ binary/ $(NGINX_ORG)/
	$(RSYNC) --delete $(foreach lang, $(LANGS), $(ZIP)/$(lang))	\
		$(NGINX_ORG)/

dev:	xslt/version.xslt sign
dev:	NGINX:=$(shell xsltproc xslt/version.xslt xml/versions.xml)

stable:	xslt/version.xslt sign
stable:	NGINX:=$(shell xsltproc --stringparam VERSION stable		\
	xslt/version.xslt xml/versions.xml)

legacy:	xslt/version.xslt sign
legacy:	NGINX:=$(shell xsltproc --stringparam VERSION legacy_stable	\
	xslt/version.xslt xml/versions.xml)

any:	sign
any:	NGINX=0.7.69


sign:
	@echo sign nginx-$(NGINX)

	gpg -sab binary/download/nginx-$(NGINX).tar.gz
	gpg -sab binary/download/nginx-$(NGINX).zip


TEMP =	temp
SITE =	nginx.org

tarball:
	rm -rf $(TEMP)
	mkdir -p $(TEMP)/$(SITE)
	cp -Rp BSDmakefile GNUmakefile umasked.sh			\
		xml xsls xslt dtd binary sources text			\
	$(TEMP)/$(SITE)

	rm -f $(SITE).tar.bz2
	tar -c -y -f $(SITE).tar.bz2					\
		--directory $(TEMP)					\
		--exclude .svn						\
		$(SITE)

dir.map:	xslt/dirmap.xslt xml/en/docs/dirindex.xml
	@xsltproc -o - xslt/dirmap.xslt xml/en/docs/dirindex.xml |	\
	sort -u -k1,1 | sed 's/^include /\\&/' > $@

ifeq ($(patsubst %.nginx.org,YES,$(shell hostname)), YES)
all:	dir.map
copy:	copy_dirmap
.PHONY:	copy_dirmap
copy_dirmap:
	/usr/local/bin/copy_dirmap.sh dir.map
endif

.DELETE_ON_ERROR:
