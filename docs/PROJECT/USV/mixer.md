# /PX4-Autopilot/ROMFS/px4fmu_common/mixers/generic_diff_rover.main.mix
    Generic differential-drive rover
    ===========================

    This mixer is suitable for controlling any differential-drive rover. That is,
    a rover where the left wheels and right wheels are driven independently,
    allowing turning in place. It outputs to channels 0 (left wheels) and
    1 (right wheels)

    Inputs to the mixer come from channel group 0 (vehicle attitude), channels 2 (yaw), and 3 (thrust).

    See the README for more information on the scaler format.


    Throttle of left wheels of rover on Output 0
    ---------------------------------------
    M: 2
    S: 0 2    10000   10000      0  -10000  10000
    S: 0 3    10000   10000      0  -10000  10000


    Throttle of right wheels of rover on Output 1
    ---------------------------------------
    M: 2
    S: 0 2   -10000  -10000      0  -10000  10000
    S: 0 3    10000   10000      0  -10000  10000

在这个差速控制模型的混控器文件中，roverpositionControl任务给出了推力和转艏的控制量。
然后这两个量被发送到了`控制组 #0`

    控制组 #0 (Flight Control)
    0：roll (-1..1)
    1：pitch (-1..1)
    2：yaw (-1..1)
    3：throttle （正常范围为 0..1，变距螺旋桨和反推动力情况下范围为 -1..1）
    4：flaps (-1..1)
    5：spoilers (-1..1)
    6：airbrakes (-1..1)
    7：landing gear (-1..1)

    M: <control count>
    O: <-ve scale> <+ve scale> <offset> <lower limit> <upper limit>

---



    Throttle of left wheels of rover on Output 0
    ---------------------------------------
    M: 2
    S: 0 2    10000   10000      0  -10000  10000
    S: 0 3    10000   10000      0  -10000  10000


    Throttle of right wheels of rover on Output 1
    ---------------------------------------
    M: 2
    S: 0 2   -10000  -10000      0  -10000  10000
    S: 0 3    10000   10000      0  -10000  10000

output 0 和 output 1对应与pwm1 和pwm2
output 0对应左推进器
output 1对应右推进器



M:2 表示有两个输入量混合

    S: <group> <index> <-ve scale> <+ve scale> <offset> <lower limit> <upper limit>

在给定的 S: 混控器配置中，ve scale 表示混控器的“反向缩放”（reverse scale）。这是混控器配置的一部分，用于将输入信号从遥控器映射到输出信号。

具体来说，ve scale 代表负方向的缩放值。在混控器的上下文中，这通常表示当输入信号在负方向时，相应的输出信号的缩放比例。这允许你调整遥控器输入的影响，以便在输出信号上实现所需的行为。

`S` simple 简单混控器






