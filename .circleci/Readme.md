# JSON Assertion Testing with Bash

A way to compare/assert handfuls of JSON with `bash` and `jq` with decent test
reporting using [bats](https://github.com/sstephenson/bats).

Mostly sugar around this core assertion:

```json-compare
#!/usr/bin/env bash
# exit 0 on success only

jq \
  --argfile actual "$1" \
  --argfile expected "$2" \
  -n -e '$actual == $expected' > /dev/null
```

Write a program that prints the JSON needed for one test/comparison, e.g.
parse some file:

```parse-pcap
#!/usr/bin/env bash
# Output JSON to stdout

set -o pipefail

tshark \
  -r "$1" \
  -X "lua_script:$2" \
  -T json \
  | jq ".[0]._source.layers.lua"
```

Write a bats compatible template for a test with bash variable expansion:

```test.tmpl
@test "${name}" {
  ./json-compare "${expected}" "${actual}"
}
```

Write a program to generate test data, this creates a directory of JSON files
to run all your tests against:

```gen-data
#!/usr/bin/env bash

set -o pipefail
set -o errexit
set -o noclobber

INPUT="$1"
OUTPUT_DIR="$2"

cat "$INPUT" | while read line; do
  pcap="$(echo $line | sed 's/:.*//')"
  lua="$(echo $line | sed 's/.*://')"
  dir=$(dirname "$pcap")
  file=$(basename "$pcap" | sed 's/\.pcap/\.json/')
  mkdir -p "$OUTPUT_DIR/$dir"
  ./parse-pcap "$pcap" "$lua" > "$OUTPUT_DIR/$dir/$file";
done
```

Generate the fixtures directory:

```
./gen-data pcap-to-lua.txt fixtures
```

Write a program to generate your tests using the template and test data:

```gen-tests
#!/usr/bin/env bash

find fixtures -name \*.json | while read line do;
  name=$(basename "$line" .json)
  expected="$line"
  actual="$(echo $line | sed 's/fixtures/actual')"
  envsubst < test.tmpl
done
```

Generate tests based on JSON files in fixtures:

```
./gen-tests fixtures > tests
```

Run tests:

```
bats tests
```
