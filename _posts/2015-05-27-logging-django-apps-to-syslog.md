---
layout: post
---
<h2 class="post-title">Logging django apps to syslog</h2>
<p class="post-meta">Wednesday, May 25th 2015</p><p>
I spent most of my day filling in the holes of my django/python logging knowledge.
I've never done logging in python or a django app before. And thus our code base, had zero logging in it.
A huge problem when it comes time to debug issues in production.
I read a few articles that I found at <a href="http://www.fullstackpython.com/logging.html">Full Stack Python</a> and 
also went through the <a href="https://docs.python.org/2.7/howto/logging.html">Python doc logging tutorials</a>.
<a href="https://hynek.me/articles/taking-some-pain-out-of-python-logging/">Taking Some Pain out of Python Logging</a>
was an eye opener when Hynek explained that <a href="http://en.wikipedia.org/wiki/Syslog">syslog</a>
can capture and record the python log records. Syslog was exactly
what I set out to do (I didn't know it was called syslog. I just knew about /var/logs, so today was a good learning experience). 
I am not doing exactly what Hynek does. But it accomplishes my goal of getting our logs into syslog.
</p>
Here is the final LOGGING dictionary in settings.py I went with.
{% highlight python %}
# settings.py
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'filters': {
        'require_debug_false': {
            '()': 'django.utils.log.RequireDebugFalse',
        },
        'require_debug_true': {
            '()': 'django.utils.log.RequireDebugTrue',
        },
    },
    'formatters': {
        'verbose': {
            'format': '%(process)-5d %(thread)d %(name)-50s %(levelname)-8s %(message)s'
        },
        'simple': {
            'format': '[%(asctime)s] %(name)s %(levelname)s %(message)s',
            'datefmt': '%d/%b/%Y %H:%M:%S'
        },
    },
    'handlers': {
        'console': {
            'level': 'DEBUG',
            'filters': ['require_debug_true'],
            'class': 'logging.StreamHandler',
            'formatter': 'simple'
        },
        'syslog': {
         'level': 'DEBUG',
         'class': 'logging.handlers.SysLogHandler',
         'facility': 'local7',
         'address': '/dev/log',
         'formatter': 'verbose'
       },
    },
    'loggers': {
        # root logger
        '':{
            'handlers': ['console', 'syslog'],
            'level': 'INFO',
            'disabled': False
        },
        'thingsforwork': {
            'handlers': ['console', 'syslog'],
            'level': 'DEBUG',
            'propagate': False,
        },
    },
}
{% endhighlight %}
Ubuntu 14.04 uses <a href="http://www.rsyslog.com/">the rocket-fast system for log processing (rsyslog)</a>
and we need to tell it about our local7 facility. 
I got this one from <a href="http://serverfault.com/questions/45402/add-a-local-application-to-syslog-excluded-from-var-log-messages">serverfault</a>.
<pre>
# /etc/rsyslog.d/30-thingsforwork.conf
local7.*                                             /var/log/thingsforwork.log
& ~  # This stops local7.* from going anywhere else.
</pre>
Do a <code>sudo service rsyslog restart</code> so rsyslog.d can pickup the new config. And ta-da!
<pre>
# /var/log/thingsforwork.log
May 28 03:19:57 vagrant 11874 140156620121856 thingsforwork.dashboard.views.home                 DEBUG    we are in the home page, this is a debug
May 28 03:19:57 vagrant 11874 140156620121856 thingsforwork.dashboard.views.home                 INFO     We are logging django! e01be3ae-e9d1-49d7-97ac-00c5454f3026
May 28 03:20:02 vagrant 11874 140156620121856 requests.packages.urllib3.connectionpool           INFO     Starting new HTTP connection (1): dashboard.reviewpush.com
May 28 03:20:02 vagrant 11874 140156605413120 requests.packages.urllib3.connectionpool           INFO     Starting new HTTPS connection (1): tartan.plaid.com
May 28 03:20:02 vagrant 11874 140156392699648 googleapiclient.discovery                          INFO     URL being requested: GET https://www.googleapis.com/discovery/v1/apis/analytics/v3/rest
May 28 03:20:02 vagrant 11874 140156375914240 requests.packages.urllib3.connectionpool           INFO     Starting new HTTPS connection (1): quickbooks.api.intuit.com
</pre>
