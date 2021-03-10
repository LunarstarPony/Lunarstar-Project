#
```
深入理解程序設計 使用Linux匯編語言
（美）巴特利特 人民郵電出版社  2014/01/01
https://www.books.com.tw/products/CN11078208?sloc=main

https://book.douban.com/subject/25789594/

https://github.com/foomur/programming-ground-up
```
```
Chapter 3 - Your first assembly program
exit.s - a first assembly program showing the functionality of the exit() syscall
maximum.s - finding the largest number in a data index
minimum.s - finding the smallest number in a data index


Chapter 4 - Functions
power.s - illustrating how functions work by solving 2^3 + 5^2
factorial.s - solving a factorial by creating a function
square.s - solving a square x^2 = ? by creating a function

Chapter 5 - Buffers & File Descriptors
toupper.s - using buffers and file descriptors to change all lowercase letters of a file to all uppercase letters


Chapter 6 - Reading and Writing Simple Records:
linux.s - common Linux definitions use .equ including syscalls, file descriptors, status codes, interrupts
record-def.s - record offsets
add-year.s - add a year to each record
count-chars.s - function to count the number of characters in a record
read-record.s - function to read a record
read-records.s - final program to read all the records
write-newline.s - function to write a newline to STDOUT
write-record.s - function to write in a record
write-records.s - final program to write all the records
```

# Chapter 3 - Your first assembly program
### exit.s - a first assembly program showing the functionality of the exit() syscall

```
#PURPOSE:  	Simple program that exits and returns a status code back to the Linux kernel

#INPUT:  	none

#OUTPUT: 	returns a status code.  This can be viewed by typing echo $? after 
#		running the program

#VARIABLES:
#		%eax holds the system call number
#		%ebx holds the return status
#

.section .data

.section .text
	.globl _start
_start:
	movl $1, %eax			# this is the linux kernel command
		 	            	# number (system call) for exiting a program
					
	movl $0, %ebx	 	   # this is the status number we will return to the operating
					           # system.  Change this around and it will return different
			       	        # things to echo $?

int $0x80				# this wakes up the kernel to run the exit command				
					
					
#TO RUN (from the command line):
#			            
#			as exit.s -o exit.o     	# Assemble the program
# 			ld exit.o -o exit		# Link the file
#			./exit				# Run the program
#			echo $?				# Should be 0, the exit status code	
```
```
as exit.c -o exit.o
ld exit.o -o exit
./exit
```
### maximum.s - finding the largest number in a data index

```
#PURPOSE: 		This program finds the maximum number of a set of data items

#INPUT:  		

#OUTPUT: 				

#VARIABLES:		The registers have the following uses:
#					%edi - Holds the index of the data item being examined	
#					%ebx - Largest data item found
#					%eax - Current data item
#
#			The following memory locations are used:
#					data_items - contains the item data.  A 0 is used to terminate the data


.section .data

data_items:				# These are the data items
.long 3,67,34,222,45,75,54,34,44,33,22,11,66,0

.section .text

.globl _start
_start:
	movl $0, %edi					# Move 0 into the index register
	movl data_items(,%edi,4), %eax			# Load the first byte of data
	movl %eax, %ebx					# Since this is the first item, %eax is the biggest
	
	start_loop:					# Starts a loop
		cmpl $0, %eax				# Checks if we are at the end
		je loop_exit
		incl %edi				# Load next value
		movl data_items(,%edi,4), %eax
		cmpl %ebx, %eax				# Compare values
		jle start_loop				# Go back to start of loop if new value isn't bigger
		movl %eax, %ebx				# Move the value as the largest
		jmp start_loop				# jump to loop beginning
	
	loop_exit:
		
		movl $1, %eax				# 1 is the exit() syscall
		int $0x80 				# Send to Linux kernel
					
					
#TO RUN (from the command line):
#			
#	as maximum.s -o maximum.o     	# Assemble the program
# 	ld maximum.o -o maximum		# Link the file
#	./maximum			# Run the program
#	echo $?				# Should return 222

```


