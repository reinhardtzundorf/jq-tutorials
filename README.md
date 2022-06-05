# JQ

## Introduction

This repository is an attempt to learn more about jq and how it can be used to perform transformations on JSON data structures.
The starting point is to complete the tutorials on the official jq website, available [here](https://stedolan.github.io/jq/tutorial/).

## Tutorial

### Getting Started

Install jq by following this link [https://stedolan.github.io/jq/download/](https://stedolan.github.io/jq/download/). On Mac OSX 
this is easily achieved by using the brew package manager: `brew install jq`. The most recent version is jq 1.6.

The follow text is summarized from the manual on the jq website: [https://stedolan.github.io/jq/manual/](https://stedolan.github.io/jq/manual/).

### About Filters

According to the manual on the website, each jq program is a "filter". It takes input and produces output. There are a number of built-in 
filters for extracting particular fields from an object, or to convert a number to a string or various other standard tasks used in data
transformation.

Filters may also be combined in various ways: output may be piped from one filter into another filter, or the output of a filter could be 
collected into an array. Some filters produce muiltiple results, for instance, there's a filter that produces all the elements of its input 
as an array. Piping the aforementioned filter into a sedcond filter which runs through each element of the array output of the first filter
is an advanced transformation with practical use and application.

In jq a filter will always have input and will always produce output. This is a core concept. Even string or integer literals like "hello" 
and 12 are filters - because they accept input and produce the same literal as output.

### JSON Streams

jq filters run on a stream of JSON data. The input you send to a jq filter is always processed as a stream. The input to jq is parsed as a sequence of whitespace-separated JSON values which are passed through the provided filter one at a time. The output(s) of the filter are written to standard out, again as a sequence of whitespace-separated JSON data. 

### Using Quotes

When using jq it is important to mind the shell's quoting rules. As a general rule, it's best to always quote with single-quotes when sending data to the jq program, as there are many characters with special meaning in jq which are also shell meta-characters. For example, `jq "foo"` will fail on most UNIX shells because that will the same as `jq foo`; throwing an error `foo is not defined`. Double quotes are evaluated in most UNIX shell environments.

### CLI Options

You can call the jq program by issuing the following command:

```bash
jq 
jq --version
jq --seq
jq --stream
jq --slurp / -s
jq --raw-input / -s
jq --nul-input / -n
jq --compact-output / -c
jq --tab					
```
In the list above,

1. Run jq without any options or arguments.

2. Run jq with the `application/json-seq` MIME type scheme for separating JSON text in the input and output. This means that an ASCII Record Separator (RS)character is printed before each value on output and an ASCII Line Feed is printed after every output. Input JSON text that fails to parse will be ignored, although a warning is issued, and subsequent input will be ignored until the next Record Separator (RS). 

The --seq format expects each value to be preceded with ASCII RS character (dec 30 / hex 1E), for example, the first line will not work yet the line thereafter which adds a `\x1e` will produce output:-

```bash
jq --seq ".product_id" '{"product_id": 2, "title": "a product"}{"product_id": 2, "title": "a product"}'
jq --seq ".product_id" '\x1e{"product_id": 2, "title": "a product"}\x1e{"product_id": 3, "title": "a product"}'
```

3. Parse the input as a stream, outputting arrays of path and leaf values (scalars and empty arrays or empty objects):

```bash

```


## Example Usage

Based on this [article](https://blog.jpalardy.com/posts/handling-broken-json-with-jq/) on "Handling Broken JSON with jq".

The article discusses a sample JSON file with a missing bracket. When you run the standard jq command without any filters, jq will reach the place in the file with syntax error and stop processing the stream. The issue is that we do not always have access to the source where the data comes from - otherwise we could simply fix the error in the JSON file.

The suggestion solution involves adding <RS> (ASCII 0x1e) in front of each record in the JSON file. This acts in a similar way to a delimiter. This allows jq to continue processing the stream and sets delimiters around the failures, making it possible to parse the rest of the data. Instead of discarding the entire JSON file, jq simply skips the section where the syntax error is.

i.e. broken JSON file:

```json
{
	"products": [
		{
			"id": 1,
			"title": "product a",
			"sku": "1234a"
		},
		{
			"id": 2,
			"title": "product a",
			"sku": "1234a"
		{
			"id": 3,
			"title": "product a",
			"sku": "1234a"
		}
	]
}
```

The above JSON is missing a bracket. 
By adding <RS> before and after each group, we are setting the boundaries for jq to process the data as a stream and eliminating the chance of loosing all of the data. Instead we'll have a missing record ({"id": "2"..}), but the rest of the data will be processed.

This may be done using the UNIX program `sed`:

```bash
cat broken.json | sed -e 's/^{/'$(printf "\x1e")'{/' | jq --seq .
```

























