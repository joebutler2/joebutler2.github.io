---
layout: post
title:  "Private Constants in Ruby"
categories: article 
tags: 
date: 2011-08-20
---

While implementing a feature the other day, I noticed a piece of code that was reaching into another class to access a constant. This would be okay for an API like `Math.PI`, but in this circumstance the calling code was tightly-coupled to the class containing the constant. Remember, we should aim for "low coupling and high cohesion". That way we can change one part of the system without hurting another.

This is solvable by hiding the constant behind a public method and then update the calling code to use the new method. This reduces coupling and makes the system more maintainable. For example, you can extend the logic of the new method without changing the calling code.

You might wonder how we can ensure the calling code doesn't access the constant directly. Can we use the `private` keyword to limit access to constants?

The short answer is no, at least not yet in Ruby 1.9.2. The core team is planning to add it soon; you can see the [support ticket](http://redmine.ruby-lang.org/issues/3908) for details. So, is there a way we can emulate a private constant today?
 
One idea that popped into my head was to use a class method rather than a constant, and then call `private_class_method` to limit access. To illustrate this idea, we’ll use an example of a computer-controlled opponent in a video game.

{% gist 4eb675b839856724d25bf0a5881caf98 0_simple.rb %}

As you can see from the output, we are unable to access the `base_points` method. If you're an experienced rubyist, you might think "well, someone could use meta-programming to access the method". While this is true, most developers will respect the private access modifier.

This is a simple case, so what problems do we run into when putting this technique to the test?

Continuing with the video game motif, let's add different base points based on the difficulty of the game. This will require updating the method to use a hash, it will map the difficulty level to the number of points.

{% gist 4eb675b839856724d25bf0a5881caf98 1_base_points_with_hash.rb %}

Notice we store the value by using the `||=` operator. Rubyists' call this idiom a nil guard. They limit the assignment to occur only once if given a "truthy" value, because of this, our `base_points` method behaves more like a constant.

Upon close inspection, this introduces a problem, the return value of `base_points` can be manipulated.

{% gist 4eb675b839856724d25bf0a5881caf98 2_private_class_method.rb %}

This obviously goes against the immutable behavior that constants have. Fortunately Ruby gives us an easy way to solve this issue, the `freeze` method.

{% gist 4eb675b839856724d25bf0a5881caf98 3_frozen.rb %}

We are getting closer to having `base_points` acting like a constant. There is the bonus of being private and having access to other class members. Note, running this example in the same file, as the previous example, will not throw an error. Most likely due to ruby's "open class" feature.

While this makeshift constant is acting like the real thing, there is one more problem you might encounter when using this technique, using it inside of instance methods. To tackle this problem, we will use a simple meta-programming technique, the `send` method.

{% gist 4eb675b839856724d25bf0a5881caf98 4_final.rb %}
  
The send method allows us to bypass the private rule and access the method. Now I agree this may seem a little hacky, but considering our goal is to mimic a private constant, this is justifiable. It achieves the goal of hiding information by only providing access to instances of the class.

You should always weigh the pros and cons of a technique like this before using it. Given the trade-offs, it seems to be a worthwhile idea to keep in mind.

I'm interested to hear what other people think. Does this have too many workarounds? Or seem too hacky? Do you see another solution? Have you used something similar before?

[Checkout all code examples](https://gist.github.com/joebutler2/4eb675b839856724d25bf0a5881caf98)