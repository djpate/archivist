This gem is no longer maintained.
=================

README.md
=================

This gem is intended as a direct replacement for acts\_as\_archive (AAA)
in Rails 3 apps with most of the same functionality and wrapping AAA's 
methods in aliases to maintain compatibility for some time. Thanks to 
[Winton Welsh](https://github.com/winton "Winton on github") for his 
original work on AAA, it is good solution to a problem that makes 
maintaining audit records a breeze.

TOC
---
1. <a href="#requirements">Requirements</a>
2. <a href="#install">Install</a>
2. <a href="#update_models">Update models</a>
2. <a href="#add_archive_tables">Add Archive tables</a>
1. <a href="#basic_usage">Basic Usage</a>
1. <a href="#additional_options">Additional Options</a>
    1. <a href="#multiple_archives">Allowing multiple archived copies</a>
    1. <a href="#associating">Associating archive to original</a>
    1. <a href="#including">Including External modules</a>
    1. <a href="#customizing">Customizing `copy_self_to_archive`</a>
1. <a href="#contributing">Contributing</a>
1. <a href="#todo">TODO</a>

<a name="requirements"></a>
Requirements
------------
This gem is intended to be used with ActiveRecord/ActiveSupport 3.0.1 and later.

<a name="install"></a>
Install
-------
**Gemfile**:
    gem 'archivist'

<a name="update_models"></a>
Update models
-------------
add `has_archive` to your models:
    class SomeModel < ActiveRecord::Base
      has_archive
    end

N.B. if you have any serialized attributes the has\_archive declaration *MUST* be after the serialization declarations or they will not be preserved and things will break when you try to deserialize the attributes. This is an unfortunate side effect of the declarative nature of both `serialize` and `has_archive` and I couldn't think of any way of getting around it that wouldn't make the entire gem really slow, any suggestions on this are welcome.

i.e.
    class AnotherModel < ActiveRecord::Base
      serialize(:some_array,Array)
      has_archive
    end

*NOT*
    class ThisModel < ActiveRecord::Base
      has_archive
      serialize(:a_hash,Hash)
    end

<a name="add_archive_tables"></a>
Add Archive tables
------------------
There are two ways to do this, the first is to use the built in updater like acts as archive.
`Archivist.update SomeModel`
Currently this doesn't support adding indexes automatically (AAA does) but I'm working on doing multi column indexes (any help is greatly appreciated)

The second way of adding archive tables is to build a migration yourself, if you're wanting to keep track of who triggered the archive or inject some other information you'll have to add those columns manually and pass a block into `copy_self_to_archive` before calling `delete!` or `destroy!`.

<a name="basic_usage"></a>
Basic Usage
-----------
Use `destroy`, `delete`, `destroy_all` as usual and the data will get moved to the archive table. If you really want the data to go away you can still do so by simply calling `destroy!` etc. This bypasses the archiving step but leaves your callback chain intact where appropriate. 

Migrations affecting the columns on the original model's table will be applied to the archive table as well.

<a name="additional_options"></a>
Additional Options
------------------
<a name="multiple_archives"></a>
###Allowing multiple archived copies
By default `copy_self_to_archive` just keeps updating a single instance of the archived record, this behavior is find if you're just trying to keep your main working table clean but can be problematic if you need a history of changes to a record.
This behavior can be changed to allow multiple copies of a archived record to be created by setting the `:allow_multiple_archives` to true in the options hash when calling `has_archive`.

####Example:
<pre>
  class SpecialModel &lt; AR::Base
    has_archive :allow_multiple_archives=&gt; true
  end
</pre>

<a name="associating"></a>
### Associating archive to original
The default here is to not associate the archived records in any way to the originals. But, if you're keeping a history of changes to a record the archived copies can be associated automatically with the 'original' by setting the `associate_with_original` option to true.

*N.B.* Using this option automatically sets `allow_multiple_archives` to true

####Example
<pre>
  class SpecialModel &lt; AR::Base
    has_archive :associate_with_original=&gt;true
  end
</pre>

allows for calls like:

`SpecialModel.first.archived_special_models`

or

`SpecialModel::Archive.first.special_model`

<a name="including"></a>
###Including External modules
If you want to include additional functionality in the `Archive` class you can pass in an array of module constants, this will allow adding scopes and methods to this class without any monkey patching.

####Example
<pre>
  class MyModel &lt; AR:Base
    has\_archive :included\_modules=&gt;BigBadModule
  end
</pre>
or if multiple modules are needed
<pre>
  class MyModel &lt; AR:Base
    has\_archive :included\_modules=&gt;[BigBadModule,MyScopes,MyArchiveMethods]
  end
</pre>

<a name="customizing"></a>
###Customizing copy\_self\_to\_archive
A block can be passed into `copy_self_to_archive` which takes a single argument (the new archived record)

####Example:
Supposing we have added an archiver\_id column to our archive table we can pass a block into the `copy_self_to_archive` method setting this value. The block gets called immediately before saving the archived record so all of the attributes have been copied over from the original and are available for use in the block.
<pre>
  class SpecialModel &lt; AR:Base
    has_archive
    def archive!(user)
      self.copy_self_to_archive do |archive|
        archive.archiver_id = user.id
      end
    end
  end
</pre>

<a name="contributing"></a>
Contributing
------------
If you'd like to help out please feel free to fork and browse the TODO list below or  add a feature that you're in need of. Then send a pull request my way and I'll happily merge in well tested changes.

Also, I use autotest and MySQL but [nertzy (Grant Hutchins)](https://github.com/nertzy "Grant on github") was kind enough to add support for testing against Postgres using the pg gem.

<a name="todo"></a>
TODO
----

 *  <del>Maintain seralized attributes from original model</del>
 *  <del>allow passing of a block into copy\_to\_archive</del>
 *  give Archive scopes from parent (may only work w/ 1.9 since scopes are Procs)
 *  <del>give subclass Archive its parent's methods (method\_missing?)</del>
 *  <del>associate SomeModel::Archive with SomeModel (if archiving more than one copy)</del>
 *  associate Archive with other models (SomeModel.reflect\_on\_all\_associations?)
 *  make archive\_all method chain-able with scopes and other finder type items
