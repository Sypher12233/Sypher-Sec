# File Uploads #

* Every time we change our social media account profile picture or add a report to the cloud or upload our work on Github we upload files to a server.
* Unfortunately, when handled badly, file uploads can also open up severe vulnerabilities in the server.

Basically if an attacker can upload files to your server at will (with no restrictions) he or she can do a lot of bad things.

## What we will learn ##

1. Overwriting existing files on the server
2. Uploading and executing shells on the server (RCE)
3. Bypassing client side filtering
4. Bypassing various kinds of server side filtering
5. Fooling content type validation checks

## 1. Overwriting ##

Basically overwriting means being able to upload and replace files on the server with malicious version of it.

So what we are doing here is trying to upload and replace an image on the website with an image of our own (with the same name).

## 2. Remote Code Execution (RCE) ##

Remote Code Execution (as the name suggests) would allow us to execute code arbitrarily on the web server. Whilst this is likely to be as a low-privileged web user account (such as www-data on Linux servers), it's still an extremely serious vulnerability. Remote code execution through a web application tends to be a result of uploading a program written in the same language as the back-end of the website (or another language which the server understands and will execute). Traditionally this would be `PHP`, however, in more recent times, other back-end languages have become more common (`Python Django` and `Javascript` in the form of `Node.js` being prime examples).

There are two basic ways to achieve `RCE` on a webserver: `webshells`, and `reverse shells`. Realistically a fully featured reverse shell is the ideal goal for an attacker; however, a webshell may be the only option available (for example, if a file length limit has been imposed on uploads). We'll take a look at each of these in turn. As a general methodology, we would be looking to upload a shell of one kind or another, then activating it, either by navigating directly to the file if the server allows it, or by otherwise forcing the webapp to run the script for us.

### Webshells ###

Lets assume we have found a webpage that allows us to upload files on it.

> What we do from here?

* We start with directory brute forcing(`gobuster`).

