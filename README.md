# Unity Object Pickup System

![ezgif-829512b22699fb16](https://github.com/user-attachments/assets/a89d8168-fe7a-4852-8a8c-7cb32c5ecad7)
![ezgif-838c7f86d76b29c4](https://github.com/user-attachments/assets/027f29d5-45fc-4b10-8759-4b2ce51aba2f)


## 项目简介

这是一个基于 Unity 实现的第一人称物体抓取与放下交互系统。

玩家可以通过摄像机前向发射射线检测场景中的可交互物体，在进入交互范围后按下指定按键抓取目标物体，并将物体移动到角色前方的抓取点位置；再次按键后可将物体放下，并恢复物体原有的物理效果。

该项目主要用于练习和展示 Unity 中常见的交互系统设计方法，涉及射线检测、物理控制、模块化脚本设计以及第一人称交互逻辑实现。该系统后续还可以扩展为搬运解谜、道具拾取、投掷系统、物理解谜关卡等玩法模块。

## 功能特点

- 第一人称视角下的物体交互
- 基于 Raycast 的目标检测
- 使用 LayerMask 限定可抓取物体
- 支持抓取与放下两种状态切换
- 抓取时关闭重力，放下时恢复重力
- 使用 Rigidbody.MovePosition 实现平滑跟随
- 玩家交互逻辑与物体逻辑分离，便于扩展

## 核心实现思路

### 1. 玩家交互检测

玩家按下交互键后，系统会从摄像机位置沿前方发射射线，用于检测指定距离内是否存在可抓取物体。

当玩家当前没有抓取物体时：
- 发射射线
- 检测命中的物体是否挂载可抓取组件
- 若满足条件，则执行抓取逻辑

当玩家当前已经抓取物体时：
- 执行放下逻辑
- 清空当前抓取引用

### 2. 物体抓取逻辑

被抓取的物体会记录一个目标抓取点 Transform，并在物理更新中持续向该位置平滑移动。

抓取时：
- 保存抓取点
- 关闭 Rigidbody 重力

放下时：
- 清空抓取点
- 恢复 Rigidbody 重力

### 3. 平滑移动实现

项目中使用 `Vector3.Lerp` 配合 `Rigidbody.MovePosition`，让物体在抓取状态下平滑向目标点靠近，而不是瞬移到指定位置。这样可以让交互表现更加自然，也更符合物理系统的使用习惯。

## 关键代码展示

### 玩家抓取与放下逻辑

```csharp
private void Update() {
    if (Keyboard.current.eKey.wasPressedThisFrame) {
        if (objectGrabbable == null) {
            float pickUpDistance = 4f;
            if (Physics.Raycast(playerCameraTransform.position, playerCameraTransform.forward, out RaycastHit raycastHit, pickUpDistance, pickUpLayerMask)) {
                if (raycastHit.transform.TryGetComponent(out objectGrabbable)) {
                    objectGrabbable.Grab(objectGrabPointTransform);
                }
            }
        } else {
            objectGrabbable.Drop();
            objectGrabbable = null;
        }
    }
}
```
该部分代码实现了交互键触发、射线检测、抓取判定以及放下状态切换，是整个系统的核心入口。

物体跟随逻辑
```
private void FixedUpdate() {
    if (objectGrabPointTransform != null) {
        float lerpSpeed = 10f;
        Vector3 newPosition = Vector3.Lerp(transform.position, objectGrabPointTransform.position, Time.deltaTime * lerpSpeed);
        objectRigidbody.MovePosition(newPosition);
    }
}
```
该部分代码负责让物体在被抓取时平滑跟随到玩家前方指定位置。

脚本结构说明
PlayerPickUpDrop.cs

负责玩家交互逻辑：

-监听交互按键
-发射射线检测物体
-调用抓取与放下方法
-管理当前抓取对象引用
ObjectGrabbable.cs

负责可抓取物体逻辑：

-维护 Rigidbody 引用
-处理抓取状态
-控制重力开关
-在 FixedUpdate 中实现物体跟随
项目适用场景

该系统可以作为以下类型项目的基础模块：

-第一人称解谜游戏
-搬运类交互玩法
-场景探索与道具拾取系统
-物理交互教学 Demo
-生存类或恐怖类游戏中的可搬动物体系统

扩展以下内容：

-增加物体旋转功能
-增加物体投掷功能
-增加交互提示 UI
-增加抓取距离限制与碰撞检测
-增加不同物体重量对移动速度的影响
-增加可拾取物品分类系统
-增加音效与高亮描边反馈
-增加键位可配置功能
项目收获

通过这个项目，我进一步熟悉了 Unity 中第一人称交互系统的常见实现方式，掌握了射线检测、物理对象控制、抓取状态管理以及模块化脚本拆分的基本思路。同时，这个项目也让我对“可扩展交互系统”的设计有了更直观的理解，为后续实现更复杂的拾取、投掷、搬运和解谜玩法打下了基础。

运行说明
使用 Unity 打开项目
在场景中创建玩家对象与摄像机
设置抓取点 objectGrabPointTransform
给可抓取物体挂载 ObjectGrabbable 脚本，并确保物体带有 Rigidbody
将可抓取物体放入指定 Layer
运行场景后，按下 E 键进行抓取/放下操作