### minimum.s - finding the smallest number in a data index
```
#PURPOSE: 		This program finds the minimum number of a set of data items

#INPUT:  		

#OUTPUT: 				

#VARIABLES:		The registers have the following uses:
#					%edi - Holds the index of the data item being examined	
#					%ebx - Largest data item found
#					%eax - Current data item
#
#			The following memory locations are used:
#					data_items - contains the item data.  A 0 is used to terminate the data


.section .data

data_items:				# These are the data items
.long 3,67,34,222,45,75,54,34,44,33,22,11,66,255

.section .text

.globl _start
_start:
	movl $0, %edi					# Move 0 into the index register
	movl data_items(,%edi,4), %eax			# Load the first byte of data
	movl %eax, %ebx					# Since this is the first item, %eax is the biggest
	
	start_loop:					# Starts a loop
		cmpl $255, %eax				# Checks if we are at the end
		je loop_exit
		incl %edi				# Load next value
		movl data_items(,%edi,4), %eax
		cmpl %ebx, %eax				# Compare values
		jge start_loop				# Go back to start of loop if new value isn't smaller
		movl %eax, %ebx				# Move the value as the smallest
		jmp start_loop				# jump to loop beginning
	
	loop_exit:
		
		movl $1, %eax				# 1 is the exit() syscall
		int $0x80 				# Send to Linux kernel
					
					
#TO RUN (from the command line):
#			
#	as minimum.s -o minimum.o     	# Assemble the program
# 	ld minimum.o -o minimum		# Link the file
#	./minimum			# Run the program
#	echo $?				# Should return 3
```
## Chapter 4 - Functions

### power.s - illustrating how functions work by solving 2^3 + 5^2
```
  
#PURPOSE: 		Program to illustrate how functions work
# 			This program will compute the value of 2^3 + 5^2
 				  
#			Everything in the main program is stored in registers,
#			so the data section doesn’t have anything.

.code32           
.section .data
.section .text
.globl _start
_start:
	pushl $3 			 # push second argument
	pushl $2 			 # push first argument
	call power 		         # call the function
	addl $8, %esp 	                 # move the stack pointer back
	pushl %eax 			 # save the first answer before
			                 # calling the next function
	pushl $2 		  	 # push second argument
	pushl $5 			 # push first argument
	call power 			 # call the function
	addl $8, %esp 	                 # move the stack pointer back
	popl %ebx 			 # The second answer is already
				         # in %eax. We saved the
					 # first answer onto the stack,
					 # so now we can just pop it
					 # out into %ebx
	addl %eax, %ebx	                 # add them together
					 # the result is in %ebx
	movl $1, %eax 	                 # exit (%ebx is returned)
int $0x80
	
#PURPOSE: 		This function is used to compute the value of a number raised to a power
 				   				
#INPUT: 	  	First argument - the base number
# 			Second argument - the power to raise it to
# 				   

#OUTPUT: 		Will give the result as a return value

#NOTES: 		The power must be 1 or greater

#VARIABLES:
# 			%ebx - holds the base number
# 			%ecx - holds the power
# 			-4(%ebp) - holds the current result
# 			%eax is used for temporary storage


.type power, @function
power:
	pushl %ebp 			  # save old base pointer
	movl %esp, %ebp 		  # make stack pointer the base pointer
	subl $4, %esp 			  # get room for our local storage
	movl 8(%ebp), %ebx 		  # put first argument in %eax
	movl 12(%ebp), %ecx 		  # put second argument in %ecx
	movl %ebx, -4(%ebp) 		  # store current result
	
power_loop_start:
	
	cmpl $1, %ecx 			  # if the power is 1, we are done
	je end_power
	movl -4(%ebp), %eax 		  # move the current result into %eax
	imull %ebx, %eax 		  # multiply the current result by
					  # the base number
	movl %eax, -4(%ebp) 	          # store the current result
	decl %ecx 		          # decrease the power
	jmp power_loop_start 	          # run for the next power
	
end_power:
	movl -4(%ebp), %eax 	          # return value goes in %eax
	movl %ebp, %esp 		  # restore the stack pointer
	popl %ebp 			  # restore the base pointer
ret



#    To run, from the command line (I'm using an x86-64 build so you may need to assemble differently)
#  		    	as --32 power.s -o power.o
#			ld -melf_i386 power.o -o power
#			./power
#			echo $?

```

