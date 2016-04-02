GHPAGES_TMP := /tmp/ghpages$(shell echo $$$$)
.INTERMEDIATE: $(GHPAGES_TMP)
ifneq (,$(CI_BRANCH))
GIT_ORIG := $(CI_BRANCH)
else
GIT_ORIG := $(shell git branch | grep '*' | cut -c 3-)
endif
ifneq (,$(findstring detached from,$(GIT_ORIG)))
GIT_ORIG := $(shell git show -s --format='format:%H')
endif

# Only run upload if we are local or on the master branch
IS_LOCAL := $(if $(CI),,true)
ifeq (master,$(CI_BRANCH))
IS_MASTER := $(findstring false,$(CI_IS_PR))
else
IS_MASTER :=
endif

Overview.html : Overview-WebCryptoAPI.xml WebIDL.xsl
	xsltproc --nodtdattr --stringparam now `date +%Y%m%d` --stringparam rev "0" -o Overview.html WebIDL.xsl Overview-WebCryptoAPI.xml 

index.html: Overview.html
	cp $< $@

ghpages: index.html Overview.html
ifneq (true,$(CI))
	@git show-ref refs/heads/gh-pages > /dev/null 2>&1 || \
	  ! echo 'Error: No gh-pages branch, run `make setup-ghpages` to initialize it.'
endif
ifneq (,$(or $(IS_LOCAL),$(IS_MASTER)))
	mkdir $(GHPAGES_TMP)
	cp -f $^ $(GHPAGES_TMP)
	git clean -qfdX
ifeq (true,$(CI))
	git config user.email "ci-bot@example.com"
	git config user.name "CI Bot"
	git checkout -q --orphan gh-pages
	git rm -qr --cached .
	git clean -qfd
	git pull -qf origin gh-pages --depth=5
else
	git checkout gh-pages
	git pull
endif
	mv -f $(GHPAGES_TMP)/* $(CURDIR)
	git add $^
	if test `git status -s | wc -l` -gt 0; then git commit -m "Script updating gh-pages. [ci skip]"; fi
ifneq (,$(CI_HAS_WRITE_KEY))
	git push https://github.com/$(CI_REPO_FULL).git gh-pages
else
ifneq (,$(GH_TOKEN))
	@echo git push -q https://github.com/$(CI_REPO_FULL).git gh-pages
	@git push -q https://$(GH_TOKEN)@github.com/$(CI_REPO_FULL).git gh-pages >/dev/null 2>&1
endif
endif
	-git checkout -qf "$(GIT_ORIG)"
	-rm -rf $(GHPAGES_TMP)
endif
