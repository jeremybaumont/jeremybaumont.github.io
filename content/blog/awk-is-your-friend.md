+++
date = "2017-07-02T15:25:53+10:00"
draft = false 
title = "Awk is your friend"
description = "Awk is an amazing operation tool."
tags = [ "awk", "ops", "tool", "unix", "log" ]
topics = [ "awk", "ops" ]

+++

## 9-iron

I remember at school doing a golf initiation course. It was fun to try a new sport, specially one that was not accessible for the wallet of my parents. I remember the first lesson where the teacher asked us to take one and only one club to execute a 9 holes course. I chose one of those wood club probably biased by [Goofy's image playing golf](http://a.dilcdn.com/bl/wp-content/uploads/sites/2/2014/08/golf.jpg). The teacher took a 9-iron. It was the best trade off from the teeing ground to the putting green even if you had to pass by a bunker. The lesson was to appreciate the values of your tools, implicitly and without knowing it, I learn this day the [pareto principle](http://en.wikipedia.org/wiki/Pareto_principle). If you need to invest 20% of your time to make 80% of results: choose wisely your tool.

I like to think that awk is a 9-iron. It is a great tool I use daily for many different kind of troubleshooting. I am learning every day new trick and it saves me a lot of hassles. 

Awk is made to search files for lines that include specific patterns (awk gives you the liberty to define the granularity that best suits your search so not only lines). When a line matches one of those patterns, awk executes associated actions on that line then it continues to process the rest of the file the same way. I won't go further explaining how patterns and actions work because there are plenty of excellent [Awk blog posts](http://http://awk.info/?Learn) out there that does the job perfectly, personnaly I would recommand the [awk in 20 minutes](htp://ferd.ca/awk-in-20-minutes.html) to get started. Just basic principles should be enough to follow the rest of this post.

## Double standard 

I recently moved down under from Europa and the lack of standard in different
industries is as a well-known burden: voltage adaptor, cup to gram conversion table, 
specific cable to load its battery... 
In software engineering, there are [more standards](https://xkcd.com/927/) than colors in the rainbow for almost every topic.
Date format is one of this 
good source of headache, notably the conversion epoch time to usual date format. Awk 
can easily convert one format to another or from one timezone to another:

```bash
$ tail -2 nagios.log

[1424893454] SERVICE NOTIFICATION ...
[1424893456] Warning: ... 
```

```bash
$ tail -2 nagios.log | awk -F'[\\[\\]]' \
> '{  cmd = "/bin/date -d @"$2" +%Y%m%d_%H:%M" ; \
> $1=$2=""; \
> if (cmd | getline var ) print var, $0; \
> close(cmd) }'

20150225_20:44    SERVICE NOTIFICATION: ... 
20150225_20:44    Warning: ... 
```

As an awk script, it will be translated as the following:
```awk
#!/bin/awk -f

BEGIN {
    FS='[\\[\\]]';
    }

{  
    cmd = "/bin/date -d @"$2" +%Y%m%d_%H:%M" ; 
    $1=$2=""; 
 
    if (cmd | getline var ) print var, $0; 
    close(cmd) 
 }
```

The ```BEGIN``` block define actions that will be executed before reading any input, it
specifies here the ```FS``` field separator with a regular expression that
means either the opening or closing brackets character: *'['* or *']'*. So the
lines will be cut in fields every time awk meets those 2 characters and
that is because the epoch time in nagios log format is between those characters,
we will always find it in the second field defined by built-in variable ```$2```. 

The following block defines a rule, where the pattern is absent so this means it will
match every line and associated actions will be applied to every lines. The ```cmd | getline var``` 
is a [well-known tip](http://awk.info/?tip/getline) in order to execute a
command on a specific field, the common canvat for getline is:
```
cmd="some command" 
do something with cmd 
close(cmd)
```

So we extract the epoch time in ```$2```, compute a human readable date with ```cmd```, print
it with ```var``` and the rest of the line with ```$0``` where we took care to
empty ```$1``` and ```$2```.

Those days most of us are on Splunk or Kibana/ELK, these tools are amazing and sky is 
the limit, logs become more and more parsed and processed for a purpose but I have never regret
investing time to learn a bit of AWK, it saved my ass several times.

## Links:
* [Awk blog post](http://ferd.ca/awk-in-20-minutes.html) that gives you the essentials to get you started with awk. 
* [Gawk FSF manual](https://www.gnu.org/software/gawk/manual/gawk.html) is a must read is you are enthusiast and serious with awk.
* [Awk related website](http://awk.info/) with plenty of resources.