### factorial.s - solving a factorial by creating a function
```
#PURPOSE -  Given a number, this program computes the factorial. For example, the factorial of
#           3 is 3 * 2 * 1, or 6. The factorial of 4 is 4 * 3 * 2 * 1, or 24, and so on.
#           This program shows how to call a function recursively.

.code32                             # you don't need this if you're not running x86-64
.section .data                      # this program has no global data
.section .text
.globl _start
.globl factorial                    # this is unneeded unless we want to share this function among other programs

_start:
  pushl $4                          # the factorial takes one argument - the number we want a factorial of. So, it gets pushed
  call factorial                    # run the factorial function
  addl $4, %esp                     # scrubs the parameter that was pushed on the stack
  movl %eax, %ebx                   #factorial returns the answer in %eax, but we want it in %ebx to send it as our exit status
  movl $1, %eax                     #call the kernel’s exit function
 int $0x80


#DEFINE THE FUNCTION:

.type factorial,@function

factorial:
  pushl %ebp                        # standard function stuff - we have to restore %ebp to its prior state before returning, 
                                    # so we have to push it
  movl %esp, %ebp                   # This is because we don’t want to modify the stack pointer, so we use %ebp.
  movl 8(%ebp), %eax                # This moves the first argument to %eax 4(%ebp) holds the return address, and
                                    # 8(%ebp) holds the first parameter
  cmpl $1, %eax                     # If the number is 1, that is our base case, and we simply return (1 is
                                    # already in %eax as the return value)
  je end_factorial
  decl %eax                         # otherwise, decrease the value
  pushl %eax                        # push it for our call to factorial
  call factorial 
  movl 8(%ebp), %ebx                # %eax has the return value, so we reload our parameter into %ebx
  imull %ebx, %eax                  # multiply that by the result of the last call to factorial (in %eax) the answer is stored in %eax, 
                                    # which is good since that’s where return values go
end_factorial:

  movl %ebp, %esp                   # standard function return stuff - we have to restore %ebp and %esp to where
  popl %ebp                         # they were before the function started
ret                                 # return to the function (this pops the return value, too)
```


### square.s - solving a square x^2 = ? by creating a function
```
#PURPOSE: 		This program will calculate x^2
# 				
# 				
#
#			Everything in the main program is stored in registers,
#			so the data section doesn’t have anything.
.code32
.section .data
.section .text
.globl _start
_start:
	pushl $8 		# the number you want to square, x=?, push second argument
	pushl $2 		# push first argument
	call square 		# call the function
	pushl %eax 		# save the first answer before
	popl %ebx
	movl $1, %eax		#exit(%ebx is returned)
int $0x80
	
#PURPOSE: 		This function is used to compute
# 			the value of a square, that is x^

#INPUT: 		First argument - the base number
# 			Second argument - the power to raise it to
# 				
#
#OUTPUT: 		Will give the result as a return value
#
#NOTES: 		The power must be 1 or greater
#
#VARIABLES:
# 				%ebx - holds the base number
# 				%ecx - holds the power
#
# 				-4(%ebp) - holds the current result
#
# 				%eax is used for temporary storage
#

.type square, @function
square:
	pushl %ebp 			#save old base pointer
	movl %esp, %ebp 		#make stack pointer the base pointer
	subl $4, %esp 			#get room for our local storage
	movl 8(%ebp), %ebx 		#put first argument in %eax
	movl 12(%ebp), %ecx 		#put second argument in %ecx
	movl %ebx, -4(%ebp) 		#store current result
	
square_loop_start:
	
	cmpl $1, %ecx 			#if the power is 1, we are done
	je end_square
	movl -4(%ebp), %eax 		#move the current result into %eax
	imull %ebx, %eax 		#multiply the current result by
					#the base number
	movl %eax, -4(%ebp) 		#store the current result
	decl %ecx 			#decrease the power
	jmp square_loop_start 		#run for the next power
	
end_square:
	movl -4(%ebp), %eax 		#return value goes in %eax
	movl %ebp, %esp 		#restore the stack pointer
	popl %ebp 			#restore the base pointer
ret



#  To run, from the command line
#  			as --32 square.s -o square.o
#			ld -melf_i386 square.o -o square
#			./square
#			echo $?
```
## Chapter 5 - Buffers & File Descriptors

