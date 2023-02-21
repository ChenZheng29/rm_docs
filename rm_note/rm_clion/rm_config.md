# rm_config

## config

### rm_control

#### rm_hw

actuator_coefficient,yaml

> configurate some actuaor informationl

engineer.yaml

> bus 
>
> configurations of the acturators

```
bus: can2
id: 002
type: cheetah
lp_cutoff_frequency: 80
need_calibration: true
```

### rm_controllers

#### engineer.yaml

> joint controllers
>
> chassis controllers
>
> pid

### rm_manual

#### engineer.yaml

> chassis accel,power
>
> gimbal
>
> mast
>
> card
>
> stone_platform
>
> controllers_list
>
> power_on_calibration
>
> ui

## launch