## python版遗传算法
更好阅读体验，请访问（http://tianle.me/2017/04/19/GA/）

遗传算法(genetic algorithm, GA)是一种进化算法，其基本原理是仿效生物界中的“物竞天择，适者生存”的演化法则。遗传算法是把问题参数编码为染色体，再利用迭代的方式进行选择、交叉以及变异等运算来交换种群中染色体的信息，最终生成符合优化目标的染色体。

### 名词解释
在遗传算法中，染色体对应的是数据或数组，通常是由一维的串结构数据来表示，串上各个位置对应基因的取值。基因组成的串就是染色体(chromosome)，或者称为基因型个体(individual)。一定数量的个体组成了群体(population)。群体中的个体数目称为群体大小(population size)，也成为群体规模。而各个个体对环境的适应程度叫适应度(fitness)。

### 基本步骤
#### 编码
GA在进行搜索之前先将解空间的解数据表示成遗传空间的基因型串结构数据，这些串结构数据的不同组合便构成了不同的点。
#### 初始群体的生成
随机产生N个初始串结构数据，每个串结构数据称为一个个体，N个个体构成了一个群体。GA以这N个串结构数据作为初始点开始进化。
#### 适应度评估
适应度表明个体或解的优劣性。不同的问题，适应度函数的定义方式也不同。
#### 选择
选择的目的是为了从当前群体中选出优良的个体，使它们有机会作为父代为下一代繁殖子孙。遗传算法通过选择过程体现这一思想，进行选择的原则是适应度强的个体为下一代贡献一个或多个后代的概率大。选择体现了达尔文的适者生存原则。
#### 交叉
交叉操作是遗传算法中最主要的遗传操作。通过交叉操作可以得到新一代个体，新个体组合了其父辈个体的特性。交叉体现了信息交换的思想。
#### 变异
变异首先在群体中随机选择一个个体，对于选中的个体以一定的概率随机地改变串结构数据中某个串的的值。同生物界一样，GA中变异发生的概率很低，通常取值很小。

### 实例详解
之前已经使用matlab实现了一次，由于现在又布置了作业，正好现在对python不是特别熟悉，那就写个代码练练手吧。
#### 目标函数
max    f (x1, x2) = 21.5 + x1·sin(4p x1) + x2·sin(20p x2)

s. t.    -3.0 <= x1 <= 12.1
          4.1 <= x2 <= 5.8