### toupper.s - using buffers and file descriptors to change all lowercase letters of a file to all uppercase letters

```
#PURPOSE: This program converts an input file
#	  to an output file with all letters 
#	  converted to uppercase.
#
#Processing: 1) Open the input file
#	     2) Open the output file
#	     3) While we're not at the end of the input file
#		a) read part of file into our memory buffer
#		b) go through each byte of memory
#			if the byte is lowercase letter,
#			convert it to uppercase
#		c) write the memory buffer to output file
.code32
.section .data

########CONSTANTS########

#system call numbers
.equ SYS_OPEN, 5
.equ SYS_WRITE, 4
.equ SYS_READ, 3
.equ SYS_CLOSE, 6
.equ SYS_EXIT, 1

#options for open
.equ O_RDONLY, 0
.equ O_CREAT_WRONLY_TRUNC, 03101

#standard file descriptors
.equ STDIN, 0
.equ STDOUT, 1
.equ STDERR, 2

#system call interrupt
.equ LINUX_SYSCALL, 0x80

.equ END_OF_FILE, 0		#return value of read when end of file
.equ NUMBER_ARGUMENTS, 2


.section .bss
#Buffer - Data from input file loaded into, 
#	  converted to uppercase and written
#	  to output file
.equ BUFFER_SIZE, 500
.lcomm BUFFER_DATA, BUFFER_SIZE	#creates buffer called buffer_data
				#with size of 500 bytes.


.section .text

#STACK POSITIONS
.equ ST_SIZE_RESERVE, 8
.equ ST_FD_IN, -4
.equ ST_FD_OUT, -8
.equ ST_ARGC, 0			#number of arguments
.equ ST_ARGV_0, 4		#name of program
.equ ST_ARGV_1, 8		#input file name
.equ ST_ARGV_2, 12		#output file name

.globl _start
_start:

####Initialize program#####
#save stack pointer

movl %esp, %ebp

# allocate space for file descriptors on the stack
subl $ST_SIZE_RESERVE, %esp	#2 words

open_files:
open_fd_in:			#open input file
movl $SYS_OPEN, %eax		#movl 5 to %eax
movl ST_ARGV_1(%ebp), %ebx	#output filename int %ebx
movl $O_RDONLY, %ecx		#set flags
movl $0666, %edx		#mode for new file
int $LINUX_SYSCALL		#call linux

store_fd_in:
movl %eax, ST_FD_IN(%ebp)	#store file descriptor


open_fd_out:
movl $SYS_OPEN, %eax
movl ST_ARGV_2(%ebp), %ebx
movl $O_CREAT_WRONLY_TRUNC, %ecx
movl $0666, %edx
int $LINUX_SYSCALL

store_fd_out:
movl %eax, ST_FD_OUT(%ebp)	#store file descriptor

read_loop_begin:
movl $SYS_READ, %eax		#read in a block from input file
				#movl 3 %eax
movl ST_FD_IN(%ebp), %ebx	#get input file descriptor
movl $BUFFER_DATA, %ecx		#the location to read into
movl $BUFFER_SIZE, %edx		#the size of the buffer
int $LINUX_SYSCALL		#size of buffer read is returned to %eax

#exit if we have reached the end
cmpl $END_OF_FILE, %eax		#check for the end of file marker
				#%eax should be 0
jle end_loop			#if found or on error, go to end
				#found = equal
				#error = negative numbers

continue_read_loop:
#convert the block to uppercase
pushl $BUFFER_DATA		#store location of buffer
pushl %eax			#store size of buffer
call convert_to_upper		#our function that converts to uppercase
popl %eax			#get the size back
addl $4, %esp			#restore stack pointer

#write the block out to the output file
movl %eax, %edx			#buffer size
movl $SYS_WRITE, %eax		#set %eax to 4
movl ST_FD_OUT(%ebp), %ebx	#the filedescriptor of the output file
movl $BUFFER_DATA, %ecx		#location of the buffer
int $LINUX_SYSCALL		#call kernel

#Continue the loop
jmp read_loop_begin


end_loop:
#Close the files
movl $SYS_CLOSE, %eax
movl ST_FD_OUT(%ebp), %ebx	#file descriptor output file
int $LINUX_SYSCALL

movl $SYS_CLOSE, %eax		#we do it again since %eax could be
				#modified by our syscall
movl ST_FD_IN(%ebp), %ebx	#file desciptor input file
int $LINUX_SYSCALL

#exit
movl $SYS_EXIT, %eax		#set %eax to 1
movl $0, %ebx			#return 0 as success value
int $LINUX_SYSCALL



#PURPOSE: This function actually does the conversion 
#	  to uppercase for a block
#
#INPUT:	  First parameter is the location fo the block of
#	  memory to convert
#	  The second parameter is the length of that buffer
#
#OUTPUT:  This function overwrites the current buffer
#	  witht the upper-casified version
#
#VARIABLES:
#	  %eax - beginning of buffer
#	  %ebx - length of buffer
#	  %edi - current buffer offset
#	  %cl - current byte being examined (first part of %ecx)

#CONSTANTS
.equ LOWERCASE_A, 'a'		#lower boundary of our search
.equ LOWERCASE_Z, 'z'		#upper boundary of our search
.equ UPPER_CONVERSION, 'A' - 'a'#Conversion between upper
				#and lower case

#STACK STUFF
.equ ST_BUFFER_LEN, 8		#length of buffer
.equ ST_BUFFER, 12		#actual buffer

convert_to_upper:
pushl %ebp
movl %esp, %ebp

#SET UP VARIABLES
movl ST_BUFFER(%ebp), %eax
movl ST_BUFFER_LEN(%ebp), %ebx
movl $0, %edi

cmpl $0, %ebx			#check if the buffer was zero
je end_convert_loop		#if so, just leave

convert_loop:
movb (%eax,%edi,1), %cl		#get the byte and move it into %cli
				#byte = (beginning of buffer,
				#	 buffer offset - how for are we
				#	 word lenght 1
				#%cl lower end %exc

cmpb $LOWERCASE_A, %cl		#check if the byte is between a
jl next_byte			#and z
cmpb $LOWERCASE_Z, %cl		#if not, go the next byte
jg next_byte

addb $UPPER_CONVERSION, %cl	#otherwise, convert to uppercase
movb %cl, (%eax,%edi,1)		#store it back into buffer

next_byte:
incl %edi			#increment buffer offset with 1
cmpl %edi, %ebx			#continue until we have reached
				#the end
jne convert_loop


end_convert_loop:
movl %ebp, %esp			#no return value, just leave
popl %ebp
ret


# To run:  From the command line (NOTE:  I'm using an x86-64 build so you may need to assebmle differently)
# Takes the file toupper.s, changes all lowercase to uppercase and saves as a new file toupper.uppercase
#			as --32 toupper.s -o toupper.o
#			ld -melf_i386 toupper.o -o toupper
#			./toupper toupper.s toupper.uppercase     
```

