#Notes on these labs.

##Release versions
The labs have been tested on Release 5.1.5.2. On other release versions, the lab descriptions and transcripts may not be 100% in accordance with the prompts of the various commands, and the order/names may change slightly.

##Notes on shell conventions used.
The Bourn Shell (bash) was used to perform the labs in the transcripts.
The constructs used are quite basic (with one possible exception - see below) and so should work with a Korn shell (ksh), or even a basic Unix shell (sh).

##Here documents
Many commands take input from either the standard input, or from a file. It is somewhat traditional to have configuration information, and data to be loaded in separate files, and to specify the command-line argument to the command to read from the appropriate file.

Rather than having multiple files to manage for these labs, a different approach has been used; the shell "here document".

This allows text to be listed in the script which is fed to the command on its standard input (just as though you had typed it). The data is in-line in the script, rather than having to be maintained in a separate file. The basic form is as follows:

	command --argument1 --argument2 <<EOF
	This text is fed
	to the standard input of the
	command up to the terminating
	string.
	EOF

The syntax: *<<EOF* indicates that the text following is to be fed to the command standard input, up to the terminator, in this case *EOF*.

The terminator has to be at the beginning of its own line.
It is possible to use indentation, but there are restrictions, most notably, indentation has to be in the form of *TAB* characters. Unfortunately, many text editors have a nasty habit of transforming *TAB* into *spaces*. For that reason, it is avoided in this lab.

Note also that shell variable interpolation is performed on the text, unless the specified terminator is enclosed in single quotes: *'EOF'*

In these labs, the most simple form of terminator is used (a single character). For example:

	./bin/ldapmodify -p 1189 -D "cn=directory manager" -w password -a <<+
	dn: uid=helpdesk,dc=example,dc=com
	changetype: modify
	add: ds-privilege-name
	ds-privilege-name: password-reset
	+

