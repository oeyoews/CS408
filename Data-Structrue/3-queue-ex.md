# 队列习题

## 顺序队列

**例题** 火车车轨入口到出口之间有$n$条轨道，列车的行进方向均为从左至右，列车可驶入任意一条轨道且每条道路可以容纳任意多数量的列车。现有编号为$1\sim9$的$9$列列车，驶入的次序依次是$8,4,2,5,3,9,1,6,7$。若期望驶出的次序依次为$1\sim9$，则$n$至少是()。

解：这个题目是一个多队列问题。根据题意：入队顺序为$8,4,2,5,3,9,1,6,7$，出队顺序为$1\sim9$。入口和出口之间有多个队列（$n$条轨道)，且每个队列（轨道）可容纳多个元素（多列列车），为便于区分，队列用字母编号。分析如下：显然先入队的元素必须小于后入队的元素（否则，若$8$和$4$入同一队列，$8$在$4$前面，则出队时也只能$8$在$4$前面），这样$8$入队列$A$，$4$入队列$B$，$2$入队列$C$，$5$入队列$B$（按照前述原则“大的元素在小的元素后面”也可将$5$入队列$C$，但这时剩下的元素$3$就必须放入一个新的队列中，无法确保“至少”，所以要最接近的一个队列），$3$入队列$C$，$9$入队列$A$，这时共占了$3$个队列，后面还有元素$1$，直接再用一个新的队列$D$，$1$从队列$D$出队后，剩下的元素$6$和$7$或入队列$B$，或入队列$C$。综上，共占用了$4$个队列。

## 循环队列

**例题** 已知循环队列存储在一维数组$A [0\cdots n-1]$中，且队列非空时$front$和$rear$分别指向队头元素和队尾元素。若初始时队列为空，且要求第一个进入队列的元素存储在$A[0]$处，则初始时$front$和$rear$的值分别是()。

$A.0$，$0$

$B.0$，$n-1$

$C.n-1$，$0$

$D.n-1$，$n-1$

解：$B$。根据题意，第一个元素进入队列后存储在$A[0]$处，此时$front$和$rear$值都为$0$。入队时由于要执行$(rear+1)\%n$操作，所以若入队后指针指向$0$，则$rear$初值为$n-1$，而由于第一个元素在$A[0]$中，插入操作只改变$rear$指针，所以$front$为$0$不变。

【2016统考真题】
