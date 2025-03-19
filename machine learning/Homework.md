**The following equations were derived for a 2-DOF RP manipulator:**

$$
\tau_1 = m_1(d_1^2 + d_2) \ddot{\theta}_1 + m_2 d_2^2 \ddot{\theta}_1 + 2m_2 d_2 \dot{d}_2 \dot{\theta}_1
$$
$$
+ g \cos(\theta_1) \big[m_1(d_1 + d_2 \dot{\theta}_1) + m_2(d_2 + \dot{d}_2) \big]
$$

$$
\tau_2 = m_1 \dot{d}_2 \ddot{\theta}_1 + m_2 \ddot{d}_2 - m_1 d_1 \dot{d}_2 - m_2 d_2 \dot{\theta}_1^2 + m_2 (d_2 + 1) g \sin(\theta_1).
$$

Some of the terms are obviously incorrect. Omit the incorrect terms and rearrange the equation to be the form:

$$
\tau = M(\Theta) \ddot{\Theta} + V(\Theta, \dot{\Theta}) + G(\Theta).
$$

Write out Matrixes: $M$, $V$ and $G$.


---



   令
   $$
   \Theta = \begin{bmatrix}\theta_1 \\ d_2\end{bmatrix}, \quad
   \dot{\Theta} = \begin{bmatrix}\dot{\theta}_1 \\ \dot{d}_2\end{bmatrix}, \quad
   \tau = \begin{bmatrix}\tau_1 \\ \tau_2\end{bmatrix},
   $$
   其中 $\theta_1$ 为旋转关节角度，$d_2$ 为平移关节（伸缩杆）的伸缩量；$\tau_1$ 为关节 1 的力矩，$\tau_2$ 为关节 2 的推拉力。

   质量矩阵为
   $$
   M(\Theta) = \begin{bmatrix}
   I_1 + m_1\,d_1^2 + m_2\,(d_1 + d_2)^2 & 0 \\[6pt]
   0 & m_2
   \end{bmatrix}.
   $$
   其中  
   - $I_1$ 为第 1 连杆关于关节 1 的转动惯量；  
   - $m_1$ 和 $m_2$ 分别为第 1 和第 2 连杆的质量；  
   - $d_1$ 为第 1 连杆质心到关节 1 旋转轴的距离。

   动能中 $(d_1+d_2)^2\dot{\theta}_1^2$ 项对时间求导可得到科里奥利和离心力项，故有
   $$
   V(\Theta,\dot{\Theta}) = \begin{bmatrix}
   2\,m_2\,(d_1 + d_2)\,\dot{\theta}_1\,\dot{d}_2 \\[6pt]
   -\,m_2\,(d_1 + d_2)\,\dot{\theta}_1^2
   \end{bmatrix}.
   $$


  第 1 连杆质心高度为 $d_1\sin\theta_1$，第 2 连杆（或末端）高度为 $(d_1+d_2)\sin\theta_1$，则势能可写为  
   $$
   U = m_1\,g\,d_1\sin\theta_1 + m_2\,g\,(d_1+d_2)\sin\theta_1.
   $$
   对 $\theta_1$ 求偏导并取负号得到关节 1 的重力力矩，而对 $d_2$ 求偏导得到关节 2 的重力分量，故有
   $$
   G(\Theta) = \begin{bmatrix}
   \bigl[m_1\,d_1 + m_2\,(d_1+d_2)\bigr]\,g\,\cos\theta_1 \\[6pt]
   -\,m_2\,g\,\sin\theta_1
   \end{bmatrix}.
   $$

