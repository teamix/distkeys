Distkeys
========
Distkeys distributes a list of SSH public keys to a list of servers. It
reaches servers behind a firewall as well.

Furthermore it executes a command or a script on a list of servers.

Distkeys is a ruby script which requires:

* Ruby ;)
* Net::SSH v2, debian package ruby-net-ssh
  (Squeeze and older: libnet-ssh2-ruby)
* Net::SSH::Gateway, debian package ruby-net-ssh-gateway
  (Squeeze and older: libnet-ssh-gateway-ruby)
* Net::SFTP v2, debian package ruby-net-sftp
  (Squeeze and older: libnet-sftp2-ruby)
* Termios, debian package ruby-termios
  (Squeeze and older: libtermios-ruby)
* SCP

Distkeys can also use SFTP, when you remove the occurences of
":via => :scp". This is not configurable via option yet.


SSH Configuration
-----------------

Using the `-F' command line option, you may specify an alternative
per-user SSH configuration file which will be used instead of the default
"~/.ssh/config" (see ssh(1)'s `-F' command line option for details).

For example, given the following "~/.ssh/multikeys.config":

  Host *
      ForwardAgent yes
      Compression yes
      StrictHostKeyChecking no
      IdentityFile ~/.ssh/tmx_rsa

Then, "multikeys.rb -F ~/.ssh/multikeys.config -h <hostlist> <action>"
will cause multikeys to enable agent forwarding and compression, disable
strict host key checking and using ~/.ssh/tmx_rsa as SSH identity file.

See the ssh_config(5) manpage for details about available configuration
options.


Known Issues
------------

### Can't add new key into hash with Ruby 1.9.3 and ruby-net-ssh 1:2.2.1-1
With Ruby 1.9.3 and Ruby Net SSH 2.2.1 the following error message
appears on uploading keys:

	Creating a backup to .ssh/authorized_keys-2012-05-16.bak if not already done today...
	Uploading keys to .ssh/authorized_keys-new...
	File does exist and has correct size, moving to .ssh/authorized_keys...
	/usr/lib/ruby/vendor_ruby/net/ssh/connection/session.rb:296:in `[]=': can't add a new key into hash during iteration (RuntimeError)
        	from /usr/lib/ruby/vendor_ruby/net/ssh/connection/session.rb:296:in `open_channel'
        	from /usr/lib/ruby/vendor_ruby/net/ssh/connection/session.rb:320:in `exec'
        	from /usr/lib/ruby/vendor_ruby/net/ssh/connection/session.rb:354:in `exec!'
        	from ./multikeys.rb:197:in `block in commit'
        	from /usr/lib/ruby/vendor_ruby/net/sftp/request.rb:87:in `call'
        	from /usr/lib/ruby/vendor_ruby/net/sftp/request.rb:87:in `respond_to'
        	from /usr/lib/ruby/vendor_ruby/net/sftp/session.rb:948:in `dispatch_request'
        	from /usr/lib/ruby/vendor_ruby/net/sftp/session.rb:911:in `when_channel_polled'
        	from /usr/lib/ruby/vendor_ruby/net/ssh/connection/channel.rb:311:in `call'
        	from /usr/lib/ruby/vendor_ruby/net/ssh/connection/channel.rb:311:in `process'
        	from /usr/lib/ruby/vendor_ruby/net/ssh/connection/session.rb:214:in `block in preprocess'
        	from /usr/lib/ruby/vendor_ruby/net/ssh/connection/session.rb:214:in `each'
        	from /usr/lib/ruby/vendor_ruby/net/ssh/connection/session.rb:214:in `preprocess'
        	from /usr/lib/ruby/vendor_ruby/net/ssh/connection/session.rb:197:in `process'
        	from /usr/lib/ruby/vendor_ruby/net/ssh/connection/session.rb:161:in `block in loop'
        	from /usr/lib/ruby/vendor_ruby/net/ssh/connection/session.rb:161:in `loop'
        	from /usr/lib/ruby/vendor_ruby/net/ssh/connection/session.rb:161:in `loop'
        	from /usr/lib/ruby/vendor_ruby/net/sftp/session.rb:802:in `loop'
        	from ./multikeys.rb:204:in `commit'
        	from ./multikeys.rb:593:in `handle_host'
        	from ./multikeys.rb:668:in `block in handle_gwhost'
        	from ./multikeys.rb:651:in `each'
        	from ./multikeys.rb:651:in `handle_gwhost'
        	from ./multikeys.rb:683:in `loop'
        	from ./multikeys.rb:771:in `<main>'

Workaround: Use Ruby 1.8 as

	ruby1.8 multikeys.rb …


### Uninitialized contant Net::SFTP::Session:StringIO in Net::STFP v2 < 2.0.5
There is an error in Net::SFTP v2 prior to 2.0.5 which needs to be added
manually:

If you get:

/usr/[...]/lib/net/sftp/session.rb:123:in `download!': uninitialized constant Net::SFTP::Session::StringIO (NameError) 

add

require 'stringio'

to the top of the session.rb file mentioned in the exception message.

For further information about this error, see:

http://toblog.bryans.org/2010/08/19/ruby-net-sftp-uninitialized-constant
