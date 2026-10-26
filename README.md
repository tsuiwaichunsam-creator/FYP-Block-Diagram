# FYP-Block-Diagram
flowchart TB
    subgraph PS["功率级 POWER STAGE"]
        DC["DC 源<br/>(PV/BESS)"] --> VSI["三相 VSI<br/>(逆变器)"]
        VSI --> LCL["LCL 滤波器"]
        LCL --> PCC["PCC 断路器"]
        PCC --> GRID["Area EPS<br/>(电网)"]
    end

    VSI -- "门极信号" --> PWM["PWM 调制器"]
    LCL -- "V_pcc(abc)" --> MEAS
    VSI -- "I_inv(abc)" --> MEAS
    PCC -- "断路器状态" --> MS

    subgraph MEAS["测量与 PLL"]
        PLL["PLL"] 
        PQ["P/Q 计算"]
        PLL -- "θ_g, ω_g, V_g_mag" --> PRE
        PQ -- "P, Q" --> DROOP
    end

    subgraph MS["模式选择器 SPDT"]
        PRE["预同步控制器<br/>(断路器=0 时激活)<br/>ΔV PI / Δf PI / Δθ PI"]
        DROOP["下垂控制器<br/>(断路器=1 时激活)<br/>P-f: f_ref=f0−mp(P−P0)<br/>Q-V: V_ref=V0−nq(Q−Q0)"]
        SW["SPDT 开关"]
        PRE --> SW
        DROOP --> SW
    end

    SW -- "V_ref, f_ref, θ_ref" --> INNER

    subgraph INNER["内环控制 INNER LOOPS"]
        INT["相位积分器<br/>θ_ref=∫2πf_ref dt"]
        VPI["电压外环 PI"]
        CPI["电流内环 PI + 解耦"]
        DQABC["dq/abc 变换"]
        INT --> VPI --> CPI --> DQABC --> PWM
    end

    PWM -- "门极信号" --> VSI

    style PS fill:#ffe0e0,stroke:#cc0000
    style MEAS fill:#e0f0ff,stroke:#0066cc
    style MS fill:#e8ffe8,stroke:#009900
    style INNER fill:#f0f0f0,stroke:#666666
