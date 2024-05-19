+++
title = "Hello, blog! Or why you should care about combinatorial enumeration."
template = "blog_page.html"
author = "Michael Gill"
tags = ["Latin squares", "Combinatorics", "Designs"]
draft = false
+++

Combinatorics is concerned with the construction, arrangement, and enumeration of elements in a discrete space. Problems in combinatorics come in two distinct flavours, existential and enumerative. Constructions and arrangements form the backbone of the solutions to the existential questions. Computation, often using a computer or a "thinking machine" as the Duneverse would put it, forms the backbone of enumeration. Using a slight tongue-in-cheek view, the existential problems ask "Is there more than one?" whereas the enumerative problems ask "How many more than one could there possibly be?". All these words boil down to a simple broad idea, combinatorics is the mathematics of counting.

So at this point, there is only one interesting question, what are we counting? Let's start simple. Let $\mathcal{N}$ be the set of natural numbers and $n$ be an element from that set, $n\in \mathcal{N}$. We can write $n$ as the sum of powers of two, for example, $410 = 2^8 + 2^7 + 2^4 + 2^3 + 2^2$. If we encode this idea using a one in position $i$ to represent that the sum contains the summand $2^i$ then we might end up with the following equivalence $410_{10} = 110011010_{2}$. We should all recognise this as a binary encoding. A binary encoding of the integer $n$ is representable in $k$ bits if $n \leq 2^{k+1}-1$. How many $k$-bit binary numbers are there with exactly $t$ ones?

Let's start by defining the function $c(k,t)$ to be the count of $k$-bit numbers with $t$ ones. In later posts, we will discuss what it means for two objects to be different, but right now it is sufficient to say that two binary numbers are different if there is some position in which they differ. For $t = 1$ it is easy to see that $c(k,1) = k$ as you just select one of $k$ bits and set it to $1$ and there are $k$ ways to this. Now moving our thinking to $t = 2$ we can see that after we select one bit and set it to $1$, we have $k-1$ positions from which to choose the second bit. This would give $k(k-1)$ choices, however, we should note that there is no difference between the first choice and the second choice. This means that we have overcounted by a factor of $2$ which gives $c(k, 2) = \frac{k(k-1)}{2}$.

For $c(k,3)$ we have $k$ positions and we select 3 of them in turn. This may lead some people to think that $c(k,3) = \frac{k(k-1)(k-2)}{3}$, but you would be off by a factor of two again. To find the correct number, let's label our choices $(b_1,b_2,b_3)$. As we know, there is no difference between choosing $b_2$ before $b_1$. This means that the choice $(b_1,b_2,b_3) = (b_2,b_1,b_3)$ and so on... If you do this on paper, you quickly find 6 possible ways of choosing bits that are all equivalent. We should note that this method of choosing bits is simply the way that we chose the first 2 bits with the additional insight that we could have chosen the third bit in one of $3$ spots: before the first choice, in between the choices, or after the last choice. Following this train of reasoning we find $c(k,t) = \frac{k(k-1)(k-2)\dots(k-t)}{t!} = \frac{k!}{t!(k-t)!}$. Some of you may recognise this as the binomial coefficient $\binom{k}{t}$.

We have now solved the enumerative problem, or have we? Our solution is slightly less than ideal. It is great if you do not care about the objects and only want to know how many there are. If you are interested in the structure or properties of the objects then you may want to generate the set of them. While the equation $c(k,t) = \binom{k}{t}$ may be useful for understanding how many $k$-bit numbers have $t$ ones, it doesn't help us generate them (At least not in a way that is meaningfully distinct from the way I am about to show you). To do this we need to think about the problem in a slightly different way. We now turn our thoughts to recursion. If we consider only the most significant bit in a $k$-bit binary number then we can consider two possible outcomes. Either the bit is a zero or a one. If the bit is a zero, then we can see that the number of $k$-bit numbers with $t$ ones is $c(k-1, t)$. If the bit is one, then the number of possible configurations is $c(k-1, t-1)$. Combining these equations into a recurrence and determining the required base cases we get,

\begin{align}
c(k,t) & = c(k-1, t) + c(k-1, t-1), \\\\
c(t,t) & = 1, \\\\
c(k, 0) & = 1.
\end{align}

