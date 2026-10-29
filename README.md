### 你好，我是 0Mints0 👋

我做 **MPI 并行程序的性能测量**——具体一点，是把"进程之间到底在不在互相等"这件事变成可复现的数字。

并行程序最常见的性能问题是这句话的变体：*每个进程都在满负荷算，但总耗时比最快的那个进程多了很多。*
多出来的时间花在等人上，可这件事很难有证据，而且极容易看错方向：同步点多不等于在等人，
等待总量大不一定是因为负载不均，最后到达次数最多的进程也不一定就是瓶颈。

我倾向于先把**定义**写清楚，再动手量：把一段等待拆成"到齐前等待"和"到齐后原语开销"，
两者之和严格等于总时长，拆分不重不漏，然后才谈归因和优化。

---

#### 🔧 我在做什么

**[lbprobe](https://github.com/0Mints0/lbprobe)** — MPI 同步点等待测量探针

给 C/C++ + MPI 项目的每个集合通信前后各加一行，得到一个能回答问题的报告：

- 进程之间**是否存在相互等待**？
- **等待时间是多少**？占墙钟多少比例？
- 这份等待该**归因给谁**——是大家在互相等，还是大家在等某一个？

它还会区分两种很容易被混为一谈的形态：**相互等待**（进度整体不一致）与
**单向等待**（单点落后）。判定按**等待量**加权而不是按次数——实测中见过某个 rank
只在 42% 的同步点上最后到达，看起来"分散"，但它那几次每次携带 2.5 秒的到达离散，
按等待量算占 99%。

```text
到齐前等待 = 最晚进入者时刻 − 本 rank 进入时刻   → 我算完了在等人
到齐后耗时 = 本 rank 退出时刻 − 最晚进入者时刻   → 集合通信自身的开销
两者之和   ≡ 该 rank 这次同步的等待时长
```

---

#### 📊 一些我信奉的测量原则

| 原则 | 为什么 |
|---|---|
| **先分段，再比较** | 启动阶段的一次性同步能占全部等待的 90%，和稳态混在一起就会得出反向结论 |
| **归因按等待量加权** | 按次数统计会被大量几乎不耗时的同步点稀释掉真实瓶颈 |
| **"等待更少"可能是坏事** | 不把负载摊开的程序既不用交换、也不会等待——"等得少"的代价是"算得慢" |
| **绝对值不可跨环境比较** | 机器负载会让同一份代码的绝对数字漂移，可信的是同一次会话内的相对关系 |

---

#### 🧰 常用

![C](https://img.shields.io/badge/C-C99-00599C?logo=c&logoColor=white)
![C++](https://img.shields.io/badge/C++-00599C?logo=cplusplus&logoColor=white)
![MPI](https://img.shields.io/badge/MPI-OpenMPI%20%7C%20MPICH-orange)
![Python](https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=white)
![Shell](https://img.shields.io/badge/Shell-4EAA25?logo=gnubash&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?logo=linux&logoColor=black)
![Git](https://img.shields.io/badge/Git-F05032?logo=git&logoColor=white)

---

#### 📁 项目

| 项目 | 说明 |
|---|---|
| **[lbprobe](https://github.com/0Mints0/lbprobe)** | MPI 同步点等待测量探针：测量、归因、出报告。C99 + Python，无第三方依赖 |
| [lbprobe 案例研究](https://github.com/0Mints0/lbprobe/blob/main/docs/case-study.md) | 一次真实系统的均衡化 A/B 对照测量——含"等待更少反而更慢"的完整推演 |

---

<sub>📫 交流或指正：欢迎开 issue。</sub>