## Chapter 6 - Reading and Writing Simple Records

### linux.s - common Linux definitions use .equ including syscalls, file descriptors, status codes, interrupts
```
#Common Linux Definitions

#system call numbers
.equ SYS_EXIT, 1
.equ SYS_READ, 3
.equ SYS_WRITE, 4
.equ SYS_OPEN, 5
.equ SYS_CLOSE, 6
.equ SYS_BRK, 45

#system call interrupt numbers
.equ LINUX_SYSCALL, 0x80

#standard file descriptors
.equ STDIN, 0
.equ STDOUT, 1
.equ STDERR, 2

#common status codes
.equ END_OF_File, 0
```

### record-def.s - record offsets
```
.equ RECORD_FIRSTNAME, 0
.equ RECORD_LASTNAME, 40
.equ RECORD_ADDRESS, 80
.equ RECORD_AGE, 320
.equ RECORD_SIZE, 324
```

### add-year.s - add a year to each record
```
.include "linux.s"
.include "record-def.s"
.code32
.section .data
input_file_name:
.ascii "test.dat\0"

output_file_name:
.ascii "testout.dat\0"

.section .bss
.lcomm record_buffer, RECORD_SIZE

#Stack offsets of local variables
.equ ST_INPUT_DESCRIPTOR, -4
.equ ST_OUTPUT_DESCRIPTOR, -8

.section .text
.globl _start
_start:
movl %esp, %ebp
subl $8, %ebp

#Open input file
movl $SYS_OPEN, %eax
movl $input_file_name, %ebx
movl $0, %ecx
movl $0666, %edx
int $LINUX_SYSCALL

movl %eax, ST_INPUT_DESCRIPTOR(%ebp)

#Error checking, neg numbers are error codes
cmpl $0, %eax
jg continue_processing

#Send the error
.section .data
no_open_file_code:
.ascii "0001: \0"
no_open_file_msg:
.ascii "Can't open input file\0"

.section .text
pushl $no_open_file_msg
pushl $no_open_file_code
call error_exit

continue_processing:


#Open file for writing
movl $SYS_OPEN, %eax
movl $output_file_name, %ebx
movl $0101, %ecx
movl $0666, %edx
int $LINUX_SYSCALL

movl %eax, ST_OUTPUT_DESCRIPTOR(%ebp)


loop_begin:
pushl ST_INPUT_DESCRIPTOR(%ebp)
pushl $record_buffer
call read_record
addl $8, %esp

#Returns the number of bytes read.
#Check for end of file or zero.
cmpl $RECORD_SIZE, %eax
jne loop_end

#Increment the age
incl record_buffer + RECORD_AGE

#Write the record out
pushl ST_OUTPUT_DESCRIPTOR(%ebp)
pushl $record_buffer
call write_record
addl $8, %esp

jmp loop_begin


loop_end:
movl $SYS_EXIT, %eax
movl $0, %ebx
int $LINUX_SYSCALL

# To run, from the command line (note:  I'm using x86-64)
#     as --32 add-year.s -o add-year.o
#     ld -melf_i386 add-year.o read-record.o write-record.o -o add-year
#     ./add-year
```

