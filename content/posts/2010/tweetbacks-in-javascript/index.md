Tweetbacks in JavaScript
========================
categories: [web]
posted: 2010-12-16
snip: A wordpress widget that uses topsy to show all of the twitterverse mentions
  of your blog posts.



Every new post on this blog gets an unsolicited [pingback][] from
[topsy][], a service that tracks which users mentioned the post on
twitter (known informally as [tweetbacks][]). On one hand, topsy is a
parasite, using sites like mine to rise in search engine rankings, but
on the other, it satisfies this blogger's curiosity to learn who reads
and enjoys my posts enough to tweet about them. In this experiment, I've
exposed tweetbacks directly on this blog using topsy's convenient 
[JSONP API][].

The implementation is entirely in JavaScript, consisting of two
scripts. The first is on the main post listing, showing the number of
tweets for each post using `http://otter.topsy.com/stats.js`. The second
is on the post page itself, showing the tweets posted in response to the
post using `http://otter.topsy.com/trackbacks.js`. Both scripts execute
after the page loads, so the only impact on page load time is the extra
kilobyte of JavaScript code. 

There are good reasons to use topsy over [twitter search][], even though
at first glance, they both provide similar information:

-   Twitter search doesn't index old tweets
-   Topsy distinguishes influential tweets (more on this later)
-   JSONP API support

It's important not to overwhelm readers by showing too many tweets.
Popular blog posts can have hundreds or even thousands of tweets, and
showing all of them at once is a bad idea. Topsy associates an influence
value with each twitter user, giving them a certain weight, making it
easier to decide which tweets to show, and which to hide. My
implementation shows at most N tweets; if there are over N total tweets,
it shows at most N 'influential' tweets, as per topsy's definition.

    var MAX_TWEETS = 10;
    var BASE = 'http://otter.topsy.com/trackbacks.js?callback=?&perpage=' + 
      MAX_TWEETS;
    var ALL = BASE + '&url=';
    var INFL = BASE + '&infonly=1&url=';
     
    function getTweets(url) {
      $.getJSON(ALL + url, function(data) {
        var response = data.response;
        if (response.total > MAX_TWEETS) {
          $.getJSON(INFL + url, function(infl) {
            processTweetList(infl.response.list);
            var count = (infl.response.total > MAX_TWEETS ? MAX_TWEETS : infl.response.total);
            updateTweetCount(count, response.total);
          });
        } else if (response.total > 0) {
          processTweetList(response.list);
          updateTweetCount(response.total);
        }
      });
    }

I also urlify the
tweet, converting all of the URLs and @mentions into <a\> elements using
this function, I found the first regular expression somewhere on the
internet – I'd never write such a beast. 

    var URL_RE = /(\b(https?|ftp|file):\/\/[-A-Z0-9+&@#\/%?=~_|!:,.;]*[-A-Z0-9+&@#\/%=~_|])/ig;
    var TWEET_RE = /@([A-Za-z0-9_]+)/g;
     
    function urlify(text) {
      return text.replace(URL_RE,"<a href='$1'>$1</a>").
                  replace(TWEET_RE, "<a href='http://twitter.com/$1'>@$1</a>");


The approach I took has some major advantages to over the [tweetback plugin][] 
for wordpress:

-   It works... the wordpress plugin doesn't
-   No wordpress comment pollution since no wordpress comments are
    created
-   No server-side load since all comments are fetched in JS
-   No wordpress required. Other blog engines or static sites work just
    as well

A drawback of this approach is that it takes some time to run the JS,
which changes the DOM after the initial page load, resulting in a
jarring experience. By using a fade effect to make twitter information
appear gradually, I try to mitigate this problem. 

I conclude with a shameless plug: tweetbacks.js is running on this blog,
so try it out by posting a tweet referring to this post's URL, and it
should appear in the list below, or if there are more than 10 tweets, at
least increment the count. If you find this concept interesting and
would like to run it on your site, let me know and I'll pack
tweetbacks.js up into a jQuery plugin or something. Thanks for reading! 

**Update:** I no longer use this code, in favor of the official twitter
button.

  [pingback]: http://en.wikipedia.org/wiki/Pingback
  [topsy]: http://topsy.com
  [tweetbacks]: http://tomsucks.wordpress.com/2008/05/27/free-idea-tweetback/
  [JSONP API]: http://otter.topsy.com
  [twitter search]: http://search.twitter.com
  [tweetback plugin]: http://yoast.com/wordpress/tweetbacks/
  [topsy-twitterized files]: topsy-tweetback.zip

