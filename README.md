
# **Stegosaurus-Lexical-Mimic-System**

	This system is designed to hide a message in plain text. The 
	system works by using the bits of a message you specify to 
	alter a fake message with key words and phrases. The process on
	how to encode and decode fake messages is entailed in this file
   
	NOTE: this technique is a form of steganography, this means that
	your messages aren't necessairly secure, but hidden so the fact
	that there is a message at all should not be known. Also note,
	the more messages sent, the easier it is to tell there is 
	some sort of hidden message, and it becomes easier to decode.



NOTE: The information here assumes the access of some sort of Linux based terminal to compile and run files. The system is usable elsewhere, but that information is not written here.

---
1.) -----------TOOLS---------------
---

	a.) parser

		This tool can take a witten fake message(with options) and split the message into the files necessary for encoding and decoding
		hidden messages. The system provided has an example message "template_full" which can be used as is. This tool allows one to make
		their own fake messages easily. The syntax rules must be followed strictly though, a slight error may result in an unreadable message.
		the syntax on how to write a fake message is defined below.

		the code for this is in parseFile.c, this code can be compiled by:

			gcc parseFile.c -o parser

		the code then can be called by 

			./parser x y z

		where:
			x - the full text of the fake message with options
			y - the file that the message with constants will be written to
			z - the file that the options will be written to

			Example: ./parser template_full template_constant template_options

	b.) countBits

		This tool is useful in checking to see if the message is being written correctly. This program can not tell if the syntax is correct
		but assumes it is. This will count how many bits of information can be encoded and also how many resulting characters the message can
		contain. ie: template_constant will allow up to 31 characters. To ensure that no garbage exists at the end of text, please make sure
		all messages have the correct number of bits (divisible by 7)

		the code for this is in countBits.c, this code can be compiled by

			gcc countBits.c -o countBits

		the code can be ran by

			./countBits x

		where:
			x - the file generated by parser with the constant text. (i.e. the "y" in the parser call)

			Example: ./countBits template_constant


	c.) stegosaurus
	
		This tool will encode your secret message! For details on how, check the code as the complexity is beyond the scope of this README.
		This program will take in parameters, and then ask for a message in the terminal. The bits of this message will determine what options
		are selected in the encoded file, this will generate a readable message that should be inconspicuous to the human eye.

		the code for this is in stegosaurus.c, this code can be compiled by

			g++ stegosaurus.cpp -o ./stegosaurus

		the code can be ran by

			./stegosaurus x y z

		where:
			x - the file that the fake message will be sent to 
			y - the file containing the constants in the fake message (i.e. the "y" in the parser call)
			z - the file containing the options for words in the fake message (i.e. the "z" in the parser call)

			Example: ./stegosaurus encoded template_constant template_options

	d.) suruasogets

		This tool will decode your secret message! For details on how, check the code as the complexity is beyond the scope of this README.
		This program will take in parameters, and then display the secret message in the terminal. The bits of this message are determined by the options
		that are written in the encoded file, this will generate the original encoded message specified in the stegosaurus program.

		the code for this is in stegosaurus.c, this code can be compiled by

			g++ suruasogets.cpp -o ./suruasogets

		the code can be ran by

			./suruasogets x y z

		where:
			x - the file that the fake message is in (i.e. the "x" in the stegosaurus call)
			y - the file containing the constants in the fake message (i.e. the "y" in the parser call)
			z - the file containing the options for words in the fake message (i.e. the "z" in the parser call)

			Example: ./suruasogets encoded template_constant template_options


---
2.) -----------MESSAGE-SYNTAX---------------
---

	The original concept for this came from spammimic, the idea behind this was to send
	a spam email with a hidden message. Why a spam email? To ensure no one would read it
	or think of it as extremely out of place other than the person the message is intended
	for. We since changed the idea to accompany any possible fake text, but the template
	and therefore the example, is a fake spam email. however, any text could be used to generate
	the encoding. That being said, if you plan on writing your own fake text the syntax neesd to 
	be specific and MUST take into consideration a few concepts.

	The general syntax for messages will look something like this:

	Consequently, my {colleagues*teammates*fellow administrators*associates} and {I*myself} {are*will be} {willing*able} to {transfer*send*wire*pay} the {total amount*full amount*entire sum*total sum} to your {account*fund} or subsequent disbursement

	The text is plain with delimiting  '{' and '}' to designate options. The options are seperated by a single *. Because of this, neither '{' nor '}' can be in the fake message AT ALL. The number of options
	must be some power of 2 since binary is base 2. this means you could have 2 options with 1 *, 4 options with 3 *, 8 options with 7*.... etc. 

	This message will then be broken down by the parser into

	Consequently, my %4 and %2 %2 %2 to %4 the %4 to your %2 or subsequent disbursement

	and 

	{colleagues*teammates*fellow administrators*associates}
	{I*myself}
	{are*will be}
	{willing*able}
	{transfer*send*wire*pay}
	{total amount*full amount*entire sum*total sum}
	{account*fund}

	Notice how the number after a % corresponds with the number of options? This means that an option statement can not be followed immediately by a number. So {1*2*3*4}00 will not work, however
	10{1*2*3*4} will as a number does not follow the '}'. There are some interesting implementations
	in the template_full so I encourage you to look at that for more information.

	Lastly, in decoding the message, we need to consider the length of the option when searching the
	fake messaeg. Because of this there is a problem with options that are shortend versions of 
	eachother. For example {help*help me} will not work. If option 2 were selected, help would still
	be found, confirming that option 1 was the correct solution when it was not. HOWEVER,
	{help me*help} is allowed. since "help me" was checked first, we only check for "help" if the message
	is not "help me". This also means that "{help me*help} me" would be an error as the text would still
	read "help me" at the exact spot that it should. However redundent wording like that should never
	truly happen unless it is seperated by a ',' so it shouldn't matter. There may be other issues in
	the writing, but these are the major ones we have identified.

	TLDR;
	Rules:
	1. No '{' or '}' are allowed in the message at all 
	2. '*' can not be used in an option, but may be used in the constant part of the text
	3. '%' can not be used in the constant text, but may be in an option
	4. No shorter versions of options are allowed to come before longer versions with a set of options
	5. The number of options in each set must be a power of 2: 2,4,8... 
	6. ASCII characters only. Only 7 bits are considered.
	7(ish). While this isn't a rule, if the total number of bits is not divisible by 7, the last
		character of information will be cut short. To avoid garbage, make sure ./countBits confirms
		that the text has the right number of bits.


---
3.) ---------OTHER-NOTES----------------- 
---
	The stegosaurus system will accept messages longer than the text can accommodate.
	However, they are cut short. For example if you can only use 5 characters 123456 
	is shortened to 12345.

	This system is not robust in the slightest. There is no error checking for files
	that are not in the correct format, or if they even exist. Be careful when using
	this program, or it may crash if things are written improperly or if files that
	need to be read from do not exist(If the file specified to write to does not exist
	it will be created).

	While this system may be updated at some point the future, it is not guaranteed.
	Support for this is not planned.


Authors:
	Austin Gordon
	Zach Patterson

Date: 10/6/2017 