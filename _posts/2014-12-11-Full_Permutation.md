---
layout: post
title: Implement full permutation in Python
tags: Algorithm python
categories: Algorithm python
---

算法题，利用Python实现一个全排列算法。要求：

1. 实现一个递归的全排列算法
2. 在1的基础上加上去重功能（有bug）
3. 实现一个非递归的全排列算法（未实现）


代码如下：


    def permutation(operate_list, m, n):
        if m == n-1:
            print operate_list
            return
        else:
            for i in range(m, n):
                operate_list[m], operate_list[i] = operate_list[i], operate_list[m]
                permutation(operate_list, m+1, n)
                operate_list[m], operate_list[i] = operate_list[i], operate_list[m]

    def deduplicated_permutation(operate_list, m, n):
        if m == n-1:
            print operate_list
            return
        else:
            for i in range(m, n):
                if is_swap(operate_list, m, i):
                    operate_list[m], operate_list[i] = operate_list[i], operate_list[m]
                    permutation(operate_list, m+1, n)
                    operate_list[m], operate_list[i] = operate_list[i], operate_list[m]

    # FIXME BUG
    def is_swap(operate_list, m, n):
        for i in range(m, n):
           if operate_list[i] == operate_list[n]:
                return False
        return True
    

    if __name__ == '__main__':
        test = ['a', 'b', 'c', 'd']
        permutation(test, 0, 4)

        test_duplicate = ['a', 'c', 'c']
        deduplicated_permutation(test_duplicate, 0, 3)
