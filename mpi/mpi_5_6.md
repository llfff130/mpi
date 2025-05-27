# MPI归约操作 #
## 基本归约函数 ##
`MPI_Reduce(sendbuf, recvbuf, count, datatype, op, root, comm)`
- 功能：将所有进程的 sendbuf 按 op 运算，结果返回 root 进程的 recvbuf
- 参数:
    - sendbuf：发送缓冲区
    - recvbuf：接收缓冲区(仅在 root 进程有效)
    - count：数据数量
    - datatype：数据类型
    - op：归约操作
    - root：接收结果的进程
    - comm：通信域
## (op) #3
|操作|	描述|
|----|-----|
|MPI_MAX	|最大值|
|MPI_MIN	|最小值|
|MPI_SUM	|求和|
|MPI_PROD	|求积|
|MPI_LAND	|逻辑与|
|MPI_BAND	|按位与|
|MPI_LOR	|逻辑或|
|MPI_BOR	|按位或|
|MPI_LXOR	|逻辑异或|
|MPI_BXOR	|按位异或|
|MPI_MAXLOC	|最大值及位置|
|MPI_MINLOC	|最小值及位置|
## 数据类型 ##
### C语言 ###
|类型	|描述|
|------|----|
|MPI_INT|	整型|
|MPI_LONG|	长整型|
|MPI_SHORT|	短整型|
|MPI_FLOAT|	单精度浮点|
|MPI_DOUBLE|	双精度浮点|
|MPI_CHAR	|字符|
### Fortran ###
|类型	|描述|
|-----|-----|
|MPI_INTEGER|	整型|
|MPI_REAL	|单精度浮点|
|MPI_DOUBLE_PRECISION	|双精度浮点|
## 组归约(Allreduce) ##
`MPI_Allreduce(sendbuf, recvbuf, count, datatype, op, comm)`
- 功能：与 MPI_Reduce 类似，但结果会广播到所有进程
- 特点：每个进程的 recvbuf 都会包含归约结果
## 归约散发(Reduce-Scatter) ##
`MPI_Reduce_scatter(sendbuf, recvbuf, recvcounts, datatype, op, comm)`
- 功能：先执行归约操作，然后将结果分散到组内所有进程
- 特点：
    - 各进程的接收缓冲区大小是发送缓冲区大小的 1/N (N为总进程数)
    - 每个进程只接收结果的一部分
## 三种归约操作对比 ##
|特性	|MPI_Reduce|	MPI_Allreduce|	MPI_Reduce_scatter|
|------|-----------|---------------|--------------------|
|结果位置	|仅 root 进程	|所有进程	|所有进程|
|结果分布	|完整结果	|完整结果	|分散的部分结果|
|应用场景	|只需一个进程有结果	|所有进程需要结果	|分散处理大数据|
## 计算π的近似值 ##
### 数值积分法 ###
```c
#include <stdio.h>
#include <math.h>
#include <mpi.h>

double f(double x) {
    return 4.0 / (1.0 + x * x);
}

int main(int argc, char *argv[]) {
    int rank, size, i;
    const int n = 1000000;  // 积分区间划分数量
    double h, sum = 0.0, pi = 0.0;
    double start_time, end_time;
    
    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);
    
    start_time = MPI_Wtime();
    
    h = 1.0 / (double)n;
    
    // 每个进程计算自己负责的部分
    for (i = rank; i < n; i += size) {
        double x = h * ((double)i + 0.5);
        sum += f(x);
    }
    
    sum *= h;
    
    // 将所有进程的部分和相加
    MPI_Reduce(&sum, &pi, 1, MPI_DOUBLE, MPI_SUM, 0, MPI_COMM_WORLD);
    
    end_time = MPI_Wtime();
    
    if (rank == 0) {
        printf("Approximation of pi: %.16f\n", pi);
        printf("Time elapsed: %.6f seconds\n", end_time - start_time);
    }
    
    MPI_Finalize();
    return 0;
}
```
![test]()
### Gauss-Seidel ###
