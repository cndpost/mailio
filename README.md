Copyrght (c) 2019. John Xu - forked since 5/16/2019.

Build instructions: 
 
   0. get the latest source from the original master repo of mailio or from my following fork location: 

             git clone https://github.com/cndpost/mailio

   1. do not use the CMake-GUI which will fail to produce the initial vs project files and solution file. Use the command line version of cmake

             cd to-path-of-mailio
             mkdir build
             cd to cmake installation path
             cmake -S path-of-mailio  -B path-of-mailio\build

      then the Visual Studio project files and solution file will be generated on the folder of path-of-mailio\build

      The default cmake will produce a link to 32 bit, debug version of boost library. If you want
      to make 64 bit version, or build release version, you need to change the boost library name accordingly,

   2. check out and build boost library, and copy boost\boost\include to c:\boost\include and copy boost\stage\lib to c:\boost\lib
      the command bootstrap.bat and .\b2.exe will build both 32 bit version and 64 bit version of the libraries. Just need to link to
      32 bit version if you compiled mailio in 32 bit, link to 64 bit version if you compiled mailio in 64 bit version
   
   3. check out and build openssl library, and copy openssl\include to c:\openssl\include, and copy openssl\lib to c:\openssl\lib if the
      build is for 32 bit. copy openssl\lib to c:\openssl\lib64 if the build is for 64 bit.
      see the Readme in https://github.com/cndpost/openssl/readme for the instruction of building openssl for windows. 

      Basically following command will build the 32 bit version:

           perl Configure no-asm VC-WIN32
           nmake

      following command will build the 64 bit version:

           perl Configure no-asm VC-WIN64A

   4. Open the mailio.sln generated in above step 1, and manually add include folder c:\OpenSSL\include and C:\Boost to all the projects which the cmake did not
      include them. And manually add c:\openssl\lib\libcrypto_static.lib and c:\openssl\lib\libssl_static.lib as the additional libraries for all the projects, assuming
      we are building 32 bit. To build for 64 bit, link to C:\openssl\lib64\libcrypto_static.lib and C:\openssl\lib\lib64\libssl_static.lib instead.
      
   5. Then the build should pass. If you checked the mailio from my fork above, it already has the working project files and solution file. Above step 4 is needed
      only if you checked out the repo from the original owner's repo. 

===================Original Readme ============================================


# mailio #

mailio is a cross platform C++ library for MIME format and SMTP, POP3 and IMAP protocols. It is based on standard C++ 11 and Boost library.


# Examples #

To send a mail, one has to create `message` object and set it's attributes as author, recipient, subject and so on. Then, an SMTP connection
is created by constructing `smtp` (or `smtps`) class. The message is sent over the connection:

```cpp
message msg;
msg.from(mail_address("mailio library", "mailio@gmail.com"));
msg.add_recipient(mail_address("mailio library", "mailio@gmail.com"));
msg.subject("smtps simple message");
msg.content("Hello, World!");

smtps conn("smtp.gmail.com", 587);
conn.authenticate("mailio@gmail.com", "mailiopass", smtps::auth_method_t::START_TLS);
conn.submit(msg);
```
    
To receive a mail, `message` object is created to store the received message. Mail can be received over POP3 or IMAP, depending of mail server setup.
If POP3 is used, then instance of `pop3` (or `pop3s`) class is created and message is fetched:

```cpp
pop3s conn("pop.mail.yahoo.com", 995);
conn.authenticate("mailio@yahoo.com", "mailiopass", pop3s::auth_method_t::LOGIN);
message msg;
conn.fetch(1, msg);
```

Receiving a message over IMAP is analogous. Since IMAP recognizes folders, then one has to be specified, like *inbox*:

```cpp
imaps conn("imap.gmail.com", 993);
conn.authenticate("mailio@gmail.com", "mailiopass", imap::auth_method_t::LOGIN);
message msg;
conn.fetch("inbox", 1, msg);
```

More advanced features are shown in `examples` directory, see below how to compile them.

