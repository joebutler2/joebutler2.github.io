---
layout: post
title:  "Private Constants in Ruby"
categories: article 
tags: 
date: 2011-08-20
---

While implementing a new feature the other week, I came across a piece of code that wasn't adhering to the principle of "information hiding". This rule was violated by one object reaching into a different class to grab a constant. This is a common code smell and leads to tightly coupled code.

This design flaw occurred when a new feature was requested. It required that piece of data to be accessed elsewhere in the system. The resulting code sacrificed good design for ease of implementation. A better design would hide the constant behind a public method, making the code less brittle. Encountering this design flaw made me wonder if Ruby supported private constants.

The short answer is no, at least not yet in Ruby 1.9.2. There is a [support ticket](http://redmine.ruby-lang.org/issues/3908) issued and it is planned for Ruby 1.9.x. But that doesn't help us solve the problem here and now. One idea that popped into my head was to use a class method rather than a constant, and then call private_class_method to limit access to it. To illustrate this idea we'll use an example of a computer-controlled opponent in a video game.

    class CPUOpponent
      def self.base_points
        1000
      end
      private_class_method :base_points
    end
    
    puts CPUOpponent.base_points
    # => NoMethodError: private method ‘base_points’ called for CPUOpponent:Class

As you can see from the output we are unable to access the base_salary without resorting to meta-programming techniques. Drawing a line in the sand like this can protect objects from becoming tightly coupled. 

Of course, this is a simple case so what problems do we start to run into when really putting this technique to use?

Continuing with the video game idea, you might want to have different base points based on the difficulty of the game. Updating the model to use a hash to map the base points to the different difficulty ranges.

    def self.base_points
      @base_points ||= { :easy => 1000, :medium => 2000, :hard => 3000 }
    end

Notice we store the value in an instance variable by using an or-equals. Also known as a nil guard. Nil guards limit the assignment to happen only once, making it behave more like a constant. This introduces a problem, someone could manipulate the value.

    class CPUOpponent
      def self.base_points
        @base_points ||= { :easy => 1000, :medium => 2000, :hard => 3000 }
      end
      private_class_method :base_points
      
      base_points[:super_hard] = 9001
      puts base_points.inspect
      # => {:medium=>2000, :hard=>3000, :super_hard=>9001, :easy=>1000}
    end

This obviously goes against the immutability behavior that constants have. Fortunately Ruby gives a an easy way to work around this issue, the freeze method.

    class CPUOpponent
      def self.base_points
        @base_points ||= { :easy => 1000, :medium => 2000, :hard => 3000 }.freeze
      end
      private_class_method :base_points
    
      base_points[:super_hard] = 9001
      puts base_points.inspect
      # => TypeError: can't modify frozen hash
    end

Now it is starting to behave more like a constant with the added bonus of being private and having access to other class members. Note that if you run this example in the same file as the previous example it will not throw an error. It seems to be a minute detail with the open class language feature. 

While this makeshift constant is starting to act like the real thing, there is one more problem that you might encounter when using this technique, using it inside of instance methods. To tackle this problem we're going to use a simple meta-programming technique, the send method.

    class CPUOpponent
      def self.base_points
        @base_points ||= { :easy => 1000, :medium => 2000, :hard => 3000 }.freeze
      end
      private_class_method :base_points
    
      def calculate_score(bonus_points)
        base_points = self.class.send(:base_points)
        bonus_points + base_points[:easy]
      end
    end
    
    opponent = CPUOpponent.new
    puts opponent.calculate_score(400)
    # => 1400
  
The send method allows us to bypass the `private` rule and access the method. Now I agree that this may seem a little hacky, but considering that our goal with this approach is to mimic a private constant (or a static constant, if you’re comparing it to a statically typed language), I feel that this is justifiable. It achieves the goal of being encapsulated by only providing access to instances of the class.

Of course, you should always weigh the pros and cons of a technique like this before using it. Given the trade-offs, it does seem to be a worthwhile idea to keep in mind.

I'm interested to hear what other people think. Do you think there are too many workarounds, or it seems too hacky? Do you see another way to tackle the problem or have you actually used something similar before?

Download code examples