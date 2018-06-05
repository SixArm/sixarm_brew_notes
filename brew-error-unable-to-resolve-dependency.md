# brew error: unable to resolve dependency for rubocop-cask and did_you_mean


If you get this error:

	$ brew cask style
	==> Installing or updating 'rubocop-cask' gem
	Error: Unable to resolve dependency: user requested 'did_you_mean (= 1.0.0)'
	Follow the instructions here:
	  https://github.com/Homebrew/homebrew-cask#reporting-bugs
	/Library/Ruby/Site/2.3.0/rubygems/resolver.rb:231:in `search_for'

Do you have Ruby?

    $ ruby --version
    ruby 2.5.1p57 (2018-03-29 revision 63029) [x86_64-darwin17]

Do you have multiple Ruby programs?

	$ which -a ruby
	/usr/local/bin/ruby
	/usr/bin/ruby

What are the versions?

    $ /usr/local/bin/ruby --version
    ruby 2.5.1p57 (2018-03-29 revision 63029) [x86_64-darwin17]

    $ /usr/bin/ruby --version
    ruby 2.3.3p222 (2016-11-21 revision 56859) [universal.x86_64-darwin17]

Do you have rubocop?

	$ gem list --exact rubocop

	*** LOCAL GEMS ***

	rubocop (0.56.0)

Do you have `rubocop-cask`?

	$ gem list --exact rubocop-cask

	*** LOCAL GEMS ***

	rubocop-cask (0.19.0)

Do you have `did_you_mean`?

	$ gem list --exact did_you_mean

	*** LOCAL GEMS ***

	did_you_mean (1.0.0)

Run doctor and fix anything:

    $ brew doctor
	Warning: You have unlinked kegs in your Cellar
	Leaving kegs unlinked can lead to build-trouble and cause brews that depend on
	those kegs to fail to run properly once built. Run `brew link` on these:
  	  ruby

    $ brew link --overwrite --force ruby
    Linking /usr/local/Cellar/ruby/2.5.1... 3288 symlinks created

Try again:

	$ brew cask style

The error message shows relevant lines when Hombrew tries to use the macOS Ruby at `/Library/Ruby/Site`:

    ...
	/Library/Ruby/Site/2.3.0/rubygems/commands/install_command.rb:156:in `execute'
	/usr/local/Homebrew/Library/Homebrew/utils.rb:228:in `install_gem!'
	...

The relevant file:

    $ more /usr/local/Homebrew/Library/Homebrew/utils.rb

The relevant code:

    def install_gem!(name, version = nil)
      # Match where our bundler gems are.
      ENV["GEM_HOME"] = "#{ENV["HOMEBREW_LIBRARY"]}/Homebrew/vendor/bundle/ruby/#{RbConfig::CONFIG["ruby_version"]}"
      ENV["GEM_PATH"] = ENV["GEM_HOME"]

Append code so we can see what's happening:

    def install_gem!(name, version = nil)
      # Match where our bundler gems are.
      ENV["GEM_HOME"] = "#{ENV["HOMEBREW_LIBRARY"]}/Homebrew/vendor/bundle/ruby/#{RbConfig::CONFIG["ruby_version"]}"
      ENV["GEM_PATH"] = ENV["GEM_HOME"]

      puts "Homebrew library: " + ENV["HOMEBREW_LIBRARY"]
      puts "Ruby version: " + RbConfig::CONFIG["ruby_version"]
      puts "Gem home: " + ENV["GEM_HOME"]
      puts "Gem path: " + ENV["GEM_PATH"]

Retry:

	$ brew cask style
	Homebrew library: /usr/local/Homebrew/Library
	Ruby version: 2.3.0
	Gem home: /usr/local/Homebrew/Library/Homebrew/vendor/bundle/ruby/2.3.0
	Gem path: /usr/local/Homebrew/Library/Homebrew/vendor/bundle/ruby/2.3.0
	...

Do we have the relevant Ruby directory or any of its parent directories?

	$ ls -d /usr/local/Homebrew/Library/Homebrew/vendor/bundle/ruby
    ls: /usr/local/Homebrew/Library/Homebrew/vendor/bundle/ruby: No such file or directory

    $ ls -d /usr/local/Homebrew/Library/Homebrew/vendor/bundle
	ls: /usr/local/Homebrew/Library/Homebrew/vendor/bundle: No such file or directory

	$ ls -d /usr/local/Homebrew/Library/Homebrew/vendor
	/usr/local/Homebrew/Library/Homebrew/vendor

What Ruby does Homebrew expect?

	$ brew config | grep Ruby
	Homebrew Ruby: 2.3.3 => /System/Library/Frameworks/Ruby.framework/Versions/2.3/usr/bin/ruby

Do we have the version that Homebrew expects?

    $ /System/Library/Frameworks/Ruby.framework/Versions/2.3/usr/bin/ruby --version
    ruby 2.3.3p222 (2016-11-21 revision 56859) [universal.x86_64-darwin17]

Try suggestion to remove brew then reinstall:

    $ sudo rm -f $(which brew)
    Password: ***

    $ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

Try a custom Ruby version:

    $ brew install rbenv
    $ brew install ruby-build 
    $ brew install rbenv-default-gems
    $ brew install rbenv-vars
    $ eval "$(rbenv init -)"
    $ rbenv install 2.5.1
    $ rbenv rehash
    $ rbenv global 2.5.1
    $ ruby --version
    ruby 2.5.1p57 (2018-03-29 revision 63029) [x86_64-darwin17]
    
Try cask upgrade and fix any errors:

    $ brew cask upgrade

    ...
    ==> Removing App '/Applications/Shortcat.app'.
    Error: Directory not empty @ dir_s_rmdir - /Applications/Shortcat.app
    ...

Is Xcode stuck waiting for a license agreement?

    $ sudo xcodebuild -license

New issue:

    $ brew doctor
    ...
	Warning: Homebrew's sbin was not found in your PATH but you have installed
	formulae that put executables in /usr/local/sbin.
	Consider setting the PATH for example like so
	  echo 'export PATH="/usr/local/sbin:$PATH"' >> ~/.bash_profile