Note for Gmail users: if 2FA is turned on, then instead of the primary password, the application password must be used. Follow
[Gmail instructions](https://support.google.com/accounts/answer/185833) to add *mailio* as trusted application and use the generated password for all three
protocols.

Note for Zoho users: if 2FA is turned on, then instead of the primary password, the application password must be used. Follow
[Zoho instructions](https://www.zoho.com/mail/help/adminconsole/two-factor-authentication.html#alink5) to add *mailio* as trusted application and use the
generated password for all three protocols.


# Requirements #

Mailio library is supposed to work on all platforms supporting C++ 11 compiler, recent Boost libraries and CMake build tool.

For Linux the following configuration is tested:

* gcc 7.3.0.
* Boost 1.66 with Regex, Date Time available.
* POSIX Threads, OpenSSL and Crypto libraries available on the system.
* CMake 3.11

For MacOS the following configuration is tested:

* Apple LLVM 9.0.0.
* Boost 1.66.
* OpenSSL 1.0.2n available on the system.
* CMake 3.10.

For Microsoft Windows the following configuration is tested:

* Windows 10.
* Visual Studio 2015 Community Edition.
* Boost 1.66.
* OpenSSL 1.0.2n available on the system.
* CMake 3.11.


# Setup #

Ensure that OpenSSL, Boost and CMake are in the path. If they are not in the path, one could set environment variables `OPENSSL_ROOT_DIR` and `BOOST_ROOT` to their
respective paths. Boost must be built with OpenSSL support. If it cannot be found in the path, set the path explicitly via `library-path` and `include`
parameters of `b2` script (after `bootstrap` finishes).


## Linux, MacOS ##

From the terminal go into the directory where the library is downloaded to, and execute:
```
mkdir build
cd ./build
cmake ..
make
```
Both static and dynamic libraries should be built in the `build` directory. If one wants to specify non-default installation directory say `/opt/mailio`, then
the last two steps should be:
```
cmake -DCMAKE_INSTALL_PREFIX=/opt/mailio ..
make install
```
Other available options are `MAILIO_BUILD_SHARED_LIBRARY` (by default is on, if turned off then the static library is built), `MAILIO_BUILD_DOCUMENTATION`
(if Doxygen documentation is generated, by default is on) and `MAILIO_BUILD_EXAMPLES` (if examples are built, by default is on).

## Microsoft Windows ##

From the command prompt go into the directory where the library is downloaded, and execute:
```
mkdir build
cd .\build
cmake ..
```
A solution file will be built, open it from Visual Studio and build the project.


# Features #

* Recursive formatter and parser of the MIME message.
* MIME message recognizes the most common headers like subject, recipients, content type and so on.
* Encodings that are supported for the message body: Seven bit, Eight bit, Binary, Base64 and Quoted Printable.
* Subject, attachment and name part of the mail address can be in ASCII or UTF-8 format.
* All media types are recognized, including MIME message embedded within another message.
* MIME message has configurable line length policy and strict mode for parsing.
* SMTP implementation with message sending. Both plain and SSL (including START TLS) versions are available.
* POP3 implementation with message receiving and removal, getting mailbox statistics. Both plain and SSL (including START TLS) versions are available.
* IMAP implementation with message receiving and removal, getting mailbox statistics. Both plain and SSL (including START TLS) versions are available.


# Issues #

The library is tested on valid mail servers, so probably there are negative test scenarios that are not covered by the code. In case you find one, please
contact me. Here is a list of issues known so far and planned to be fixed in the future.

* Non-ASCII subject is assumed to be UTF-8.
* Non-ASCII attachment name is assumed to be UTF-8.
* Header attribute cannot contain space between name and value.
* SSL certificate is not verified.


# Roadmap #

* version 0.8.0 (Q4/2016): More advanced test scenarios, more examples.
* version 0.9.0 (Q4/2016): Binary content transfer encoding, strict mode for codec parsers.
* version 0.10.0 (Q2/2017): Strict mode for message parser.
* version 0.11.0 (Q2/2017): Q codec to support non-ASCII message subjects.
* version 0.12.0 (Q3/2017): Non-ASCII name part of a mail address.
* version 0.13.0 (Q4/2017): UTF-8 filename of an attachment.
* version 0.14.0 (Q4/2017): Mail content is sent with attachments as another MIME part.
* version 0.15.0 (Q1/2018): Line policy applied to the header. Clang/MacOS build support in Makefile.
* version 0.16.0 (Q2/2018): Cmake scripts tested on Linux, MacOS and Windows. Sender header added.
* version 0.17.0 (Q3/2018): Timeouts for I/O operations. Fetching only message header with IMAP.
* version 0.18.0 (Q1/2019): Fixes of IMAP parser. Managing folders in IMAP.


# Contributors #

* [Trevor Mellon](https://github.com/TrevorMellon): CMake build scripts.
* [Kira Backes](mailto:kira.backes[at]nrwsoft.de): Fix for correct default message date.
* [sledgehammer_999](mailto:hammered999[at]gmail.com): Replacement of Boost random function with the standard one.
* [Paul Tsouchlos](mailto:developer.paul.123[at]gmail.com): Modernizing build scripts.
* [Anton Zhvakin](mailto:a.zhvakin[at]galament.com): Replacement of deprecated Boost Asio entities.


# Contact #

In case you find a bug, please drop me a mail to contact (at) alepho.com. Since this is my side project, I'll do my best to be responsive.