![function](http://img.tianle.me/image/20170419/maxfunction.jpg)

```python
def func(self):
        self.decoding(self.code_x1, self.code_x2)
        self.y = 21.5 + self.x1 * math.sin(4 * math.pi * self.x1) + self.x2 * math.sin(20 * math.pi * self.x2)
```

#### 二进制编码
在刚刚提到的遗传算法中，我们首先要将数据进行编码，这里我们采用二进制的方式进行编码。第一步，我们根据题目的介绍可以得知该函数含有两个变量，以及各自的定义域。在二进制编码中，我们首先要先计算它的编码长度。计算公式如下：
$${2^{{m_j} - 1}} < ({b_j} - {a_j})*precision \le {2^{{m_j}}} - 1$$
其中precision为精度，如小数点后5位，则precision=10^5，m<sub>j</sub>为编码长度，${x_j} \in [{a_j},{b_j}]$
#### 二进制解码
解码即编码的逆过程:
$${x_j} = {a_j} + {\rm{decimal}}(substrin{g_j}) \times \frac{{{b_j} - {a_j}}}{{{2^{{m_j}}} - 1}}$$
![function](http://img.tianle.me/image/20170419/coding.jpg)

```python
def decoding(self, code_x1, code_x2):
        self.x1 = self.bounds[0][0] + int(code_x1, 2) * (self.bounds[0][1] - self.bounds[0][0]) / (
        2 ** self.code_x1_length - 1)
        self.x2 = self.bounds[1][0] + int(code_x2, 2) * (self.bounds[1][1] - self.bounds[1][0]) / (
        2 ** self.code_x2_length - 1)
```

#### 种群初始化
编码完成那我们就开始对种群初始化吧，为了简便我采用了随机地方式进行初始化。
```python
def __init__(self, bounds, precision):
        self.x1 = 1
        self.x2 = 1

        self.y = 0

        self.code_x1 = ''
        self.code_x2 = ''

        self.bounds = bounds

        temp1 = (bounds[0][1] - bounds[0][0]) * precision
        self.code_x1_length = math.ceil(math.log(temp1, 2))

        temp2 = (bounds[1][1] - bounds[1][0]) * precision
        self.code_x2_length = math.ceil(math.log(temp2, 2))

        self.rand_init()
        self.func()

def rand_init(self):
        for i in range(self.code_x1_length):
            self.code_x1 += str(random.randint(0, 1))

        for i in range(self.code_x2_length):
            self.code_x2 += str(random.randint(0, 1))
```
#### 选择
选择我们采用轮盘赌方式进行选择，主要思想是适应度高的，被选择到的概率大。
![function](http://img.tianle.me/image/20170419/selection.jpg)
没怎么优化，用了一堆for循环。。。。
```python
    def select(self):
        """
        轮盘赌选择
        :return:
        """
        # calculate fitness function
        sum_f = 0
        for i in range(self.pop_size):
            self.pop[i].func()

        # guarantee fitness > 0
        min = self.pop[0].y
        for i in range(self.pop_size):
            if self.pop[i].y < min:
                min = self.pop[i].y
        if min < 0:
            for i in range(self.pop_size):
                self.pop[i].y = self.pop[i].y + (-1) * min

        # roulette
        for i in range(self.pop_size):
            sum_f += self.pop[i].y
        p = [0] * self.pop_size
        for i in range(self.pop_size):
            p[i] = self.pop[i].y / sum_f
        q = [0] * self.pop_size
        q[0] = 0
        for i in range(self.pop_size):
            s = 0
            for j in range(0, i+1):
                s += p[j]
            q[i] = s
        # start roulette
        v = []
        for i in range(self.pop_size):
            r = random.random()
            if r < q[0]:
                v.append(self.pop[0])
            for j in range(1, self.pop_size):
                if q[j - 1] < r <= q[j]:
                    v.append(self.pop[j])
        self.pop = v
```
#### 变异
这里的变异，我们先以变异概率，从种群中选一个，然后对选中的个体，随机选一个变异位点进行变异。
![function](http://img.tianle.me/image/20170419/mutation.jpg)
```python
    def mutation(self):
        """
        变异
        :return:
        """
        for i in range(self.pop_size):
            if self.pm > random.random():
                pop = self.pop[i]
                # select mutation index
                index1 = random.randint(0, pop.code_x1_length-1)
                index2 = random.randint(0, pop.code_x2_length-1)

                i = pop.code_x1[index1]
                i = self.__inverse(i)
                pop.code_x1 = pop.code_x1[:index1] + i + pop.code_x1[index1+1:]

                i = pop.code_x2[index2]
                i = self.__inverse(i)
                pop.code_x2 = pop.code_x2[:index2] + i + pop.code_x2[index2+1:]
```
#### 交叉
这里采用单点交叉法。随机从种群中选两个个体，然后再随机选一个交叉点，交换位置。看图 = . =
![function](http://img.tianle.me/image/20170419/crossover.jpg)
![function](http://img.tianle.me/image/20170419/crossoverCode.jpg)
```python
    def cross(self):
        """
        交叉
        :return:
        """
        for i in range(int(self.pop_size / 2)):
            if self.pc > random.random():
                # randon select 2 chromosomes in pops
                i = 0
                j = 0
                while i == j:
                    i = random.randint(0, self.pop_size-1)
                    j = random.randint(0, self.pop_size-1)
                pop_i = self.pop[i]
                pop_j = self.pop[j]

                # select cross index
                pop_1 = random.randint(0, pop_i.code_x1_length - 1)
                pop_2 = random.randint(0, pop_i.code_x2_length - 1)

                # get new code
                new_pop_i_code1 = pop_i.code_x1[0: pop_1] + pop_j.code_x1[pop_1: pop_i.code_x1_length]
                new_pop_i_code2 = pop_i.code_x2[0: pop_2] + pop_j.code_x2[pop_2: pop_i.code_x2_length]

                new_pop_j_code1 = pop_j.code_x1[0: pop_1] + pop_i.code_x1[pop_1: pop_i.code_x1_length]
                new_pop_j_code2 = pop_j.code_x2[0: pop_2] + pop_i.code_x2[pop_2: pop_i.code_x2_length]

                pop_i.code_x1 = new_pop_i_code1
                pop_i.code_x2 = new_pop_i_code2

                pop_j.code_x1 = new_pop_j_code1
                pop_j.code_x2 = new_pop_j_code2
```

#### 算法主流程
至此，遗传的主要框架已经完毕，下面展示主流程，及画图部分代码。
```python
    def ga(self):
        """
        算法主函数
        :return:
        """
        self.init_pop()
        best = self.find_best()
        self.g_best = copy.deepcopy(best)
        y = [0] * self.pop_size
        for i in range(self.max_gen):
            self.cross()
            self.mutation()
            self.select()
            best = self.find_best()
            self.bests[i] = best
            if self.g_best.y < best.y:
                self.g_best = copy.deepcopy(best)
            y[i] = self.g_best.y
            print(self.g_best.y)

        # plt
        plt.figure(1)
        x = range(self.pop_size)
        plt.plot(x, y)
        plt.ylabel('generations')
        plt.xlabel('function value')
        plt.show()
```
#### 实验结果图
![function](http://img.tianle.me/image/20170419/result.png)

### 总结
在编码的时候，我偷懒了一下，把两个变量拆开写，x1和x2，导致之后的操作变得异常复杂，并且不利于代码重构。
程序中过多的使用了for循环，并没有对此进行优化。
针对上述两个问题，在此记录一下。

### 程序完整代码
[Genetic-Algorithms](https://github.com/zhangtianle/Genetic-Algorithms)

### 参考资料
《MATLAB智能算法-30个案例分析》