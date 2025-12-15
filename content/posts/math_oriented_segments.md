+++
title = "math: oriented segments"
date = 2025-12-15
+++

i wanted to learn vectors, but to learn them i must learn oriented segments first...again, i'll use my 9th grade textbook as a guide to learn them

**note**: on wikipedia i found different notations than the ones in my textbook, for this post im gonna use the ones from my textbook

if we have two points in the same plane, we can define the following:

- a straight line <math>AB</math> or <math>BA</math>
- a closed line segment <math>[AB]</math> or <math>[BA]</math> (includes <math>A</math> and <math>B</math>)
- an open line segment <math>(AB)</math> or <math>(BA)</math> (_excludes_ <math>A</math> and <math>B</math>)
- the distance between <math>A</math> and <math>B</math> (a real number, noted <math>AB</math>)

the segment <math>[AB]</math> can be traversed both ways, from <math>A</math> to <math>B</math> and from <math>B</math> to <math>A</math>, because of that we introduce _oriented_ or _directed_ line segments which are noted as: <math>\overline{AB}</math> (from <math>A</math> to <math>B</math>) and <math>\overline{BA}</math> (from <math>B</math> to <math>A</math>)

for <math>\overline{AB}</math>, <math>A</math> is called the _origin_ and <math>B</math> is called the _extremity_ (or head and tail)

if <math>A=B</math> then we have <math>\overline{AB}=\overline{AA}</math> which we call a _null_ segment. if <math>A\not=B</math> we call the segment a _non-null_ segment

for oriented segments, we can give the following definitions:

- the length of <math>\overline{AB}</math> is equal to the length of <math>[AB]</math> and is noted as <math>|\overline{AB}|</math> or, simply <math>AB</math>. <math>|\overline{AA}|=0</math> (the length of a null segment is always <math>0</math>)
- the _supporting line_ for <math>\overline{AB}</math> is <math>AB</math>
- we can say that two oriented segments have the same _direction_ if their supporting lines either are parraler or are equal

    from definition, a null oriented segment has the same direction as any other oriented segment, since it only has one point, there are infinite lines that can go through that point
- we can say that <math>\overline{AB}</math> and <math>\overline{A^{\prime}B^{\prime}}</math> have the same _sense_ if they satisfy the following:
    - have the same direction
    - if <math>AB\not=A^{\prime}B^{\prime}</math> then <math>B</math> and <math>B^{\prime}</math> are in the same semi-plane as <math>AA^{\prime}</math>
    - if <math>AB=A^{\prime}B^{\prime}</math>, then the semi-line <math>[A^{\prime}B^{\prime}</math> is included in the semi-line <math>[AB</math> (or inverse)
- we will say that two oriented segments have _opposite_ sense if they have the same _direction_ but _not_ the same _sense_

## equipollence

what a word...have a look at the [dictionary](https://www.merriam-webster.com/dictionary/equipollent)

but basically, two oriented segments are _equipollent_ if they have the same _length_ and _sense_. we use the _tilde_ (~) to denote that: <math>\overline{AB}\text{\textasciitilde}\overline{CD}</math>

next we're gonna look at three properties:

- let <math>A, B, C</math> and <math>D</math> be points
    1. <math>\overline{AB}\text{\textasciitilde}\overline{CD}\iff ABCD</math> is a parallelogram or <math>A, B, C, D</math> are on the same line and the segments <math>[AD], [BC]</math> have the same middle
    2. <math>\overline{AB}\text{\textasciitilde}\overline{CD}\iff\overline{AC}\text{\textasciitilde}\overline{BC}</math>
- basic properties of equivalence
    1. <math>\overline{AB}\text{\textasciitilde}\overline{AB}</math>
    2. <math>\overline{AB}\text{\textasciitilde}\overline{CD}\Rightarrow\overline{CD}\text{\textasciitilde}\overline{AB}</math>
    3. <math>\overline{AB}\text{\textasciitilde}\overline{CD}</math> and <math>\overline{CD}\text{\textasciitilde}\overline{EF}\Rightarrow\overline{AB}\text{\textasciitilde}\overline{EF}</math>
- let <math>\overline{AB}</math> be an oriented segment. for every point <math>M</math> inside the same plane, there exists only one point <math>N</math> in the same plane such that <math>\overline{MN}\text{\textasciitilde}\overline{AB}</math>

and...thats all for segments!
