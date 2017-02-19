---
layout: post
title: 基數排序（正整數）
date: 2017-02-19 19:50
comments: true
external-url:
categories: sort
---

Radix Sort!

用 ZeroJudge 上的題目測一下正確性。

## 傳送門
[ZJa233](https://zerojudge.tw/ShowProblem?problemid=a233)

## 限制

<center>
$$1 \leq N \leq 1000000$$
</center>
測資會卡教科書上的 Quick Sort

## 程式碼:
其實是亂寫的~~

{% highlight cpp linenos=table %}
#include <bits/stdc++.h>
using namespace std;

inline void rxd(vector<unsigned int> &arr) {
    unsigned int maxv = *max_element(arr.begin(), arr.end());
    if(maxv==0) return;
    maxv = 32-__builtin_clz(maxv);
    vector<unsigned int> bucket[2];
    for(unsigned int d=0; d<maxv; ++d) {
        bucket[0].clear(); bucket[1].clear();
        for(vector<unsigned int>::iterator i=arr.begin();\
            i!=arr.end(); ++i) {
            bucket[ ((*i)>>d)&1u ].push_back(*i);
        }
        arr.clear();
        #define UPDATE(X,Y) \
            X.insert( X.end(), Y.begin(), Y.end())
        UPDATE(arr, bucket[0]);
        UPDATE(arr, bucket[1]);
    }
}

vector<unsigned int> arr;

int main(void) {
    ios::sync_with_stdio(false);cin.tie(0);
    int N;
    while(cin>>N) {
        for(int i=0; i<N; ++i) {
            unsigned int temp; cin>>temp;
            arr.push_back(temp);
        }
        rxd(arr);
        vector<unsigned int>::iterator i=arr.begin();                          
        cout<<*i;
        for(++i; i!=arr.end(); ++i) cout<<' '<<*i;
        cout<<'\n';
        arr.clear();
    }
    return 0;
}
{% endhighlight %}

順手改一下，讓他也支援 int

{% highlight cpp linenos=table %}
#include <bits/stdc++.h>
using namespace std;

inline void rxd(vector<unsigned int> &arr) {
    unsigned int maxv = *max_element(arr.begin(), arr.end());
    if(maxv==0) return;
    maxv = 32-__builtin_clz(maxv);
    vector<unsigned int> bucket[2];
    for(unsigned int d=0; d<maxv; ++d) {
        bucket[0].clear(); bucket[1].clear();
        for(vector<unsigned int>::iterator i=arr.begin();\
            i!=arr.end(); ++i) {
            bucket[ ((*i)>>d)&1u ].push_back(*i);
        }
        arr.clear();
        #define UPDATE(X,Y) \
            X.insert( X.end(), Y.begin(), Y.end())
        UPDATE(arr, bucket[0]);
        UPDATE(arr, bucket[1]);
    }
}

void split_rxd(vector<int> &arr) {
    vector<unsigned int> pos, neg;
    for(vector<int>::iterator i=arr.begin(); i!=arr.end() ; ++i ) {
        if(*i<0) neg.push_back((unsigned int)-*i);
        else pos.push_back((unsigned int)*i);
    }
    arr.clear();
    if(!pos.empty()) rxd(pos);
    if(!neg.empty()) rxd(neg);
    for(vector<unsigned int>::reverse_iterator i=neg.rbegin();\
        i!=neg.rend(); ++i) {
        arr.push_back((int)(-*i));
    }
    neg.clear();
    arr.insert(arr.end(), pos.begin(), pos.end() );
    pos.clear();
    //end of sorting
}
vector<int> arr;

int main(void) {
    ios::sync_with_stdio(false);
    //cin.tie(0);
    int N;
    while(cin>>N) {
        for(int i=0; i<N; ++i) {
            int temp; cin>>temp;
            arr.push_back(temp);
        }
        split_rxd(arr);
        vector<int>::iterator i=arr.begin();
        cout<<*i;
        for(++i; i!=arr.end(); ++i) cout<<' '<<*i;
        cout<<'\n';
        arr.clear();
    }
    return 0;
}
{% endhighlight %}

順便附上之前亂寫的 Quick-Sort (3-way partition)
{% highlight cpp linenos=table %}

#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <limits.h>
#include <string.h>
#define MAX 1000000

/*
*   Sorting Practice from 581 course;
*   Author: Kuang-Yu Jeng;
*   Date: 2016/5/23;
*	Note: <stdio.h>, <string.h> and <time.h> are needed; So does the custom comparison function ;)
*         And it's better to include <limits.h> too.
*/
#if RAND_MAX==32767
#define ENLARGE_RAND
#endif // RAND_MAX

#ifndef INCLUDE_SORT
#define INCLUDE_SORT

/*It's seems that Heap-sort does not help...*/
#if 0
#define ENABLE_HEAP
#endif

#ifndef S_SLOW_MODE
#define ENABLE_SHORT
#define ENABLE_INSERT
#define NOMID
#endif

#ifdef ENABLE_HEAP
#ifndef GROUP
#define GROUP 15 //Default Heap size-1
#endif
#endif // ENABLE_HEAP

void real_sort(int left, const int right, char *ptr, const size_t block, const size_t word_len, int (*cmp)(const void *, const void *))
{
    //if(left>=right)return;/*The size of partition <= 0, exit.*/
    int i, j, rd_m;
    register char *t2;
    int k, pa;
    int left_p, right_p;
    char *temp;
    while(left<right)
    {

#ifdef ENABLE_SHORT
        if( right - left == 1 ) //If there are only 2 elements, no sorting is needed.
        {
            temp = (char*)malloc(word_len);/*Allocate temporary variable.*/
            if( cmp(ptr+left*block, ptr+right*block) > 0 )
            {
                memcpy(temp, ptr+left*block, word_len);
                memcpy(ptr+left*block, ptr+right*block, word_len);
                memcpy(ptr+right*block, temp, word_len);
            }
            free(temp);
            return;
        }
#endif
#ifdef ENABLE_INSERT
        if( right - left < 10 ) // Less than 10 elements
        {
            temp = (char*)malloc(word_len);
            for(i=left+1; i<=right; i++)
            {
                memcpy(temp,ptr+i*block,word_len);
                j = i-1;
                while(j>=0 && cmp(temp, ptr+j*block) < 0)
                {
                    memcpy(ptr+(j+1)*block, ptr+j*block, word_len);
                    j--;
                }
                memcpy(ptr+(j+1)*block, temp, word_len);
            }
            free(temp);
            return;
        }
#endif

#ifdef ENABLE_HEAP
        if( right - left < GROUP ) //If there are GROUP elements, switch to heap sort
        {
            t2 = (char*)malloc(word_len);
            temp = (char*)malloc(word_len*(GROUP));
            j = -1;
            for(i=left; i<=right; i++)
            {
                memcpy(temp+(++j)*block,ptr+i*block,word_len);
                if(j==0)continue;
                k = j;
                while(k>0)
                {
                    pa = (k-1)>>1;
                    if(cmp(temp+k*block,temp+pa*block)<=0)
                    {
                        memcpy(t2,temp+k*block,word_len);
                        memcpy(temp+k*block,temp+pa*block,word_len);
                        memcpy(temp+pa*block, t2, word_len);
                        k = pa;
                    }
                    else break;;
                }
            }

            for(i=left; i<=right; i++)
            {
                memcpy(ptr+i*block, temp, word_len);

                pa = k = 0;
                if(j==-1)break;
                memcpy(temp,temp+j*block,word_len);
                j--;
                while((pa=(k<<1)+1) <= j)
                {
                    if(pa+1 <= j && cmp(temp+pa*block, temp+(pa+1)*block) > 0 ) pa+=1;
                    if(cmp(temp+k*block, temp+pa*block) > 0)
                    {
                        memcpy(t2, temp+k*block, word_len);
                        memcpy(temp+k*block, temp+pa*block, word_len);
                        memcpy(temp+pa*block, t2, word_len);
                        k = pa;
                    }
                    else break;
                }
            }

            free(temp);
            free(t2);
            return;
        }
#endif // 1

#ifdef S_SLOW_MODE
        rd_m = left;
#endif

#ifndef S_SLOW_MODE
#ifndef ENLARGE_RAND
        rd_m = left + 1 + (((int)rand())%(right-left-1)); //Get random pivot
#endif
#ifdef ENLARGE_RAND //Enlarge to 2147483647
        rd_m = left + 1 + ((int)((rand()<<16)|(rand()<<1)|rand()&1) % (right-left-1)); //Get random pivot
#endif
#ifdef MID
        /*Choose better pivot*/
        if( ( (k=cmp(ptr+rd_m*block,ptr+left*block)) < 0 && (pa=cmp(ptr+left*block, ptr+right*block)) < 0) ||\
                ( k>0 && pa>0 ) ) //Suppose LEFT is mid;
        {
            rd_m = left;
        }
        else if( ( (k=cmp(ptr+rd_m*block,ptr+right*block)) < 0 && (pa=cmp(ptr+right*block, ptr+left*block)) < 0) ||\
                 ( k>0 && pa>0 ) ) //Suppose RIGHT is mid;
        {
            rd_m = right;
        }
#endif
#endif
        temp = (char*)malloc(word_len);
        t2 = (char*)malloc(word_len);
        i = left;
        j = right;
        k = left+1;

        memcpy(t2, ptr+rd_m*block, word_len);
        memcpy(ptr+rd_m*block, ptr+left*block, word_len);
        memcpy(ptr+left*block, t2, word_len);
        while( k <= j )
        {
            if( (pa=cmp(ptr+k*block, t2)) < 0 )
            {
                memcpy(temp, ptr+k*block, word_len);
                memcpy(ptr+k*block, ptr+i*block, word_len);
                memcpy(ptr+i*block, temp, word_len);
                k++;
                i++;
            }
            else if( pa > 0 )
            {
                memcpy(temp, ptr+k*block, word_len);
                memcpy(ptr+k*block, ptr+j*block, word_len);
                memcpy(ptr+j*block, temp, word_len);
                j--;
            }
            else k++;
        }

        free(t2);
        free(temp);
        real_sort(left, i-1, ptr, block, word_len, cmp);
        left = j+1;
    }
    //real_sort(j+1, right, ptr, block, word_len, cmp);
}

void my_qsort(void *ptr, size_t n, size_t word_len, int (*cmp)(const void *, const void *))
{
    //Initialize random seed;
    srand((unsigned)time(NULL));
    real_sort(0, n-1, (char*)ptr, (word_len/sizeof(char)), word_len, cmp);//All cast to char;
}
#endif

int cmp(const void *a, const void *b)
{
    return *(int*)a - *(int*)b;
}

int main(void)
{
    int *arr = (int*)malloc(sizeof(int)*MAX);
    int n, i;
    while(scanf("%d", &n)!=EOF)
    {
        for(i=0; i<n; i++)scanf("%d", arr+i);
        my_qsort(arr,n,sizeof(int),cmp);
        printf("%d",arr[0]);
        for(i=1; i<n; i++)printf(" %d",arr[i]);
        printf("\n");
    }
    free(arr);
    return 0;
}

{% endhighlight %}
> 寫壞了

還有內建的 qsort
{% highlight cpp linenos=table %}

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#define MAX 1000000
int arr[MAX];

int cmp(const void *a, const void *b) {
    return *(const int*)a - *(const int*)b;
}

int main(void) {
    int n, i;
    while(scanf("%d", &n)!=EOF) {
        for(i=0; i<n; i++)scanf("%d", arr+i);
        qsort(arr,n,sizeof(int),cmp);
        printf("%d",arr[0]);
        for(i=1; i<n; i++)printf(" %d",arr[i]);
        printf("\n");
    }
    return 0;
}

{% endhighlight %}

## 速度
qsort 大概 0.1s

radix sort 大概 80ms

quick sort 大概 0.2s
