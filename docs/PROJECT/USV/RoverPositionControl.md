# 对px4中的RoverPositionControl任务的解读

## RoverPositionControl.hpp


    class RoverPositionControl final : public ModuleBase<RoverPositionControl>, public ModuleParams, public px4::WorkItem

这是`C++`中的类定义语法，其中`RoverPositionControl`是一个类名，它继承自三个类：`ModuleBase<RoverPositionControl>`、`ModuleParams`、和 `px4::WorkItem`。

`ModuleBase<RoverPositionControl>`: 这是类 `ModuleBase` 的模板实例化，使用 `RoverPositionControl` 作为模板参数。这意味着 `RoverPositionControl` 类会继承自`ModuleBase`，并可能使用模板中的一些特定实现或配置。

`ModuleParams`: 这是另一个类，`RoverPositionControl` 类直接继承自它。这意味着 `RoverPositionControl` 类会继承 `ModuleParams` 类的成员和行为。

px4::WorkItem: 这是另一个类，RoverPositionControl 类也直接继承自它。这表示 RoverPositionControl 类会继承 px4::WorkItem 类的成员和行为。

public 关键字表示这些基类中的成员在派生类中保持它们的访问权限（即 public 成员在派生类中仍然是 public 访问权限）。

final 关键字表示 RoverPositionControl 类是一个终极类，不能再被其他类继承。

综合起来，这个类的定义表示 RoverPositionControl 是一个终极类，直接继承自三个类，其中包括一个模板类。这种多继承的语法通常用于在一个类中组合多个不同的功能和特性。

---

uORB::SubscriptionCallbackWorkItem _vehicle_angular_velocity_sub{this, ORB_ID(vehicle_angular_velocity)};

	uORB::SubscriptionInterval _parameter_update_sub{ORB_ID(parameter_update), 1_s};

这行代码做了以下几件事情：

1. `uORB::SubscriptionCallbackWorkItem`: 这是一个类，用于创建订阅 uORB 主题的回调函数。SubscriptionCallbackWorkItem 通常用于异步处理接收到的消息。

2. `_vehicle_angular_velocity_sub`: 这是一个类成员变量，它的类型是 uORB::SubscriptionCallbackWorkItem。通常，以 _ 开头的变量名表示这是一个类的私有成员。

3. `{this, ORB_ID(vehicle_angular_velocity)}`: 这是使用初始化列表初始化 _vehicle_angular_velocity_sub 成员变量。this 表示当前类的实例，ORB_ID(vehicle_angular_velocity) 表示订阅的主题 ID（在 uORB 中，每个主题都有一个唯一的 ID）。这行代码的效果是**创建了一个用于订阅 vehicle_angular_velocity 主题的回调对象。**
   

	uORB::Publication<vehicle_attitude_setpoint_s>	_attitude_sp_pub{ORB_ID(vehicle_attitude_setpoint)};
	uORB::Publication<position_controller_status_s>	_pos_ctrl_status_pub{ORB_ID(position_controller_status)};  /**< navigation capabilities publication */

4. 这行代码声明了一个 `uORB::Publication` 类型的成员变量 `_attitude_sp_pub`。这个类是用于发布特定类型消息的 uORB 发布者。在这里，它发布的消息类型是 vehicle_attitude_setpoint_s，这是一个结构体类型，描述了期望的飞行器姿态设定点。

ORB_ID(vehicle_attitude_setpoint) 表示该发布者将发布 vehicle_attitude_setpoint 主题的消息。ORB_ID 宏用于获取主题的 ID。

	uORB::Subscription _control_mode_sub{ORB_ID(vehicle_control_mode)}; /**< control mode subscription */

1. 这行代码声明了一个 uORB::Subscription 类型的成员变量 _control_mode_sub，用于订阅 vehicle_control_mode 主题的消息。vehicle_control_mode 主题通常包含有关飞行器当前控制模式的信息。
	uORB::Subscription _global_pos_sub{ORB_ID(vehicle_global_position)};
	uORB::Subscription _local_pos_sub{ORB_ID(vehicle_local_position)};
	uORB::Subscription _manual_control_setpoint_sub{ORB_ID(manual_control_setpoint)}; /**< notification of manual control updates */
	uORB::Subscription _pos_sp_triplet_sub{ORB_ID(position_setpoint_triplet)};
	uORB::Subscription _att_sub{ORB_ID(vehicle_attitude)};
	uORB::Subscription _att_sp_sub{ORB_ID(vehicle_attitude_setpoint)};
	uORB::Subscription _trajectory_setpoint_sub{ORB_ID(trajectory_setpoint)};

> 上述代码使用到了`uORB`相关代码。这是px4中用于处理发布-订阅通信的机制，用于在不同模块之间传递消息。

    manual_control_setpoint_s		_manual_control_setpoint{};			    /**< r/c channel data */
	position_setpoint_triplet_s		_pos_sp_triplet{};		/**< triplet of mission items */
	vehicle_attitude_setpoint_s		_att_sp{};			/**< attitude setpoint > */
	vehicle_control_mode_s			_control_mode{};		/**< control mode */
	vehicle_global_position_s		_global_pos{};			/**< global vehicle position */
	vehicle_local_position_s		_local_pos{};			/**< global vehicle position */
	vehicle_attitude_s				_vehicle_att{};
	trajectory_setpoint_s _trajectory_setpoint{};

