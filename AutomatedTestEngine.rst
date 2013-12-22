================================================================================
Automated Test Engine
================================================================================

.. contents::

What is it?
================================================================================
MarkUs Automated Test Engine (ATE) is a major new feature of MarkUs, currently
under development. As a part of the MarkUs system, ATE is a testing framework
that test students’ submission files using test scripts written by the
instructor, then collect and analyze the results before putting into the
rubrics.

How it works
================================================================================
An instructor uploads the test scripts and test support files using the config
UI, then MarkUs saves the file in the file system and all the options in the
database. When a student / grader / instructor request to run a test, test
launcher is invoked. The test launcher copies the student files, test scripts
and test support files to the designated location, where test runner is
(probably on another machine). It then calls the test runner with a list of
names of the test scripts. The test runner execute the list of test scripts,
concatenate the XML formatted output return from the test scripts, and return to
MarkUs. The parser takes the output and parse the result into useful information
and save them into the database. Finally, the test result will be displayed to
the student and grader when the test is finished. The level of details the user
can see depends on the options in the test scripts as well as the role of the
user (student, TA or admin).

MarkUs ATE Installation/Setup Guide
================================================================================
This guide assumes that you have a working MarkUs development environment on
your machine. Currently, MarkUs ATE is not in the release version of MarkUs,
because it is still under development. Thus, if you want to use the automated
test engine and work on developing MakrUs ATE, there are more setup steps to
perform.

Checkout ATE Branch
--------------------------------------------------------------------------------
On MarkUs upstream, ATE is currently not located in the master branch. Because
this is a new major feature which will be available in a future release, we do
not want it to mess up with the current release version. Hence, all the ATE
development is done on another branch on MarkUs upstream. This branch is called
test-framework.

Run the following commands in the MarkUs root directory in the terminal ::

    git checkout test-framework

ATE Database Migration
--------------------------------------------------------------------------------
Because the database schema is updated in the ATE branch, the database needs to
be updated to reflect the changes.

If you have not yet created much data in your database, it is recommended to
drop all the MarkUs databases and re-create them. In the Markus root directory,
execute ::

    bundle exec rake db:drop:all
    bundle exec rake db:create:all
    bundle exec rake db:schema:load

You may also want to re-seed the database and perform unit and function tests on
this branch too. ::

    bundle exec rake db:seed
    bundle exec rake db:test:prepare
    bundle exec rake test:units
    bundle exec rake test:functionals

Another option is to migrate the database schema, so that the current data is
not erased. However, migration sometimes fails.

Setup Redis-Server and Resque (gems)
--------------------------------------------------------------------------------
After you pull the source code to your local working directory, you need to
reinstall the gems as there are a few new gems required by MarkUs ATE. Execute::

    bundle install

in the terminal. “libxml-ruby” for XML parsing and “resque” for background jobs
are installed.

Also, Redis server is required for Resque to run. Go to http://redis.io/download
to download the latest stable version and follow the installation guide to
install Redis. After that you can start Redis-server from anywhere by
executing::

    redis-server

as it is added to $PATH.

Setup Test Server
--------------------------------------------------------------------------------
MarkUs ATE assumes all the tests are done on another machine (test server). In
development, it can be another account on the local machine. When a test is
requested, the test launcher creates a secure shell (SSH) connection to the test
server, securely copy all the files over and invoke the test runner. However,
SSH connection is protected using password, but we do not want to enter password
every time MarkUs sends test request. Hence, we need to skip authentication by
adding a public key to the machine running MarkUs.

Assume that MarkUs is running on the local-host (for example,
markus@cs.university.ca), and the test server is running on remote-host (for
example, test@cs.university.ca). When GitHub is set up, an SSH key is generated
and stores in /home/.ssh/id_rsa.pub . Copy the public key to remote-host using
ssh copy-id. Execute in the home directory of the local-host::

    ssh-copy-id -i ~/.ssh/id_rsa.pub remote-host

You need to enter the password of remote-host. And then test the connection::

    ssh remote-host

If you can successfully connect with remote-host without using a password, then
you are good.

You can set up more than one test servers by repeating the above steps for each
remote-host. You can use the same public key for all the test servers.

