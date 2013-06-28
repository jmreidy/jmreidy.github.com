---
title: attr_accessor Shortcut in Ruby
layout: post
comments: true
---
Pivotal Labs' blog provides one of the most consistent sources of quality code writing
in my RSS reader. Whether in their Daily Standup posts, or their longer-form articles,
I'm constantly finding tidbits of quality Ruby and JS code. A couple weeks ago, [Jeff
Dean wrote a solid piece on breaking out your form logic](http://pivotallabs.com/users/jdean/blog/articles/1706-form-backing-objects-for-fun-and-profit) in Rails models and controllers. If you're a
Ruby developer, you should read the post in entirety. What most stuck out to me, though,
was a little syntactic sugar that I hadn't enountered before for stating your public
attributes: 
``` ruby
ATTRIBUTES = [:name, :email, :terms_of_service, :anti_bot_question_id, :anti_bot_answer, :ip_address]

attr_accessor *ATTRIBUTES
```
I'll definitely be adding that to my shortcut list.