So we have seen that we can count the number of $k$-bit binary numbers with $t$ ones in two different ways. The first way invokes the use of the binomial coefficient and gives us an easy-to-compute number of objects, but is not actually useful if we would like to find the objects themselves. The second solution is less direct and requires us to do some more work to find the solution, but it does inform a simple algorithm for generating the set of objects. Let's get started with backtracking.

## Enumeration and backtracking: A love affair

Let's make our goal more concrete. The recursive formulation works by considering both possible values of the bit $k$ in a $k$-bit number. First, it assumes the current bit at position $k$ is a zero and then asks how many $(k-1)$-bit numbers exist when this is the case, the term $c(k-1, t)$. It then assumes that the current bit, at position $k$, is one and asks how many $(k-1)$-bit numbers exist when this is the case, the term $c(k-1, t-1)$. This strategy exposes an interesting idea, what if we got it to build the numbers, instead of counting them? Instead of asking what happens if the $k$<sup>th</sup> bit is one, let's set it to one and then come back and change it later. This is the key insight of recursive backtracking, we keep a partial solution and at each stage fill a segment of the partial solution in with the intention of returning to the partial solution later and updating it to generate a different partial solution. We then detect when our partial solution is a real solution, using the base cases, and store it for later use. For those of you familiar with search algorithms, we are effectively depth-first searching the space of all possible solutions.

<pre class="mermaid">
graph TD;
    A(5,3)-->|0|B(4,3);
    A-->|1|C(4,2);
    B-->|0|D(3,3);
    B-->|1|E(3,2);
    C-->|0|F(3,2);
    C-->|1|G(3,1);
    D-->|111|H(00111);
    E-->|0|I(2,2);
    E-->|1|J(2,1);
    F-->|0|K(2,2);
    F-->|1|L(2,1);
    G-->|0|M(2,1);
    G-->|1|N(2,0);
    I-->|11|O(01011);
    J-->|0|P(1,1);
    J-->|1|Q(1,0);
    K-->|11|R(10011);
    L-->|0|S(1,1);
    L-->|1|T(1,0);
    M-->|0|U(1,1);
    M-->|1|V(1,0);
    N-->|00|X(11100);
    P-->|1|Y(01101);
    Q-->|0|Z(01110);
    S-->|1|AA(10101);
    T-->|0|AB(10110);
    U-->|1|AC(11001);
    V-->|0|AD(11010);
</pre>

