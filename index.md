## How To Setup Tor Browser


### For Windows

Navigate to the Tor Browser download page.

Download the Windows .exe file.

(Recommended) Verify the file's signature.

When the download is complete, double click the .exe file. Complete the installation wizard process.

### For macOS

Navigate to the Tor Browser download page.

Download the macOS .dmg file.

(Recommended) Verify the file's signature.

When the download is complete, double click the .dmg file. Complete the installation wizard process.

### For GNU/Linux

Navigate to the Tor Browser download page.

Download the GNU/Linux .tar.xz file.

(Recommended) Verify the file's signature.

Now follow either the graphical or the command-line method:

### Graphical method

When the download is complete, extract the archive using an archive manager.

You'll need to tell your GNU/Linux that you want the ability to execute shell scripts. Navigate to the newly extracted Tor Browser directory. Right click on start-tor-browser.desktop, open Properties or Preferences and change the permission to allow executing file as program. Double-click the icon to start up Tor Browser for the first time.

## Note: On Ubuntu and some other distros if you try to launch start-tor-browser.desktop a text file might open up. To change this behavior and launch Tor Browser instead, follow this:

Launch "Files" (GNOME Files/Nautilus)

Click on Preferences.

Navigate to the 'Behavior' Tab.

Select "Run them" or "Ask what to do" under "Executable Text Files".

If you choose the latter click on "Run" after launching the start-tor-browser.desktop file.

## Command-line method

When the download is complete, extract the archive with the command tar -xf [TB archive].

From inside the Tor Browser directory, you can launch Tor Browser by running:

‪./start-tor-browser.desktop

Note: If this command fails to run, you probably need to make the file executable. From within this directory run: chmod +x start-tor-browser.desktop

Some additional flags that can be used with start-tor-browser.desktop from the command-line:

### Flag	Description

‪--register-app	To register Tor Browser as a desktop application.

‪--verbose	To display Tor and Firefox output in the terminal.

‪--log [file]	To record Tor and Firefox output in file (default: tor-browser.log).

‪--detach	To detach from terminal and run Tor Browser in the background.

‪--unregister-app To unregister Tor Browser as a desktop application.

### Configuring Onion Services for Tor

Tor allows clients and relays to offer onion services. That is, you can offer a web server, SSH server, etc., without revealing your IP address to its users. In fact, because you don't use any public address, you can run an onion service from behind your firewall.

If you have Tor installed, you can see onion services in action by visiting this sample site.

This page describes the steps for setting up your own onion service website. For the technical details of how the onion service protocol works, see our onion service protocol page.

Step Zero: Get Tor working

Step One: Install a web server locally

Step Two: Configure your onion service

Step Three: More advanced tips

Step Four: Set up next-gen (v3) onions

Step Zero: Get Tor working

Before you start, you need to make sure:

Tor is up and running,

You actually set it up correctly.

Windows users should follow the Windows howto, OS X users should follow the OS X howto, and Linux/BSD/Unix users should follow the Unix howto.

Step One: Install a web server locally

First, you need to set up a web server locally, for example nginx or lighttpd (apache is not the best option for anomymity, see Step Three below). Setting up a web server can be complex. We're not going to cover how to set up a web server here. If you get stuck or want to do more, find a friend who can help you. We recommend you install a new separate web server for your onion service, since even if you already have one installed, you may be using it (or want to use it later) for a normal website.

You need to configure your web server so it doesn't give away any information about you, your computer, or your location. Be sure to bind the web server only to localhost (if people could get to it directly, they could confirm that your computer is the one offering the onion service). Be sure that its error messages don't list your hostname or other hints. Consider putting the web server in a sandbox or VM to limit the damage from code vulnerabilities.

Once your web server is set up, make sure it works: open your browser and go to http://localhost:8080/, where 8080 is the webserver port you chose during setup (you can choose any port, 8080 is just an example). Then try putting a file in the main html directory, and make sure it shows up when you access the site.

Step Two: Configure your onion service

Next, you need to configure your onion service to point to your local web server.

First, open your torrc file in your favorite text editor. (See the torrc FAQ entry to learn what this means.) Go to the middle section and look for the line

    ############### This section is just for location-hidden services ###

    