- 位置期望(_manual_control_setpoint), 订阅自manual模式,即遥控控制信号
- 位置期望(_pos_sp_triplet), 订阅自misson模式,即自动巡航的航点 
- 姿态期望(_att_sp)
- 控制模式(_control_mode)
- 目前位置、全球坐标系(_global_pos)
- 局部坐标系(_local_pos)
- 目前姿态(_vehicle_att)
- 轨迹期望,包含线速度加速度等信息(_trajectory_setpoin)




## RoverPositionControl.cpp

### RoverPositionControl::Run()
    parameters_update(true);
开启参数更新

    vehicle_angular_velocity_s angular_velocity;
新建一个角速度变量

    if (_vehicle_angular_velocity_sub.update(&angular_velocity))
`Run()`函数里在外侧的一个判断语句
角速度发生更新，返回1并且赋值给`angular_velocity`

    /* check vehicle control mode for changes to publication state */
		vehicle_control_mode_poll();
		attitude_setpoint_poll();
		vehicle_attitude_poll();
		manual_control_setpoint_poll();

		_vehicle_acceleration_sub.update();

		/* update parameters from storage */
		parameters_update();
`poll()`函数 轮询功能
`vehicle_control_mode_poll();` 控制模式更新轮询
`attitude_setpoint_poll();` 姿态期望更新轮询 

`vehicle_attitude_poll();` 物体姿态更新赋值

`manual_control_setpoint_poll();` 这段代码主要处理手动控制模式下的逻辑，生成相应的飞行控制指令并发布到系统中。它根据飞行器的控制模式和手动控制通道的输入来调整姿态和油门控制。

    if (_local_pos_sub.update(&_local_pos))
检查本地位置数据是否有更新。如果有更新，表示飞行器的位置发生了变化，执行下面的位置控制逻辑。

    position_setpoint_triplet_poll()
期望位置的轮询，期望的位置订阅自mission模式

    if (!_global_local_proj_ref.isInitialized()
        || (_global_local_proj_ref.getProjectionReferenceTimestamp() != _local_pos.ref_timestamp)) {
        _global_local_proj_ref.initReference(_local_pos.ref_lat, _local_pos.ref_lon, _local_pos.ref_timestamp);
    }
初始化全局本地投影参考点
如果全局本地投影参考点未初始化，或者参考点的时间戳与本地位置的参考时间戳不匹配，则初始化全局本地投影参考点。

    // Convert Local setpoints to global setpoints
			if (_control_mode.flag_control_offboard_enabled) {
				_trajectory_setpoint_sub.update(&_trajectory_setpoint);

				// local -> global
				_global_local_proj_ref.reproject(
					_trajectory_setpoint.position[0], _trajectory_setpoint.position[1],
					_pos_sp_triplet.current.lat, _pos_sp_triplet.current.lon);

				_pos_sp_triplet.current.valid = true;
			}
如果启用了外部控制（Offboard mode），则将期望轨迹（trajectory_setpoint）转换为全局设定点，使用全局本地投影参考点进行投影

    matrix::Vector3f ground_speed(_local_pos.vx, _local_pos.vy,  _local_pos.vz);
    matrix::Vector2d current_position(_global_pos.lat, _global_pos.lon);
    matrix::Vector3f current_velocity(_local_pos.vx, _local_pos.vy, _local_pos.vz);
从本地位置数据中提取飞行器的地速、当前位置（经纬度）、当前速度。

    if (!_control_mode.flag_control_manual_enabled && _control_mode.flag_control_position_enabled) 
如果非手动模式并且启用了位置控制，执行位置控制逻辑。调用control_position函数进行位置控制，并在成功执行后发布位置控制器的状态信息。

    } else if (!_control_mode.flag_control_manual_enabled && _control_mode.flag_control_velocity_enabled) {
        _trajectory_setpoint_sub.update(&_trajectory_setpoint);
        control_velocity(current_velocity);
    }
如果非手动模式并且启用了速度控制，执行速度控制逻辑。调用control_velocity函数进行速度控制。

这段代码的目的是根据当前位置和控制模式执行相应的位置或速度控制逻辑，并发布相应的控制器状态信息。其中，control_position和control_velocity函数负责实际的位置和速度控制逻辑的实现。

以上从`if (_local_pos_sub.update(&_local_pos)) `开始的大循环是run()函数中开启手动控制或位置控制或速度控制的循环

---

`RoverPositionControl::control_position` 的代码分析


    bool
    RoverPositionControl::control_position(const matrix::Vector2d &current_position,
                        const matrix::Vector3f &ground_speed, const position_setpoint_triplet_s &pos_sp_triplet)
位置控制函数    

















