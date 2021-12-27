[toc]

# Coverage of Known Spaces : The Boustrophedon Cellular Decomposition

## 1. Introduction

1. exact celluar decomposition : the union of non-interesting regions composing the target environment.

2. boustrophedon : **the way of ox**.

3. Typically, whe an ox drags a plow in a field, it crosses the full length of the field in a straight line, turns around, and then traces a new straight path adjacent to the previous one.

   <img src="/home/pikachu/.config/Typora/typora-user-images/image-20211227134844120.png" alt="image-20211227134844120" style="zoom:67%;" />

4. **The Boustrophedon Cellular Decomposition** : 

   1. the robot's free space is broken down into cells;
   2. the robot can cover each cell with back-and-forth boustrophedon motion; (来来回回牛耕式运动)
   3. once a robot covers each cell, it has covered the entire free region of environment.

## 2. Background Work

### 2.1 Prior Work in Coverage

1. floor maintenance : 1994 Colegrave and Branch
   * the path must be explicitly programmed into the robot,
   * do not generate the coverage path,
2. Dmeter project : 1996 Ollis and Stentz
   * used to harvest large fields,
   * use vision to guide its path alongside ghe previous cut crop line
   * only cover rectangular fields.

### 2.2 Exact Cellular Decomposition

1. 旅行商问题；
2. trapezoid decomposition, 不规则分解，1991
3. The trapezoid decomposition approach assumes that ==a vertical line, termed a slice, sweeps left to right== through a bounded environment which is populated with polygonal obstacles.
4. Cells are formed via a sequence of open and close operations which occur when the slice encounters an event, an instance in which a slice intersects a vertex of a polygon.
5. Three types of events : IN, OUT and MIDDLE;

## 3. Contributions

<img src="/home/pikachu/.config/Typora/typora-user-images/image-20211227142210451.png" alt="image-20211227142210451" style="zoom:50%;" />

1. Boustrophedon Decomposition has a fewer number of cells;
2. Critical Point : changes in connectivity of a slice;

## 4. Algorithm Overview

1. negative infinity
2. depth-first-like graph search algoritm

## 5. Experiments

1. problems near obstacle edges that form acute angles with the boustrophedon back and forth paths.
2. different mechanism near the boundary of the environment
3. use sensor information to guide the coverage operation.

## 6. Conclusion and Future Work

1. 

