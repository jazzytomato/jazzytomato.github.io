---
layout: post
title: Hacking Tinder
---

![Geek stick pic]({{ site.baseurl }}/images/tinder_hack/geek_stick.png){: .f-r }


In this post I will demonstrate how to programmatically query your [Tinder](https://www.gotinder.com/, "Tinder's website") matches and send a batch of messages to them. 

You might find this creepy, but to the point where we are now, I find that swiping tens *(hundreds ?)* of people a minute based on their appearance is a bit scary already. It's a funny world we live in.

Believe me or not, I wasn't looking for hookups. I've sent a batch of messages and met one girl. I also told her that the initial message was automatic and she didn't care. Anyway, she is the only person I have ever met from an online dating app. That would be a good story but I can't tell you yet if I met the love of my life by writing a ruby script because we've only been together for 2 weeks.

**Whatever your intent is, you should be respectful and honest. This little hack is just a way to save your time and meet great people.** :v:


## Why?
1. Learning
2. Laziness
3. Send messages from my laptop

It is a bit tricky to get responses on Tinder, my matches were simply ignoring my messages because:

- They have tons of people talking to them already
- They found love and don't use the app anymore (but I had no way to know that from the app itself)
- Tinder servers were down
- They noticed that I skip both torso and leg days

So I felt like wasting my time, trying to be nice to a girl and just simply being ignored. Complete radio silence can be painful, to quote *Elie Wiesel* :

> The opposite of love is not hate, it's indifference. The opposite of art is not ugliness, it's indifference. The opposite of faith is not heresy, it's indifference. And the opposite of life is not death, it's indifference.

Though sometimes they would just take their time because [this is the way it works now](http://nautil.us/issue/33/attraction/shell-text-me-shell-text-me-not).

## How?

Tinder doesn't provide an open API, but by intercepting the traffic between our phone and the Tinder API, we can mimic the phone behavior and send out similar HTTP requests from a computer, namely [a Man-in-the-middle attack](https://en.wikipedia.org/wiki/Man-in-the-middle_attack "man-in-the-middle attack"). This will allow us to download our list of matches and send the messages.

There are many different tools that can help us achieve this. In this post I will explain **how I did it**. Here is my setup:

- a Macbook
- an iPhone 6s

Your phone needs to be connected to the same network as your computer because your phone network traffic will pass through your computer.

In terms of software, I have used :

- [Mitmproxy](http://mitmproxy.org/)
- [Ruby](https://www.ruby-lang.org/en/) (and the [http gem](https://rubygems.org/gems/http))

### Setup

#### On your computer

I use [Homebrew](http://brew.sh/) as a package manager which allows me to install mitmproxy by running the following command in a terminal:
{% highlight bash %}
brew install mitmproxy
{% endhighlight %}
If you plan to do more ruby, I recommend using `rbenv` that you can install with Homebrew as well. Otherwise, and if you don't already have ruby :
{% highlight bash %}
brew install ruby
{% endhighlight %}
And once you have ruby installed:
{% highlight bash %}
gem install http
{% endhighlight %}

Now, let's start mitmproxy
{% highlight bash %}
mitmproxy
{% endhighlight %}

A blank screen will appear, everything is fine. mitmproxy is now running and listening for incoming requests on the port `8080` (by default). You will see activity once your phone is configured.

#### On your phone

Uninstall the Tinder app and remove its local data. This will force the app to redownload the list of your matches. You will not lose all your matches as they are stored on the Tinder servers. Then reinstall the app but **don't open it yet**. *If you know a better way to do it without reinstalling the app please leave a comment and I will update the post. I haven't dug too much into that, to be honest.*

This is important that you reinstall the app before setting up the proxy because the AppStore use [certificate pinning](http://media.blackhat.com/bh-us-12/Turbo/Diquet/BH_US_12_Diqut_Osborne_Mobile_Certificate_Pinning_Slides.pdf, "certificate pinning") which make it unaccessible when going through mitmproxy.

Go in your network settings, and set up the http proxy to use our mitmproxy server. It looks like this on my iPhone:

![mitmproxy settings]({{ site.baseurl }}/images/tinder_hack/mitmproxy_settings.png)

In the field "server" set your computer local IP address (you can get it with `ifconfig`) and the port to `8080`.

Now, from your phone visit [mitm.it](http://mitm.it/), and install the certificate.

Once the certificate is installed, try launching your web browser on your phone and you should see the HTTP/HTTPS traffic being monitored on the mitmproxy screen. If it doesn't work, visit the [mitmproxy documentation](http://docs.mitmproxy.org/en/stable/certinstall.html#quick-setup) to get further assistance.


### Hack

Open the Tinder app, and log in. Now your mitmproxy console might go crazy because the app is about to redownload everything that it needs, including the pictures. We want to find our list of matches. Tinder poll their API every second to get the updated content, this is done via a POST request to `https://api.gotinder.com/updates`. We can filter the mitmproxy view by pressing <kbd>L</kbd> and then entering a regular expression, [here is a reference of the expressions you can use](http://docs.mitmproxy.org/en/stable/features/filters.html "filter expression reference"). Here I want to filter by URL so I use `~u` followed by the regexp.

{% highlight bash %}
~u /.*(tinder).*(update).*/
{% endhighlight %}

Here is what it looks like on my computer :

![mitm_tinder_update]({{ site.baseurl }}/images/tinder_hack/mitm_tinder_update.png)

Now try to spot the biggest request (or the one that took the longest to load), it should be the first one. You can navigate in mitmproxy using the arrow keys. Press enter to view the request details. The first tab is already interesting since it contains the request header. 

![mitm_request_header]({{ site.baseurl }}/images/tinder_hack/mitm_request_header.png)

Copy and save the authorization token (the part that I have blanked out from the picture). We will send our requests using almost the same header (but don't bother copying it just yet).

Then hit <kbd>TAB</kbd> to go in the response, then <kbd>B</kbd> to save the output to a file in the current directory. You will be prompted for a file name, you can save it to `matches.json` for example.

Have a quick glance at the file, it should contain all your matches and the full history of your messages and activity.

Now, using the same technique of intercepting requests, I found that sending a message to a match is done via a POST request to `https://api.gotinder.com/user/matches/:match_id` with the request body being `{ message: 'Hello.' }`

To send a batch of messages to the matches I had no messages with yet, I wrote a short ruby script:

{% highlight ruby%}
#!/usr/bin/env ruby

gem 'http'
require 'http'
require 'json'

# Your token goes here
TOKEN = ''

HEADERS = { 'host' => 'api.gotinder.com',
            'Authorization' => "Token token='#{TOKEN}'",
            'x-client-version' => '47217',
            'app-version' => '467',
            'Proxy-Connection' => 'keep-alive',
            'Accept-Encoding' => 'gzip, deflate',
            'Accept-Language' => 'en-GB;q=1, fr-FR;q=0.9',
            'platform' => 'ios',
            'Content-Type' => 'application/json',
            'User-Agent' => 'Tinder/4.7.2 (iPhone; iOS 9.2.1; Scale/2.00)',
            'Connection' => 'keep-alive',
            'X-Auth-Token' => TOKEN,
            'os_version' => '90000200001' }

# Here you should specify the path of your matches.json file
def matches
  JSON.parse(File.read('matches.json'))['matches']
end

def new_matches
  matches.select { |m| !m['pending'] && m['messages'].size == 0 }
end

new_matches.each_with_index do |m, n|
  puts "sending message ##{n} to #{m['person']['name']} id (#{m['person']['_id']})"
  res = HTTP.headers(HEADERS)
        .post("https://api.gotinder.com/user/matches/#{m['_id']}",
              json: { message: "Hello #{m['person']['name']}. If I had a flower for every time I thought of you... I could walk through my garden forever." })

  puts res.body
  puts res.status == 200 ? 'OK' : 'FAILED'
  sleep(2)
end
{% endhighlight %}

This is quite straightforward ruby code. I use the `http` gem because I never remember how to use the native [Net::HTTP library](http://ruby-doc.org/stdlib-2.3.0/libdoc/net/http/rdoc/Net/HTTP.html "Ruby Net::HTTP reference"). I let the thread sleep for 2 seconds between each request just in case they have some kind of request rate/throttling protection.

Save this code to a file, ie. `tinder.rb`. Don't forget to set your token at the top of the script and to customize your message.

Execute the ruby script to send out the messages:
{% highlight bash %}
ruby tinder.rb
{% endhighlight %}

Voilà !

## Conclusion

This is a simple demonstration on how we can leverage [reverse engineering](https://en.wikipedia.org/wiki/Reverse_engineering "reverse engineering on Wikipedia") to unlock features that are not accessible through a mobile app. The data we get from the API calls also give us more information than the app, for example, we can see the last ping date of the match or its birthday date... That could unlock more potential for further hacking, but use it wisely :smile:
