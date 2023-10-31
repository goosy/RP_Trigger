# relay and pulse trigger

## 说明

本库是为了 `s7-gen-scl` 中补足 RP 功能所调用的 FB，RP 功能要求实现以下延时或脉冲：

- onRelay 开延时
  in保持指定时间内为开时，Q在该时间之后输出开；
  由内置 SFB4 TON 实现；
- offRelay 关延时
  in保持指定时间内为关时，Q在该时间之后输出关；
  由内置 SFB5 TOF 实现；
- onPulse 上升沿脉冲
  在in的上升沿触发Q输出一个脉冲，脉冲周期为指定时长，在脉冲周期内如果再遇到in的上升沿，不会影响该脉冲周期；
  由内置 SFB3 TP 实现；
- onDPulse 防颤(debounce)上升沿脉冲
  在in的上升沿触发Q输出一个脉冲，脉冲周期为指定时长，在脉冲周期内如果再遇到in的上升沿，该脉冲周期重新计时延长脉冲；
  由本库的 FB3 DP 实现
- changePulse 变化脉冲
  in有变化时，即上升沿或下降沿时，触发Q输出一个脉冲，在脉冲周期内如果再遇到in的变化，不会影响该脉冲周期；
  由本库的 FB4 CP 实现
- changeDPulse 防颤(debounce)变化脉冲
  in有变化时，即上升沿或下降沿时，触发Q输出一个脉冲，在脉冲周期内如果再遇到in的变化，该脉冲周期重新计时延长脉冲；
  由本库的 FB3 DP 实现

## 用法

```SCL
// RP背景块: 涨潮退潮通知
DATA_BLOCK "tidenotice"
{ S7_m_c := 'true' }
AUTHOR:Goosy
FAMILY:GooLib
"CP"
BEGIN
  PT := TIME#1H; // 脉冲时长
END_DATA_BLOCK

"CP"."tidenotice"(
  IN := "tidesignal",
  ); // 涨潮退潮通知
M10.0 = tidenotice.Q;

 / 声控灯光
"DP"."on_sound_light"(
  IN := "onsound",
  PT := TIME#2M);

// 泵状态屏蔽联锁，上升沿下降沿都产生脉冲
"DP"."pumpCD"(
  IN := "pump_run",
  PT := TIME#5M,
  IncludeFallingEdge := true);
```