![gobuster](https://i.imgur.com/OftwAIE.png)

And we have two directories uploads and assets!

* So basically we can use the uploads page to try and upload a malicious code and try to get RCE.

To do that we are gonna upload this `php` script in a `.php` file to the website

```php
<?php echo "<pre>" .shell_exec($_GET["cmd"]). "</pre>"; ?>
```

A simple webshell works by taking a parameter and executing it as a system command. In PHP, the syntax for this would be:

```php
<?php
    echo system($_GET["cmd"]);
?>
```

This code takes a `GET` parameter and executes it as a system command. It then echoes the output out to the screen.

## 3. Filtering ##

First up, let's discuss the differences between client-side filtering and server-side filtering.

> Client-Side Filtering;

When we talk about a script being "Client-Side", in the context of web applications, we mean that it's running in the user's browser as opposed to on the web server itself. JavaScript is pretty much ubiquitous as the client-side scripting language, although alternatives do exist.  Regardless of the language being used, a client-side script will be run in your web browser. In the context of file-uploads, this means that the filtering occurs before the file is even uploaded to the server. Theoretically, this would seem like a good thing, right? In an ideal world, it would be; however, because the filtering is happening on our computer, it is trivially easy to bypass. As such client-side filtering by itself is a highly insecure method of verifying that an uploaded file is not malicious.

> Server-Side Filtering;

Conversely, as you may have guessed, a server-side script will be run on the server. Traditionally PHP was the predominant server-side language (with Microsoft's ASP for IIS coming in close second); however, in recent years, other options (C#, Node.js, Python, Ruby on Rails, and a variety of others) have become more widely used. Server-side filtering tends to be more difficult to bypass, as you don't have the code in front of you. As the code is executed on the server, in most cases it will also be impossible to bypass the filter completely; instead we have to form a payload which conforms to the filters in place, but still allows us to execute our code.

---

With that in mind, let's take a look at some different kinds of filtering.

> Extension Validation:

File extensions are used (in theory) to identify the contents of a file. In practice they are very easy to change, so actually don't mean much; however, MS Windows still uses them to identify file types, although Unix based systems tend to rely on other methods, which we'll cover in a bit. Filters that check for extensions work in one of two ways. They either blacklist extensions (i.e. have a list of extensions which are not allowed) or they whitelist extensions (i.e. have a list of extensions which are allowed, and reject everything else).

> File Type Filtering:

Similar to Extension validation, but more intensive, file type filtering looks, once again, to verify that the contents of a file are acceptable to upload. We'll be looking at two types of file type validation:

MIME validation: MIME (Multipurpose Internet Mail Extension) types are used as an identifier for files -- originally when transferred as attachments over email, but now also when files are being transferred over HTTP(S). The MIME type for a file upload is attached in the header of the request, and looks something like this:

MIME types follow the format `<type>/<subtype>`. In the request above, you can see that the image "spaniel.jpg" was uploaded to the server. As a legitimate JPEG image, the MIME type for this upload was "image/jpeg". The MIME type for a file can be checked client-side and/or server-side; however, as MIME is based on the extension of the file, this is extremely easy to bypass.

Magic Number validation: Magic numbers are the more accurate way of determining the contents of a file; although, they are by no means impossible to fake. The "magic number" of a file is a string of bytes at the very beginning of the file content which identify the content. For example, a PNG file would have these bytes at the very top of the file: 89 50 4E 47 0D 0A 1A 0A.

Unlike Windows, Unix systems use magic numbers for identifying files; however, when dealing with file uploads, it is possible to check the magic number of the uploaded file to ensure that it is safe to accept. This is by no means a guaranteed solution, but it's more effective than checking the extension of a file.

> File Length Filtering:

File length filters are used to prevent huge files from being uploaded to the server via an upload form (as this can potentially starve the server of resources). In most cases this will not cause us any issues when we upload shells; however, it's worth bearing in mind that if an upload form only expects a very small file to be uploaded, there may be a length filter in place to ensure that the file length requirement is adhered to. As an example, our fully fledged PHP reverse shell from the previous task is 5.4Kb big -- relatively tiny, but if the form expects a maximum of 2Kb then we would need to find an alternative shell to upload.

> File Name Filtering:

As touched upon previously, files uploaded to a server should be unique. Usually this would mean adding a random aspect to the file name, however, an alternative strategy would be to check if a file with the same name already exists on the server, and give the user an error if so. Additionally, file names should be sanitised on upload to ensure that they don't contain any "bad characters", which could potentially cause problems on the file system when uploaded (e.g. null bytes or forward slashes on Linux, as well as control characters such as ; and potentially unicode characters). What this means for us is that, on a well administered system, our uploaded files are unlikely to have the same name we gave them before uploading, so be aware that you may have to go hunting for your shell in the event that you manage to bypass the content filtering.

> File Content Filtering:

More complicated filtering systems may scan the full contents of an uploaded file to ensure that it's not spoofing its extension, MIME type and Magic Number. This is a significantly more complex process than the majority of basic filtration systems employ, and thus will not be covered in this room.

It's worth noting that none of these filters are perfect by themselves -- they will usually be used in conjunction with each other, providing a multi-layered filter, thus increasing the security of the upload significantly. Any of these filters can all be applied client-side, server-side, or both.

Similarly, different frameworks and languages come with their own inherent methods of filtering and validating uploaded files. As a result, it is possible for language specific exploits to appear; for example, until PHP major version five, it was possible to bypass an extension filter by appending a null byte, followed by a valid extension, to the malicious .php file. More recently it was also possible to inject PHP code into the exif data of an otherwise valid image file, then force the server to execute it. These are things that you are welcome to research further, should you be interested.

### Bypassing client-side filtering ###

So according to the above notes, we can first check the page source of the website and look if there are any filters, in our case we have a `js` filter which prevents us from uploading anything except a `.png` file.

Knowing that we can upload our shell as a `.png` file and capture the request using burpsuite and edit the `.png code with .php`.

And then we can forward the request and find the upload dir using `gobuster`.

After that we need to execute it, and we get a RCE if we have a netcat listener running.

### Bypassing server-side filtering: File Extensions ###

Client-side filters are easy to bypass -- you can see the code for them, even if it's been obfuscated and needs processed before you can read it; but what happens when you can't see or manipulate the code? Well, that's a server-side filter. In short, we have to perform a lot of testing to build up an idea of what is or is not allowed through the filter, then gradually put together a payload which conforms to the restrictions.