### count-chars.s - function to count the number of characters in a record

```
#PURPOSE: Count the characters until a null byte is reached.
#
#INPUT:   The address of the character string
#
#OUTPUT: Returns the count in %eax
#
#PROCESS:
#  Registers used:
#	%ecx - chararcter count
#	%al - current character
#	%edx - current character address
.code32
.type count_chars, @function
.globl count_chars

.equ ST_STRING_START_ADDRESS, 8
count_chars:
pushl %ebp
movl %esp, %ebp

movl $0, %ecx					#Counter starts at 0
movl ST_STRING_START_ADDRESS(%ebp), %edx

count_loop_begin:
movb (%edx), %al
cmpb $0, %al					#Current char zero?
je count_loop_end				#If yes, we're done
incl %ecx					#If no, increment counter
incl %edx					
jmp count_loop_begin				#Loop again

count_loop_end:
movl %ecx, %eax					#Return our value

popl %ebp
ret
```

### read-record.s - function to read a record

```
.include "record-def.s"
.include "linux.s"

#PURPOSE: This function reads a record from the file descriptor
#
#INPUT:	  The file descriptor and a buffer
#
#OUTPUT:  This function writes the data to the buffer 
#	  and returns a status code.
#
#STACK LOCAL VARIABLES
.equ ST_READ_BUFFER, 8
.equ ST_FILEDES, 12
.section .text
.globl read_record
.type read_record, @function

read_record:
pushl %ebp			#Save old base pointer
movl %esp, %ebp			#Reset the base pointer

pushl %ebx			#Save value of %ebx
movl ST_FILEDES(%ebp), %ebx
movl ST_READ_BUFFER(%ebp), %ecx
movl $RECORD_SIZE, %edx		#Defined in record-def.s which 
				#is included
movl $SYS_READ, %eax
int $LINUX_SYSCALL		#execute read command

popl %ebx			#%eax has the return value, which we will
				#give back to our calling programming

movl %ebp, %esp			#Move the stack pointer back
popl %ebp			#Restore previous base pointer
ret
```

