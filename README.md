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

This script reads a config file, which describes a list
of files in order. It then reads those files and parses them
for comments matching spcified patterns. The combination of
parsed comments, plus extra info pulled from the config file
is combined into a single output stream.

The comment-doc script uses a number of global variables that can be overridden with command ;ine options. These options are as follows:
- --config: This is the configuration. It is an object, with specific headers and a list of files.
- --headers: This determines whether to display pandoc markdown headers. The default is no.
- --output: This defaults to stdout, but can be used to store items in a file.

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
If it finds a special `start_line`, it will begin searching for comments.
By default the `start_line` is set to match any line.
If it finds a comment delineated by the `include_marker`,
it removes the `include_marker` and adds the comment to the list
It also checks for a special `end_line`, which will stop it from finding further comments.
Finally, it appends `post_lines` as it did with the `pre_lines`.

The output can be written to an output file or to stdout.

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
    "files" : [
        {
            "file_name" : "README.md",
            "start_line" : "Comment-Doc",
            "end_line" : "^---$"
        },        
        {
            "file_name" : "comment-doc",
            "start_line" : "##",
            "include_marker" : "^.*##"
        },
        {
            "file_name" : ".comment-doc.json",
            "pre_lines" : [
            "The following is a sample .comment-doc.json file:",
            "```" ],
            "post_lines" : [ "```" ]
        }
    ]
}
```
