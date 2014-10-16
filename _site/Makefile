bin = $(shell npm bin)
stylus = $(bin)/stylus
stylus-paths = -I node_modules/nib/lib -I node_modules/entypo-stylus -I node_modules

css:
	mkdir -p css
	$(stylus) $(stylus-paths) $$STYLUS_OPTIONS -o css stylus

css-watch:
	STYLUS_OPTIONS="--watch" $(MAKE) css

.PHONY: css css-watch
