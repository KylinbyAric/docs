# Vector3

该类可能代表向量、坐标、旋转、缩放

# transform



| API                | 类型         | 说明                                            |
| ------------------ | ------------ | ----------------------------------------------- |
| `position`         | `Vector3`    | 游戏对象在世界空间中的位置。                    |
| `localPosition`    | `Vector3`    | 游戏对象相对于父级的位置。                      |
| `rotation`         | `Quaternion` | 游戏对象在世界空间中的旋转（四元数）。          |
| `localRotation`    | `Quaternion` | 游戏对象相对于父级的旋转（四元数）。            |
| `eulerAngles`      | `Vector3`    | 旋转的欧拉角表示（以度为单位）。                |
| `localEulerAngles` | `Vector3`    | 相对于父级的欧拉角旋转（以度为单位）。          |
| `lossyScale`       | `Vector3`    | 全局缩放值（只读）。                            |
| `localScale`       | `Vector3`    | 相对于父级的缩放。                              |
| `parent`           | `Transform`  | 获取或设置父级 Transform。                      |
| `root`             | `Transform`  | 返回层级视图中最顶层的 Transform。              |
| `childCount`       | `int`        | 子对象的数量。                                  |
| `forward`          | `Vector3`    | 世界空间中对象的蓝轴（Z轴）方向。               |
| `right`            | `Vector3`    | 世界空间中对象的红轴（X轴）方向。               |
| `up`               | `Vector3`    | 世界空间中对象的绿轴（Y轴）方向。               |
| `hasChanged`       | `bool`       | 检查自上一帧以来 Transform 是否已被修改。<br /> |


### 🏃 动作方法 (Action Methods)


| API                                                             | 说明                                                                          |
| --------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| `Translate(Vector3 translation, Space relativeTo = Space.Self)` | 按指定方向移动对象（常用参数为`Space.Self`自身坐标或`Space.World`世界坐标）。 |
| `Rotate(Vector3 eulerAngles, Space relativeTo = Space.Self)`    | 按欧拉角旋转。                                                                |
| `RotateAround(Vector3 point, Vector3 axis, float angle)`        | 让对象绕世界坐标系中的点`point` 和轴 `axis` 旋转 `angle` 度。                 |
| `LookAt(Transform target)`                                      | 旋转对象使其`forward` 朝向 `target` 对象。                                    |



### 👨‍👧‍👦 父子级操作 (Hierarchy Methods)


| API                                          | 说明                                                                                                         |
| -------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `SetParent(Transform parent)`                | 设置当前 Transform 的父级。                                                                                  |
| `DetachChildren()`                           | 清除所有子对象的父级关系（让子对象变成独立的根对象）。                                                       |
| `Find(string name)`                          | 按名称查找子物体。支持路径格式（如`"Arm/Hand"`），但如果指定了路径，**只会返回末端对象，不会返回中间对象**。 |
| `GetChild(int index)`                        | 按索引获取子物体的 Transform 对象（索引从 0 开始）。                                                         |
| `GetSiblingIndex()`                          | 获取当前对象在父级子列表中的索引位置。                                                                       |
| `SetAsFirstSibling()` / `SetAsLastSibling()` | 将当前对象移动到父级子列表的最前面/最后面。                                                                  |
| `IsChildOf(Transform parent)`                | 判断当前对象是否为参数 parent 的子物体（或子物体链中的一员）。                                               |



### 🔄 空间转换 (Coordinate Transform Methods)


