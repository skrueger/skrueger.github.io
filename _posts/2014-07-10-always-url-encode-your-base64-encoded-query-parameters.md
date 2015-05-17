---
layout: post
---
<h2 class="post-title">Always URL encode your Base64 encoded query parameters</h2>
<p class="post-meta">Thursday, July 10th 2014</p>
<blockquote><p>"I have not failed. I've just found 10,000 ways that won't work."</p><p><small>Thomas A. Edison</small></p></blockquote>
<p>I spent a large portion of the day at work debugging an issue where I did not URL encode a Base64 encoded value I was sending to a server as a query parameter value.  I want to make sure I never waste time on this issue again.  I am writing a basic explanation of the problem and how to avoid it as reminder to myself.  I hope this may help some one else in the future too.</p>
<p>Base64 encoded strings can contain the <code>+</code> character. If the <code>+</code> character is placed on a URL query parameter, it is interpreted as a space.  This is problematic because a space (<code> </code>) character is NOT a <code>+</code>. In fact, the space (<code> </code>) character is <b>not</b> a valid Base64 encoding character.</p>
<p>To circumvent this behavior make sure you URL encode the Base64 encoded value so that <code>+</code> characters are encoded as <code>%2B</code>.</p>
<p>For example, say I have to send <code>QUJD+REVG+0hJY=</code> to a server. If I put this on a URL <i>without</i> URL encoding the value, the URL will look like this: <code>http://simonkrueger.com?q=QUJD+REVG+0hJY=</code>. This URL is perfectly valid, but when a request with this URL is sent to a server the query parameter, <code>q</code>, is seen with a value of <code>QUJD REVG 0hJY=</code>. This is <i>not</i> what I wanted. Do you <i>see</i> those spaces? </p>
<p>To fix this, I can URL encode this value <i>before</i> it is put on the URL. The URL now looks like this: <code>http://simonkrueger.com?q=QUJD%2BREVG%2B0hJY=</code>. Now the server sees <code>q</code>'s value as <code>QUJD+REVG+0hJY=</code>. Exactly what I wanted :).</p>
<p>This is a pretty simple mistake that I am sure has been made a million times. I hope to not to make it a million and one. Anyway, always URL encode your <s>Base64 encoded</s> query parameters!</p>
