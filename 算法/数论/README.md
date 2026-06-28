# 数论索引

本目录包含数论相关算法的详细讲解，每个文件包含原理说明、实现代码、Mermaid 可视化图表。

## 算法关系总图

```mermaid
flowchart TD
    subgraph 基础工具
        A[素数] --> A1[素数筛<br/>埃氏/欧拉筛]
        A --> A2[素数判定<br/>Miller-Rabin]
        B[最大公约数] --> B1[欧几里得算法]
        B --> B2[扩展欧几里得]
        C[模运算] --> C1[快速幂]
    end

    subgraph 核心定理
        B2 --> D1[线性丢番图方程]
        B2 --> D2[模逆元]
        C1 --> D3[欧拉定理/降幂]
        C1 --> D4[费马小定理]
        D3 --> D5[中国剩余定理]
    end

    subgraph 进阶工具
        E[欧拉函数 φ] --> E1[积性函数体系]
        E1 --> E2[狄利克雷卷积]
        E2 --> E3[莫比乌斯反演]
        E3 --> E4[数论分块]
        D5 --> F[原根与阶]
        F --> G[离散对数 BSGS]
    end

    subgraph 大数运算
        A2 --> H[质因数分解<br/>Pollard-Rho]
        G --> H
        H --> I[二次剩余<br/>Cipolla/Tonelli-Shanks]
    end

    subgraph 密码学应用
        D4 --> J[RSA]
        D5 --> J
        F --> K[Diffie-Hellman]
        F --> L[ElGamal]
        G --> K
        I --> M[椭圆曲线 ECC]
    end
```

## 学习路径

### 路径一：基础入门（数论常识）

```
素数筛 → GCD/扩展欧几里得 → 快速幂 → 欧拉函数
```

### 路径二：进阶必备（算法竞赛）

```
路径一 → 中国剩余定理 → 莫比乌斯反演 → 原根 → 二次剩余
```

### 路径三：密码学方向（安全/加密）

```
路径一 → 费马小定理 → Miller-Rabin → Pollard-Rho → 离散对数 → 原根
```

## 符号说明

| 符号 | 含义 |
|------|------|
| gcd(a,b) | 最大公约数 |
| lcm(a,b) | 最小公倍数 |
| φ(n) | 欧拉函数 |
| μ(n) | 莫比乌斯函数 |
| τ(n) | 约数个数 |
| σ(n) | 约数和 |
| ord_m(a) | a 的阶 |
| (a/p) | Legendre 符号 |
| a ≡ b (mod m) | a 与 b 模 m 同余 |
| ⌊x⌋ | 向下取整 |
| [P] | Iverson 括号（P 真为 1） |
