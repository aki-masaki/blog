+++
title = "math: sequences"
date = 2025-12-14
+++

i have a math textbook for grade 9 (im in grade 12 but i dont know most of the theory even from 9th grade), i opened it, looked at the table of contents and saw sequences (actually i saw _"șiruri"_ since its in romanian) and decided to learn them (and ofcourse write a blog post about them)

**note**: many chances are that im wrong in this post

if <math>f(x):\N^*\rightarrow E</math> then <math>f</math> is the sequence of the set <math>E</math>

the _rank_ of an element its basically its index in the sequence

sequences are commonly notated as <math>(a_n)_n</math> or simply <math>(a_n)</math>. to get an element from the sequence we "index" it like this: <math>a_n</math> (thats like saying `a[n]`) where <math>n\in\N^*</math> is the rank of the number. we can use any other letter

note: unlike in sets, numbers can appear multiple times in a sequence: <math>1, 1, 2, 3, 4</math> is a valid sequence

indexing can also start from <math>0</math>, for example mapping <math>\N</math> to its successor in <math>\N^*</math> (<math>0 \rightarrow 1; 1 \rightarrow 2; 2 \rightarrow 3; ...</math>)

sequences can be defined:

- in words

<math>d_1=1, d_2=11, d_3=111, ...</math>

here, <math>(d_n)_n</math> is defined as "each element in the sequence has n digits of 1"

- with a formula

<math>b_n=2^n</math>

that gives: <math>b_1=2^1=2, b_2=2^2=4, b_3=2^3=8, ...</math>

- using recursion

in sequences, recursion means each element is given by a formula that uses the previous elements

for example, let <math>(c_n)_n</math> be a sequence in which <math>c_1=1, c_2=2, c\_{n+2}=c_n+c\_{n+1}</math>, for <math>n\geqslant 1</math>

<math>c_3=c\_{1+2}</math> (in which case <math>n=1</math>) so <math>c\_{1+2}=c_1+c\_{1+1}=c_1+c_2=1+2 \Rightarrow c_3=3</math>

a sequence is _bound_ if there exists an interval <math>[a, b]</math> such that <math>a \leqslant a_n \leqslant b</math>

an example of a bound sequence is <math>(\frac{1}{n})_n, n \geqslant 1</math> because all values fall in the interval <math>[0, 1]</math>

sequences can be _increasing_ if <math>a_n \leqslant a\_{n+1}</math> and _decreasing_ if <math>a_n \geqslant a\_{n+1}</math>. if a sequence is either increasing or decreasing, it is called a _monotone_ sequence. if we do _not_ allow repeated values (we use <math><</math> instead of <math>\leqslant</math> and <math>></math> instead of <math>\geqslant</math>) it is called _strictly_ increasing or _strictly_ decreasing.

there are two special types of sequences called _arithmetic_ sequences (or progressions) and _geometric_ sequences (or progressions). _arithmetic_ sequences just means the next element is the previous element plus a number. _geoemtric_ sequences are the same thing but instead of adding a number, we multiply by one

arithmetic sequences are defined like this: <math>a\_{k+1}=a_k+r, k \geqslant 1</math>

<math>r</math> is constant and is called _common difference_

from the definition, <math>a\_{k+1}-a_k=r</math>

for arithmetic sequences we only need the first element and the common difference, and from that we can compute every number in the sequence

for arithmetic sequences, we have some interesting properties

- mean

for <math>n \geqslant 2</math> we have <math>a_n=\frac{a\_{n-1}+a\_{n+1}}{2}</math>

basically, every element <math>n</math> starting from the second, it is equal to the _mean_ of its neighboring elements (<math>a\_{n-1}</math> and <math>a\_{n+1}</math>)

- formula for general term

in a arithmetic sequence, <math>a_n=a_1+(n-1)r</math>

for <math>a_2=a_1+r, a_3=a_2+r=a_1+r+r, ...</math> this means that an element <math>n</math> is <math>kr</math> far from an element <math>n - k</math> (how far means <math>a_n-a\_{n-k}=kr</math>), and to get how far it is from <math>0</math> we just add <math>a_1</math>

now for geometric sequences, <math>a\_{k+1}=a_k \cdot q</math>

<math>q\not=0</math> is constant and is called common ratio

from the definition, <math>\frac{a\_{k+1}}{a_k}=q</math>

just as with arithmetic sequences, in geometric sequences you only need the first element and the common ratio to compute any element <math>n</math>

- geometric mean

<math>a_n=\sqrt{a\_{n-1} \cdot a\_{n+1}}</math>

basically, every element <math>n</math> starting from the second, is equal to the _geometric_ mean of its neighboring elements (<math>a\_{n-1}</math> and <math>a\_{n+1}</math>)

- formula for general term

<math>a_n=a_1 \cdot q</math>

for <math>a_2=a_1 \cdot q, a_3=a_2 \cdot q=a_1 \cdot q \cdot q</math> therefore <math>a_3=a_1 \cdot q^2</math> so each element <math>n</math> is <math>q^k</math> far from <math>n-k</math> (how far means <math>a_n-a\_{n-k}=q^k</math>)

that's all for sequences!