### read-records.s - final program to read all the records
```
.include "linux.s"
.include "record-def.s"
.code32
.section .data
file_name:
.ascii "test.dat\0"

record_buffer_ptr:
.long 0


.section .text
.globl _start
_start:
.equ ST_INPUT_DESCRIPTOR, -4
.equ ST_OUTPUT_DESCRIPTOR, -8

call allocate_init				#Initalize memory manager
pushl $RECORD_SIZE
call allocate
movl %eax, record_buffer_ptr

movl %esp, %ebp
subl $8, %esp					#Make room for local vars

movl $SYS_OPEN, %eax				#Open the file
movl $file_name, %ebx
movl $0, %ecx					#Open read-only
movl $0666, %edx
int $LINUX_SYSCALL

movl %eax, ST_INPUT_DESCRIPTOR(%ebp)		#Save file descriptor

movl $STDOUT, ST_OUTPUT_DESCRIPTOR(%ebp)	#Save output file descriptor
						#makes it easy to change to diff file

record_read_loop:
pushl ST_INPUT_DESCRIPTOR(%ebp)
pushl record_buffer_ptr				#Pass the pointer to our memory
call read_record
addl $8, %esp

cmpl $RECORD_SIZE, %eax				#returns number of bytes read
jne finished_reading 				#if it's not the same number as requested
						#its and end of file or error, so quit
						#print out first name, first know its size
movl record_buffer_ptr, %eax
addl $RECORD_FIRSTNAME, %eax
pushl %eax
call count_chars
addl $4, %esp
movl %eax, %edx
movl ST_OUTPUT_DESCRIPTOR(%ebp), %ebx
movl $SYS_WRITE, %eax
movl record_buffer_ptr, %ecx
addl $RECORD_FIRSTNAME, %ecx
int $LINUX_SYSCALL

push ST_OUTPUT_DESCRIPTOR(%ebp)
call write_newline
addl $4, %esp

jmp record_read_loop

finished_reading:
pushl record_buffer_ptr
call deallocate
movl $SYS_EXIT, %eax
movl $0, %ebx
int $LINUX_SYSCALL


# To run: (NOTE: I'm using x86-64 so yours may be different)

# as --32 read-record.s -o read-record.o
# as --32 count-chars.s -o count-chars.o
# as --32 write-newline.s -o write-newline.o
# as --32 read-records.s -o read-records.o
# ld -melf_i386 read-record.o count-chars.o write-newline.o \
# read-records.o -o read-records
# ./read-records
```

### write-newline.s - function to write a newline to STDOUT

```
.include "linux.s"
.code32
.globl write_newline
.type write_newline,@function
.section .data
newline:
.ascii "\n"
.section .text
.equ ST_FILEDES, 8
write_newline:
pushl %ebp
movl %esp, %ebp
movl $SYS_WRITE, %eax
movl ST_FILEDES(%ebp), %ebx
movl $newline, %ecx
movl $1, %edx
int $LINUX_SYSCALL
movl %ebp, %esp
popl %ebp
ret
```

### write-record.s - function to write in a record
```
.include "linux.s"
.include "record-def.s"

#PURPOSE: This function writes a record to the given
#	  the given file descriptor
#
#INPUT:   The file descriptor and a buffer
#
#OUTPUT:  This function produces a status code
#
#STACK LOCAL VARIABLES
.equ ST_WRITE_BUFFER, 8
.equ ST_FILEDESCRIPTOR, 12
.section .text
.globl write_record
.type write_record,@function
write_record:
pushl %ebp				#Save base pointer	
movl %esp, %ebp				#Set new base pointer

pushl %ebx				#against c calling convention?
movl $SYS_WRITE, %eax
movl ST_FILEDESCRIPTOR(%ebp), %ebx
movl ST_WRITE_BUFFER(%ebp), %ecx
movl $RECORD_SIZE, %edx
int $LINUX_SYSCALL

popl %ebx

movl %ebp, %esp
popl %ebp
ret
```

