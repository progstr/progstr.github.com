---
title: Ruby on Rails and Direct File Uploads
layout: filer
permalink: rails-direct-upload.html
---
## Table of Contents

* [Prerequisites](#prerequisites)
* [How it works](#how_it_works)
* [Register scripts](#register_scripts)
* [The view helper](#the_view_helper)
* [Processing uploads](#processing_uploads)
* [Upload validations](#upload_validations)
* [UI Localization](#ui_localization)
* [View helper option reference](#view_helper_option_reference)


Progstr Filer exposes a file upload API that lets you upload files from any part of your application. That sort of power requires greater responsibility though - your application needs to handle file uploads. For the typical user file upload that does not need any processing from your code, that is entirely unnecessary. That is why Progstr Filer offers a custom upload UI that lets end users upload files directly to the Progstr Filer servers.

### Prerequisites

You have to have the `progstr-filer` Ruby gem added to your `Gemfile` and have your access key and secret key set up in your environment configuration file as described in the ["Getting Started" article](/filer_getting_started.html).


### How it works

Ajax HTTP requests have been around for some time, but uploading files over them is a relatively new addition. With the advent of cross-origin (CORS) HTTP request support <sup>1</sup> it is now possible to have users visit your application and upload files to the Progstr Filer servers directly. The integration is seamless and most users will not even notice the difference. That lets you build forms that have users select and upload files to Progstr Filer, then post file details along with other form values to your application.

The direct upload UI is built in JavaScript, but you do not need to be a JavaScript expert to use it. Integrating it requires no JavaScript code and is all handled by convenient Rails view helpers.

--

Almost all major browsers support Ajax file uploads and cross-origin HTTP request. For those that do not, Progstr Filer falls back to using a Flash-based uploader component.


### Register scripts

You need to pick a place on your HTML page where the direct upload UI scripts will be included. The recommended place for that is at the end of your page - just before the closing `</body>` tag. This way, your page will load and render before starting to load the Filer scripts.

Typically, you would edit your layout file and add the following code at the desired location:

    <%= yield :filer_scripts %>
{.prettyprint .lang-ruby .language-ruby}

That will register a layout content area that will contain the script registrations. The Filer upload UI view helper is smart enough to include the scripts only on pages that require the Filer upload UI. You can also register the scripts using the `filer_scripts` raw view helper:

    <%= filer_scripts %>
{.prettyprint .lang-ruby .language-ruby}

... but that is not recommended since, if used on a layout page, it will unconditionally include the script registration statements on all your application pages.


### The view helper

Rendering the upload UI on a typical `form_for`-generated form is as simple as using the `filer_upload_field` builder. For example, to render an uploader that will accept avatar images for a hypothetical `User` model object, you can use:

    <%= f.filer_upload_field :avatar %>
{.prettyprint .lang-ruby .language-ruby}

If you do not use `form_for` to layout your forms, you can use the raw HTML `filer_upload` view helper:

    <%= filer_upload @user.avatar, :file_input_name => "user[avatar]" %>
{.prettyprint .lang-ruby .language-ruby}

Note how we need to do extra work and pass the file attachment object `@user.avatar` as well as specify the file input name that will be rendered and later used when processing form data. That is why we recommend that you use the `form_for` approach most of the time.



Note: make sure your database table has an `avatar` *string* column created already. Future versions of the `progstr-filer` gem will let you generate migrations automatically.

### Processing uploads

Each direct upload UI renders a hidden input field that will post back details about the file being uploaded by the user. The field contains the uploaded file ID, its name and size encoded in JSON. You shouldn't worry about that though, since our model integration takes care of it automatically.

Provided you set up an insert/edit web form using the `form_for` helper, and you initialize your newly-edited model object from the `params` dictionary, you need just a few simple checks when processing your form post data:

    @user = User.new(params[:user])

    # Check if the user actually uploaded a file
    need_to_upload = @user.avatar.blank?
    # Get the avatar file name and size
    name = @user.avatar.path
    size = @user.avatar.size
{.prettyprint .lang-ruby .language-ruby}

### Upload validations

Outsourcing uploads directly to the Progstr Filer servers has some implications too. The main one is that you do not control the upload endpoint and cannot validate uploaded files before accepting them. This is where upload validations step in.

You configure upload validation rules in the Filer admin backend. Validations are specified per uploader and you can set up rules that may reject files if they are larger than a specified size or are of a type that you do not allow.

![Upload validations](images/uploads/upload-validations.png)

By default all files will get accepted for uploaders that do not have validation rules specified.

### UI Localization

The upload UI displays a bunch of text messages to the user that may be improved upon in your application, or just need to be localized in a target language. To do that, you need to pass extra options to the view helper function like this:


    <%= f.filer_upload_field :avatar, :browse_button_text => "Pick files..." %>
{.prettyprint .lang-ruby .language-ruby}

Here is a list of the options and their default values. Note that the `{file}` part will be replaced with the respective file name:

* `browse_button_text`: The caption of the file selection dialog button. Default is "\[Browse\]"
* `upload_button_text`: The caption of the upload start button. Default is "\[Upload\]"
* `file_size_error_text`: Shown when a user tries to upload a file bigger than the allowed size. Default is "File '{file}' exceeds maximum allowed size."
* `allowed_extensions_error_text`: Shown when a user tries to upload a file of a disallowed type. Default is "File '{file}' not allowed."
* `multiple_not_allowed_error_text`: Shown when the user selects multiple files in the file selection dialog. Default is "Multiple files not allowed. Dropping '{file}'."
* `upload_failed_error_text`: Shown in case an upload fails. Default is "Could not upload file: '{file}'."
* `upload_in_progress_text`: Shown if the user tries to navigate away from the page while an upload is in progress. Default is: "File upload in progress."

### View helper option reference

Some of the upload UI behavior can be controlled by passing options to the view helper.

    <%= f.filer_upload_field :avatar, :max_file_size => "1 MB" %>
{.prettyprint .lang-ruby .language-ruby}

You saw some of the options that control messages displayed to the user. Here are the rest of the supported options:

* `max_file_size`: The maximum allowed file size that will be accepted. You can use freeform values such as "200 kb", "2 mb", etc.
* `allowed_file_extensions`: A whitelist of allowed extensions.

Note that both `max_file_size` and `allowed_file_extensions` are just UI hints and are designed to save the user from a server roundtrip. A resourceful and possibly malicious user could find a way to circumvent those restrictions, and that is why you should define server-side upload validations for your uploader too.
