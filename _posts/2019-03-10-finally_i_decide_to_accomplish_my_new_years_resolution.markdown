---
layout: post
title:      "defining my prime number methods and compare them "
date:       2019-03-10 03:54:35 -0400
permalink:  finally_i_decide_to_accomplish_my_new_years_resolution
---

A prime number is a whole number greater than 1 whose only factors are 1 and itself. When I try to finish a method to check if a number is a prime number, it is very easy to come up with the solution to try to divide the number n with numbers in a range of 2 to (n-1). So the basic idea is to find factors other than 1 and the number itself within 2 to (n-1) . And yes, that is the first way I can think of to deal with the problem. Here is my code:

```

def prime0?(number)
  if number<2
     false
    else
      i=2
     while i<number
       if number%i==0
         return false
      end
     i=i+1
   end
   true
 end
  end
```

And yes, it works just fine! But then when I look at the code, I really think it is not efficient enough. And then, the following idea occur to me that for sure if a number is not a prime number, in another way, a composite number, then one of the factors will fall in [2,n/2] since factors always come up as a pair, one smaller than n/2, the other bigger. So I only have to check in the range of [2,n/2], which reduces half of the work that my codes need to do. Here is how I inplement my second thought:


```
def prime1?(number)
  if number<2
     false
    else
      i=2
     while i<=number/2
       if number%i==0
         return false
      end
     i=i+1
   end
   true
 end
  end

```

Even it works, however, I still try to find a better solution to the problem. It is then when the idea stucked me: will that be a lot of waste if I start the counter i as 2, and then increment it by one everytime? Think about it, if the number n can be divided by 4, for sure it can be divided by 2, right? So that means, if the number, from the very beginning cannot be divided by 2, then we can save our time by setting i as 3, and increment it by 2, saving the trouble to check the even numbers. So here is my code for it :
```
def prime2?(number)
if number<2||number==4
   false
 elsif number==2
    true
   else
     i=3
     while i<=number/2
     if number%i==0
       return false
    end
   i=i+2
 end
 true
end
end
```
But also, I am not sure if you have noticed, I made more than changing i to the code this time. Since the condition for my `while` loop is` i<=number/2`, so for number 4, whose half is 2,  smaller than what the i is set initially, right after the number 4 gets to the` if` statement, it will jump over it and jump to the end of the `while` loop, which is true. And obviously, that is not true. That is why I have to hard code it in the first` if` statement: 

```
if number<2||number==4
   false
	 
```

Ok, so I test it and this method is fine too. And here is how I test if I am right about the prime2 method is faster than the prime1, which is faster than prime0. I use the Benchmark#bm method to help me, after reading this article from Jesse Storimer (http://rubylearning.com/blog/2013/06/19/how-do-i-benchmark-ruby-code/). So basically, I simply define 3 methods to check prime numbers within [2,10000] and then use the #bm method to generate reports side by side, showing me how fast each method will take and thus easier to compare which is more efficient. Here are my codes：

```
def checkprime0
  i=2
  while i<10000
    prime0?(i)
    i=i+1
  end
end

def checkprime1
  i=2
  while i<10000
    prime1?(i)
    i=i+1
  end
end

def checkprime2
  i=2
  while i<10000
    prime2?(i)
    i=i+1
  end
end
  Benchmark.bm do |bm|
  bm.report { checkprime0}
  bm.report { checkprime1}
  bm.report { checkprime2}

end
```
The following are the results:

```
 user     system      total        real
   0.244000   0.004000   0.248000 (  0.246886)
   0.164000   0.000000   0.164000 (  0.163082)
   0.144000   0.000000   0.144000 (  0.145867)
```
And if I try to make it looks more different, I can try to make a wider range to [2,100000], and the result is:

```
user     system      total        real
  19.628000   0.000000  19.628000 ( 19.635854)
  12.876000   0.000000  12.876000 ( 12.875490)
  11.240000   0.000000  11.240000 ( 11.241820)
```

Yes, the results, compared to each other, have bigger differences now! So at least now I am sure I make the right guess about which one will be faster(even not much difference between the second and the third one). And I am glad that I have tried, that is very fun!

So in the end, thanks for reading my blog. I am still new to coding so what i write above is not perfect. Therefore, if you have more suggestions on how to check prime number faster(I am sure you will!) or if you think anything is wrong in this blog, please do email me so I can correct it in this blog! Thanks!
My E-mail  is [chanwingkeihaha@gmail.com](http://). 
    
