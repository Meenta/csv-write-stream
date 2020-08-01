# csv-write-stream

A CSV encoder stream that produces properly escaped CSVs.

[![NPM](https://nodei.co/npm/csv-write-stream.png)](https://nodei.co/npm/csv-write-stream/)

[![browser support](http://ci.testling.com/maxogden/csv-write-stream.png)](http://ci.testling.com/maxogden/csv-write-stream)

A through stream. Write arrays of strings (or JS objects) and you will receive a properly escaped CSV stream out the other end.

## usage

### var writer = csvWriter([options])

```javascript
var csvWriter = require('csv-write-stream')
var writer = csvWriter()
```

`writer` is a duplex stream -- you can pipe data to it and it will emit a string for each line of the CSV

### default options

```javascript
{
  separator: ',',
  newline: '\n',
  headers: undefined,
  sendHeaders: true
}
```

`headers` can be an array of strings to use as the header row. if you don't specify a header row the keys of the first row written to the stream will be used as the header row IF the first row is an object (see the test suite for more details). if the `sendHeaders` option is set to false, the headers will be used for ordering the data but will never be written to the stream.

`headers` can also be an array of objects containing a header title, field and default. The `title` will replace the field in header. The `field` defines the object key. The `default` set the default value if a key is not found in a given row.

example of auto headers:

```javascript
var writer = csvWriter()
writer.pipe(fs.createWriteStream('out.csv'))
writer.write({hello: "world", foo: "bar", baz: "taco"})
writer.end()

// produces: hello,foo,baz\nworld,bar,taco\n
```

example of specifying headers:

```javascript
var writer = csvWriter({ headers: ["hello", "foo"]})
writer.pipe(fs.createWriteStream('out.csv'))
writer.write(['world', 'bar'])
writer.end()

// produces: hello,foo\nworld,bar\n
```

example of not sending headers:

```javascript
var writer = csvWriter({sendHeaders: false})
writer.pipe(fs.createWriteStream('out.csv'))
writer.write({hello: "world", foo: "bar", baz: "taco"})
writer.end()

// produces: world,bar,taco\n
```

example of with headers objects (title, field and defaults):
note: header objects does not work in the cli.

```javascript
var headers = [
  { title: 'SSN', field: 'ssn_value', default: 'NA'},
  { title: 'First Name', field: 'fName', default: ''},
  { title: 'Last Name', field: 'lName', default: ''}
]
var writer = csvWriter({ headers: headers })
writer.pipe(fs.createWriteStream('out.csv'))
writer.write({ fName: 'Joe', lName: 'Dirt', ssn_value: '111-22-3333'})
writer.end()

// produces:
// SSN,First Name,Last Name\n
// 000-11-0000,Joe,Dirt\n
```

see the test suite for more examples

## run the test suite

```bash
$ npm install
$ npm test
```

## cli usage

This module also includes a CLI, which you can pipe [ndjson](http://ndjson.org) to stdin and it will print csv on stdout. You can install it with `npm install -g csv-write-stream`.

```bash
$ csv-write --help
usage: csv-write [-h] [-v] [--separator SEPARATOR] [--newline NEWLINE]
                 [--headers HEADERS [HEADERS ...]] [--no-send-headers]


A CSV encoder stream that produces properly escaped CSVs. JSON is read from
STDIN, formatted to CSV, and written to STDOUT.

Optional arguments:
  -h, --help            Show this help message and exit.
  -v, --version         Show program's version number and exit.
  --separator SEPARATOR
                        The separator character to use. Defaults to ','.
  --newline NEWLINE     The newline character to use. Defaults to $'\n'.
  --headers HEADERS [HEADERS ...]
                        The list of headers to use. If omitted, the keys of
                        the first row written to STDIN will be used
  --no-send-headers     Don't print the header row.
```

```bash
$ cat example.ndjson | csv-write > example.csv
```
