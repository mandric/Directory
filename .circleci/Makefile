#
# Depends on wireshark-lua repo, set LUA_BASE if it's already cloned somewhere,
# otherwise we clone in the current directory.
#

SHELL=/usr/bin/env bash -o pipefail -o errexit

all: tests

deps: jq tshark wireshark-lua fixtures

jq:
	which jq > /dev/null || sudo apt-get -y install jq

tshark:
	which tshark > /dev/null || sudo apt-get -y install tshark

wireshark-lua:
	test -n "$(LUA_BASE)" || \
	  git clone --depth=1 \
            git@github.com:Open-Markets-Initiative/wireshark-lua.git

# Generate the data we use to compare against new builds.  There should be one
# JSON file per pcap data file (1:1).
fixtures:
	#test -d fixtures || ./gen-data ./pcap-to-lua.txt fixtures
	./gen-data ./pcap-to-lua.txt fixtures

# Generate new dissector data then compare against existing set in fixtures.
tests: deps
	./gen-data ./pcap-to-lua.txt actual
	find fixtures -name \*.json | while read expected; do \
	 ./compare "$$expected" "$$(echo $$expected | sed 's/fixtures/actual/')"; \
	done

clean:
	rm -rf actual

reset: clean
	rm -rf wireshark-lua fixtures