### write-records.s - final program to write all the records
```
.include "linux.s"
.include "record-def.s"

.code32

.section .data

#Constant data of the records we want to write
#Each text data item is padded to the proper
#length with null (i.e. 0) bytes.

#.rept is used to pad each item. .rept tells
#the assembler to repeat the section between
#.rept and .endr the number of times specified.
#This is used in this program to add extra null
#characters at the end of each field to fill
#it up
record1:
.ascii "Frederick\0"
.rept 31				#Padding to 40 bytes
.byte 0
.endr

.ascii "Barlett\0"
.rept 31
.byte 0
.endr

.ascii "4242 S Prairie\nTulsa, OK 55555\0"
.rept 209
.byte 0
.endr

.long 45

record2:
.ascii "Marilyn\0"
.rept 32
.byte 0
.endr

.ascii "Taylor\0"
.rept 33
.byte 0
.endr

.ascii "2224 S Johannan St\nChicago, IL 12345\0"
.rept 203
.byte 0
.endr

.long 29

record3:
.ascii "Derrick\0"
.rept 32
.byte 0
.endr

.ascii "McIntire\0"
.rept 31
.byte 0
.endr

.ascii "500 W Oakland\nSan Diego, CA 54321\0"
.rept 206
.byte 0
.endr

.long 36

file_name:
.ascii "test.dat\0"			#The name of the file we will write to
.equ ST_FILE_DESCRIPTOR, -4
.globl _start
_start:
movl %esp, %ebp				#Copy the stack pointer to the
					#base pointer
subl $4, %esp				#Reserve space for file desciptor

movl $SYS_OPEN, %eax			#Open the file
movl $file_name, %ebx
movl $0101, %ecx			#Open if doesn't exit, open to write
movl $0666, %edx			#File permissions
int $LINUX_SYSCALL

movl %eax, ST_FILE_DESCRIPTOR(%ebp)	#Store file desciptor as local var

pushl ST_FILE_DESCRIPTOR(%ebp)		#Write first record
pushl $record1				#This is a pointer to the data
call write_record
addl $8, %esp

pushl ST_FILE_DESCRIPTOR(%ebp)		#Write second record
pushl $record2
call write_record
addl $8, %esp

pushl ST_FILE_DESCRIPTOR(%ebp)		#Write third record
pushl $record3
call write_record
addl $8, %esp

movl $SYS_CLOSE, %eax			#Close file descriptor
movl ST_FILE_DESCRIPTOR(%ebp), %ebx
int $LINUX_SYSCALL

#PURPUSE: This function writes a record to the given
#	  the given file descriptor
#
#INPUT:   The file descriptor and a buffer
#
#OUTPUT:  This function produces a status code
#
#STACK LOCAL VARIABLES
.equ ST_WRITE_BUFFER, 8
.equ ST_FILEDESCRIPTOR, 12
.section .text
.globl write_record
.type write_record,@function
write_record:
pushl %ebp				#Save base pointer	
movl %esp, %ebp				#Set new base pointer

pushl %ebx				#against c calling convention?
movl $SYS_WRITE, %eax
movl ST_FILEDESCRIPTOR(%ebp), %ebx
movl ST_WRITE_BUFFER(%ebp), %ecx
movl $RECORD_SIZE, %edx
int $LINUX_SYSCALL

popl %ebx

movl %ebp, %esp
popl %ebp
ret
#PURPOSE: This function reads a record from the file descriptor
#
#INPUT:	  The file descriptor and a buffer
#
#OUTPUT:  This function writes the data to the buffer 
#	  and returns a status code.
#
#STACK LOCAL VARIABLES
.equ ST_READ_BUFFER, 8
.equ ST_FILEDES, 12
.section .text
.globl read_record
.type read_record, @function

read_record:
pushl %ebp			#Save old base pointer
movl %esp, %ebp			#Reset the base pointer

pushl %ebx			#Save value of %ebx
movl ST_FILEDES(%ebp), %ebx
movl ST_READ_BUFFER(%ebp), %ecx
movl $RECORD_SIZE, %edx		#Defined in record-def.s which 
				#is included
movl $SYS_READ, %eax
int $LINUX_SYSCALL		#execute read command

popl %ebx			#%eax has the return value, which we will
				#give back to our calling programming

movl %ebp, %esp			#Move the stack pointer back
popl %ebp			#Restore previous base pointer
ret

movl $SYS_EXIT, %eax			#Exit the program
movl $0, %ebx
int $LINUX_SYSCALL


# To run:  From the command line (NOTE:  I'm using an x86-64 build so you may need to assebmle differently)
# Takes the file toupper.s, changes all lowercase to uppercase and saves as a new file toupper.uppercase
#			as --32 write-records.s -o write-records.o
#			as --32 write-record.s -o write-record.o
#			ld -melf_i386 write-record.o write-records.o -o write-records
#			./write-records
```