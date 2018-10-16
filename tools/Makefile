VALIDATOR = ./node_modules/.bin/mip-validator
htmls   = $(shell ls cases/*.html)
results = $(patsubst cases/%.html,cases/%.json, $(htmls))

.PHONY: test 

test : $(results)

$(results):cases/%.json:cases/%.html ./mip.rules.json
	$(VALIDATOR) -c "./mip.rules.json" < $< > $@

