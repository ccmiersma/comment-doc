Comment-Doc
===========

This a small perl script for generating documentation from comments in scripts.
The primary use case is generating markdown documents form comments in bash,
perl, and Ansible scripts, as well as configuration files. Ordinarily, the
more sophisticated tools such as javadoc and doxygen should be used for bigger projects.

This tool is designed to be used in conjunction with pandoc. It generates headers
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

The comment-doc script uses a number of global variables. These are as follows:
- $config: This is the configuration. It is an object, with specific headers and a list of files.
- $output: This is the final concatenated output string from the files and config.
- $headers: This determines whether to display pandoc markdown headers. The default is no.
- $output_filename: This defaults to stdout, but can be used to store items in a file.

The configuration file must be a hidden file
in the current directory called .comment-doc.json
The script opens and reads the configuration file.
It converts the JSON to a $config object and validates it.

The configuration validation ensures that a complete
configuration is present with default values if necessary.

The script reads each file in the list in order and returns
the comments extracted at the end.
For each file, it first inserts the configured `pre_lines`
that precede the file contents, such as metadata about the file
or lines including an entire block of code.
This step is skipped only if the `pre_lines` are empty.

The script then reads the file.
If it finds a special `start_line`, ir will begin searching for comments.
By default the `start_line` is set to match any line.
If it finds a comment delineated by the `include_marker`,
it removes the `include_marker` and adds the comment to the list
It also checks for a special `end_line`, which will stop it from finding further comments.
Finally, it appends `post_lines` as it did with the `pre_lines`.

The script allows two options, --headers, and --output $file_name

The output can be written to an output file or to stdout.

The following is a sample .comment-doc.json file:
```
{
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
