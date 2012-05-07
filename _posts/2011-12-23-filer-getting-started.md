---
title: Home
layout: filer
permalink: filer-getting-started.html
---
## Table of Contents

* [Setting up the Ruby gem](#setting_up_the_ruby_gem)
* [Credentials](#credentials)
* [Defining an uploader](#defining_an_uploader)
* [Associating uploaders with your models](#associating_uploaders_with_your_models)
* [Public and private files](#public_and_private_files)
* [Feeding data to your uploaders](#feeding_data_to_your_uploaders)
* [Uploading directly to Progstr Filer](#uploading_directly_to_progstr_filer)
* [Generating URLs for files](#generating_urls_for_files)
* [Validation] (#validation)
* [Source code](#source_code)


Progstr Filer is a developer-friendly file hosting platform built specifically for web apps. It lets you easily associate file attachments with your ActiveRecord models and removes the hassle of actually hosting the files yourself.

### Setting up the Ruby gem

Bundler makes that all too easy - add this line to your `Gemfile` to have the gem pulled into your app and required automatically.


    gem "progstr-filer", :require => "progstr-filer"
{.prettyprint .lang-ruby .language-ruby}


### Credentials

Progstr Filer uses two keys similar to Amazon's cloud services. The access key is a public string that gets rendered publicly in URL's and web pages. We use it to identify your account. The secret key is used to sign and encrypt sensitive data. You should keep it, well, secret.

Provisioning your add-on with Heroku gets you two environment variables: `PROGSTR_FILER_ACCESS_KEY` and `PROGSTR_FILER_SECRET_KEY`. The Ruby gem is smart enough to pick them up automatically, so you don't have to do anything about them. Yet, if you want to override them, you could modify your `config/environments/production.rb` file as follows:


    # Note, sample/invalid keys below - substitute for your real keys
    Progstr::Filer.access_key = "31322b06758444309c62452f35105318"
    Progstr::Filer.secret_key = "01f0db3bd2f640cebfcbd0134360d4c4"
{.prettyprint .lang-ruby .language-ruby}

Note: Non-Heroku deployments must have the keys set in the respective environment config file.


### Defining an uploader

Every attachment can have a specific set of options that take effect while uploading and manipulating it. Those settings get configured via your own class that will extend `Progstr::Filer::Uploader`. You could store uploader classes in your `app/uploaders` folder. Here is a sample class:

    class AvatarUploader < Progstr::Filer::Uploader
      #uploader options
    end
{.prettyprint .lang-ruby .language-ruby}


### Associating uploaders with your models

Progstr Filer extends ActiveRecord models and lets you use the `has_file` method in class definitions. Here is a User model class that has an avatar image managed as a Progstr Filer attachment via its `avatar` property:

    class User < ActiveRecord::Base
        has_file :avatar, AvatarUploader
    end
{.prettyprint .lang-ruby .language-ruby}


Note: make sure your database table has an `avatar` *string* column created already. Future versions of the `progstr-filer` gem will let you generate migrations automatically.

### Public and private files

Each file as an associated URL that you can use to refer to it in your web application. Typically you will get a list of the files you need to display and generate image references that point to them. By default files are private and are accessible only through a special URL that has an authentication token parameter. That URL is temporary and will expire after a certain amount of time (the default is 30 minutes). This makes it easy for you to implement private areas with separate files that are visible only to certain users. You control the access settings by deciding which URLs will be visible to each user.

You can also mark files as public. Public files can be accessed without the authentication token part in the URL and are visible to everyone.

Controlling permissions requires no coding on your part and is done in the Progstr Filer control panel. Instead of marking individual files as public or private you define rules that match files according to their uploader names. This way you can easily mark avatar images as public and private gallery files as, well, private. Here is a sample rule that allows public access to files uploaded by `AvatarUploader`:

![Public avatars rule](images/heroku/public-avatars-rule.jpg)

Again, by default, all files are private unless there is an explicit rule that makes them public.

You can control private file URL expiration through the `Progstr::Filer.session_timeout` setting. Timeouts are specified in seconds. Here is how you can set the timeout to one hour:

    Progstr::Filer.session_timeout = 60 * 60 # 1 hour

### Feeding data to your uploaders

That is easy - all you need is assign a Ruby File object to the uploader property and save the model object:

    @user.avatar = uploaded_image_file
    @user.avatar.save 
{.prettyprint .lang-ruby .language-ruby}


Or, Rails can get most of the job done for you automatically. If you create a file upload form:

    <div class="field">
      <%= f.label :avatar %>
      <%= f.file_field :avatar %>
    </div>
{.prettyprint .lang-html .language-html}


... and then create your model from the params hash:

    @user = User.new(params[:user])
    @user.save
{.prettyprint .lang-ruby .language-ruby}

That will automatically extract the attachment file object from the params hash and set it as the avatar.

### Uploading directly to Progstr Filer

Handling uploads yourself, validating and processing them before sending them to Progstr Filer is a fine approach, but often it is too much. In cases where you do not need to process files before storing them, you can just use the direct upload UI and have users upload files directly to the Progstr Filer servers. You can still restrict uploads with validation rules that can reject files by size or file type.

For details on how to do that, see our [Direct file uploads article](/rails-direct-upload.html).


### Generating URLs for files

Just use the `url` method on your attachments, say `@user.avatar.url`. Here is a sample view that generates an `img` tag pointing to the avatar image:

    <p>
      <b>Avatar:</b><br>
      <%= image_tag @user.avatar.url, :style => "max-width: 600px; margin: 10px 0px;" %>
    </p>
{.prettyprint .lang-html .language-html}

The `url` method always generates private URLs. To generate a public url, use the `public_url` method.

### Validation

Users may upload all types of files to your application and it might be a good idea to restrict that to the set of files that your application knows how to process. For example it would not be a good idea to allow non-image file types as an avatar picture. It might be a good idea to disable executable content that might spread malware too. To help with that the `progstr-filer` gem currently supports attachment validation according to an extension whitelist through the `validates_file_extension_of` method that is available to model objects:

    class User < ActiveRecord::Base
      has_file :avatar, AvatarUploader
      validates_file_extension_of :avatar, :allowed => ["jpg", "png"]
    end
{.prettyprint .lang-ruby .language-ruby}

In addition you can restrict the file size using `validates_file_size_of`:

    class User < ActiveRecord::Base
      has_file :avatar, AvatarUploader
      validates_file_size_of :avatar, :less_than => 1 * 1024 * 1024
    end
{.prettyprint .lang-ruby .language-ruby}

The example above will not allow files larger than 1 MB. You can specify a lower bound using the `:greater_than` option or even pass a numeric range using the `:in` option.

Of course, requiring users to always upload a file when saving a model object, use the Rails built-in `validates_presence_of` validator:

    class User < ActiveRecord::Base
      has_file :avatar, AvatarUploader
      validates_presence_of :avatar
    end
{.prettyprint .lang-ruby .language-ruby}

You can pass a custom error message for all validators using the `:message` option.

### Deleting stale files

Every time you set a new file and save your model, the old file will get scheduled for deletion. The same happens when you `destroy` your model. Note that calling `delete` for your model will not delete any associated files.

### Source code

The `progstr-filer` [gem](https://rubygems.org/gems/progstr-filer) is open source and its code is hosted on [Github](https://github.com/progstr/progstr-filer-gem).

All the techniques outlined in this document are available as a fully-functional Rails project on [Github](https://github.com/progstr/progstr-filer-demo) too.