Configuration Settings
--------------------------------------------------------------------------------
The next step is to fill in the configuration settings in MarkUs. Using any text
editor, open the three files located in /MARKUS-ROOT/config/environments/. These
three files contain some configuration settings of MarkUs in development, test,
or production environments. If you are a MarkUs developer, you need to change
the settings in development.rb and test.rb, and maybe consider changing in
production.rb too.

Look for the “Automated Testing Engine settings” in the file. There are six
values in this section, and the documentation in the file should explain well
what each of them does. Change the values to what you desire.

Test Runner
--------------------------------------------------------------------------------
The test runner is a (ruby) script, provided as part of MarkUs, that runs one or
more test scripts in a protected/secure environment, and outputs XML formatted
content which can be parsed by MarkUs. Note that the test runner can be used
without MarkUs.

The source code of test runner is located at ::

    MARKUS-ROOT/automated-tests-files/test_runner/testrunner.rb

Copy this file to the designated location on the test server, specified in the
configuration settings.

Writing Test Scripts
--------------------------------------------------------------------------------
A test script, or test suite, is simply a file uploaded by the course instructor
to test the student-submitted code.

A test script can consist of any number of individual tests. Each of these tests
can be a single unit test, or a collection of unit tests.

Each test must contain the following information in the results (REQUIRED):

* The test name
* The numerical mark that the student earned for the test
* The total number of marks it is possible to earn for the test

Additionally, each test can contain the following additional information
(OPTIONAL):

* Test description
* Test feedback. This field can be used by the instructor to provide any 
  comments/feedback to the student (why the test failed, exception stack trace, 
  etc).

The information that is returned must use the following XML formatting. If the
formatting is not followed, then the parser will be unable to parse the results.

The test must return a string with the format <test> … </test>, where the
ellipsis represents the information to be returned.

The test name must have the format <name>n</name>, where n is any string.

The student mark must have the format <marksearned>i</marksearned> where i is an
integer.

The total marks available must have the format
<marksavailable>i</marksavailable> where i is an integer.

As stated, the following information is optional, and can be omitted entirely.

The test description must have the format <description>s</description> where s
is a string.

The test feedback must have the format <feedback>s</feedback> where s is a
string.

Once all of the tests have finished running, all <test>...</test> strings must
be printed to stdout. These can be printed as a single string containing all of
the test data, or as a number of smaller strings.

Finally, all test scripts must start with the line #!/usr/bin/env p, where p is
the program that would be used to execute the script (ruby, python, sh, scheme,
etc.). At the moment, the test runner does not support test scripts written in
languages that must be compiled.

Sample test scripts that can be used as a template are located in
MARKUS-ROOT/automated-tests-files/test_runner/sample_data

Test Supports and Helpers
--------------------------------------------------------------------------------
In addition to test scripts, there are also test supports and test helpers. 

Test supports are files that will be available to all tests run on the 
assignment. They are placed into the directory where each script is run. Test 
supports are good for code that is shared between tests (i.e. custom 
libraries), or common resources (files used for testing input/reading in
multiple tests).

Test helpers are similar to supports, but are instead associated with a 
specific test. They are only placed into the folder of the test script that
they have been added to. While the test runner doesn't support test scripts 
written in compiled languages, you can use an executable test script to compile
test helpers as a work-around.

Starting Redis-Server and Resque Workers
--------------------------------------------------------------------------------
In development, if you want to start MarkUs, you just need to start the rails
server in a terminal. However, because MarkUs ATE requires Resque to support
background jobs, and Resque runs on Redis server, you need to start them too.

Open a new terminal, and start Redis-server ::

    redis-server

Open a new terminal, navigate to MarkUs root directory, and start Resque worker
::

     VVERBOSE=1 QUEUE=test_waiting_list rake environment resque:work

You can start more than one worker, but you need a new terminal for each worker.
Also note that the number of workers running should equal to the number of
test servers multiplied by the maximum number of running tests on a server.

Finally, in a new terminal, you can start rails server to start MarkUs. ::

    bundle exec rails server

Launching Tests
--------------------------------------------------------------------------------
By pressing the “Run Test” button in the student UI or admin / grader UI, a test
request is send to Resque’s waiting list. It is then sent to the test server and
the test runner runs the test (providing that there are test scripts uploaded).
The result is returned to MarkUs and display to the user.
