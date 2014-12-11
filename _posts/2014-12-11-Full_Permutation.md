


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
