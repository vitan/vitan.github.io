---
layout: post
title: Implement python stack in which the operation push/pop/max must match O(1) time complexity
tags: algorithm python
categories: algorithm python
---

直入主题，在python下利用list实现一个stack，要求操作push/pop/max的时间复杂度为O(1)。
问题的关键是怎么保证stack.max()的时间复杂度是O(1)，这里程序维护了一个__max的list：
 
 - 在push操作时，会比较\_\_max最后一个值与将要压入的值的大小，大于等于 则append到\_\_max中
 - 在pop操作时，会比较\_\_max最后一个值与弹出的值的大小，等于则remove掉\_\_max[-1]

类定义如下：

    class Stack(object):
        def __init__(self, size):
            self.__stack = []
            self.__max = []
            self.__size = size
            self.__top = -1

        def push(self, value):
            if self.is_full():
                raise "stack already full"
            else:
                self.__stack.append(value)
                self.__top += 1

                #Part-1: Tricky solution to match the O(1) max()
                if self.__max == [] or self.__max[-1] <= value:
                    self.__max.append(value)

        def pop(self):
            if self.is_empty():
                raise "stack is empty"
            else:
                value = self.__stack[-1]
                self.__top = self.__top - 1
                del self.__stack[-1]

                #Part-2: Tricky solution to match the O(1) max()
                if value == self.__max[-1]:
                    del self.__max[-1]

                return value

        def max(self):
            if self.is_empty():
                raise "stack is empty"
            else:
                #Part-3: Tricky solution to match the O(1) max()
                return self.__max[-1]

        def top(self):
            if self.is_empty():
                raise "stack is empty"
            else:
                return self.__stack[-1]

        def is_full(self):
            return True if self.__top+1 == self.__size else False

        def is_empty(self):
            return True if self.__top == -1 else False
