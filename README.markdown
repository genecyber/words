# Words - A fast, easy to use interface to WordNet® with cross ruby distribution compatibility. #

## About ##

Words implements a fast interface to [Wordnet®](http://wordnet.princeton.edu) which provides both a pure ruby and an FFI powered backend over the same easy-to-use API. The FFI backend makes use of [Tokyo Cabinet](http://1978th.net/tokyocabinet/) and the FFI interface, [rufus-tokyo](http://github.com/jmettraux/rufus-tokyo), to provide cross ruby distribution compatibility and blistering speed. The pure ruby interface operates on a special ruby optimised index along with the basic dictionary files provided by WordNet®. I have attempted to provide ease of use in the form of a simple yet powerful api and installation is a sintch!

* Version 0.2 Introduced Pure Ruby Backend
* Version 0.3 Introduced Evocation Support (see examples & below) as developed by the [Wordnet® Evocation Project](http://wordnet.cs.princeton.edu/downloads/evocation/release-0.4/README.TXT) 
* Version 0.4 Substantial performance increase in pure mode (now faster at some things than the tokyo backend) and simplification of use! Full refactoring. Move to RSpec for testing. API CHANGES: Words::Words -> Words::Wordnet, close -> close!, connected -> connected? and evocations_enabled? -> evocations?

Documentation: [Yardoc Live](http://yardoc.org/docs/roja-words)

## Pre-Installation ##

First ensure you have a copy of the wordnet data files. This is generally available from your Linux/OSX package manager:

    #Ubuntu
    sudo apt-get install wordnet-base
    
    #Fedora/RHL
    sudo yum update wordnet

    #OSX HomeBrew
    sudo brew install wordnet
    
    #MacPorts
    sudo port install wordnet
    
or you can simply download and install (Unix/OSX):

	wget http://wordnetcode.princeton.edu/3.0/WNdb-3.0.tar.gz
	sudo mkdir /usr/local/share/wordnet
	sudo tar -C /usr/local/share/wordnet/ -xzf WNdb-3.0.tar.gz
	
or (Windows)

	Download http://wordnetcode.princeton.edu/3.0/WNdb-3.0.tar.gz
	Unzip

        # due to the way windows tends not to have a folder for things you will
        # have to specify the location of the wordnet files when using the gem

## For Tokyo Backend Only ##

Unless you want to use the tokyo backend you are now ready to install Words (and build the data if you want extras), otherwise if you want to use the tokyo backend (FAST!) you will also need [Tokyo Cabinet](http://1978th.net/tokyocabinet/) installed. It should be nice and easy... something like:

    # osx users should, if ports is installed, simply do
    sudo ports install tokyocabinet

    # otherwise the best route is from source
    wget http://1978th.net/tokyocabinet/tokyocabinet-1.4.41.tar.gz
    tar -xzf tokyocabinet-1.4.41.tar.gz
    cd tokyocabinet-1.4.41/
    ./configure
    make
    sudo make install
    
## GEM Installation ##

After this it should be just a gem to install. For those of you with old rubygems versions first:

    gem install gemcutter  # These two steps are only necessary if you haven't
    gem tumble             # yet installed the gemcutter tools
    
Otherwise and after it's simply:

	gem install words
	
Then your ready to rock and roll. :)

## Build Data ##

If all you want to do is use wordnet in it's standard form you don't have to do any databuilding and can skip this section. If however you either
want to take advantage of evocations ([Wordnet® Evocation Project](http://wordnet.cs.princeton.edu/downloads/evocation/release-0.4/README.TXT)) or want to use the tokyo backend, read on!

To build the wordnet dataset file yourself, from the original wordnet files, you can use the bundled "build_wordnet" command

	build_wordnet -h # this will give you the usage information & additional options/features
	
	# this would attempt to build the tokyo backend data locating the original wordnet
        # files through a search...
	sudo build_wordnet --build-tokyo
	
	# this would attempt to build the tokyo backend locating the original wordnet files
        # through a search with the addition of evocation support...
	sudo build_wordnet --build-tokyo-with-evocations
	
	# this would attempt to build evocation support for the pure backend
        # (remember no dataset needs to be built to use wordnet with the pure backend)
	sudo build_wordnet --build-pure-evocations

## Usage ##

Heres a few little examples of using words within your programs.

    require 'rubygems'
    require 'words'
    
    data = Words::Wordnet.new # or: data = Words::Wordnet.new(:tokyo) for the tokyo backend

    # to specify a wordnet path Words::Words.new(:pure, '/path/to/wordnet')
    # to specify the tokyo dataset Words::Words.new(:pure, :search, '/path/to/data.tct')

    # play with connections
    data.connected? # => true
    data.close!
    data.connected? # => false
    data.open!
    data.connected? # => true
    data.connection_type # => :pure or :tokyo depending...

    # locate a word
    lemma = data.find("bat")
    
    lemma.to_s # => bat, noun/verb
    lemma.available_pos.inspect # => [:noun, :verb]
    
    lemma.synsets(:noun) # => array of synsets which represent nouns of the lemma bat
    # or
    lemma.nouns # => array of synsets which represent nouns of the lemma bat
    lemma.noun_ids # => array of synsets ids which represent nouns of the lemma bat
    lemma.verbs? #=> true
    
    # specify a sense
    sense = lemma.nouns.last
    sense2 = lemma.nouns[2]
    
    sense.gloss # => a club used for hitting a ball in various games
    sense2.words # => ["cricket bat", "bat"]
    sense2.lexical_description # => a description of the lexical meaning of the synset
    sense.relations.first # => "Semantic hypernym relation between n02806379 and n03053474"

    sense.relations(:hyponym) # => Array of hyponyms associated with the sense
    # or
    sense.hyponyms # => Array of hyponyms associated with the sense
    sense.hyponyms? # => true
    
    sense.relations.first.is_semantic? # => true
    sense.relations.first.source_word # => nil
    sense.relations.first.destination # => the synset of n03053474
    
    sense.derivationally_related_forms.first.is_semantic? # => false
    sense.derivationally_related_forms.first.source_word # => "bat"
    sense.derivationally_related_forms.first.destination_word # => "bat"
    sense.derivationally_related_forms.first.destination # => the synset of v01413191

    if data.evocations? # check for evocation support
       data.find("broadcast").senses.first.evocations # => sense relevant evocations
       data.find("broadcast").senses.first.evocations[1] # => the evocation at index 1
       data.find("broadcast").senses.first.evocations[1][:destination].words # => synset
    end

    data.close!
    
These and more examples are available from within the examples.rb file!        

## Note on Patches/Pull Requests ##
 
* Fork the project.
* Make your feature addition or bug fix.
* Add tests for it. This is important so I don't break it in a
  future version unintentionally.
* Commit, do not mess with rakefile, version, or history.
  (if you want to have your own version, that is fine but bump version in a commit by itself I can ignore when I pull)
* Send me a pull request. Bonus points for topic branches.

## Copyright ##

Copyright (c) 2010 Roja Buck. See LICENSE for details.
