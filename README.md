Comment-Doc
===========

This a small perl script for generating documentation from comments in scripts.
The primary use case is generating markdown documents form comments in bash,
perl, and Ansible scripts, as well as configuration files. Ordinarily, the
more sophisticated tools such as javadoc, doxygen, perldoc, etc. should be used for bigger projects.

This tool is designed to be used in conjunction with pandoc. It can generate headers
that can be used for manpages etc.

Installation Instructions
-------------------------

Copy the comment-doc script to a location in your path.

---

Description
-----------

This script reads a config file, which describes a list
of inputs and outputs. For each output file, it matches the associated
inputs. The inputs are defined by the output of an arbitrary command,
but default to using the `find` command to search a path for a given pattern.
The output is parse for comments matching the specified include marker. The combination of
parsed comments, plus extra info pulled from the config file
is combined into an output stream which is written to the corresponding output.

The comment-doc script uses a number of global variables that can be overridden with command line options. These options are as follows:
- --config: This is the configuration. It is an object, with specific headers and a list of files.
- --debug: This determines whether to dump certain variables for debugging. The default is no.
- --headers: This determines whether to display pandoc markdown headers. The default is no.
- --output: This defaults to /dev/stdout, but can be used to store items in a file. This option only modifies the default output. It does not override outputs defined in the config.

Default config files. This is a list of default config files in order of precedence: ./.comment-doc.json, $HOME/.comment-doc.json, and /etc/comment-doc.json.

The configuration file must be correctly formatted json passed at the command line
or loaded from a config file.
The script first checks to see if a config has been passed
at the command line.
If it has, it tries to open the file name or parses the 
string as JSON and returns without trying to open a config file.
If no special config is found, it tries to open and read the configuration files in order.

As soon as it opens a config file successfully, it uses it, and ignores the other files.
It either case, it converts the JSON to a $config object and validates it.

The configuration validation ensures that a complete
configuration is present with default values if necessary.

The script reads each file in the list in order and returns
the comments extracted at the end.
For each file, it first inserts the configured `pre_lines`
that precede the file contents, such as metadata about the file
or lines including an entire block of code.
This step is skipped only if the `pre_lines` are empty.

The script then reads the file.
If it finds a special `begin` regexp, it will begin searching for comments.
By default `begin` is set to match any line.
If it finds a comment delineated by the `include_marker`,
it removes the `include_marker` and adds the comment to the list
It also checks for a special `end` regexp, which will stop it from finding further comments.
Finally, it appends `post_lines` as it did with the `pre_lines`.

Because multiple outputs can be defined, processing is done on a per output basis.
A list of unique outputs is calculated.
As each output is processed, it processes all the inputs.
For each input, if it matches the current output, it process the input and adds it to the output stream.

The default output is /dev/stdout, but can be any arbitrary file.

The following is a sample .comment-doc.json file:
```
{
    "defaults": {
        "output" : "README.md"
    },
    "headers": { 
        "title" : "comment-doc(1)",
        "author" : "Christopher Miersma"
    },    
    "inputs" : [
        {
            "pattern" : "README.md",
            "begin" : "Comment-Doc",
            "end" : "^---$"
        },        
        {
            "pattern" : "comment-doc",
            "begin" : "##",
            "include_marker" : "^.*##"
        },
        {
            "pattern" : ".comment-doc.json",
            "pre_lines" : [
            "The following is a sample .comment-doc.json file:",
            "```" ],
            "post_lines" : [ "```" ]
        }
    ]
}
```
