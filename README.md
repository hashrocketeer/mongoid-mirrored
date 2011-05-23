Mongoid::Mirrored
====================

Helper module for mirroring root documents in embedded collections. Works with *[mongoid](https://github.com/mongoid/mongoid). 

When adopting the mindset of document base storages we can understand the advantages of representing a document with all its related content as opposed to the relational database approach of referencing information scattered across a number of other collections. That doesn't only help in understanding better the data model we have in our hands, but also usually provides better read performances in our applications. Although the document based mindset can come naturally for most of us, some still struggle when trying to adopt this new paradigm (myself included) and end up modeling our data closely of what we would have done in a relational database. Sometimes that happens because can't be sure of that data access patterns in advance or we just fear denormalization. 

Mongoid provides an intuitive interface to reference documents among different collections, but that comes at a cost. As MongoDB doesn't support joins and relying heavily on relational data reads, that could be a problem for some applications. Unfortunately, Mongoid doens't support embedding collections while maintaining an independent root collection and that is exactly the intention with mirrored documents.  

I couldn't find anything that helped me with that, but this *[discussion](http://groups.google.com/group/mongoid/browse_thread/thread/b5e2bccf77457043) at the mongoid group where Durran stated that it was unlikely that mongoid would go in that direction. 

This gem has been inspired by that conversation and also *[Mongoid::Denormalize](https://github.com/logandk/mongoid_denormalize) which you should definitelly check out if you have similar needs.

I'm not an experienced developer(in fact I work as a Product Manager and develop only to bootstrap proofs of concept) and I'm new to document database storages. Please feel free to contribute to this gem or point out the correct approach for this.

Installation
------------

Add the gem to your Bundler `Gemfile`:

    gem 'mongoid_mirrored'

Or install with RubyGems:

    $ gem install mongoid_mirrored


Usage
-----

In your root(master) model:

    # Include the helper module
    include Mongoid::Mirrored
    
    # Define wich fields and methods will be shared among root and embedded classes
		mirrored_in :articles, :users, :sync_direction => :both, :sync_events => :all do
			field :contents, :type => String
			field :vote_ratio, :type => Float
			
			def calculate_vote_ratio
				# do something with votes
			end
		end
    

Example
-------

		# The conventioned name for the mirrored class is "#{embedding_class}::{root_class}"
		
		  class Post
		    embeds_many :comments, :class_name => "Post::Comment"
		  end

		  class User
		    embeds_many :comments, :class_name => "User::Comment"
		  end

		# The Root Class should establish with which classes it is supposed to sync documents.
		# Everything declared within the block will be shared between the root and mirrored 
		# documents (eg. fields, instance methods, class methods)
		
		  class Comment
		   mirrored_in :post, :user, :sync_direction => :both do
		     field :contents
		     field :vote_ratio, :type => Integer
		   end
		  end
		 
			class Comment
		   mirrored_in :post, :sync_events => :create do
		     field :contents
		     field :vote_ratio, :type => Float
		   end
			 mirrored_in :user, :sync_events => [:create, :update] do
		     field :contents
		     field :vote_ratio, :type => Float
		   end
		  end

Options
-------

sync_events => :all(default), :create, :update, :destroy
sync_direction => :both(default), :from_root, :from_mirror
replicate_to_siblings => true(default)


Rake tasks
----------

should include tasks to re-sync


Known issues
------------

shuld include known issues

Performance
------------

perf/performance.rb

Credits
-------

Copyright (c) 2011 Alexandre Angelim, released under the MIT license.