I will be writing and presenting code in [Rust](https://www.rust-lang.org/). My desire to master Rust is a large contributing factor to this blog, so all code will be provided in Rust. We should note that while our function allows for $k$-bit numbers where $k$ is arbitrarily large, to keep our computation relatively simple we will limit ourselves to the number of bits contained in a `usize` and assume that no data of a larger size is given.

```Rust
fn bin_k_t(k: u32, t: u32, current: usize, results: &mut Vec<usize>) {
    // Base case 1, we need to put all 1's in the remainder of the integer.
    if k == t {
        let mask = usize::pow(2, k+1) - 1;
        let current = current | mask;

        results.push(current);
    } else if t == 0 {
        // Base case 2, we take the integer as it is, we are guaranteed that it
    // contains k 0's at the end.
        results.push(current);
    } else {
        let mask = usize::pow(2, k);
        bin_k_t(k-1, t, current, results);
        bin_k_t(k-1, t-1, current | mask, results);
    }
}

fn main() {
    let mut results: Vec<usize> = vec![];

    bin_k_t(10, 4, 0, &mut results);

    // There are exactly 10*9*8*7/24 = 10*3*7 = 210 10-bit binary numbers with 4 ones.
    assert_eq!(results.len(), 210);

    for val in results {
        println!("{}", val);
    }
}
```

## Harder problems

So far this is a relatively simple example. It shows how to generate a simple combinatorial object, but it does not help us understand the full depth of techniques to understand combinatorial enumeration. So, let's choose a more interesting example. We will think about $k\times k$ $(0,1)$-arrays where each row and each column is a $k$-bit binary number containing exactly $t$ ones. The natural number $t$ is called the strength of the array, while $k$ is its order. If $k = 5$ and $t = 3$, then we might have the following example:

$$
\left[
\begin{array}{ccccc}
    1 & 1 & 1 & 0 & 0 \\\\
    1 & 0 & 0 & 1 & 1 \\\\
    1 & 0 & 1 & 0 & 1 \\\\
    0 & 1 & 1 & 1 & 0 \\\\
    0 & 1 & 0 & 1 & 1
\end{array}
\right]
$$

It should be obvious at this point that this array is more complicated to generate than the $k$-bit binary numbers with $t$ ones. But the process is roughly the same. There are several ways to think about this enumeration, but let's start with the most obvious. We are going to start with a $(0,1)$-array containing only zeros. From this, we will recursively backtrack. For each position in a row, we will check if that position could be a one. We will then set it to a one in the partial solution and recurse to find all partial solutions that have that position fixed. After we return from our recursive call we set the bit back to zero and move to the next cell.

To do this in Rust we will use the `Vec<bool>` type. This type is similar to the bit array type we see in other languages. It is an efficient implementation of a vector of bools, storing each boolean value as a singular bit, so it is both a space- and time-efficient implementation. As we want a 2-dimensional array we will wrap the `Vec` datastructure in a 2-dimensional list implementation.

```Rust
use std::ops::{Index, IndexMut};
use std::fmt;

pub struct Vec2D<T> {
    data: Vec<T>,
    n_rows: usize,
    n_cols: usize,
}

impl <T: Default + Clone> Vec2D<T> {
    pub fn new(n_rows: usize, n_cols: usize) -> Vec2D<T> {
        let data = vec![T::default(); n_rows * n_cols];
        Vec2D { data, n_rows, n_cols }
    }
}

impl <T> Index<(usize, usize)> for Vec2D<T> {
    type Output = T;

    fn index(&self, index: (usize, usize)) -> &T {
        assert!(index.0 < self.n_rows);
        assert!(index.1 < self.n_cols);

        &self.data[index.0 * self.n_cols + index.1]
    }
}

impl <T> IndexMut<(usize, usize)> for Vec2D<T> {
    fn index_mut(&mut self, index: (usize, usize)) -> &mut T {
        assert!(index.0 < self.n_rows);
        assert!(index.1 < self.n_cols);

        &mut self.data[index.0 * self.n_cols + index.1]
    }
}

impl <T> fmt::Display for Vec2D<T> where T: fmt::Display {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        for i in 0..self.n_rows {
            for j in 0..self.n_cols {
                match write!(f, "{} ", self[(i, j)]) {
                    Err(e) => return Err(e),
                    _ => {}
                }
            }
            match write!(f, "\n") {
                Err(e) => return Err(e),
                _ => {}
            }
        }

        write!(f, "")
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test() {
        let mut test = Vec2D::<usize>::new(3, 3);

        test[(0, 0)] = 1;
        test[(0, 1)] = 2;
        test[(0, 2)] = 3;
        test[(1, 0)] = 4;
        test[(1, 1)] = 5;
        test[(1, 2)] = 6;
        test[(2, 0)] = 7;
        test[(2, 1)] = 8;
        test[(2, 2)] = 9;

        assert!(test.data == vec![1, 2, 3, 4, 5, 6, 7, 8, 9]);
    }
}
```

We can now enumerate the objects by recursively backtracking over each position of a `Vec2D`.

```Rust
mod collections;
use collections::vec2d::Vec2D;

fn bin_square(dim: usize, strength: usize) -> usize {
    // We are going to use a helper function to do the recursion as Rust does
    // not support overloading functions.
    let mut partial = Vec2D::new(dim, dim);

    let agg = extend_partial(dim, strength, &mut partial, (0, 0));
    return agg;
}

fn extend_partial(
    dim: usize,
    strength: usize,
    partial: &mut Vec2D<bool>,
    current_loc: (usize, usize),
) -> usize {
    // Determine if we just finished a row. This is required so that we know
    // if the previous bit in the row was set correctly. If the previous
    // row does not have the desired strength, then we reject this square.
    if current_loc.0 > 0 && current_loc.1 == 0 {
        if row_strength(partial, (current_loc.0 - 1, dim)) < strength {
            return 0;
        }
    // The only way for our current row to be the dimension of the square is
    // if we have completed a square.
    } else if current_loc.0 == dim {
        // If we were interested in the output of this program we would print
        // here.
        
        // println!("{}", partial);

        return 1;
    // Last base case is an early rejection condition. It checks to make sure
    // that there is room left in the square for the current row and column to
    // have the desired strength.
    } else if strength - row_strength(partial, current_loc) > dim - (current_loc.1) &&
              strength - col_strength(partial, current_loc) > dim - (current_loc.0) {
        return 0;
    }

    let mut agg = 0;
    let next_loc = if current_loc.1 == dim - 1 {
        (current_loc.0 + 1, 0)
    } else {
        (current_loc.0, current_loc.1 + 1)
    };

    // Compute the row strength and the column strength and decide if there is
    // room remaining in the square for the 1
    if row_strength(partial, current_loc) < strength
        && col_strength(partial, current_loc) < strength
    {
        partial[current_loc] = true;
        agg += extend_partial(dim, strength, partial, next_loc);
        partial[current_loc] = false;
    }

    // We might always be able to extend with a 0. So we do this here.
    agg += extend_partial(dim, strength, partial, next_loc);

    return agg;
}

// We compute the row strength by finding the number of 1's contained in the
// current row.
fn row_strength(partial: &Vec2D<bool>, current_loc: (usize, usize)) -> usize {
    let mut strength = 0;

    for i in 0..current_loc.1 {
        if partial[(current_loc.0, i)] {
            strength += 1;
        }
    }

    return strength;
}

// We compute the column strength by finding the number of 1's contained in
// the current column.
fn col_strength(partial: &Vec2D<bool>, current_loc: (usize, usize)) -> usize {
    let mut strength = 0;

    for i in 0..current_loc.0 {
        if partial[(i, current_loc.1)] {
            strength += 1;
        }
    }

    return strength;
}
```

Using this code we find the counts for the number of $k\times k$ $(0,1)$-arrays with $1\leq t \leq \lfloor k/2 \rfloor$ ones, at least for all $2 \leq k \leq 8$:

| (k, t)   | Count       | Time elapsed |
| -------: | ----------: | -----------: |
| (2, 1)   | 2           |      0.000   |
| (3, 1)   | 6           |      0.001   |
| (4, 1)   | 24          |      0.006   |
| (4, 2)   | 90          |      0.031   |
| (5, 1)   | 120         |      0.034   |
| (5, 2)   | 2,040       |      0.785   |
| (6, 1)   | 720         |      0.236   |
| (6, 2)   | 67,950      |     34.149   |
| (6, 3)   | 297,200     |    262.386   |
| (7, 1)   | 5,040       |      1.619   |
| (7, 2)   | 3,110,940   |   1519.783   |
| (7, 3)   | 68,938,800  |  74330.333   |
| (8, 1)   | 40,320      |     17.610   |
| (8, 2)   | 187,530,840 | 121685.176   |

The first thing you will notice is that $(8,3)$ and $(8,4)$ are missing. At our current stage, these are taking too long to compute and the values don't really add a lot, so let's save them as stretch goals for a later blog post. Moving on, a quick sanity check shows that $(k, 1) = k!$. This makes sense, these arrays are simply the family of permutation matrices. We can also see $(k, 2)$ and $(k, 3)$ match the integer sequences [A001499](https://oeis.org/A001499) and [A001501](https://oeis.org/A001501), respectively. We can be relatively convinced that the code we have seen above is computing the desired values. At current, this is as far as we can get within a reasonable time. There is just not enough juice in our computer to get us any further using the current methods.

## What next?

In the coming blog posts, I plan to improve on these methods. We will see that we can compute these values using less direct techniques that exploit symmetries and other structural properties of our objects. We will also utilise techniques that will rely on our computer architecture, using operations that are fast compared to the operations seen so far. We will also discuss the role that precomputation has to play in combinatorial enumeration and how, with some simple tricks, we can restrict the branching factor of a search, reducing the costs even further. We will also talk about notions of similarity, observing how they factor into our searches.

The code developed in this post is available at my GitHub repository, [enumeration1](https://github.com/orthosquare/enumeration1). Happy counting.

<script type="module">
    import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.esm.min.mjs';
</script>
