---
layout: post
title: Different solutions of getting max prime in the given range
tags: Algorithm Python
categories: practical python algorithm
---

Challenged again and again by the interviewers since my poor solution of algorithm >_<. here I am coding out the solutions of getting max prime in the given range.


Support functions
-----------------

Import modules, implemented *timeit* function:

        import time
        import math
        
        def timeit(method):
            def calculate(*args, **kwargs):
                print 'Start running function %s' % method.__name__
                start = time.time()
                result = method(*args, **kwargs)
                end = time.time()
                print 'End and cost %f' % (end-start)
                return result
            return calculate


Solution 1: Record the history primes(sacrificing space for performance)
---------------------------------------------------------------------------------------
blabla

        RANGE = 1000

        @timeit
        def record_history_prime(length):
              if length < 2:
                  return None
              
              history = [2]
              if length == 2:
                  return history

              for i in xrange(3, length):
                   if _is_prime(history, i):
                       history.append(i)
              return history[-1]

        def _is_prime(history, num):
              for i in history:
                  if num % i == 0:
                      return False
              for i in xrange(history[-1]+1, int(math.sqrt(num))):
                  if num % i == 0:
                      return False
              return True


        if __name__ == "__main__":
            primes = record_history_prime(RANGE)
