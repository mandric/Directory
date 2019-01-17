
SHELL=/usr/bin/env bash -o pipefail -o errexit

.PHONY: test

all: test

deps:
	which jq > /dev/null || sudo apt-get -y install jq
	which bats > /dev/null || sudo apt-get -y install bats
	which tshark > /dev/null || sudo apt-get -y install tshark
	# Depends on wireshark-lua repo, set LUA_BASE to the path, otherwise we
	# clone in current directory.
	test -d "$(LUA_BASE)" || \
	  git clone --depth=1 \
            git@github.com:Open-Markets-Initiative/wireshark-lua.git
	# Depends on pcap files, set PCAP_BASE if you need to specify
	test -d "$(PCAP_BASE)" || test -d ../Data

fixtures: deps
	./gen-data ./pcap-to-lua.txt fixtures

actual: deps
	./gen-data ./pcap-to-lua.txt actual

tests:
	./gen-tests fixtures > tests

test: actual tests
	bats tests

clean:
	rm -rf actual

reset: clean
	rm -rf wireshark-lua fixtures
