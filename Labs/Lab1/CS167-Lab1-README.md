# Lab 1

## Student information
* Full name: David Ryan
* E-mail: dryan011@ucr.edu
* UCR NetID: dryan011
* Student ID: 862203891

## Answers

* (Q1) What is the name of the created directory?
dryan011_lab1

* (Q2) What do you see at the console output?
Hello World!

* (Q3) What do you see at the output?
I see an "ArrayIndexOutOfBoundsException" error on line 61, in the main function.

log4j:WARN No appenders could be found for logger (org.apache.hadoop.metrics2.lib.MutableMetricsFactory).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
Exception in thread "main" java.lang.ArrayIndexOutOfBoundsException: 0
	at edu.ucr.cs.cs167.dryan011.App.main(App.java:61)

* (Q4) What is the output that you see at the console?
but	1
cannot	3
crawl	1
do	1
fly,	1
forward	1
have	1
if	3
keep	1
moving	1
run	1
run,	1
then	3
to	1
walk	1
walk,	1
whatever	1
you	5

* (Q5) Does it run? Why or why not?
It does run, because it takes in the .txt files as proper parameters, which is why the program runs through Hadoop