This section of the file consists of groups of lines, each representing one onion service. Right now they are all commented out (the lines start with #), so onion services are disabled. Each group of lines consists of one HiddenServiceDir line, and one or more HiddenServicePort lines:

HiddenServiceDir is a directory where Tor will store information about that onion service. In particular, Tor will create a file here named hostname which will tell you the onion URL. You don't need to add any files to this directory. Make sure this is not the same directory as the hidserv directory you created when setting up thttpd, as your HiddenServiceDir contains secret information!

HiddenServicePort lets you specify a virtual port (that is, what port people accessing the onion service will think they're using) and an IP address and port for redirecting connections to this virtual port.

Add the following lines to your torrc:

    HiddenServiceDir /Library/Tor/var/lib/tor/hidden_service/

    HiddenServicePort 80 127.0.0.1:8080

    

You're going to want to change the HiddenServiceDir line, so it points to an actual directory that is readable/writeable by the user that will be running Tor. The above line should work if you're using the OS X Tor package. On Unix, try "/home/username/hidden_service/" and fill in your own username in place of "username". On Windows you might pick:

 HiddenServiceDir C:\Users\username\Documents\tor\hidden_service

	HiddenServicePort 80 127.0.0.1:8080 

Note that since 0.2.6, both SocksPort and HiddenServicePort support Unix sockets. This means that you can point the HiddenServicePort to a Unix socket:

    HiddenServiceDir /Library/Tor/var/lib/tor/hidden_service/

    HiddenServicePort 80 unix:/path/to/socket

    

Now save the torrc and restart your tor.

If Tor starts up again, great. Otherwise, something is wrong. First look at your logfiles for hints. It will print some warnings or error messages. That should give you an idea what went wrong. Typically there are typos in the torrc or wrong directory permissions (See the logging FAQ entry if you don't know how to enable or find your log file.)

When Tor starts, it will automatically create the HiddenServiceDir that you specified (if necessary), and it will create two files there.

private_key

First, Tor will generate a new public/private keypair for your onion service. It is written into a file called "private_key". Don't share this key with others -- if you do they will be able to impersonate your onion service.

hostname

The other file Tor will create is called "hostname". This contains a short summary of your public key -- it will look something like duskgytldkxiuqc6.onion. This is the public name for your service, and you can tell it to people, publish it on websites, put it on business cards, etc.

If Tor runs as a different user than you, for example on OS X, Debian, or Red Hat, then you may need to become root to be able to view these files.

Now that you've restarted Tor, it is busy picking introduction points in the Tor network, and generating an onion service descriptor. This is a signed list of introduction points along with the service's full public key. It anonymously publishes this descriptor to the directory servers, and other people anonymously fetch it from the directory servers when they're trying to access your service.

Try it now: paste the contents of the hostname file into your web browser. If it works, you'll get the html page you set up in step one. If it doesn't work, look in your logs for some hints, and keep playing with it until it works.

Step Three: More advanced tips

If you plan to keep your service available for a long time, you might want to make a backup copy of the private_key file somewhere.

If you want to forward multiple virtual ports for a single onion service, just add more HiddenServicePort lines. If you want to run multiple onion services from the same Tor client, just add another HiddenServiceDir line. All the following HiddenServicePort lines refer to this HiddenServiceDir line, until you add another HiddenServiceDir line:

    HiddenServiceDir /usr/local/etc/tor/hidden_service/

    HiddenServicePort 80 127.0.0.1:8080

    HiddenServiceDir /usr/local/etc/tor/other_hidden_service/

    HiddenServicePort 6667 127.0.0.1:6667

    HiddenServicePort 22 127.0.0.1:22

    

To set up an onion service on Raspbian have a look at Alec Muffett's Enterprise Onion Toolkit.

Client Authorization

To set up Cookie Authentication for v2 services see the entries for the HidServAuth and HiddenServiceAuthorizeClient options in the manual. First add following line to the torrc file of your onion service:

    HiddenServiceAuthorizeClient [auth-type] [service-name]

    

Restart/reload tor and read the cookie from the hostname file of your onion service, for example in

/var/lib/tor/hidden_service_path/hostname.

To access it with a tor client add following line to torrc and (re)start/reload it:

    HidServAuth [onion-address] [auth-cookie] [service-name]

    

You are now able to browse to the onion service address.

To set up Client Authorization for v3 ("next-gen") services as specified in rend-spec-v3.txt for the tor service running the onion follow the instructions in Client Authorization. Note that to revoke clients you need to restart the tor service (see #28275). At the moment you need to create the keys yourself with a script (like these written in bash or rust).

To access it with a tor client make sure you have ClientOnionAuthDir set in torrc. In the <ClientOnionAuthDir> directory, create an .auth_private file for the onion service corresponding to this key (i.e. 'bob_onion.auth_private').

The contents of the <ClientOnionAuthDir>/<user>.auth_private file should look like:

    <56-char-onion-addr-without-.onion-part>:descriptor:x25519:BBBEAUAO3PIFAH7SBGBI6A2QFAZBXG2NVN7HMBXFCZENJVF6C5AQ

    

Then (re)start/reload it and you should be able to browse to the onion service address.

Operational security

Onion services operators need to practice proper operational security and system administration to maintain security. For some security suggestions please make sure you read over Riseup's "Tor Hidden (Onion) Services Best Practices" document. Also, here are some more anonymity issues you should keep in mind:

As mentioned above, be careful of letting your web server reveal identifying information about you, your computer, or your location. For example, readers can probably determine whether it's thttpd or Apache, and learn something about your operating system.

If your computer isn't online all the time, your onion service won't be either. This leaks information to an observant adversary.

It is generally a better idea to host onion services on a Tor client rather than a Tor relay, since relay uptime and other properties are publicly visible.

The longer an onion service is online, the higher the risk that its location is discovered. The most prominent attacks are building a profile of the onion service's availability and matching induced traffic patterns.

Another common issue is whether to use HTTPS on your relay or not. Have a look at this post on the Tor Blog to learn more about these issues.

You can use stem to automate the management of your onion services.

Finally, feel free to use the [tor-onions] mailing list to discuss the secure administration and operation of Tor onion services.

Step Four: Set up next-gen (v3) onions

Since Tor 0.3.2 and Tor Browser 7.5.a5 56-character long v3 onion addresses are supported and should be used instead. This newer version of onion services ("v3") features many improvements over the legacy system:

Better crypto (replaced SHA1/DH/RSA1024 with SHA3/ed25519/curve25519)

Improved directory protocol, leaking much less information to directory servers.

Improved directory protocol, with smaller surface for targeted attacks.

Better onion address security against impersonation.

More extensible introduction/rendezvous protocol.

A cleaner and more modular codebase.

For details see Why are v3 onions better?. You can identify a next-generation onion address by its length: they are 56 characters long, as in 4acth47i6kxnvkewtm6q7ib2s3ufpo5sqbsnzjpbi7utijcltosqemad.onion. The specification for next gen onion services can be found here.

How to setup your own prop224 service

It's easy! Just use your regular onion service torrc and add HiddenServiceVersion 3 in your onion service torrc block. ` Here is an example torrc designed for testing:

SocksPort auto

HiddenServiceDir /home/user/tmp/hsv3

HiddenServiceVersion 3

HiddenServicePort 6667 127.0.0.1:6667

    

Then your onion address is in /home/user/tmp/hsv3/hostname. To host both a v2 and a v3 service using two onion service torrc blocks:

HiddenServiceDir /home/user/tmp/hsv2

HiddenServicePort 6667 127.0.0.1:6667

HiddenServiceDir /home/user/tmp/hsv3

HiddenServiceVersion 3

HiddenServicePort 6668 127.0.0.1:6667

    

Please note that tor is strict about directory permissions and does not like to share its files. Make sure to restrict read and write access to the onion services directory before restarting tor. For most linux based systems

chmod 700 -R /var/lib/tor

should be intended.

To restart tor it's safer to not use SIGHUP directly (see bug #21818), but to check the validity of the config first. On Debian based systems the services management tool does this for you:

    service tor restart

    

How to help the next-gen onion development

Please let us know if you find any bugs! We are still in testing & development stage so things are very liquid and in active development. If you want to help with development, check out the list of open prop224 bugs.

For researchers our wiki page Onion Service Naming Systems could be of value. If you are more of the bug hunting type, please check our code and spec for errors and inaccuracies. We would be thrilled to know about them!

For debugging and to send us more helpful log files, turn on info logging:

SafeLogging 0

Log notice file /home/user/tmp/hs/hs.log

Log info file /home/user/tmp/hs/hsinfo.log
























You can use the [editor on GitHub](https://github.com/ProfessorRah/darkweb.engineer/edit/gh-pages/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [Basic writing and formatting syntax](https://docs.github.com/en/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/ProfessorRah/darkweb.engineer/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
