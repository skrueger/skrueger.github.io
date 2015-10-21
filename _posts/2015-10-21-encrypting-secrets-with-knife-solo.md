---
layout: post
---
<h2 class="post-title">Encrypting secrets with knife-solo</h2>
<p class="post-meta">Wednesday, October 21st 2015</p><p>
We've had some secrets laying around in our database that needed to be changed and encrypted.
We have been using chef to deploy our service. With chef data bags we can <a href="https://docs.chef.io/data_bags.html#encrypt-a-data-bag-item">encrypt</a> and decrypt our secrets.
There is one problem though. It expects you to have a chef server.
However, we are using a pretty simple setup with no chef server at the moment.
Luckily there is <a href="https://github.com/matschaffer/knife-solo">knife-solo</a> and <a href="https://github.com/thbishop/knife-solo_data_bag">knife-solo_data_bag</a>.
</p>
<p>Create a data bag named secrets with an item named database</p>
{% highlight bash %}
$ knife solo data bag create secrets database --json-file data_bags/secrets/database.json --data-bag-path chef/data_bags/ --secret-file ~/secrets_key
{% endhighlight %}

<p>Edit the existing data bag item.</p>
{% highlight bash %}
$ knife solo data bag edit secrets database --data-bag-path chef/data_bags/ --secret-file ~/secrets_key
{% endhighlight %}