| API                                                 | 说明                                                       |
| --------------------------------------------------- | ---------------------------------------------------------- |
| `TransformPoint(Vector3 localPoint)`                | 将**位置**从局部坐标转换到世界坐标（受缩放影响）。         |
| `InverseTransformPoint(Vector3 worldPoint)`         | 将**位置**从世界坐标转换到局部坐标（受缩放影响）。         |
| `TransformDirection(Vector3 localDirection)`        | 将**方向**从局部坐标转换到世界坐标（不受缩放和位置影响）。 |
| `InverseTransformDirection(Vector3 worldDirection)` | 将**方向**从世界坐标转换到局部坐标（不受缩放和位置影响）。 |
| `TransformVector(Vector3 localVector)`              | 将**矢量**从局部坐标转换到世界坐标（受缩放影响）。         |
| `InverseTransformVector(Vector3 worldVector)`       | 将**矢量**从世界坐标转换到局部坐标（受缩放影响）。         |



### 💡 实用技巧与性能优化

* **高效设置 Position 和 Rotation**：同时设置位置和旋转时，推荐使用 `SetPositionAndRotation(Vector3 position, Quaternion rotation)`，只需一次操作，比分别赋值性能更好。
* **高效查找子物体**：
  * 使用 `GetChild(i)` 遍历比 `Find` 更高效，因为 `Find` 是字符串查找，性能消耗较大。
  * `Find` 方法仅查找当前物体下的子物体，而非所有后代。若需要递归查找所有后代，需自己实现递归逻辑。
