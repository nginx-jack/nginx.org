
OUT =		libxslt
TEXT =		text
CSS =		css
IMG =		img
ZIP =		gzip
NGINX_ORG =	/data/www/nginx.org
SHELL =		tools/umasked.sh

XSLS ?=		tools/xslscript.pl
RSYNC =		rsync -v -rpc
CHMOD =		/bin/chmod -R g=u


define	XSLScript
	$(XSLS) -o $(2) $(1)
endef

define	XSLT
	xmllint --noout --valid $2
	xsltproc -o $3							\
		$(shell ff="$(strip $2)"; f=$${ff#xml/*/};		\
		if [ "$$f" != "$$ff" ]; then				\
		[ -f xml/en/$$f ] && echo --stringparam ORIGIN "en/$$f";\
		t=; for l in $(LANGS); do				\
		[ -f "xml/$$l/$$f" ] && t="$$t$$l "; done;		\
		echo --stringparam TRANS "\"$$t\"";			\
		fi)							\
		$(if $4,--stringparam $4 $5)				\
		$1 $2
endef

define 	JPEGNORM
	jpegtopnm $1							\
		| pamscale -width=150					\
		| pnmtojpeg -quality=95 -optimize -dct=float		\
		> $2
endef


COMMON_DEPS =								\
		xml/banner.xml						\
		xml/menu.xml						\
		xml/i18n.xml						\
		dtd/content.dtd						\
		xslt/dirname.xslt					\
		xslt/link.xslt						\
		xslt/style.xslt						\
		xslt/body.xslt						\
		xslt/menu.xslt						\
		xslt/content.xslt					\

ARTICLE_DEPS =								\
		$(COMMON_DEPS)						\
		xml/versions.xml					\
		dtd/article.dtd						\
		dtd/module.dtd						\
		xslt/article.xslt					\
		xslt/books.xslt						\
		xslt/directive.xslt					\
		xslt/donate.xslt					\
		xslt/download.xslt					\
		xslt/security.xslt					\
		xslt/versions.xslt					\
		xslt/projects.xslt					\

NEWS_DEPS =								\
		$(COMMON_DEPS)						\
		dtd/news.dtd						\
		xslt/news.xslt						\

LANGS =		en ru

YEARS = 								\
		2009 2010 2011 2012 2013 2014 2015 2016 2017 2018 2019	\
		2020 2021 2022 2023 2024 2025

all:		homepage news arx 404 css $(LANGS)

homepage:	$(OUT)/index.html
news:		$(OUT)/news.html $(OUT)/index.rss
arx:		$(foreach year,$(YEARS),$(OUT)/$(year).html)
404:		$(OUT)/404.html
css:		$(foreach f,$(wildcard css/*.css),$(OUT)/$(f))


DIRIND_DEPS =
VARIND_DEPS =

define lang-specific

TOP=
DOCS=
REFS=
FAQ=
include xml/$(lang)/GNUmakefile

$(lang):								\
		$$(foreach f,index $$(TOP),$(OUT)/$(lang)/$$(f).html)	\
		$$(foreach f,index $$(DOCS) $$(REFS) $$(FAQ),		\
		$(OUT)/$(lang)/docs/$$(f).html)

$(OUT)/$(lang)/docs/index.html:						\
		$$(foreach f,$$(DOCS) $$(REFS),				\
		$(OUT)/$(lang)/docs/$$(f).html)

$(OUT)/$(lang)/docs/faq.html:						\
		$$(foreach f,$$(FAQ),$(OUT)/$(lang)/docs/$$(f).html)

ifneq (,$$(filter dirindex,$$(DOCS)))
DIRIND_DEPS +=	xml/$(lang)/docs/dirindex.xml
xml/$(lang)/docs/dirindex.xml:						\
		$$(foreach f,$$(REFS),xml/$(lang)/docs/$$(f).xml)	\
		xslt/dirindex.xslt
	echo "<modules>$$(patsubst %,					\
	<module name=\"%\"/>, $$(filter %.xml,$$^))</modules>" |	\
	xsltproc -o - --stringparam LANG $(lang)			\
	xslt/dirindex.xslt - |						\
	sed 's;xml/[^/]*/docs/;;g' > $$@
endif

ifneq (,$$(filter varindex,$$(DOCS)))
VARIND_DEPS +=	xml/$(lang)/docs/varindex.xml
xml/$(lang)/docs/varindex.xml:						\
		$$(foreach f,$$(REFS),xml/$(lang)/docs/$$(f).xml)	\
		xslt/varindex.xslt
	echo "<modules>$$(patsubst %,					\
	<module name=\"%\"/>, $$(filter %.xml,$$^))</modules>" |	\
	xsltproc -o - --stringparam LANG $(lang)			\
	xslt/varindex.xslt - |						\
	sed 's;xml/[^/]*/docs/;;g' > $$@
endif

endef

$(foreach lang, $(LANGS), $(eval $(call lang-specific)))

$(foreach lang, $(LANGS), $(OUT)/$(lang)/docs/dirindex.html): $(DIRIND_DEPS)

$(foreach lang, $(LANGS), $(OUT)/$(lang)/docs/varindex.html): $(VARIND_DEPS)

$(OUT)/index.html:							\
		xml/homepage.xml					\
		xml/index.xml						\
		$(ARTICLE_DEPS)
	$(call XSLT, xslt/article.xslt, $<, $@)

$(OUT)/index.rss:							\
		xml/index.xml						\
		$(NEWS_DEPS)						\
		xslt/rss.xslt
	$(call XSLT, xslt/rss.xslt, $<, $@)

$(OUT)/news.html:							\
		xml/index.xml						\
		$(NEWS_DEPS)
	$(call XSLT, xslt/news.xslt, $<, $@)

$(foreach year,$(YEARS),$(OUT)/$(year).html):				\
		xml/index.xml						\
		$(NEWS_DEPS)
	$(call XSLT, xslt/news.xslt, $<, $@, YEAR, $(basename $(notdir $@)))

$(OUT)/404.html:							\
		xml/404.xml						\
		$(COMMON_DEPS)						\
		dtd/error.dtd						\
		xslt/error.xslt
	$(call XSLT, xslt/error.xslt, $<, $@)

$(OUT)/%.html:	xml/%.xml						\
		$(ARTICLE_DEPS)
	$(call XSLT, xslt/article.xslt, $<, $@)


# Prevent intermediate .xslt files from being removed.
$(patsubst xsls/%.xsls,xslt/%.xslt,$(wildcard xsls/*.xsls)):

xslt/%.xslt:	xsls/%.xsls
	mkdir -p $(dir $@)
	$(call XSLScript, $<, $@)

$(OUT)/css/%.css:      css/%.css
		mkdir -p $(dir $@)
		cp $< $@

genapi:
	$(MAKE) -C yaml


images:									\
		binary/books/complete_nginx_cookbook_2019.jpg		\
		binary/books/deploying_nginx_as_api_gateway.jpg		\
		binary/books/nginx_cookbook.jpg				\
		binary/books/nginx_http_server_3rd_ed.jpg		\
		binary/books/nginx_troubleshooting.jpg			\
		binary/books/nginx_richtig_konfigurieren.jpg		\
		binary/books/practical_nginx_guide_jp.jpg		\
		binary/books/nginx_pocket_reference_jp.jpg		\
		binary/books/nginx_http_server_jp.jpg			\
		binary/books/nginx_1_web_server.jpg			\
		binary/books/nginx_http_server.jpg			\
		binary/books/nginx_in_practice.jpg			\
		binary/books/mastering_nginx.jpg			\
		binary/books/nginx_http_server_2nd_ed.jpg		\
		binary/books/instant_nginx_starter.jpg			\
		binary/books/nginx_module_extension.jpg			\
		binary/books/nginx_high_performance.jpg			\
		binary/books/nginx_essentials.jpg			\
		binary/books/nginx_http_server_4th_ed.jpg

binary/books/complete_nginx_cookbook_2019.jpg:				\
		sources/ebk-ORM-NGINX-Cookbook-mega-2019-150x185.jpg
	mkdir -p $(dir $@)
	$(call JPEGNORM, $<, $@)

binary/books/deploying_nginx_as_api_gateway.jpg:			\
		sources/ebk-Deploying-NGINX-Plus-as-API-Gateway-150x185.jpg
	mkdir -p $(dir $@)
	$(call JPEGNORM, $<, $@)

binary/books/nginx_cookbook.jpg:	sources/B05431_0.jpg
	mkdir -p $(dir $@)
	$(call JPEGNORM, $<, $@)

binary/books/nginx_http_server_3rd_ed.jpg:				\
		sources/0337OS_4846_Nginx.jpg
	mkdir -p $(dir $@)
	$(call JPEGNORM, $<, $@)

binary/books/nginx_troubleshooting.jpg:					\
		sources/51T7ds6JdBL._SX404_BO1,204,203,200_.jpg
	mkdir -p $(dir $@)
	$(call JPEGNORM, $<, $@)

binary/books/nginx_richtig_konfigurieren.jpg:	sources/5106%2B0b2pbL.jpg
	mkdir -p $(dir $@)
	$(call JPEGNORM, $<, $@)

binary/books/practical_nginx_guide_jp.jpg:	sources/9784774178660.jpg
	mkdir -p $(dir $@)
	$(call JPEGNORM, $<, $@)

binary/books/nginx_pocket_reference_jp.jpg:				\
		sources/51JYTdy8jrL._SX335_BO1,204,203,200_.jpg
	mkdir -p $(dir $@)
	$(call JPEGNORM, $<, $@)

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

binary/books/mastering_nginx.jpg:	sources/3311OS_4851_Mastering\ NGINX_0.jpg
	mkdir -p $(dir $@)
	$(call JPEGNORM, "$<", $@)

binary/books/nginx_http_server_2nd_ed.jpg:	sources/2322OS_cov.jpg
	mkdir -p $(dir $@)
	$(call JPEGNORM, $<, $@)

binary/books/instant_nginx_starter.jpg:	sources/5125OS_cov.jpg
	mkdir -p $(dir $@)
	$(call JPEGNORM, $<, $@)

binary/books/nginx_module_extension.jpg:	sources/3046OS_cover.jpg
	mkdir -p $(dir $@)
	$(call JPEGNORM, $<, $@)

binary/books/nginx_high_performance.jpg:	sources/1839OS.jpg
	mkdir -p $(dir $@)
	$(call JPEGNORM, $<, $@)

binary/books/nginx_essentials.jpg:	sources/B04282_MockupCover_Normal.jpg
	mkdir -p $(dir $@)
	$(call JPEGNORM, $<, $@)

binary/books/nginx_http_server_4th_ed.jpg:	sources/9781788623551.jpg
	mkdir -p $(dir $@)
	$(call JPEGNORM, $<, $@)

.PHONY:	gzip
gzip:	rsync_gzip
	$(MAKE) do_gzip

rsync_gzip:
	$(CHMOD) $(OUT) $(TEXT)
	$(RSYNC) --delete --exclude='*.gz' $(OUT)/ $(TEXT)/ $(IMG) $(ZIP)/

do_gzip:	$(addsuffix .gz, $(wildcard $(ZIP)/*.html))		\
		$(addsuffix .gz,					\
			$(foreach lang, $(LANGS),			\
			$(foreach dir, . docs docs/dev docs/faq docs/http docs/mail docs/njs docs/stream, \
			$(wildcard $(ZIP)/$(lang)/$(dir)/*.html))))	\
		$(ZIP)/index.rss.gz					\
		$(ZIP)/LICENSE.gz					\
		$(ZIP)/en/CHANGES.gz					\
		$(addsuffix .gz, $(wildcard $(ZIP)/en/CHANGES-?.?))	\
		$(addsuffix .gz, $(wildcard $(ZIP)/en/CHANGES-?.??))	\
		$(ZIP)/ru/CHANGES.ru.gz					\
		$(addsuffix .gz, $(wildcard $(ZIP)/ru/CHANGES.ru-?.?))	\
		$(addsuffix .gz, $(wildcard $(ZIP)/ru/CHANGES.ru-?.??))	\
		$(addsuffix .gz, $(wildcard $(ZIP)/keys/*.key))		\
		$(addsuffix .gz, $(wildcard $(ZIP)/css/*.css))		\
		$(addsuffix .gz, $(wildcard $(ZIP)/img/*.svg))		\

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
legacy:	NGINX:=$(shell xsltproc --stringparam VERSION legacy		\
	xslt/version.xslt xml/versions.xml)

any:	sign
any:	NGINX=0.7.69


sign:
	@echo sign nginx-$(NGINX)

	gpg -sab binary/download/nginx-$(NGINX).tar.gz
	gpg -sab binary/download/nginx-$(NGINX).zip


dir.map:	xslt/dirmap.xslt xml/en/docs/dirindex.xml		\
		xml/en/docs/varindex.xml
	@xsltproc -o - xslt/dirmap.xslt xml/en/docs/dirindex.xml	\
	xml/en/docs/varindex.xml > $@

ifeq ($(patsubst %.nginx.com,YES,$(shell hostname)), YES)
all:	images

ifeq ($(NGINX_ORG), /data/www/nginx.org)
all:	dir.map
copy:	copy_dirmap
.PHONY:	copy_dirmap
copy_dirmap:
	/usr/local/bin/copy_dirmap.sh dir.map $(NGINX_ORG)
endif

draft:	copy_draft
.PHONY:	copy_draft
copy_draft:
	/usr/local/bin/copy_draft.sh $(NGINX_ORG)
endif


#
# Targets and rules for generating Markdown files from XML using XSLT transformations.
# Includes module documentation and directory and variable indexes.
#

MODULE_MD_XSLT = xslt/module-md.xslt
XML_MODULE_DIR = xml/en/docs
MD_OUT_DIR     = libxslt-md/
MODULE_MD_DEPS = $(MODULE_MD_XSLT) dtd/module.dtd

define MODULE_MD_XSLT_PROC
	xsltproc --stringparam sourceFileName "$(1)" $(MODULE_MD_XSLT) "$(XML_MODULE_DIR)/$(1)" > "$(MD_OUT_DIR)/module-reference/$(basename $(1)).md"
endef

# Get list of XML module files (excluding the API head)
XML_MODULE_FILES := $(shell grep -rl 'dtd/module.dtd' "$(XML_MODULE_DIR)" | grep -v 'http/ngx_http_api_module_head.xml' | sed 's|^$(XML_MODULE_DIR)/||')
MD_MODULE_FILES := $(patsubst %.xml,$(MD_OUT_DIR)/module-reference/%.md,$(XML_MODULE_FILES))

# API module related paths
NGX_API_HEAD_XML   = $(XML_MODULE_DIR)/http/ngx_http_api_module_head.xml
NGX_API_FIXED_XML  = $(XML_MODULE_DIR)/http/ngx_http_api_module_head_fixed.xml
NGX_API_DEST_XML   = $(XML_MODULE_DIR)/http/ngx_http_api_module.xml
NGX_API_MD         = $(MD_OUT_DIR)/module-reference/http/ngx_api_module.md

# Hacks for inplace ngx_http_api file manipulation
# Step 1: Fix original file (add </module> if missing)
$(NGX_API_FIXED_XML): $(NGX_API_HEAD_XML)
	@echo "Fixing $< (ensuring </module>) -> $@"
	@cp $< $@
	@grep -q '</module>' $@ || echo '</module>' >> $@

# Step 2: Copy fixed file to final location
$(NGX_API_DEST_XML): $(NGX_API_FIXED_XML)
	@echo "Copying fixed XML to $@"
	@cp -f $< $@

# Step 3: Process via XSLT to markdown
$(NGX_API_MD): $(NGX_API_DEST_XML) $(MODULE_MD_DEPS)
	@mkdir -p "$(dir $@)"
	@echo "Processing ngx_http_api_module.xml -> ngx_api_module.md"
	$(call MODULE_MD_XSLT_PROC,http/ngx_http_api_module.xml)

# Entry point for API module handling
.PHONY: ngx-api-prepare
ngx-api-prepare: $(NGX_API_MD)

# Generic XSLT processing for all other modules
$(MD_OUT_DIR)/module-reference/%.md: $(XML_MODULE_DIR)/%.xml $(MODULE_MD_DEPS) ngx-api-prepare
	@if [ "$<" = "$(NGX_API_HEAD_XML)" ]; then \
		echo "Skipping $< (handled separately)"; \
		exit 0; \
	fi
	@mkdir -p "$(dir $@)"
	@echo "Processing $< -> $@"
	$(call MODULE_MD_XSLT_PROC,$*.xml)

# Index generation macros
define generate-index-md
	@mkdir -p "$(dir $3)"
	@echo "Processing $1 -> $3"
	xsltproc -o $3 $2 $1
endef

# Directory index
.PHONY: dirindex-markdown
DIRINDEX_XML = $(XML_MODULE_DIR)/dirindex.xml
DIRINDEX_MD = $(MD_OUT_DIR)/directives.md
DIRINDEX_XSLT = xslt/dirindex-md.xslt

dirindex-markdown: $(DIRINDEX_MD)

$(DIRINDEX_MD): $(DIRINDEX_XML) $(DIRINDEX_XSLT)
	$(call generate-index-md,$(DIRINDEX_XML),$(DIRINDEX_XSLT),$(DIRINDEX_MD))

# Variable index
.PHONY: varindex-markdown
VARINDEX_XML = $(XML_MODULE_DIR)/varindex.xml
VARINDEX_MD = $(MD_OUT_DIR)/variables.md
VARINDEX_XSLT = xslt/varindex-md.xslt

varindex-markdown: $(VARINDEX_MD)

$(VARINDEX_MD): $(VARINDEX_XML) $(VARINDEX_XSLT)
	$(call generate-index-md,$(VARINDEX_XML),$(VARINDEX_XSLT),$(VARINDEX_MD))

# All-in-one target
.PHONY: markdown
markdown: ngx-api-prepare $(MD_MODULE_FILES) dirindex-markdown varindex-markdown
	@echo "All module, dirindex, varindex, and API module XML files converted successfully."

# Cleanup
clean:
	rm -rf $(ZIP) $(OUT) $(MD_OUT_DIR) xml/*/docs/dirindex.xml dir.map \
	xml/*/docs/varindex.xml \
	$(NGX_API_FIXED_XML) $(NGX_API_DEST_XML)

.DELETE_ON_ERROR:
