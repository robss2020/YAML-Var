Version 0.9

![ Puzzle pieces with := sign in front of them](https://raw.githubusercontent.com/robss2020/YAML-Var/main/0.9/banner.png "The key to the puzzle")

# Welcome

Welcome to YAML-Var! The easiest way to add variables to your YAML files. Set by using := and use by enclosing in \{braces\}.  YAML-Var converts to YAML after making the substitutions.

This is the specification file.  You can try [YAML-Var online here.](https://taonexus.com/publicfiles/playground/yaml-var/yaml-var-converter)

# Motivation and benefits

When using a YAML configuration file, you may want to use placeholder values or simple variables to cut down on redundancy.

Here is a simple example:

`# Setting a variable`  
`name := John Smith`

Note the lack of any quotation marks.

`# Using the variable`  
`username: {name}`

This would set the username to "John Smith" without quotation marks but including the space.

Here is a more typical example:

`# Setting variables for a base path`  
`   base := /path/to/my/project`

`# Using the variables giving a configuration file`  
`configuration1: {base}/config1.yaml`  
`configuration2: {base}/config2.yaml`

Now you know everything you need to use YAML-Var with ease.

The rest of this document is the formal specification.


# Requirements and constraints for the product design specifications:

Requirements:  
-	 "Some way to specify variables for reusable value"  
-	 "variables: specify and populate placeholders"
	 
Constraints:  
-	"Not having to learn a radical new config file format."  
-	"Concise, unambiguous, and simple."
	
For this reason, YAML was chosen, with a simple addition of being able to specify variables or placeholders using the concise := syntax.

Besides this, it was deemed appropriate for the specification to be easily understood in its entirety within 15 seconds, such as with the line:  
-   "Set variables for substitution using := syntax and substitute them by placing the variable name in \{braces\}"

A user should feel confident writing their first script in YAML-Var format within 60 seconds and have a complete understanding of the format within 5 minutes.

# Comments, whitespace, and quotation marks in YAML-Var

- When declaring a variable, anything after a # is stripped in case you want to leave a comment at the end of the line. # is illegal in a YAML-Var variable name and there is no way to include one in the value either.
- Lines that set variables won't be output after conversion to YAML.
- Whitespace is stripped from both ends of the value
- Quotation marks of any kind are included literally, so don't include them because they will be substituted wherever you use the YAML-Var, which is probably not what you want.
- Substitutions are made throughout the file, including in variable assignments. So this is valid:

`  base := /base/path/to/location`  
`  file := {base}/program`  
`  file-location: {file}`

  When this is converted from YAML-Var to YAML the output would be:
  
`  file-location: /base/path/to/location/program`
  
  The first of the three lines above will set the value of `base`, the second line will set the value of `file` while making use of the value of `base`, and the third line actually substitutes a variable into the YAML file.
  Since the first two lines are mere assignments as part of the YAML-Var file, they would not be printed into the YAML file.

# Uses

Here are a few examples of how YAML-Var can simplify your YAML files:

Simply set a variable with := and start using it with \{\}. Note that variable values are set once upon being read - it is a simple substitution. They are not re-evaluated at runtime.

A simple example:

```
basepath := /usr/path/to/shared/folder
username := jsmith
users:
  {username}:
    pubdir: {basepath}/{username}
```

The result will be

```
users:
  jsmith:
    pubdir: /usr/path/to/shared/folder/jsmith
```


A more complicated example:

```
basepath := /usr/local/lib/python3.10/site-packages/tensorflow/python/keras/engine
username := jsmith
users:
  {username}:
    pubdir: {basepath}/pub/{username}
```

The result will be

```
users:
  jsmith:
    pubdir: /usr/local/lib/python3.10/site-packages/tensorflow/python/keras/engine/pub/jsmith
```



# Suggested file extension

It is suggested that the extension .yamlv be used for YAML-Var files.


# Formal specification:

A compliant YAML-Var processor MUST:
- In each line where := occurs, set the variable with the identifier on the left-hand side of the := with the value on the right-hand side of the := up to any occurrence #, stripping whitespace from both sides of the value.
- In each line, for each variables that was set and in the order it was set, at any occurrence of a variable name surrounded by braces, substitute that variable with its current value if and only if that variable name is currently defined.
- Not make substitutions after any #, but retain the original comment line.
- Allow variables to be redeclared with a new value without any warning.
- After making the substitution, output each line if it did not originally contain an assignment. (i.e. suppress lines from the output that set the YAML-Var variables.)


A compliant YAML-Var processor MAY:
- Ignore or include quotation marks and whitespace as it sees fit
- Apply any type of formating to its YAML output (e.g. beautifying)
- Warn the user if a YAML-Var variable is set but never used.
- Warn the user if there is a value in the format /\{[^{%].+?\}/ without that variable having been set. (i.e. a {} that does not have a second brace or % sign inside with a variable that was not set.)
- Warn the user if more than one := appears on a line.
- Warn the user if a variable value is empty

There are no performance specifications and compliant YAML-Var parsers may take any amount of time, space, and memory to run and be any size.

# Online demo

You can try out YAML-Var here: https://taonexus.com/publicfiles/playground/yaml-var/yaml-var-converter

# Implementers

If you would like to implement YAML-Var yourself, the following are some decisions to keep in mind:
- A line may contain only one :=, more than one := is undefined. The reference implementation warns the user.
- If a variable is set but never used, it may be worth telling the user.
- If there is a \{\} that does not seem to reference a YAML-Var variable it is worth warning the user, as it may be a typo. However, if it starts with a second opening brace,  {{like this}} or a percent sign {%like this%} then it is probably part of another templating language and not worth warning about.
- Remember that a variable assignment may itself have parts, for example `fullpath := {basepath}/{username}` should set the `fullpath` after making both substitutions if they have been set. These values should be evaluated just once.

# Design history
A configuration file format was needed and the above requirements and constraints were collected. A JavaScript playground implementation was first done to explore edge cases and usability in an interactive environment,
this is [still online and available here](https://taonexus.com/publicfiles/playground/yaml-var/yaml-var-converter).
The results were then formalized into the specification.  For such a simple substitution there were a surprising number of cases that seemed obvious but needed
to be specified, such as having more than one := syntax on the same line, redeclaring variables, dealing with comments in the source code.

The proper design choice vis-a-vis quotation marks posed an especially large challenge, since YAML supports a wide range of quotation marks and elsewhere we were advised "the type of quotes
etc shouldn't be syntactically relevant".  However, there are many types of quotes available and matching all available quotation types would greatly complicate the format, therefore it was decided
to specify that quotation marks should not be used, and if present to include them in the variable values literally.

# Contributing

We invite suggestions of any kind to rviragh+yamlvar@gmail.com

# Version history

0.9 - Public feedback.

# License
CC0 "No Rights Reserved", like public domain. Free to use in any commercial or non-commercial way without attribution:

https://creativecommons.org/share-your-work/public-domain/cc0/