* **注意坐标空间**：许多 API 如 `Translate`, `Rotate` 都支持选择基于 **自身坐标系 (Space.Self)** 或 **世界坐标系 ([Space.World](https://space.world/))**。选错坐标空间会导致物体移动方向错误。
* **重设 Transform**：右键点击 Inspector 面板的 Transform 组件，选择 "Reset" 可快速将 Position 重置为 (0,0,0)，Rotation 重置为 (0,0,0)，Scale 重置为 (1,1,1)。


| 写法                                          | 速度     | 适用场景                         |
| --------------------------------------------- | -------- | -------------------------------- |
| `this.transform`                              | 快       | 偶尔访问（如每帧 1\~2 次）       |
| 缓存为局部变量`Transform t = this.transform;` | **更快** | 循环、每帧多次访问、性能关键代码 |

> **最佳实践**：
>
> * 默认在 `Start` 或 `Awake` 中缓存 `this.transform` 到私有字段 `private Transform m_transform`，之后全部使用该字段。
> * 这几乎不增加代码复杂度，但能避免所有潜在的属性访问开销，并且让代码意图更清晰（表明这个引用会被重复使用）。



## 性能优化建议总结


| 优化点                          | 建议                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| **频繁访问 Transform 组件**     | 在`Start` 中缓存引用：`private Transform m_Trans;`           |
| **同时修改位置和旋转**          | 使用`SetPositionAndRotation` / `SetLocalPositionAndRotation` |
| **避免在 Update 中使用 `Find`** | `Find` 只应在初始化时调用一次，或改用 `GetChild` 遍历        |
| **减少 Transform 变化通知**     | 批量修改后手动调用`SendTransformChanged`（极少需要）         |
| **使用 `Transform.HasChanged`** | 仅当需要检测变化时读取，不要每帧无意义轮询                   |

---

## 常见陷阱总结


| 陷阱                                    | 解决办法                                            |
| --------------------------------------- | --------------------------------------------------- |
| **误用 `Find` 找不到深层子物体**        | 使用路径`"Arm/Hand"` 或递归查找                     |
| **子物体旋转后，移动方向错误**          | 明确使用`Space.Self` 或 `Space.World`               |
| **销毁物体后仍访问其 Transform**        | 销毁前移除引用，或检查`== null`                     |
| **欧拉角直接赋值导致万向锁**            | 使用四元数运算，只在需要限制角度时读写`eulerAngles` |
| **忘记 `SetParent` 的世界位置保持参数** | 明确传入`worldPositionStays` 参数                   |


# input


### 旧版 Input Manager API 速查表

* **`Input.GetKey(KeyCode.W)`**：检测特定按键是否处于按下状态，每帧都返回 `true`。
* **`Input.GetKeyDown(KeyCode.Space)`**：特定按键在本帧被按下时返回 `true`，只触发一次。
* **`Input.GetKeyUp(KeyCode.Space)`**：特定按键在本帧被松开时返回 `true`，只触发一次。
* **`Input.GetAxis("Horizontal")`**：获取虚拟轴的值（范围 -1 到 1），数值平滑渐变，手感丝滑。主要用于键盘的 A/D 或手柄摇杆。
* **`Input.GetAxisRaw("Horizontal")`**：获取虚拟轴未经平滑处理的原始值（只有 -1, 0, 1），响应直接。
* **`Input.GetButton("Jump")`**：检测在 Input Manager 中定义的“按钮”（如跳跃）是否被按住。
* **`Input.GetButtonDown("Jump")`**：检测定义的“按钮”在本帧是否被按下。
* **`Input.GetMouseButton(0)`**：检测鼠标按键是否按下，0: 左键, 1: 右键, 2: 中键。
* **`Input.GetMouseButtonDown(0)`**：检测鼠标按键在本帧被按下。
* **`Input.GetMouseButtonUp(0)`**：检测鼠标按键在本帧被抬起。
* **`Input.mousePosition`**：获取鼠标当前的屏幕像素坐标，左下角为 (0,0)。
* **`Input.GetAxis("Mouse X")`**：获取鼠标在水平方向的移动增量（每帧位移）。
* **`Input.GetAxis("Mouse Y")`**：获取鼠标在垂直方向的移动增量（每帧位移）。
* **`Input.touches`**：获取当前屏幕上所有触摸点的结构体数组，用于移动端开发。
* **`Input.acceleration`**：获取设备在三维空间中的加速度值，用于重力感应控制。
* **`Input.anyKey`**：检测是否有任何按键或鼠标按钮被按下。
* **`Input.anyKeyDown`**：检测在本帧是否有任何按键或鼠标按钮被按下。


### 🎮 核心 API 实战指南

#### 1. 角色移动：GetAxis 实现“丝滑”手感

用 `Input.GetKey` 处理移动是初学者常踩的“性能坑”。它虽快，但手感僵硬，且在高帧率下容易丢帧。`Input.GetAxis` 会返回一个平滑渐变的值，使起步和停止更自然，并统一了键盘的离散输入和手柄摇杆的连续输入。

**csharp**

```
void Update() {
    // 丝滑移动
    float moveX = Input.GetAxis("Horizontal");
    float moveZ = Input.GetAxis("Vertical");
    Vector3 move = transform.right * moveX + transform.forward * moveZ;
    transform.position += move * speed * Time.deltaTime;
  
    // 僵硬移动（不推荐）
    if (Input.GetKey(KeyCode.W)) transform.position += transform.forward * speed * Time.deltaTime;
}
```

#### 2. 跳跃与射击：GetKeyDown/GetButtonDown 保证单次触发

实现跳跃、射击等逻辑时，必须使用 `GetXxxDown` 方法，它只在按键按下的那一帧返回一次 `true`。如果错误使用 `GetKey`，代码会每一帧都执行，导致逻辑被疯狂触发。

**csharp**

```
void Update() {
    if (Input.GetButtonDown("Jump")) {
        rb.AddForce(Vector3.up * jumpForce, ForceMode.Impulse);
    }
}
```



#### 3. 鼠标操作：从屏幕坐标到世界坐标

鼠标输入主要分三部分：屏幕位置 (`mousePosition`)、按键状态和滚轮 (`mouseScrollDelta`)。用 `Camera.ScreenPointToRay` 从摄像机发射射线，检测鼠标在3D世界中点击了哪个物体。

**csharp**

```
void Update() {
    // 鼠标左键点击
    if (Input.GetMouseButtonDown(0)) {
        Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);
        if (Physics.Raycast(ray, out RaycastHit hit, 100f)) {
            Debug.Log("你点击了：" + hit.collider.name);
        }
    }
  
    // 检测滑轮滚动
    float scroll = Input.mouseScrollDelta.y;
    if (scroll != 0) camera.fieldOfView -= scroll * 5f;
}
```


### ⚠️ 常见误区与优化技巧

1. **误区：轮询代码杂乱无章**：直接将 `Input.GetKeyDown` 代码散落在项目各处，后期维护会非常痛苦。**优化**：设计一个**输入管理器（Input Manager）** 类，统一检测所有输入，并通过 C# 事件或方法将输入逻辑广播出去。
2. **误区：在 Update 中频繁调用 Input.GetAxis**：`GetAxis` 带有平滑算法，频繁调用不如一次读取结果并缓存。**优化**：在 `Update` 中读取一次 `Input.GetAxis`，将结果存入一个 `float` 变量，供本帧内所有逻辑使用。
3. **误区：在 FixedUpdate 中读取输入**：`FixedUpdate` 的调用频率与帧率不同步，会导致输入读取不稳定。**优化**：推荐在 `Update` 中读取所有输入并存储，然后在 `FixedUpdate` 中使用这些值进行物理运算。
4. **性能要点**：在非必要情况下，避免在 `Update` 中调用 `Input.GetJoystickNames()` 和频繁调用 `Input.ResetInputAxes()`。


# touch


### 📋 触摸系统的核心角色

它主要由三部分构成：负责“问”的 `Input` 类、负责“描述”的 `Touch` 结构体，以及负责“状态机”的 `TouchPhase` 枚举。

#### 🎬 `TouchPhase` 枚举：触摸的生命周期

触摸就像一个有生命周期的临时状态机，这也是理解触摸输入的关键。


| 触摸阶段 (Phase) | 描述                                                     |
| ---------------- | -------------------------------------------------------- |
| **Began**        | 手指刚触碰到屏幕，**这是唯一确定的起始帧**。             |
| **Moved**        | 手指在屏幕上发生了位移，这个阶段会连续触发。             |
| **Stationary**   | 手指停留在屏幕上，这一帧没有移动。                       |
| **Ended**        | 手指从屏幕上抬起，**这是唯一的结束帧**。                 |
| **Canceled**     | 触摸被系统意外取消（如锁屏、弹窗），视为一种特殊的结束。 |

> 每个触摸点都按 `Began → Moved/Stationary → Ended/Canceled` 的顺序走完它的生命周期，这构成了触摸逻辑的基础。

#### ⚙️ `Touch` 结构体：单个触摸点的说明书

`Touch` 是一个结构体，它存储了单个触摸点在一帧内的全部快照信息。


| 属性                            | 说明                                                 |
| ------------------------------- | ---------------------------------------------------- |
| **`fingerId`**                  | 每个手指的唯一编号，用于在多指触摸时跟踪同一根手指。 |
| **`phase`**                     | 触摸的当前阶段 (`TouchPhase` 枚举)。                 |
| **`position`**                  | 当前帧的触摸位置，单位是屏幕坐标系中的像素。         |
| **`deltaPosition`**             | 自从上一帧位置的变化量，常用于计算滑动速度。         |
| **`deltaTime`**                 | 自上次触摸状态改变以来经过的时间。                   |
| **`tapCount`**                  | 连续点击次数，用于判断双击。注意：仅 iOS 设备可靠。  |
| **`pressure`**                  | 触摸压力，需要设备支持，如 3D Touch，否则为 1.0。    |
| **`radius` / `radiusVariance`** | 触摸区域的估算半径，可用于模拟“指绘”笔刷效果。     |

#### 🏃‍♂️ `Input` 类：触摸数据的“询问器”

`Input` 类是访问所有输入设备的统一入口，对于触摸，主要通过以下静态方法和属性来获取数据。


| 方法 / 属性                        | 说明                                                                   |
| ---------------------------------- | ---------------------------------------------------------------------- |
| **`Input.touchCount`**             | 返回当前帧的触摸点数量。循环访问触摸数据前，总会先用它判断。           |
| **`Input.GetTouch(int index)`**    | 通过索引获取`Touch` 结构体。官方推荐，因为它不分配临时变量，性能更好。 |
| **`Input.touches`**                | 一个包含了所有触摸点的`Touch` 数组，通常与 `touchCount` 配合使用。     |
| **`Input.multiTouchEnabled`**      | 是否启用多点触摸，默认为 true，可设为 false 强制单点。                 |
| **`Input.touchPressureSupported`** | 检测当前设备是否支持压力感应。                                         |

### 🎯 核心逻辑模式与技巧

明白了生命周期，就可以用它构建处理流程了：

* **每帧查询**：触摸输入是“轮询”式的，必须在每帧的 `Update` 函数中主动查询和响应。生命周期中的 `Began`、`Moved` 等状态，必须主动判断才能知道手指正在做什么。
* **用 `fingerId` 跟踪手指**：多点触摸时，通过 `fingerId` 而非数组索引来跟踪特定手指，因为上一帧的索引 0 的手指，下一帧可能因为插入新触摸而索引改变。
* **简单的单点示例**：
  **csharp**

  ```
  void Update()
  {
      // 1. 检查是否有触摸
      if (Input.touchCount > 0)
      {
          // 2. 获取第一个触摸点
          Touch touch = Input.GetTouch(0);

          // 3. 根据触摸的不同阶段，做不同的事
          if (touch.phase == TouchPhase.Began)
          {
              Debug.Log("触摸开始，位置：" + touch.position);
          }
          else if (touch.phase == TouchPhase.Moved)
          {
              // 处理移动逻辑，touch.deltaPosition 是自上一帧的移动量
              Debug.Log("触摸移动，增量：" + touch.deltaPosition);
          }
          else if (touch.phase == TouchPhase.Ended || touch.phase == TouchPhase.Canceled)
          {
              Debug.Log("触摸结束");
          }
      }
  }
  ```

### 💻 常用场景与实现思路

了解了基础，你就能把这些生命周期知识应用到更复杂的逻辑中。

* **场景 1：物体拖拽**
  * 难点：保持物体和手指相对位置，以及碰撞检测。**建议在拖拽时禁用`Physics`以节省性能，并在松开时恢复。**
  * 思路：在 `TouchPhase.Began` 用射线检测选中物体并计算偏移；`TouchPhase.Moved` 时把屏幕坐标转成世界坐标并加上偏移去更新物体位置；在 `TouchPhase.Ended` 时清除状态。
* **场景 2：双指缩放（Pinch）**
  * 难点：精确追踪两个特定手指。**必须用`fingerId`记录，因为双指触摸时，`touches`数组的顺序可能不稳定。**
  * 思路：当有两个触摸点时，记录下各自的 `fingerId`。每当 `Moved` 时，根据 `fingerId` 找到这两个点，通过它们之间距离的变化量，来缩放摄像机视野或物体大小。
* **场景 3：滑动判断（Swipe）**
  * 难点：区分“滑动”和“点击”，并有方向敏感性。
  * 思路：在 `TouchPhase.Began` 时记录起始位置和时间；在 `TouchPhase.Ended` 时计算位置差和时间差。如果速度或距离超过阈值，再根据 `deltaPosition` 的正负来判断方向。
* **场景 4：复杂的触摸交互**
  * 难点：管理复杂的多点手势逻辑。
  * 思路：使用成熟的第三方解决方案，如 **Lean Touch** 或 **Easy Touch**，它们提供了摇杆、手势识别等开箱即用的组件，可以极大地简化开发工作。

### ⚠️ 常见误区与实战技巧

* **误区1：触摸与 UI 打架**
  * 起因：UI 系统的射线检测会拦截触摸事件，导致点击 UI 时，也被游戏世界的逻辑识别到。
  * 解法：在处理游戏逻辑前，用 `EventSystem.current.IsPointerOverGameObject(touch.fingerId)` 判断。若返回 `true`，说明点在 UI 上，应忽略本次触摸。
* **误区2：多点触摸无法限制**
  * 起因：在 `EventSystem` 体系下，`Input.multiTouchEnabled = false` 是无效的。
  * 解法：在代码中过滤。用 `touchCount` 判断，并建立一个“主控手指”的概念，只跟踪最先或最后触摸的那根手指的 `fingerId`。
* **误区3：混淆屏幕坐标与世界坐标**
  * 起因：`touch.position` 是屏幕像素坐标（如 iPhone 屏幕大小），而 3D 物体的位置是世界坐标。
  * 解法：坐标转换用 `Camera.main.ScreenToWorldPoint(touch.position)`。**注意要提供合理的 Z 轴值**，代表该点距离摄像机的距离。






# 欧拉角

俯仰角:pitch。绕着X轴旋转

翻滚角:roll。绕着Y轴旋转

偏航角:Yaw。绕着z轴旋轴。

## 一、欧拉角（Euler Angles）

**欧拉角**是用三个角度值（绕三个坐标轴的旋转分量）来描述物体朝向的方法。通常顺序为：绕Z轴（偏航/yaw）、绕X轴（俯仰/pitch）、绕Y轴（翻滚/roll），或者根据坐标系不同顺序有所差异（Unity中为Z-X-Y）。

* **表示法**：`(x, y, z)` 三个独立的角度值（通常范围 -180 到 180 或 0-360）。
* **直观理解**：就像飞机的“航向-俯仰-滚转”，人脑非常容易理解。
* **在Unity中**：`Transform` 的 `eulerAngles` 属性返回欧拉角，Inspector面板显示的也是欧拉角。

### ✅ 欧拉角的优点

* **直观性**：三个数值直接对应绕轴旋转的角度，美术、设计师容易理解。
* **存储小巧**：只需3个浮点数。
* **人机交互友好**：用于输入角度、编辑器调节。

### ❌ 欧拉角的缺点

* **万向锁（Gimbal Lock）**：当两个旋转轴对齐（例如俯仰达到±90°时，偏航和翻滚变得不可区分。），导致一个自由度丢失，产生不期望的旋转行为。
* **插值困难**：直接对三个角度分别插值会造成路径扭曲、速度不均。
* **表示不唯一**：`(30°, 0°, 0°)` 和 `(30°, 360°, 0°)` 表示同一朝向；不同顺序得到的数值也不同。
* **连续旋转难以处理**：从角度A到角度B的旋转方向可能不是最短路径。比如(0,0,0)旋转到(0,359,0)，直观感受是Y轴旋转-1度即可，但欧拉角方式就必须要旋转359，绕了很大的远路。

## 二、四元数（Quaternion）

**四元数**是一种扩展复数到四维空间的数学工具，用于表示三维空间中的旋转，形式为 `q = (x, y, z, w)`，满足 `x²+y²+z²+w²=1`。

* **核心思想**：四元数表示绕某轴旋转某角度。数学上：`q = (sin(θ/2)*axis, cos(θ/2))`。
* **在Unity中**：`Transform` 的 `rotation` 属性是四元数类型，所有旋转操作底层都使用四元数。

## ✅ 四元数的优点

* **无万向锁**：可平滑表示任意旋转，不受奇异点影响。旋转瞬间是生成一个新的三维旋转向量，围绕该向量进行旋转
* **稳定插值（Slerp）**：可在两个旋转之间进行球面线性插值，得到平滑的最短路径旋转。
* **高效组合**：两个四元数的乘法即可组合旋转，运算速度快，无误差累积。
* **连续性**：可轻松表示旋转序列，无歧义。

## ❌ 四元数的缺点

* **不直观**：四个数难以直接理解旋转效果。
* **调试困难**：直接看数值无法知道朝向。
* **需要归一化**：非单位四元数会导致缩放变换；计算中需定期归一化以避免浮点误差。
