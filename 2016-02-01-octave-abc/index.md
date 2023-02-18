# 吴恩达 ML 公开课笔记(4)-Octave abc

Too lazy to explain some of the commands...orz

<!--more-->

## Basic
 - `PS1('sign')` Change prompt to 'sign'
 - Matrix Assignment
     - `v = [1 2; 3 4; 5 6]`
     - `v = [1.1 1.2 1.3] = [1:<0.1:>1.3]` (default step is 1)
 - Matrix Generation commands
     - `ones(2,3) = [1 1 1; 1 1 1]` (ones/zeros/rand/randn)
     - `eye(n)`: generate n by n identical matrix
     - `magic(n)`: generate n by n magic matrix
 - `size(M, n)`: return the size of the n=1:row n=2:coloum
 - `length(M)`: max dimention of M

## Moving data around
 - `pwd`: show current path
 - `load('xxx')` `load <filename>`
 - `who`: display current scope. whos: details
 - `clear <variable>`: clear the space of the variable
 - `M(start:end)`: count from start to end coloum by coloum
 - `save <filename> <variable> <encode>`: save variable to the file using enconde like ascii

## Manipulate
 - `M(x,y)`: indexing
 - `M(r1:r2,c1:c2)` sub-matrix block: coloumn1-2/row1-2.
 - `M(rn, :) = [a b c]`: assign [a b c] to row n of M
 - `M([r1 r2], :)`: copy row1 and row2
 - `M(M, [a b c])`: append [a b c] to M
 - `M(:)`: put all elements of M into a single vector
 - `M = [X Y] = [X,Y]`: concatenating X & Y into M(horizontally)
 - `M = [X;Y]`: concatenating X & Y into M (vertically)

## Computing on data
 - `M'`: transpose
 - `pinv(M)`: inverse of M
 - Element-wise calculation
     - `A*b`, `A .* B`: element-wise multiply, as well as `.^`, `./`, `log(v)`, `exp(v)`, `abs(v)` and so on
     - `M < a`: element-wise comparison, which returns a boolean matrix
 - `max()`
     - `[value, index] = max(M)`
     - `max(M,N)`: M and N are of the same size, do element-wise comparison, then generate a new matrix
     - `max(M) = max(M,[],1)`: 1->coloum-wise; 2->per row-wise.
 - `[r, c] = find(M < a)`: returns the index of the elements
 - `sum()`
     - `sum(M)`: summarize all the elements
     - `sum(M,1)`: summarize each coloum
     - `sum(M,2)`: summarize each row
 - `prod(M) = prod(M,1)`: multiply each coloum. 1->coloum-wise; 2->per row-wise.
 - `floor(M)`: rounds down
 - `ceil(M)`: rounds up
 - `flipud(M)`: flip upside-down

## Plotting data
 - `hist(v, n)`: plot v with n buckets histogram
 - `plot(x, y, <color>)`
 - `plot(x,y1) hold; plot(x,y2)`
 - `xlabel('X')  ylabel('Y')`
 - `legend('y1','y2')`  `title('title')`
 - `print -dpng 'filename.png'`
 - `close: close figure window`
 - `figure <number>; plot(x,y1);figure <number>; plot(x,y2);` ...
 - `subplot(x,y,index)`: access the index
 - `axis([xmin xmax ymin ymax])`
 - `clf`: clear all figure
 - `imagesc(M), colorbar, colormap gray;`

## Control statements
 - `for i=1:10, v(i) = 2^i; end;`
 - `break` `continue`
 - `while i<=5, v(i)=100; i=i+1; end;`
 - `if i==6, <stat>; elseif i==7, <stat2>; else <stat3>; end;`

## Functions
 - `<functionname.m>`: format of the function file
 - Example:
 ```
 function [y1, y2] = squareAndCubeThisNumber(x)
 y1 = x^2;
 y2 = x^3;
 ``` 
 - addpath('path')

