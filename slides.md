The Stack
=========
 &
===
The Hack
========

### Nathan Hessler

---

## The Stack

* FILO - First In Last Out
* LIFO - Last In First Out

---

## The Stack

### Method Scope
```ruby
class Person
  attr_accessor :first_name, :last_name
  #...
  def full_name
    # sorry, I know this can be interpolated
    first_name + ' ' + last_name
  end
end

person = Person.new("Nathan", "Hessler")

# Important part
person.full_name
```

---

## The Stack

### Stackoverflow

* The Stack is not infinite
* Recursion is often what *'blows'* the stack
* What is our stack size?

```shell
  ulimit -s
```

--- 

## The Stack

### Recursion

```ruby
# factioral(3) => 3 * 2 * 1 => 6
def factorial(n)
  return 1 if n <= 1
  n * factorial(n -1)
end
```
---

## The Stack

### Tail Call Recursion

```ruby
# factioral(3) => 3 * 2 * 1 => 6
def factorial(n) # maintain interface
  factorial_helper(1, n)
end

# never call directly
def factorial_helper(accumulator, n)
  return accumulator if n <= 1
  factorial_helper(accumulator * n, n - 1)
end
```
---

## The Stack

### Tail Opmtimization

**Explanation**: Since we make the recursive call last, there is no other work to be done or context to keep around. This means we don't have to keep the current frame around and can pop it before placing the recursive frame on the stack. This effectively makes the stack infinite in these cases.

**Notes**: This is usually implemented by functional Languages, and not by OO Languages.

---

## The Hack

### Ruby and Tail Optimization

* A Muggle's Guide To Tail Call Optimization
  * Talk by Danny Guinther at Rubyconf 2015
  * http://rubyconf.org/program#prop_1447
* Summary: Ruby has it
  * Not turned on by default
  * A bit finicky. Rails test fails with it on

--- 

## The Hack

### Insipiration: Clojure

* Java is OO and does not implement tail optimized recursion
* Clojure is functional and a Lisp and really wants it

```clojure
(defn factorial [n]
  (factorial-helper 1 n)))

(defn factorial-helper [acc cnt]
  (if (zero? cnt) 
    acc 
    (recur (dec cnt) (* acc cnt)))))
```

---

## The Hack

### How does this work?

* There are a couple possibilities..
* They could be messing with the AST, it's Lisp after all
* They could just be doing some stack management
* Both are options in ruby, but we'll focus on stack management

---

## The Hack

### Ruby Equivalent

```ruby
# factioral(3) => 3 * 2 * 1 => 6
def factorial(n) # maintain interface
  factorial_helper(1, n)
end

# never call directly
def factorial_helper(accumulator, n)
  return accumulator if n <= 1
  # replace tail call with recurse method call
  recurse(accumulator * n, n - 1)
end
```

---


## The Hack

### 3 Phases of Recurse

---

## The Hack

### Phase 1: The First Call

* determine calling method name
* add state management args to method arg list
* start loop to 'recurse'
* run calling method with new arg list

---

## The Hack

### Phase 2: The Inbetween Calls
* add state management args to arg list and return
* basically a short curcuit of the recursive call
* thus allowing the method frame to pop the stack

---

## The Hack

### Phase 3: The Last Call
* Never gets to recurse
* No stack management args get added to arg list
* return results from loop

---

## The Hack

### Recurse Defined

```ruby
def recurse(*args)
      return [:recurse, args] if instance_variable_defined? :@caller
      begin
        @caller = binding.of_caller(1).eval('__method__')
        caller_args = [:start, args]
        loop do
          caller_args = __send__ @caller, *caller_args.last
          return caller_args unless caller_args.respond_to? :first
          return caller_args unless caller_args.first == :recurse
        end
      ensure
        remove_instance_variable :@caller if instance_variable_defined? :@caller
      end
    end
```

---

## The Hack

### Torc Gem 0.1.0 
##### Tail Optimized Recursive Call

* Wanted to call it torch, but already taken
* Josh Cheek was a big help
* He also submitted a PR that mucks with AST
* No benchmarks, probably not faster
* But, we have an effectively infinite stack
* and beautiful tail recursion
* Probably not production worthy. Use at own risk

---

## The End

#### Nathan Hessler

* twitter: @spune
* github: nhessler
* employment: customink

---

## The Q&A

### THank You!
